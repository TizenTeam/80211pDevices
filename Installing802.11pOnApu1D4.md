


# Installing 802.11p on APU 1D4
##Introduction
In this document we describe how to get 802.11p device on the APU 1D4 board.
The steps to install an 802.11p device in the APU 1D4 are:

 1. Getting the hardware.
 2. Installing Debian.
 2. Installing a linux kernel with 802.11p support.
 3. Installing user-space tools (iw) to configure the device properly.
 4. Installing the new Regulatory Database to allow configuring the device.
 5. Setting up the interface for startup configuration.

## The hardware
### The board
We decided to buy a APU 1D4 board from [pccengines](http://pcengines.com). The characteristics of this board are:

 - CPU: AMD G series T40E, 1 GHz dual Bobcat core with 64 bit support, 32K data + 32K instruction + 512K L2 cache per core
 - DRAM: 2 or 4 GB DDR3-1066 DRAM
 - Storage: Boot from SD card (connected through USB), external USB or m-SATA SSD. 1 SATA + power connector.
 - 12V DC, about 6 to 12W depending on CPU load. Jack = 2.5 mm, center positive
 - Connectivity: 3 Gigabit Ethernet channels (Realtek RTL8111E)
 - I/O: DB9 serial port, 2 USB external + 2 internal, three front panel LEDs, pushbutton
 - Expansion: 2 miniPCI express (one with SIM socket), LPC bus, GPIO header, I2C bus, COM2 (3.3V RXD / TXD) 
The key points to chose this board were the amount of RAM, the mini PCIe connector, and the m-Sata interface.

This board allows us to install Debian on it and it is able to run java and i386 binaries.
### The wireless interface
The most important module in our configuration is the wireless card. We bought a "802.11 a/b/g/n miniPCI express radio" this card has a 	Qualcomm Atheros AR9280 chipset supported by the driver ath9k. This driver is well known to support 5.9Ghz and OCB operations.
### Shopping list
The final component list is:
From [PcEngines](http://pcengines.com)
 - APU.1D4 system board 4GB	EUR 115.02
 - Enclosure 3 LAN, red, USB	EUR 8.78
 - AC adapter 12V 2A euro for IT equipment	EUR 3.86
 - SSD M-Sata 16GB MLC Phison	EUR 14.93
 - Compex WLE200NX miniPCI express card	EUR 16.68
From [Hobbitronics](http://www.hobbytronics.co.uk/gps-gp-20u7)
 - GP20u7 GPS Receiver EUR 15.96
## Installing Debian
Installing Debian in the Apu board is quite simple. First we need to use a serial cable to connect a computer with the board and get a terminal, then we only need to install Debian from a usb stick as we would install it in any other computer.
### Serial connection to the board
To connect with the APU board we need a null-modem cable. A null-modem cable is a DB9 serial cable whose pins 2-3 have been crossed. There are USB-DB9 cables you can use if your computer does not have a DB9 port.
Once the board is connected to your computer you can open a terminal using minicom and the following configuration:
    
    A - Serial Device         : /dev/ttyUSB0
    B - Lockfile Location     : var/lock
    C - Callin Program        :
    D - Callout Program       :
    E - Bps/Par/Bits          : 115200 8N1
    F - Hardware Flow Control : Yes
    G - Software Flow Control : No

### Creating a Debian USB Installation Stick
The installation process assumes that the APU1D4 is connected to the Internet through the eth0 (the closest to the serial port) interface.
1. Download the last Debian Image from [lisovy's github repository](https://github.com/ssinyagin/pcengines-apu-debian-cd). 
2. Write to an USB stick using dd:
`dd if=/path/image.iso of=/dev/sdb`
3. Plug it into the APU1D4 and boot the board.
4. Follow the instructions.
5. Unplug the USB stick and restart.
6. Log in `user:root password:pcengines` 
7. Install the following packages:
    
    #apt-get install git build-essential  libncurses-dev sharutils pkg-config
    
8. Now you have a Debian system.

Now the board is ready to install the 802.11p drivers and applications. Next
sections follow this [manual](https://gist.github.com/lisovy/80dde5a792e774a706a9).
## Installing 802.11p linux kernel
To install a 802.11p kernel follow these steps:
1. Git clone [this repository](https://github.com/CTU-IIG/802.11p-linux.git).
    
    git clone https://github.com/CTU-IIG/802.11p-linux.git

2. Cd into the directory and checkout the `its-g5_v3` branch

    $cd 802.11p-linux
    $git checkout its-g5_v3
    
3. Create a new directory for the build
    
    $mkdir _build

4. Copy the current kernel configuration included with your Debian:
    
    $cp cp /boot/config-3.16.0-4-amd64 .config
    
5. Configure your kernel, be sure enable MAC80211_*_DEBUG options:

   $make 0=_build oldconfig # Update the oldconfig
   $make 0=_build menuconfig # Configure your kernel
   $grep MAC80211_.*_DEBUG < .config # Check MAC80211_*_DEBUG configuration.
   CONFIG_MAC80211_VERBOSE_DEBUG=y
   CONFIG_MAC80211_MLME_DEBUG=y
   CONFIG_MAC80211_STA_DEBUG=y
   CONFIG_MAC80211_HT_DEBUG=y
   CONFIG_MAC80211_OCB_DEBUG=y
   CONFIG_MAC80211_IBSS_DEBUG=y
   CONFIG_MAC80211_PS_DEBUG=y
   CONFIG_MAC80211_MPL_DEBUG=y
   CONFIG_MAC80211_MPATH_DEBUG=y
   CONFIG_MAC80211_MHWMP_DEBUG=y
   CONFIG_MAC80211_MESH_SYNC_DEBUG=y
   CONFIG_MAC80211_MESH_CSA_DEBUG=y
   CONFIG_MAC80211_MESH_PS_DEBUG=y
   CONFIG_MAC80211_TDLS_DEBUG=y


6. Generate debian kernel packages:

   $make deb-pkg # Compile the kernel and generate Debian packages 

7. Install the new kernel

   #
## Installing Iw
## Installing the Regulatory Database
## Configuring interfaces
## Installing the GeoNetworking Stack
