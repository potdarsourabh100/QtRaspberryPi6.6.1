# Qt 6.6.1 cross compilation for Raspberry pi 4

In this page, you can find related steps to compile Qt6.6.1 for raspberr pi 4
As a host machine, ubuntu virtual machine is used. 

If you do not follow the steps exactly or add any extra step, process may fail.
I recommend you to first follow this insturctions then apply your different.

# Prepare Raspberry pi 4
I used 64 bit image 2023-12-05-raspios-bookworm-arm64.img. This is important because
in the next we compile the gcc and we need to configure it correctly. If you use 32 bit image
then you should use kernel7 instead of kernel8, so becareful. Steps can be different also for this case.

We need to update raspberry pi because later while compiling the qt crossly on ubuntu host machine, we need sysroot directory where all headers/libraries from raspberry pi.

Update the raspberry pi.

```bash
sudo apt update
sudo apt full-upgrade
sudo reboot
sudo rpi-update
sudo reboot
```
After this you can check your firmware revision number. 

```bash
cat /boot/firmware/.firmware_revision 
```
For me it is 1eba1172652d63bc90d9345ea890c2930ce5a9cd
If you want to be sure, you can set user this revision with 

```bash
rpi-update 1eba1172652d63bc90d9345ea890c2930ce5a9cd
```

but it is up to you.



Install the dependencies

```bash
sudo apt-get install \
libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev \
libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev \
libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev \
mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx \
libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-alsa \
libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" \
flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev \
libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev \
libsqlite3-dev libxslt1-dev libavcodec-dev libavformat-dev libswscale-dev \
libx11-dev freetds-dev libpq-dev libiodbc2-dev firebird-dev \
libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev \
libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev \
libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev \
libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev \
libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev \
libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev \
libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev \
libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev \
libdirectfb-dev libaudio-dev libxkbcommon-x11-dev gdbserver
```
Install the dependencies

```bash
sudo apt-get install \
libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev \
libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev \
libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev \
mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx \
libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-alsa \
libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" \
flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev \
libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev \
libsqlite3-dev libxslt1-dev libavcodec-dev libavformat-dev libswscale-dev \
libx11-dev freetds-dev libpq-dev libiodbc2-dev firebird-dev \
libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev \
libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev \
libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev \
libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev \
libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev \
libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev \
libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev \
libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev \
libdirectfb-dev libaudio-dev libxkbcommon-x11-dev gdbserver\
libgstreamer1.0-dev gdbserver libxcb1 libxcb-cursor0
```
for QT based packages please provide following commands:
```bash
sudo apt-get remove --purge qt6-base-dev
sudo apt-get install qt6-base-dev
```

After this all, you should be able to see opengl libraries under /opt/vc/lib/ directory. This is important for future steps. Check like this:

```bash
ls /opt/vc/lib/
```
list will be as follows

```bash
libbcm_host.so         libelftoolchain.so     libopenmaxil.so
libbrcmEGL.so          libGLESv1_CM.so        libOpenVG.so
libbrcmGLESv2.so       libGLESv2.so           libvchiq_arm.so
libbrcmOpenVG.so       libGLESv2_static.a     libvchostif.a
libbrcmWFC.so          libkhrn_client.a       libvcilcs.a
libcontainers.so       libkhrn_static.a       libvcos.so
libdebug_sym.so        libmmal_components.so  libvcsm.so
libdebug_sym_static.a  libmmal_core.so        libWFC.so
libdtovl.so            libmmal.so             pkgconfig
libEGL.so              libmmal_util.so        plugins
libEGL_static.a        libmmal_vc_client.so
```
# Host machine Prepartion
I used ubuntu Ubuntu 24.04.1 LTS You can find out yours using
```bash
uname -a
```
it gives output as
```bash
Linux TEPL-PUN-WS108 6.8.0-41-generic #41-Ubuntu SMP PREEMPT_DYNAMIC Fri Aug  2 20:41:06 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

Update host machine
```bash
sudo apt-get update
sudo apt-get upgrade
```
Install dependencies
```bash
$ sudo apt-get install make build-essential cmake libclang-dev ninja-build gcc git bison \
python3 gperf pkg-config libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev \
libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-glx0-dev \
libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev \
libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev \
libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev \
libatspi2.0-dev libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev libgmp-dev libmpfr-dev libmpc-dev
```
### Compile toolchain

We need toolchain to compile our qt or applications for cross compilation.
This can be done in many ways. In this context, we will follow a way that I took as reference in this script. https://docs.slackware.com/howtos:hardware:arm:gcc-10.x_aarch64_cross-compiler

Simply we will download the source code of gcc and compile it according to our needs. It is very tricky process. The steps for the process can vary for different version of gcc. 

Create a directory for compilation process and install gcc, glibc and binutils. I choose this version to match with the one in raspberry pi but it is not necessary, IF you use stable and quite latest one, it should work either.

```bash
mkdir crossTools && cd crossTools
wget https://mirror.lyrahosting.com/gnu/binutils/binutils-2.40.tar.gz
wget https://ftp.nluug.nl/pub/gnu/glibc/glibc-2.36.tar.gz
wget https://ftp.nluug.nl/pub/gnu/gcc/gcc-12.2.0/gcc-12.2.0.tar.gz
git clone --depth=1 https://github.com/raspberrypi/linux

tar xf binutils-2.40.tar.gz
tar xf gcc-12.2.0.tar.gz
tar xf glibc-2.36.tar.gz
```
Create a directory under opt for installation

```bash
sudo mkdir -p /opt/cross-pi-gcc
sudo chown $USER /opt/cross-pi-gcc
export PATH=/opt/cross-pi-gcc/bin:$PATH
```
We need kernel interfaces for compilation for related architecture
```bash
cd linux/
KERNEL=kernel8
make ARCH=arm64 INSTALL_HDR_PATH=/opt/cross-pi-gcc/aarch64-linux-gnu headers_install
```
Compile binutils

```bash
cd ..
mkdir build-binutils && cd build-binutils
../binutils-2.40/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --with-arch=armv8 --disable-multilib
make -j 8
make install
```
Add PATH_MAX define into to asan_linux.cpp (gcc-12.2.0/libsanitizer/asan/asan_linux.cpp)
Otherwise you will have compilation error. Add it on line no 68

```bash
#ifndef PATH_MAX
#define PATH_MAX 4096
#endif
```

Compile gcc and glibc partly
```bash
cd ..
 mkdir build-gcc && cd build-gcc
../gcc-12.2.0/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --enable-languages=c,c++ --disable-multilib
make -j8 all-gcc
make install-gcc
```
for glibc
```bash
cd ..
mkdir build-glibc && cd build-glibc
../glibc-2.36/configure --prefix=/opt/cross-pi-gcc/aarch64-linux-gnu --build=$MACHTYPE --host=aarch64-linux-gnu --target=aarch64-linux-gnu --with-headers=/opt/cross-pi-gcc/aarch64-linux-gnu/include --disable-multilib libc_cv_forced_unwind=yes
make install-bootstrap-headers=yes install-headers
make -j8 csu/subdir_lib
install csu/crt1.o csu/crti.o csu/crtn.o /opt/cross-pi-gcc/aarch64-linux-gnu/lib
aarch64-linux-gnu-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /opt/cross-pi-gcc/aarch64-linux-gnu/lib/libc.so
touch /opt/cross-pi-gcc/aarch64-linux-gnu/include/gnu/stubs.h
```
Compile glibc and gcc completely

```bash
cd ../build-gcc/
make -j8 all-target-libgcc
make install-target-libgcc
cd ../build-glibc/
make -j8
make install
cd ../build-gcc/
make -j8
make install
```
Now you have croos toolchain, if you like, you can compile simple hello world application to test :)

### Qt compilation
First we need to compile Qt for host. 

```bash
cd ~
wget https://download.qt.io/official_releases/qt/6.6/6.6.1/submodules/qtbase-everywhere-src-6.6.1.tar.xz
mkdir qt6 qt6/host qt6/pi qt6/host-build qt6/pi-build qt6/src
tar xf qtbase-everywhere-src-6.6.1.tar.xz 
cd qt6/host-build/
cmake ../../qtbase-everywhere-src-6.6.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$HOME/qt6/host
cmake --build . --parallel 8
cmake --install .
```
Binaries will be in $HOME/qt6/host
### Qt Cross compilation

We need to pull some libraries/headers from raspberry pi

```bash
cd ~
mkdir rpi-sysroot rpi-sysroot/usr rpi-sysroot/opt
rsync -avz --rsync-path="sudo rsync" pi@192.168.29.23:/usr/include rpi-sysroot/usr
rsync -avz --rsync-path="sudo rsync" pi@192.168.29.23:/lib rpi-sysroot
rsync -avz --rsync-path="sudo rsync" pi@192.168.29.23:/usr/lib rpi-sysroot/usr 
rsync -avz --rsync-path="sudo rsync" pi@192.168.29.23:/opt/vc rpi-sysroot/opt
```
Correct symbollic links

```bash
cd ~
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
chmod +x sysroot-relativelinks.py 
python3 sysroot-relativelinks.py rpi-sysroot
```

### Create toolchain.cmake under qt6 directory
create a file as follows
```bash
cd ~
cd qt6
touch toolchain.cmake
```
copy all the contents from following into the newly created file
check all paths and verify whether those exist in your PC in same way or not. 
If not make appropriate changes

```cmake
cmake_minimum_required(VERSION 3.20)
include_guard(GLOBAL)

# Set the system name and processor for cross-compilation
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# Set the target sysroot and architecture
set(TARGET_SYSROOT /home/tepladmin/rpi-sysroot) ## update this path accordingly
set(TARGET_ARCHITECTURE aarch64-linux-gnu)
set(CMAKE_SYSROOT ${TARGET_SYSROOT})

# Configure the pkg-config environment variables
set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig")
set(ENV{PKG_CONFIG_LIBDIR} "/usr/lib/pkgconfig:/usr/share/pkgconfig:${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig:${CMAKE_SYSROOT}/usr/lib/pkgconfig")
set(ENV{PKG_CONFIG_SYSROOT_DIR} "${CMAKE_SYSROOT}")

# Set the C and C++ compilers
set(CMAKE_C_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-gcc)
set(CMAKE_CXX_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-g++)

# Define additional compiler flags
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/${TARGET_ARCHITECTURE}")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")

# Set Qt-specific compiler and linker flags
set(QT_COMPILER_FLAGS "-march=armv8-a -mfpu=crypto-neon-fp-armv8 -mtune=cortex-a72 -mfloat-abi=hard")
set(QT_COMPILER_FLAGS_RELEASE "-O2 -pipe")
set(QT_LINKER_FLAGS "-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -Wl,-rpath-link=${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE} -Wl,-rpath-link=$HOME/qt6/pi/lib")

# Configure CMake find root path modes
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

# Set the install RPATH use and build RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_BUILD_RPATH ${TARGET_SYSROOT})

# Initialize CMake configuration variables with the specified flags.
set(CMAKE_C_FLAGS_INIT "${QT_COMPILER_FLAGS} -march=armv8-a")
set(CMAKE_CXX_FLAGS_INIT "${QT_COMPILER_FLAGS} -march=armv8-a")
set(CMAKE_EXE_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS} -Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed")
set(CMAKE_SHARED_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS} -Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed")
set(CMAKE_MODULE_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS} -Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed")

# Set paths for XCB and OpenGL libraries
set(XCB_PATH_VARIABLE ${TARGET_SYSROOT})
set(GL_INC_DIR ${TARGET_SYSROOT}/usr/include)
set(GL_LIB_DIR ${TARGET_SYSROOT}:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}:${TARGET_SYSROOT}/usr:${TARGET_SYSROOT}/usr/lib)
set(EGL_INCLUDE_DIR ${GL_INC_DIR})
set(EGL_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libEGL.so)
set(OPENGL_INCLUDE_DIR ${GL_INC_DIR})
set(OPENGL_opengl_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libOpenGL.so)
set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
set(GLIB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)
set(GLESv2_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

# Correct the setting for gbm_LIBRARY
set(gbm_INCLUDE_DIR ${GL_INC_DIR})
set(gbm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libgbm.so)

set(Libdrm_INCLUDE_DIR ${GL_INC_DIR})
set(Libdrm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libdrm.so)
set(XCB_XCB_INCLUDE_DIR ${GL_INC_DIR})
set(XCB_XCB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libxcb.so)

# Append to CMake library and prefix paths
list(APPEND CMAKE_LIBRARY_PATH ${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE})
list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${TARGET_ARCHITECTURE}/cmake")
```

### Compile the qt6.6.1 crossly
```bash
cd qt6/pi-build
cmake ../../qtbase-everywhere-src-6.6.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DINPUT_opengl=es2 -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DQT_HOST_PATH=$HOME/qt6/host -DCMAKE_STAGING_PREFIX=$HOME/qt6/pi -DCMAKE_INSTALL_PREFIX=/usr/local/qt6 -DCMAKE_TOOLCHAIN_FILE=$HOME/qt6/toolchain.cmake -DQT_QMAKE_TARGET_MKSPEC=devices/linux-rasp-pi4-aarch64 -DQT_FEATURE_xcb=ON -DFEATURE_xcb_xlib=ON -DQT_FEATURE_xlib=ON
```

Output shall look  like this
```bash
-- The CXX compiler identification is GNU 12.2.0
-- The C compiler identification is GNU 12.2.0
-- The ASM compiler identification is GNU
-- Found assembler: /opt/cross-pi-gcc/bin/aarch64-linux-gnu-gcc
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /opt/cross-pi-gcc/bin/aarch64-linux-gnu-g++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /opt/cross-pi-gcc/bin/aarch64-linux-gnu-gcc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
            -DCMAKE_C_FLAGS= -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/aarch64-linux-gnu
            -DCMAKE_C_FLAGS_DEBUG=-g
            -DCMAKE_C_FLAGS_RELEASE=-O3 -DNDEBUG
            -DCMAKE_C_FLAGS_RELWITHDEBINFO=-O2 -g -DNDEBUG
            -DCMAKE_CXX_FLAGS= -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/aarch64-linux-gnu
            -DCMAKE_CXX_FLAGS_DEBUG=-g
            -DCMAKE_CXX_FLAGS_RELEASE=-O3 -DNDEBUG
            -DCMAKE_CXX_FLAGS_RELWITHDEBINFO=-O2 -g -DNDEBUG
            -DCMAKE_OBJCOPY=/opt/cross-pi-gcc/bin/aarch64-linux-gnu-objcopy
            -DCMAKE_EXE_LINKER_FLAGS=-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -Wl,-rpath-link=/home/tepladmin/rpi-sysroot/usr/lib/aarch64-linux-gnu -Wl,-rpath-link=$HOME/qt6/pi/lib -Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed
            -DCMAKE_TOOLCHAIN_FILE=/home/tepladmin/qt6/toolchain.cmake
            -DCMAKE_C_STANDARD=11
            -DCMAKE_C_STANDARD_REQUIRED=ON
            -DCMAKE_CXX_STANDARD=17
            -DCMAKE_CXX_STANDARD_REQUIRED=ON
            -DCMAKE_MODULE_PATH:STRING=/home/tepladmin/qtbase-everywhere-src-6.6.1/cmake/platforms

-- Configuration summary shown below. It has also been written to /home/tepladmin/qt6/pi-build/config.summary
-- Configure with --log-level=STATUS or higher to increase CMake's message verbosity. The log level does not persist across reconfigurations.
 
-- Configure summary:

Building for: devices/linux-rasp-pi4-aarch64 (arm64, CPU features: cx16 neon)
Compiler: gcc 12.2.0
Build options:
  Mode ................................... release
  Optimize release build for size ........ no
  Fully optimize release builds (-O3) .... no
  Building shared libraries .............. yes
  Using ccache ........................... no
  Unity Build ............................ no
  Using new DTAGS ........................ yes
  Relocatable ............................ yes
  Using precompiled headers .............. yes
  Using Link Time Optimization (LTCG) .... no
  Using Intel CET ........................ no
  Target compiler supports:
    ARM Extensions ....................... NEON
  Sanitizers:
    Addresses ............................ no
    Threads .............................. no
    Memory ............................... no
    Fuzzer (instrumentation only) ........ no
    Undefined ............................ no
  Build parts ............................ libs
  Install examples sources ............... no
Qt modules and options:
  Qt Concurrent .......................... yes
  Qt D-Bus ............................... yes
  Qt D-Bus directly linked to libdbus .... yes
  Qt Gui ................................. yes
  Qt Network ............................. yes
  Qt PrintSupport ........................ yes
  Qt Sql ................................. yes
  Qt Testlib ............................. yes
  Qt Widgets ............................. yes
  Qt Xml ................................. yes
Support enabled for:
  Using pkg-config ....................... yes
  Using vcpkg ............................ no
  udev ................................... yes
  OpenSSL ................................ yes
    Qt directly linked to OpenSSL ........ no
  OpenSSL 1.1 ............................ no
  OpenSSL 3.0 ............................ yes
  Using system zlib ...................... yes
  Zstandard support ...................... yes
  Thread support ......................... yes
Common build options:
  Linker can resolve circular dependencies  yes
Qt Core:
  backtrace .............................. yes
  DoubleConversion ....................... yes
    Using system DoubleConversion ........ no
  CLONE_PIDFD support in forkfd .......... yes
  GLib ................................... yes
  ICU .................................... yes
  Using system libb2 ..................... no
  Built-in copy of the MIME database ..... yes
  Application permissions ................ yes
  Defaulting legacy IPC to POSIX ......... no
  Tracing backend ........................ <none>
  OpenSSL based cryptographic hash ....... no
  Logging backends:
    journald ............................. no
    syslog ............................... no
    slog2 ................................ no
  PCRE2 .................................. yes
    Using system PCRE2 ................... yes
Qt Sql:
  SQL item models ........................ yes
Qt Network:
  getifaddrs() ........................... yes
  IPv6 ifname ............................ yes
  libproxy ............................... no
  Linux AF_NETLINK ....................... yes
  DTLS ................................... yes
  OCSP-stapling .......................... yes
  SCTP ................................... no
  Use system proxies ..................... yes
  GSSAPI ................................. no
  Brotli Decompression Support ........... yes
  qIsEffectiveTLD() ...................... yes
    Built-in publicsuffix database ....... yes
    System publicsuffix database ......... yes
Core tools:
  Android deployment tool ................ no
  macOS deployment tool .................. no
  Windows deployment tool ................ no
  qmake .................................. yes
Qt Gui:
  Accessibility .......................... yes
  FreeType ............................... yes
    Using system FreeType ................ yes
  HarfBuzz ............................... yes
    Using system HarfBuzz ................ no
  Fontconfig ............................. yes
  Image formats:
    GIF .................................. yes
    ICO .................................. yes
    JPEG ................................. yes
      Using system libjpeg ............... yes
    PNG .................................. yes
      Using system libpng ................ yes
  Text formats:
    HtmlParser ........................... yes
    CssParser ............................ yes
    OdfWriter ............................ yes
    MarkdownReader ....................... yes
      Using system libmd4c ............... no
    MarkdownWriter ....................... yes
  EGL .................................... yes
  OpenVG ................................. no
  OpenGL:
    Desktop OpenGL ....................... no
    OpenGL ES 2.0 ........................ yes
    OpenGL ES 3.0 ........................ yes
    OpenGL ES 3.1 ........................ yes
    OpenGL ES 3.2 ........................ yes
  Vulkan ................................. yes
  Session Management ..................... yes
Features used by QPA backends:
  evdev .................................. yes
  libinput ............................... yes
  HiRes wheel support in libinput ........ yes
  INTEGRITY HID .......................... no
  mtdev .................................. yes
  tslib .................................. yes
  xkbcommon .............................. yes
  X11 specific:
    xlib ................................. yes
    XCB Xlib ............................. yes
    EGL on X11 ........................... yes
    xkbcommon-x11 ........................ yes
    xcb-sm ............................... no
QPA backends:
  DirectFB ............................... no
  EGLFS .................................. yes
  EGLFS details:
    EGLFS OpenWFD ........................ no
    EGLFS i.Mx6 .......................... no
    EGLFS i.Mx6 Wayland .................. no
    EGLFS RCAR ........................... no
    EGLFS EGLDevice ...................... yes
    EGLFS GBM ............................ yes
    EGLFS VSP2 ........................... no
    EGLFS Mali ........................... no
    EGLFS Raspberry Pi ................... no
    EGLFS X11 ............................ yes
  LinuxFB ................................ yes
  VNC .................................... yes
  VK_KHR_display ......................... yes
  QNX:
    lgmon ................................ no
    IMF .................................. no
  XCB:
    Using system-provided xcb-xinput ..... no
    GL integrations:
      GLX Plugin ......................... no
        XCB GLX .......................... no
      EGL-X11 Plugin ..................... yes
  Windows:
    Direct 2D ............................ no
    Direct 2D 1.1 ........................ no
    DirectWrite .......................... no
    DirectWrite 3 ........................ no
Qt Widgets:
  GTK+ ................................... no
  Styles ................................. Fusion Windows
Qt Testlib:
  Tester for item models ................. yes
  Batch tests ............................ no
Qt PrintSupport:
  CUPS ................................... yes
Qt Sql Drivers:
  DB2 (IBM) .............................. no
  InterBase .............................. yes
  MySql .................................. no
  OCI (Oracle) ........................... no
  ODBC ................................... no
  PostgreSQL ............................. yes
  SQLite ................................. yes
    Using system provided SQLite ......... no
  Mimer .................................. no
 

Note: Due to CMAKE_STAGING_PREFIX usage and an unfixed CMake bug,
      to ensure correct build time rpaths, directory-level install
      rules like ninja src/gui/install will not work.
      Check QTBUG-102592 for further details.

-- 

Qt is now configured for building. Just run 'cmake --build . --parallel'

Once everything is built, you must run 'cmake --install .'
Qt will be installed into '/usr/local/qt6'

To configure and build other Qt modules, you can use the following convenience script:
        /home/tepladmin/qt6/pi/bin/qt-configure-module

If reconfiguration fails for some reason, try removing 'CMakeCache.txt' from the build directory
Alternatively, you can add the --fresh flag to your CMake flags.

-- Configuring done (9.1s)
-- Generating done (0.2s)
-- Build files have been written to: /home/tepladmin/qt6/pi-build
```
Then you can start build process.

```bash
cmake --build . --parallel 8
cmake --install .
```

Send binaries to Rasperry pi

```bash
rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@192.168.29.23:/usr/local/qt6
```




















