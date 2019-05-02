# How To build VLC in msys2

Building VLC on Windows is done using [msys2](http://www.msys2.org/). You need to install it first (x64 or x86, I use the former).

Then building is done in 3 steps:
* The build tools
* The contribs
* The VLC code

## Msys2 Packages

**It is important that you don't run `pacman -Syu` to update `msys2` to the latest packages. The current `gcc 8.2` produces VLC executables that don't run**

VLC uses a lot of external code. And also many different tools to build this code. A lot of these tools are common in dev environment and some are not found in msys2 or needs some patching to work with VLC.

The first step after you install `msys2` is to install the basic tools and toolchain to be able to build anything.

The basic things you need can be installed with
```
pacman -S make automake autoconf pkg-config libtool git patch dos2unix unzip yasm git gperf bison autogen python3 help2man
pacman -S mingw-w64-x86_64-pkg-config mingw-w64-x86_64-extra-cmake-modules mingw-w64-x86_64-python3 mingw-w64-x86_64-meson
```

You will also need a compiler to build native msys2 executable:
```
pacman -S gcc
```

You don't need this if you are using a [custom LLVM-clang toolchain](http://martin.st/temp/llvm-mingw-x86_64.zip) which comes with all the compilers you need. Otherwise you need a compiler for your target:
* `x86_64-w64-mingw32-gcc`
* `i686-w64-mingw32-gcc`

So you need either these packages for **x64**:
```
pacman -S mingw-w64-x86_64-gcc mingw-w64-x86_64-headers-git mingw-w64-x86_64-tools-git mingw-w64-x86_64-make mingw-w64-x86_64-libwinpthread-git mingw-w64-x86_64-gdb mingw-w64-x86_64-nasm
```
Or for **x86**:
```
pacman -S mingw-w64-i686-gcc mingw-w64-i686-headers-git mingw-w64-i686-tools-git mingw-w64-i686-make mingw-w64-i686-libwinpthread-git mingw-w64-i686-gdb mingw-w64-i686-nasm
```

If you plan to build libbluray with menu support you will also need to [install a Java environment](http://jdk.java.net/java-se-ri/8) and have `JAVA_HOME` set properly. **It should not contain spaces otherwise it won't be used correctly by msys2.**

## Get the VLC sources

In a `mingw64.exe` shell (or `mingw86.exe` if you plan on build x86 binaries) you clone the VLC repository:
```
git config --global core.autocrlf false
git clone git://git.videolan.org/vlc.git
```

## Build Tools

The first step in building VLC is building build tools that are  missing or have a different version. Go **in `<path/to/vlc/root>/extra/tools`** and do the following in the shell:
```
export PATH=`cygpath -a build/bin`:$PATH
./bootstrap
make
```

Make sure it builds at least CMake. If it doesn't do it manually: `make .buildcmake`. This is required to have pkg-config work properly in msys2.


## Contribs

In a `mingw64.exe` shell (or `mingw86.exe` for i686 output) first you need to set the environment variable to set the compiler. It may not be found correctly by `make` and `CMake` otherwise:
```
export CC='x86_64-w64-mingw32-gcc'; export CXX='x86_64-w64-mingw32-g++'; export AR='x86_64-w64-mingw32-gcc-ar.exe'; export RANLIB='x86_64-w64-mingw32-gcc-ranlib'
```
or for **x86**
```
export CC='i686-w64-mingw32-gcc'; export CXX='i686-w64-mingw32-g++'; export AR='i686-w64-mingw32-gcc-ar.exe'; export RANLIB='i686-w64-mingw32-gcc-ranlib'
```

Then go in **in `<path/to/vlc/root>/contrib`** you create a folder where you will build and then build all of them. For **x64**:
```
mkdir win64
cd win64
../bootstrap --host=x86_64-w64-mingw32
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make fetch
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make
```

For **x86**:
```
mkdir win32
cd win32
../bootstrap --host=i686-w64-mingw32
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make fetch
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make
```

This will take a **long** time. You can use more threads to make it faster by adding `-j4` to the make command. You can adjust the number to amount of threads you want to use.

Once all the contribs are built you will have all the libraries in **`<path/to/vlc/root>/contrib/x86_64-w64-mingw32/`** (or **`<path/to/vlc/root>/contrib/i686-w64-mingw32/`**).


## Building VLC

In a `mingw64.exe` shell (or `mingw86.exe` for x86 output) you first need to boostrap the repository so it can be built. 

First Make sure you have `<path/to/vlc/root/extra/tools/build/bin>` in your `PATH`:
```
export PATH=</absolute/path/to/vlc/root>/extra/tools/build/bin:$PATH
```

And boostrap:

```
cd <path/to/vlc/root>
./boostrap
```

Then can create a folder anywhere you want and build VLC in it. First make sure your environment variables are set:
```
export CC='x86_64-w64-mingw32-gcc'; export CXX='x86_64-w64-mingw32-g++'; export AR='x86_64-w64-mingw32-gcc-ar.exe'; export RANLIB='x86_64-w64-mingw32-gcc-ranlib'
export CONFIG_SITE=/dev/null
```
or for **x86**
```
export CC='i686-w64-mingw32-gcc'; export CXX='i686-w64-mingw32-g++'; export AR='i686-w64-mingw32-gcc-ar.exe'; export RANLIB='i686-w64-mingw32-gcc-ranlib'
export CONFIG_SITE=/dev/null
```

Then you configure the build:
```
cd <build_folder>
<relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=x86_64-w64-mingw32 --enable-debug --disable-nls --disable-ncurses
```
or for **x86**
```
cd <build_folder>
<relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=i686-w64-mingw32 --enable-debug --disable-nls --disable-ncurses
```

And you're ready to build
```
make
```
