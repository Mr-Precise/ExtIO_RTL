cmake_minimum_required(VERSION 3.7.2...4.6)

if(${CMAKE_VERSION} VERSION_LESS "3.12.0")
    project(ExtIO_RTL)
else()
    project(ExtIO_RTL CXX)
endif()

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
endif()

find_package(Threads)
find_package(PkgConfig)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/rtl-sdr/CMakeLists.txt")
    add_subdirectory(rtl-sdr EXCLUDE_FROM_ALL)
    set(rtlsdr_FOUND TRUE)
elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/librtlsdr/CMakeLists.txt")
    add_subdirectory(librtlsdr EXCLUDE_FROM_ALL)
    set(rtlsdr_FOUND TRUE)
else()
    find_package(rtlsdr)
endif()

if (NOT rtlsdr_FOUND)
    message(STATUS "RTL-SDR development files not found...")
    set(RTLSDR_LIBRARIES "" CACHE STRING "manual RTL-SDR library path")
    set(RTLSDR_INCLUDE_DIRS "" CACHE STRING "manual RTL-SDR include path")
endif ()

message(STATUS "RTLSDR_INCLUDE_DIRS - ${RTLSDR_INCLUDE_DIRS}")
message(STATUS "RTLSDR_LIBRARIES - ${RTLSDR_LIBRARIES}")

if(PKG_CONFIG_FOUND)
    pkg_check_modules(LIBUSB libusb-1.0 IMPORTED_TARGET)
else()
    set(LIBUSB_LIBRARIES "" CACHE STRING "manual libusb path")
    set(LIBUSB_INCLUDE_DIRS "" CACHE STRING "manual libusb include path")
endif()

if(MSVC)
    set(THREADS_PTHREADS_LIBRARY "" CACHE STRING "manual pthread-win32 path")
    set(THREADS_PTHREADS_INCLUDE_DIR "" CACHE STRING "manual pthread-win32 includepath")
else()
    set(THREADS_PTHREADS_LIBRARY "" CACHE INTERNAL "manual pthread-win32 path")
    set(THREADS_PTHREADS_INCLUDE_DIR "" CACHE INTERNAL "manual pthread-win32 includepath")
endif()

if(NOT MSVC)
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--allow-multiple-definition -Wl,--enable-stdcall-fixup")
endif()

option(LINK_STATIC_RTLSDR "Link statically with librtlsdr" ON)

if(LINK_STATIC_RTLSDR)
    option(EXPORT_RTLSDR_SYMBOLS "Export librtlsdr functions in ExtIO_RTL.dll (for static rtl-sdr)" OFF)
    if(EXPORT_RTLSDR_SYMBOLS)
        set(DEF_FILE src/ExtIO_RTL_static.def)
    else()
        set(DEF_FILE src/ExtIO_RTL.def)
    endif()

    if(TARGET rtlsdr::rtlsdr_static)
        set(RTLSDR_LIB rtlsdr::rtlsdr_static)
    elseif(TARGET rtlsdr_static)
        set(RTLSDR_LIB rtlsdr_static)
    elseif(TARGET rtlsdr::rtlsdr)
        set(RTLSDR_LIB rtlsdr::rtlsdr)
    elseif(TARGET rtlsdr)
        set(RTLSDR_LIB rtlsdr)
    elseif(RTLSDR_LIBRARIES)
        set(RTLSDR_LIB ${RTLSDR_LIBRARIES})
    else()
        set(RTLSDR_LIB "")
    endif()
else()
    set(DEF_FILE src/ExtIO_RTL.def)

    if(TARGET rtlsdr::rtlsdr_shared)
        set(RTLSDR_LIB rtlsdr::rtlsdr_shared)
    elseif(TARGET rtlsdr)
        set(RTLSDR_LIB rtlsdr)
    elseif(TARGET rtlsdr::rtlsdr)
        set(RTLSDR_LIB rtlsdr::rtlsdr)
    elseif(RTLSDR_LIBRARIES)
        set(RTLSDR_LIB ${RTLSDR_LIBRARIES})
    else()
        set(RTLSDR_LIB "")
    endif()
endif()

include(CheckCXXSymbolExists)

set(CMAKE_REQUIRED_INCLUDES ${CMAKE_CURRENT_SOURCE_DIR}/src ${RTLSDR_INCLUDE_DIRS})
if(TARGET ${RTLSDR_LIB})
    get_target_property(_target_inc ${RTLSDR_LIB} INTERFACE_INCLUDE_DIRECTORIES)
    if(_target_inc)
        list(APPEND CMAKE_REQUIRED_INCLUDES ${_target_inc})
    endif()
    get_target_property(_target_source_dir ${RTLSDR_LIB} SOURCE_DIR)
    if(_target_source_dir AND EXISTS "${_target_source_dir}/../include")
        list(APPEND CMAKE_REQUIRED_INCLUDES "${_target_source_dir}/../include")
    endif()
endif()
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/rtl-sdr/include")
    list(APPEND CMAKE_REQUIRED_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/rtl-sdr/include")
elseif(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/librtlsdr/include")
    list(APPEND CMAKE_REQUIRED_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/librtlsdr/include")
endif()

if(LINK_STATIC_RTLSDR)
    set(CMAKE_REQUIRED_DEFINITIONS "-DRTLSDR_STATIC_DEFINE")
endif()

# Check symbols at compile time without linking an executable.
# This ensures checks succeed on Windows XP/MSVC and when rtl-sdr is added
# as a git submodule (add_subdirectory) before the library is compiled.
set(_prev_try_compile_target_type ${CMAKE_TRY_COMPILE_TARGET_TYPE})
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

check_cxx_symbol_exists(rtlsdr_set_dithering "stddef.h;rtl-sdr.h" HAVE_RTLSDR_SET_DITHERING)
check_cxx_symbol_exists(rtlsdr_set_bias_tee "stddef.h;rtl-sdr.h" HAVE_RTLSDR_SET_BIAS_TEE)
check_cxx_symbol_exists(rtlsdr_set_tuner_bandwidth "stddef.h;rtl-sdr.h" HAVE_RTLSDR_SET_TUNER_BANDWIDTH)

set(CMAKE_TRY_COMPILE_TARGET_TYPE ${_prev_try_compile_target_type})

add_library(ExtIO_RTL SHARED
    src/ExtIO_RTL.cpp
    src/ExtIO_RTL.h
    ${DEF_FILE}
    src/dllmain.cpp
    res/resource.h
    res/resource.rc
)

target_include_directories(ExtIO_RTL PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/res
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${RTLSDR_INCLUDE_DIRS}
)

set_target_properties(ExtIO_RTL PROPERTIES
    PREFIX ""
    OUTPUT_NAME "ExtIO_RTL"
)

if(MSVC_VERSION AND MSVC_VERSION LESS 1900)
    set_property(TARGET ExtIO_RTL PROPERTY CXX_STANDARD 11)
    set_property(TARGET ExtIO_RTL PROPERTY CXX_STANDARD_REQUIRED OFF)
else()
    set_property(TARGET ExtIO_RTL PROPERTY CXX_STANDARD 11)
endif()

target_compile_definitions(ExtIO_RTL PUBLIC _CRT_SECURE_NO_WARNINGS)

if(HAVE_RTLSDR_SET_DITHERING)
    target_compile_definitions(ExtIO_RTL PRIVATE HAVE_RTLSDR_SET_DITHERING=1)
endif()
if(HAVE_RTLSDR_SET_BIAS_TEE)
    target_compile_definitions(ExtIO_RTL PRIVATE HAVE_RTLSDR_SET_BIAS_TEE=1)
endif()
if(HAVE_RTLSDR_SET_TUNER_BANDWIDTH)
    target_compile_definitions(ExtIO_RTL PRIVATE HAVE_RTLSDR_SET_TUNER_BANDWIDTH=1)
endif()

target_link_libraries(ExtIO_RTL PRIVATE
    ${RTLSDR_LIB}
    comctl32
    gdi32
    user32
)

if(LIBUSB_LIBRARIES)
    target_link_libraries(ExtIO_RTL PRIVATE ${LIBUSB_LIBRARIES})
endif()

if(THREADS_PTHREADS_LIBRARY)
    target_link_libraries(ExtIO_RTL PRIVATE ${THREADS_PTHREADS_LIBRARY})
endif()
