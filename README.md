# Pintos

This repo contains an implementation of Pintos, a simple operating system for the 80x86 architecture that supports kernel threads, loading and running user programs, and a file system.



## Acknowledgements

This project adopts from the course project of Operating Systems at Stanford University, and was completed with [Cunhan You](https://github.com/youcunhan) under the instruction and guidance of *Pintos_Official_Doc* and *Pintos_Guide*.



## Configuration

Here, I will demonstrate how to configure and test Pintos in Ubuntu:

1. Choose a directory for Pintos, for example, `/home/username/` will be used in this section (`username` is the username of Ubuntu)

2. Clone the framework to the chosen directory: `git clone git://pintos-os.org/pintos-anon`

3. Comment out `#include <stropts.h>` in both `squish-pty.c` and `squish-unix.c` inside the directory `/home/username/pintos-anon/src/utils/`, comment out lines 288-293 in `squish-pty.c` inside the directory `/home/username/pintos-anon/src/utils/`

4. Run the following bash command in your console:
   
   ```bash
   sudo apt install build-essential libncurses5-dev texinfo libx11-dev libxrandr-dev
   ```
   
5. Copy the following contents to the file `/home/username/pintos-anon/src/misc/bochs-build.sh` (You may need to create that file), run bash command `./bochs-build.sh ./bochs` inside the `misc/` folder

   ```bash
   #! /bin/bash
   
   if [ $# -ne 1 ] || [ $1 == "-h" -o $1 == "--help" ]; then
     echo "Usage: $0 DSTDIR"
     exit 1
   fi
   
   CWD="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
   if [ ! -f $CWD/bochs-2.6.2-jitter-plus-segv.patch -a -f $CWD/bochs-2.6.2-xrandr-pkgconfig.patch -a -f $CWD/bochs-2.6.2-banner-stderr.patch -a -f $CWD/bochs-2.6.2-block-device-check.patch -a -f $CWD/bochs-2.6.2-const-char.patch ]; then
     echo "Could not find the patch files for Bochs in $CWD."
     exit 1
   fi
   
   mkdir -p $1
   if [ ! -d $1 ]; then
     echo "Could not find or create the destination directory"
     exit 1
   fi
   DSTDIR=$(cd $1 && pwd)
   
   cd /tmp
   mkdir $$
   cd $$
   wget https://sourceforge.net/projects/bochs/files/bochs/2.6.2/bochs-2.6.2.tar.gz/download -O bochs-2.6.2.tar.gz 
   tar xzf bochs-2.6.2.tar.gz
   cd bochs-2.6.2
   cat $CWD/bochs-2.6.2-jitter-plus-segv.patch | patch -p1
   cat $CWD/bochs-2.6.2-xrandr-pkgconfig.patch | patch -p1
   cat $CWD/bochs-2.6.2-banner-stderr.patch | patch -p1
   cat $CWD/bochs-2.6.2-block-device-check.patch | patch -p1
   cat $CWD/bochs-2.6.2-const-char.patch | patch -p1
   CFGOPTS="--with-x --with-x11 --with-term --with-nogui --prefix=$DSTDIR"
   os="`uname`"
   if [ $os == "Darwin" ]; then
     if [ ! -d /opt/X11 ]; then
       echo "Error: X11 directory does not exist. Have you installed XQuartz https://www.xquartz.org?" 
       exit 1
     fi
     # Bochs will have trouble finding X11 header files and library
     # We need to set the pkg config path explicitly.
     export PKG_CONFIG_PATH=/opt/X11/lib/pkgconfig
   fi
   WD=$(pwd)
   mkdir plain && cd plain
   ../configure $CFGOPTS --enable-gdb-stub && make -j8
   if [ $? -ne 0 ]; then
     echo "Error: build bochs failed"
     exit 1
   fi
   echo "Bochs plain successfully built"
   make install
   cd $WD
   mkdir with-dbg && cd with-dbg 
   ../configure --enable-debugger --disable-debugger-gui $CFGOPTS && make -j8
   if [ $? -ne 0 ]; then
     echo "Error: build bochs-dbg failed"
     exit 1
   fi
   cp bochs $DSTDIR/bin/bochs-dbg 
   rm -rf /tmp/$$
   echo "Done. bochs and bochs-dbg has been built and copied to $DSTDIR/bin"
   ```

6. Add `export PATH=$HOME/pintos-anon/src/utils:$HOME/pintos-anon/src/misc/bochs/bin:$PATH` to the end of file `.bashrc`

7. Modify `GDBMACROS` inside the file `/home/username/pintos-anon/src/utils/pintos-gdb` to `/home/username/pintos-anon/src/misc/gdb-macros`

8. Type `make` in `/home/username/pintos-anon/src/utils/` (Type `make` in `/home/username/pintos-anon/src/threads/`)

9. Replace `loader.bin` inside the file `/home/username/pintos-anon/src/utils/Pintos.pm` with `/home/username/pintos-anon/src/threads/build/loader.bin`; Replace `kernel.bin` inside the file `/home/username/pintos-anon/src/utils/pintos` with `/home/username/pintos-anon/src/threads/build/kernel.bin`

10. Run tests by invoking bash command `make check` in the directory `home/username/pintos-anon/src/threads/`, should fail 20/27 if your configuration is correct



## Implementation

You can find the design documents (Markdown version) of the four projects in the corresponding folders.



## Warning

**You may get inspirations from our work but you should NEVER copy any part of my code or assignments.**

**PLEASE FOLLOW YOUR HORNOR CODE STRICTLY.**
