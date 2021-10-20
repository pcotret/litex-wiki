LiteX relies on several external tools to load the FPGA bitstream in SRAM or flash it in SPI Flash. This page provide information to install them.

## OpenOCD

Please use and install **our OpenOCD fork** for boards using OpenOCD programmer:

    git clone https://github.com/enjoy-digital/openocd
    cd openocd  
    ./bootstrap  
    ./configure --enable-ftdi  
    make  
    sudo make install

## openFPGALoader

    apt-get install libftdi1-2 libftdi1-dev libhidapi-hidraw0 libhidapi-dev libudev-dev cmake pkg-config make g++
    git clone https://github.com/trabucayre/openFPGALoader
    cd openFPGALoader
    mkdir build
    cd build
    cmake ../
    make
    sudo make install
