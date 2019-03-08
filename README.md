# How To build libVLC for Universal Windows in msys2 with LLVM

Building VLC on Windows is done using [msys2](http://www.msys2.org/). You need to install it first (x64 or x86, I use the former).

Then building is done in 3 steps:
* The build tools
* The contribs
* The VLC code

## Msys2 Packages

Run `pacman -Syu` (twice) to update `msys2` to the latest packages.

VLC uses a lot of external code. And also many different tools to build this code. A lot of these tools are common in dev environment and some are not found in msys2 or needs some patching to work with VLC.

The first step after you install `msys2` is to install the basic tools and toolchain to be able to build anything.

The basic things you need can be installed with
```
pacman -S make automake autoconf libtool git patch dos2unix unzip yasm nasm git gperf bison autogen python3 help2man
```
And either (if running mingw64.exe)
```
pacman -S mingw-w64-x86_64-pkg-config mingw-w64-x86_64-extra-cmake-modules mingw-w64-x86_64-python3 mingw-w64-x86_64-meson mingw-w64-x86_64-ragel
```
or if you plan on using the mingw32.exe
```
pacman -S mingw-w64-i686-pkg-config mingw-w64-i686-extra-cmake-modules mingw-w64-i686-python3 mingw-w64-i686-meson mingw-w64-i686-ragel
```

Then you need to install the LLVM compiler on top from (https://github.com/mstorsjo/llvm-mingw/).

For a mingw64 environment you just uncompress the latest llvm-mingw-<date>-x86_64.zip in the root of your msys2 installation.
```
wget https://github.com/mstorsjo/llvm-mingw/releases/download/20190124/llvm-mingw-20190124-x86_64.zip
unzip llvm-mingw-20190124-x86_64.zip -d /
```
Or for **x86**:
```
wget https://github.com/mstorsjo/llvm-mingw/releases/download/20190124/llvm-mingw-20190124-i686.zip
unzip llvm-mingw-20190124-i686.zip -d /
```

From this point you will need to have `/llvm-mingw/bin` in you msys2 PATH:
```
export PATH=/llvm-mingw/bin:$PATH
```

If you plan to build libbluray with menu support you will also need to [install a Java environment](http://jdk.java.net/java-se-ri/8) and have `JAVA_HOME` set properly. **It should not contain spaces otherwise it won't be used correctly by msys2.**

## Get the VLC sources

In a `mingw64.exe` shell (or `mingw32.exe` if you plan on build x86 binaries) you clone the VLC repository:
```
git clone git://git.videolan.org/vlc.git
```

## Build Tools

The first step in building VLC is building build tools that are  missing or have a different version. Go **in `<path/to/vlc/root>/extra/tools`** and do the following in the shell:
```
export PATH=`cygpath -a build/bin`:$PATH
./bootstrap
make
```

*Make sure it builds at least `libtool` and `automake`*. If it doesn't do it manually: `make .buildautomake .buildlibtool`. This is required to have llvm link properly.


## Contribs

In a `mingw64.exe` shell (or `mingw86.exe` for i686 output) first you need to set the environment variable to set the compiler. It may not be found correctly by `make`, `CMake` or `meson` otherwise:
```
export CC="x86_64-w64-mingw32-gcc.exe"; export CXX="x86_64-w64-mingw32-g++.exe"; export AR="x86_64-w64-mingw32-ar.exe"; export STRIP="x86_64-w64-mingw32-strip.exe"
```
or for **x86**
```
export CC="i686-w64-mingw32-gcc.exe"; export CXX="i686-w64-mingw32-g++.exe"; export AR="i686-w64-mingw32-ar.exe"; export STRIP="i686-w64-mingw32-strip.exe"
```

and the flags to make sure the APIs allowed by the Windows Store are used:
```
export CFLAGS="-DWINAPI_FAMILY=WINAPI_FAMILY_APP -D_WIN32_WINNT=0x0A00 -DWINVER=0x0A00 -DWINSTORECOMPAT -D_UNICODE -DUNICODE"; export CXXFLAGS=$CFLAGS; export CPPFLAGS=$CPPFLAGS
export LDFLAGS="-lnormaliz -lwinstorecompat -lruntimeobject -lmincore"
```


Go in **in `<path/to/vlc/root>/contrib`** you create a folder where you will build and then build all of them. For **x64**:
```
mkdir win64
cd win64
../bootstrap --host=x86_64-w64-mingw32 --build=x86_64-w64-mingw32 --prefix=`cygpath -a ../x86_64-w64-mingw32uwp` --enable-d3d11 --enable-libdsm --disable-gnuv3 --disable-qt --disable-qtdeclarative --disable-qtgraphicaleffects --disable-qtquickcontrols2 --disable-qtsvg
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make fetch
PKG_CONFIG_PATH="" CONFIG_SITE=/dev/null make
```

For **x86**:
```
mkdir win32
cd win32
../bootstrap --host=i686-w64-mingw32 --build=i686-w64-mingw32 --prefix=`cygpath -a ../i686-w64-mingw32uwp` --enable-d3d11 --enable-libdsm --disable-gnuv3 --disable-qt --disable-qtdeclarative --disable-qtgraphicaleffects --disable-qtquickcontrols2 --disable-qtsvg
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
export CONFIG_SITE=/dev/null
```

Then you configure the build:
```
cd <build_folder>
<relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=x86_64-w64-mingw32 --enable-winstore-app --disable-vlc --enable-debug --disable-nls --disable-ncurses
```
or for **x86**
```
cd <build_folder>
<relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=i686-w64-mingw32 --enable-winstore-app --disable-vlc --enable-debug --disable-nls --disable-ncurses
```

If you want to generate PDB files for debugging should add the extra configure option `--enable-pdb`:
```
<relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=x86_64-w64-mingw32 --enable-debug --disable-nls --disable-ncurses --enable-pdb
```

And you're ready to build
```
make
```

