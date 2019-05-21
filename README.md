# How To build VLC in WSL with LLVM

Building VLC on Windows is done using [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10). You need to install it first (x64 or x86, I use the former).

Then building is done in 3 steps:
* The build tools
* The contribs
* The VLC code

## LLVM Packages

You need to install the LLVM compiler on top from (https://github.com/mstorsjo/llvm-mingw/).

You just uncompress the latest `llvm-mingw-<date>-x86_64.tar.xz` in a local folder installation.
```
wget https://github.com/mstorsjo/llvm-mingw/releases/download/20190504/llvm-mingw-20190504-ubuntu-16.04.tar.xz
xz -d llvm-mingw-20190504-ubuntu-16.04.tar.xz
tar xvf llvm-mingw-20190504-ubuntu-16.04.tar -C ~/
```

From this point you will need to have `~/llvm-mingw-20190504-ubuntu-16.04/bin` in you PATH:
```
export PATH="`realpath ~/llvm-mingw-20190504-ubuntu-16.04/bin`":$PATH
```

## Get the VLC sources

In shell you clone the VLC repository:
```
git clone git://git.videolan.org/vlc.git
```

## Build Tools

The first step in building VLC is building build tools that are  missing or have a different version. Create a local directory where the tools will be built:

```
mkdir tools
cd tools
```

And do the following in the shell:
```
export VLC_TOOLS="$PWD"
export PATH="$VLC_TOOLS":$PATH
<path/to/vlc/root>/extra/tools/bootstrap
make
```

*Make sure it builds at least `libtool`*. If it doesn't do it manually: `make .buildlibtool`. This is required to have llvm link properly.


## Contribs

In your build root you create a folder where you will build and then build all of them. For **x64**:
```
mkdir contrib
cd contrib
<path/to/vlc/root>/contrib/bootstrap --host=x86_64-w64-mingw32 --enable-pdb
make fetch
VLC_TOOLS="`realpath -a tools/build/bin`" PKG_CONFIG_PATH="" make
```

For **x86**
```
mkdir contrib
cd contrib
<path/to/vlc/root>/contrib/bootstrap --host=i686-w64-mingw32 --enable-pdb
make fetch
VLC_TOOLS="`realpath -a tools/build/bin`" PKG_CONFIG_PATH="" make
```

This will take a **long** time. You can use more threads to make it faster by adding `-j4` to the make command. You can adjust the number to amount of threads you want to use.

Once all the contribs are built you will have all the libraries in **`<path/to/vlc/root>/contrib/x86_64-w64-mingw32/`** (or **`<path/to/vlc/root>/contrib/i686-w64-mingw32/`**).


## Building VLC

In a shell you first need to boostrap the git repository so it can be built.

First Make sure you have `</absolute/path/to/build/dir>/tools/build/bin` in your `PATH`:
```
export PATH=</absolute/path/to/build/dir>/tools/build/bin:$PATH
```

And boostrap:

```
(cd <path/to/vlc/root> && ./boostrap)
```

Then can create a folder anywhere you want and build VLC in it. First make sure your environment variables are set:
```
export VLC_TOOLS="`realpath -a tools/build/bin`"
export PATH="$VLC_TOOLS":$PATH
```

Then you configure the build:
```
cd <build_folder>
PKG_CONFIG="`which pkg-config`" <relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=x86_64-w64-mingw32 --prefix=`realpath ./_win64` --enable-debug --with-contrib=contrib/x86_64-w64-mingw32 --disable-nls --disable-ncurses
```
or for **x86**
```
cd <build_folder>
PKG_CONFIG="`which pkg-config`" <relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=i686-w64-mingw32 --prefix=`realpath ./_win64` --enable-debug --with-contrib=contrib/i686-w64-mingw32 --disable-nls --disable-ncurses
```

If you want to generate PDB files for debugging should add the extra configure option `--enable-pdb`:
```
PKG_CONFIG="`which pkg-config`" <relative/path/to/vlc/root>/extras/package/win32/configure.sh --host=x86_64-w64-mingw32 --prefix=`realpath ./_win64` --enable-debug --with-contrib=contrib/x86_64-w64-mingw32 --disable-nls --disable-ncurses --enable-pdb
```

And you're ready to build
```
make
```

