## ExtIO_RTL

ExtIO wrapper for librtlsdr for use with [**HDSDR**](https://hdsdr.de/) and other [**Winrad**](https://www.i2phd.org/winrad/) compatible programs.  

### Fork for modified library by [Mr-Precise/rtl-sdr](https://github.com/Mr-Precise/rtl-sdr)
---
* Original code from [**jorgem-seq/ExtIO_RTL**](https://github.com/jorgem-seq/ExtIO_RTL), which was derived from [**josemariaaraujo/ExtIO_RTL**](https://github.com/josemariaaraujo/ExtIO_RTL)
* removed makefiles and added cmake support to compile with Visual Studio or MinGW
* merged 'Add Bias Tee option'
---
### For manual compilation from Visual Studio 2010/2019 or MinGW.  
[Visual Studio](https://visualstudio.microsoft.com/) or MinGW/[msys](https://www.msys2.org/)/[LLVM Clang MinGW](https://github.com/mstorsjo/llvm-mingw) etc...  
libusb [libusb/releases](https://github.com/libusb/libusb/releases)  
[pthread-win32 library](https://github.com/GerHobbelt/pthread-win32)  
Latest [CMake](https://cmake.org/download/) or [Old CMake](https://github.com/Kitware/CMake/releases/tag/v3.13.4) for 2000/XP  
librtlsdr [rtl-sdr](https://github.com/Mr-Precise/rtl-sdr)  
Recommented for use: [Precompiled static libs (build kit)](https://github.com/Mr-Precise/SDR-binary-builds-stuff/releases/tag/windows)

---
### Download latest pre-built binary available in [Release](https://github.com/Mr-Precise/ExtIO_RTL/releases/latest) section.

### LICENSE

GPL-2.0, see [**LICENSE file**](COPYING)
