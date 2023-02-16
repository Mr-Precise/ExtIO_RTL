cmake_minimum_required(VERSION 3.7.2)

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
find_package(RTLSDR)

if (NOT RTLSDR_FOUND)
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

if(PKG_CONFIG_FOUND AND NOT LIBUSB_FOUND)
    message(FATAL_ERROR "LibUSB 1.0 required to compile ExtIO_RTL")
endif()
if(NOT THREADS_FOUND)
    message(FATAL_ERROR "pthreads(-win32) required to compile ExtIO_RTL")
endif()

include_directories(${RTLSDR_INCLUDE_DIRS})

include_directories(ExtIO_RTL PRIVATE
    ${CMAKE_SOURCE_DIR}/res
)

add_library(ExtIO_RTL SHARED
    src/ExtIO_RTL.cpp src/ExtIO_RTL.h src/ExtIO_RTL.def src/dllmain.cpp
    res/resource.h  res/resource.rc
)

set_property(TARGET ExtIO_RTL PROPERTY CXX_STANDARD 17)
set_property(TARGET ExtIO_RTL PROPERTY CXX_STANDARD_REQUIRED ON)
target_compile_definitions(ExtIO_RTL PUBLIC _CRT_SECURE_NO_WARNINGS)

target_link_libraries(ExtIO_RTL PRIVATE
    ${LIBUSB_LIBRARIES}
    ${RTLSDR_LIBRARIES}
    ${THREADS_PTHREADS_LIBRARY}
)
