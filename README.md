mspgcc-install
==============

Script to install [mspgcc](http://mspgcc.sourceforge.net/) (C compiler for [msp430](http://www.ti.com/msp430) microcontrollers) in Ubuntu. It installs the LTS (Long Term Support) version of the compiler, 
which is the most up-to-date version. Currently, this is LTS 20120406.

To install got to the terminal app and run:
    $ git clone https://github.com/jlhonora/mspgcc-install.git
    $ cd mspgcc-install
    $ chmod +x install-all
    $ ./install-all

That should download, patch and install the current stable version into /usr/local/msp430 (modifyiable in the script). If you already have contents in this folder I recommend you to move it to a backup folder, i.e.:

    $ sudo mv /usr/local/msp430 /usr/local/msp430-backup

You should be careful to watch for error messages. In the following sections you will find how to build each part of the project in order.
    
This guide is based on [Sergio Campama's](https://github.com/sergiocampama/Launchpad) guide.

## mspgcc basics ##
    # Installation path. This will overwrite your
    # changes to the INSTALL_PATH folder
    INSTALL_PATH=/usr/local/msp430

    # get LTS mspgcc
    wget http://sourceforge.net/projects/mspgcc/files/mspgcc/mspgcc-20120406.tar.bz2
    tar xvf mspgcc-20120406.tar.bz2
    cd mspgcc-20120406

## binutils ##
    echo "Downloading binutils" 
    wget ftp://ftp.gnu.org/pub/gnu/binutils/binutils-2.21.1a.tar.bz2
    echo "Extracting binutils" 
    tar xjf binutils-2.21.1a.tar.bz2
    echo "Installing binutils" 
    ( cd binutils-2.21.1 ; patch -p1 < ../msp430-binutils-2.21.1a-20120406.patch )
    mkdir -p BUILD/binutils
    cd BUILD/binutils
    ../../binutils-2.21.1/configure \
      --target=msp430 \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with binutils" 
    cd ../../
    
## gcc ##
    echo "Downloading gcc" 
    wget ftp://ftp.gnu.org/pub/gnu/gcc/gcc-4.6.3/gcc-4.6.3.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-gcc-4.6.3-20120406-sf3540953.patch
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-gcc-4.6.3-20120406-sf3559978.patch
    echo "Extracting gcc" 
    tar xjf gcc-4.6.3.tar.bz2
    echo "Installing gcc" 
    cd gcc-4.6.3
    patch -p1 < ../msp430-gcc-4.6.3-20120406.patch
    patch -p1 < ../msp430-gcc-4.6.3-20120406-sf3540953.patch
    patch -p1 < ../msp430-gcc-4.6.3-20120406-sf3559978.patch
    cd ..
    mkdir -p BUILD/gcc
    cd BUILD/gcc
    ../../gcc-4.6.3/configure \
      --target=msp430 \
      --enable-languages=c,c++ \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with gcc" 
    
## gdb ##
    echo "Downloading gdb" 
    wget ftp://ftp.gnu.org/pub/gnu/gdb/gdb-7.2a.tar.bz2
    echo "Extracting gdb" 
    tar xjf gdb-7.2a.tar.bz2
    echo "Installing gdb" 
    ( cd gdb-7.2 ; patch -p1 < ../msp430-gdb-7.2a-20111205.patch )
    mkdir -p BUILD/gdb
    cd BUILD/gdb
    ../../gdb-7.2/configure \
      --target=msp430 \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with gdb" 
    errors=`cat co mo moi | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ../../
## msp430mcu ##
    echo "Downloading msp430mcu" 
    wget http://sourceforge.net/projects/mspgcc/files/msp430mcu/msp430mcu-20120406.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430mcu-20120406-sf3522088.patch
    echo "Extracting msp430mcu" 
    tar xvf msp430mcu-20120406.tar.bz2
    cd msp430mcu-20120406
    patch -p1 < ../msp430mcu-20120406-sf3522088.patch
    echo "Installing msp430mcu" 
    sudo MSP430MCU_ROOT=`pwd` ./scripts/install.sh $INSTALL_PATH | tee so
    cd ..
## msp430-libc ##
    OLDPATH=$PATH
    export PATH=$INSTALL_PATH/bin:$PATH
    echo "Downloading libc" 
    wget https://sourceforge.net/projects/mspgcc/files/msp430-libc/msp430-libc-20120224.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-libc-20120224-sf3522752.patch
    echo "Extracting libc" 
    tar xvf msp430-libc-20120224.tar.bz2
    echo "Installing libc" 
    cd msp430-libc-20120224
    patch -p1 < ../msp430-libc-20120224-sf3522752.patch
    ./configure
    cd src/
    make | tee mo
    sudo make PREFIX=$INSTALL_PATH install | tee mio
    echo "Done with libc" 
    errors=`cat mo mio | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ..
    PATH=$OLDPATH


## Install script ##
Here's the script in case you don't want to clone the repo. Just copy the following text to a script file and run it.


    #/usr/bin/env bash

    # A simple (and not very nice)
    check_for_errors(){
        echo "$1" | tail 
        if [ -n "$1" ]
        then
            echo "There seems to be errors, exiting"
            exit
        else
            echo "All OK"
        fi
    }

    # Installation path. This will overwrite your
    # changes to the INSTALL_PATH folder
    INSTALL_PATH=/usr/local/msp430

    # get LTS mspgcc
    wget http://sourceforge.net/projects/mspgcc/files/mspgcc/mspgcc-20120406.tar.bz2
    tar xvf mspgcc-20120406.tar.bz2
    cd mspgcc-20120406

    #binutils
    echo "Downloading binutils" 
    wget ftp://ftp.gnu.org/pub/gnu/binutils/binutils-2.21.1a.tar.bz2
    echo "Extracting binutils" 
    tar xjf binutils-2.21.1a.tar.bz2
    echo "Installing binutils" 
    ( cd binutils-2.21.1 ; patch -p1 < ../msp430-binutils-2.21.1a-20120406.patch )
    mkdir -p BUILD/binutils
    cd BUILD/binutils
    ../../binutils-2.21.1/configure \
      --target=msp430 \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with binutils" 
    errors=`cat co mo moi | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ../../

    #gcc
    echo "Downloading gcc" 
    wget ftp://ftp.gnu.org/pub/gnu/gcc/gcc-4.6.3/gcc-4.6.3.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-gcc-4.6.3-20120406-sf3540953.patch
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-gcc-4.6.3-20120406-sf3559978.patch
    echo "Extracting gcc" 
    tar xjf gcc-4.6.3.tar.bz2
    echo "Installing gcc" 
    cd gcc-4.6.3
    patch -p1 < ../msp430-gcc-4.6.3-20120406.patch
    patch -p1 < ../msp430-gcc-4.6.3-20120406-sf3540953.patch
    patch -p1 < ../msp430-gcc-4.6.3-20120406-sf3559978.patch
    cd ..
    mkdir -p BUILD/gcc
    cd BUILD/gcc
    ../../gcc-4.6.3/configure \
      --target=msp430 \
      --enable-languages=c,c++ \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with gcc" 
    errors=`cat co mo moi | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ../../

    #gdb
    echo "Downloading gdb" 
    wget ftp://ftp.gnu.org/pub/gnu/gdb/gdb-7.2a.tar.bz2
    echo "Extracting gdb" 
    tar xjf gdb-7.2a.tar.bz2
    echo "Installing gdb" 
    ( cd gdb-7.2 ; patch -p1 < ../msp430-gdb-7.2a-20111205.patch )
    mkdir -p BUILD/gdb
    cd BUILD/gdb
    ../../gdb-7.2/configure \
      --target=msp430 \
      --prefix=$INSTALL_PATH \
    2>&1 | tee co
    make 2>&1 | tee mo
    sudo make install 2>&1 | tee moi
    echo "Done with gdb" 
    errors=`cat co mo moi | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ../../

    #msp430-mcu
    echo "Downloading msp430mcu" 
    wget http://sourceforge.net/projects/mspgcc/files/msp430mcu/msp430mcu-20120406.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430mcu-20120406-sf3522088.patch
    echo "Extracting msp430mcu" 
    tar xvf msp430mcu-20120406.tar.bz2
    cd msp430mcu-20120406
    patch -p1 < ../msp430mcu-20120406-sf3522088.patch
    echo "Installing msp430mcu" 
    sudo MSP430MCU_ROOT=`pwd` ./scripts/install.sh $INSTALL_PATH | tee so
    cd ..

    OLDPATH=$PATH
    export PATH=$INSTALL_PATH/bin:$PATH

    #libc
    echo "Downloading libc" 
    wget https://sourceforge.net/projects/mspgcc/files/msp430-libc/msp430-libc-20120224.tar.bz2
    wget http://sourceforge.net/projects/mspgcc/files/Patches/LTS/20120406/msp430-libc-20120224-sf3522752.patch
    echo "Extracting libc" 
    tar xvf msp430-libc-20120224.tar.bz2
    echo "Installing libc" 
    cd msp430-libc-20120224
    patch -p1 < ../msp430-libc-20120224-sf3522752.patch
    ./configure
    cd src/
    make | tee mo
    sudo make PREFIX=$INSTALL_PATH install | tee mio
    echo "Done with libc" 
    errors=`cat mo mio | grep "Error [0-9]"`
    check_for_errors "$errors"
    cd ..

    PATH=$OLDPATH
    echo "Done installing, installed at $INSTALL_PATH"
