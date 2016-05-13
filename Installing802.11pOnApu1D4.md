


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
    
    ```
    #apt-get install git build-essential libncurses-dev sharutils pkg-config \\
    libnl-dev libnl-genl-3-dev python-dev python-pip python-m2crypto libgcrypt11-dev
    ```
    
8. Now you have a Debian system.

Now the board is ready to install the 802.11p drivers and applications. Next
sections follow this [manual](https://gist.github.com/lisovy/80dde5a792e774a706a9).

## Installing 802.11p linux kernel

To install a 802.11p kernel follow these steps:
1. Git clone [this repository](https://github.com/GRCDEV/802.11p-linux.git).
    
    `git clone https://github.com/CTU-IIG/802.11p-linux.git`

2. Cd into the directory and checkout the `its-g5_v3` branch

```
$cd 802.11p-linux
$git checkout its-g5_v3
```
    
3. Create a new directory for the build
    
    `$mkdir _build`

4. Copy the current kernel configuration included with your Debian:
    
    `$cp cp /boot/config-3.16.0-4-amd64 .config`
    
5. Configure your kernel, be sure enable MAC80211_*_DEBUG options:

    ```
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
    ```
    
6. Generate debian kernel packages:

    `$make deb-pkg # Compile the kernel and generate Debian packages`

7. Install the new kernel

    ```
    # ls ../*.deb
    ../linux-firmware-image-3.18.0+_3.18.0+-1_amd64.deb
    ../linux-headers-3.18.0+_3.18.0+-1_amd64.deb
    ../linux-image-3.18.0+_3.18.0+-1_amd64.deb
    ../linux-image-3.18.0+-dbg_3.18.0+-1_amd64.deb
    ../linux-libc-dev_3.18.0+-1_amd64.deb
    # for i in ../*.deb; do dpkg --install $i; done
    ```

Now you are ready to install userspace tools to use the new features in your
kernel.

## Installing Iw
To install `iw` follow these steps:

1. Clone the [iw repository](https://github.com/CTU-IIG/802.11p-iw.git)

    ```
    git clone https://github.com/CTU-IIG/802.11p-iw.git
    ```

2. Get into the directory and checkout the its-g5_v3 branch 

    ```
    $cd 802.11p-iw
    $checkout its-g5_v3
    ```
    
3. There is a bug on the iw Makefile we need to fix:

    ```
    #Replace 
    $(Q)$(CC) $(LDFLAGS) $(OBJS) $(LIBS) -o iw
    #With
    $(Q)$(CC) $(LDFLAGS) $(OBJS) $(LIBS) -I /usr/include/libnl3 -o iw
    ```

3. Build and install it

    ```
    $make
    $sudo make install
    ```

4. Test it

    ```
    $iw |grep -i ocb
    dev <devname> ocb join <freq in MHz> <5MHZ|10MHZ>
    dev <devname> ocb leave
    ```

## Installing the Regulatory Database

Now it is time to install a new regulatory database so we can configure our
wireless device in OCB mode and tune it at 5.9Ghz. To install the new regulatory
database follow these steps:

1. Clone the [802.11p-wireless-regdb](802.11p-wireless-regdb) repository in your
working directory.

    `git clone https://github.com/CTU-IIG/802.11p-wireless-regdb.git`

2. Get into it and checkout the branch its-g5_v3

    ```
    $cd 802.11p-wireless-regdb
    $git checkout its-g5_v3
    ```

3. Change the db.txt file to copy the OCB line into your country section which
is originally included only for Germany.

    ```
    #For ITS-G5 evaluation
    (5850 – 5925 @ 20), (100 mW), NO-CCK, OCB-ONLY
    ```

4. Build it and install it.

    ```
    $make
    $sudo make install
    ```

5. Go back to your working directory and clone [this repository](https://github.com/CTU-IIG/802.11p-crda.git)
    
    ```
    $cd ..
    $git clone https://github.com/CTU-IIG/802.11p-crda.git
    ```

6. Checkout the `its-g5_v1` branch.

    `git checkout its-g5_v1`

7. Copy your recently created pubkey to the pubkeys directory.

    `# cp /usr/lib/crda/pubkeys/root.key.pub.pem pubkeys/`

8. Build it and install it

```
    $make
    $make install
```

9. Check your regulatory configuration.

```
$regdbdump /usr/lib/crda/regulatory.bin | less
country ES: DFS-ETSI
        (2400.000 - 2483.500 @ 40.000), (20.00)
        (5150.000 - 5250.000 @ 80.000), (20.00), NO-OUTDOOR
        (5250.000 - 5350.000 @ 80.000), (20.00), NO-OUTDOOR
        (5470.000 - 5725.000 @ 80.000), (26.98), DFS
        (5850.000 - 5925.000 @ 20.000), (20.00), NO-CCK, OCB-ONLY
        (57240.000 - 65880.000 @ 2160.000), (40.00), NO-OUTDOOR
```

## Configuring interfaces
Now we can configure our interface so it will be configured at startup time.
Edit the `/etc/network/interfaces` file and add an entry for your wireless
interface (`wlan0` in our case) like this.

```
# JdJ20151204: OCB entry (802.11p)
auto wlan0
iface wlan0 inet static
address 192.168.97.101
# replace with your IPv4 address
netmask 255.255.255.0
broadcast 192.168.97.255
pre-up iw reg set ES
# replace with your CC
pre-up iw dev wlan0 set type ocb
post-up iw dev wlan0 ocb join 5900 10MHZ
```

Restart your system now and check the status of your interface.

1. Check the low level configuration and be sure 5.9 Ghz channels are enabled.

```
# iw phy0 info
Wiphy phy0
[....]
                        * 5855 MHz [171] (disabled)
                        * 5860 MHz [172] (20.0 dBm) (no IR)
                        * 5865 MHz [173] (20.0 dBm) (no IR)
                        * 5870 MHz [174] (20.0 dBm) (no IR)
                        * 5875 MHz [175] (20.0 dBm) (no IR)
                        * 5880 MHz [176] (20.0 dBm) (no IR)
                        * 5885 MHz [177] (20.0 dBm) (no IR)
                        * 5890 MHz [178] (20.0 dBm) (no IR)
                        * 5895 MHz [179] (20.0 dBm) (no IR)
                        * 5900 MHz [180] (20.0 dBm) (no IR)
                        * 5905 MHz [181] (20.0 dBm) (no IR)
                        * 5910 MHz [182] (20.0 dBm) (no IR)
                        * 5915 MHz [183] (20.0 dBm) (no IR)
                        * 5920 MHz [184] (disabled)
                        * 5925 MHz [185] (disabled)
[....]
```

2. Check the wlan0 status
```
# iw wlan0 info
Interface wlan0
        ifindex 5
        wdev 0x1
        addr 04:f0:21:1e:48:05
        type outside context of a BSS
        wiphy 0
        channel 180 (5900 MHz), width: unknown, center1: 5900 MHz
```

## Next Steps

Now we can use the IP stack to check the performance of our new 802.11p nic.
In the case we want to install a ITS-G5 stack we can follow this 
[document](http://gcdc.net/images/doc/REP.GN-with-OpenVPN.pdf) from the GCDC 
project.

## Resources

* https://gist.github.com/lisovy/80dde5a792e774a706a9
* https://github.com/ssinyagin/pcengines-apu-debian-cd
* http://gcdc.net/images/doc/REP.GN-with-OpenVPN.pdf
* http://gcdc.net/images/doc/REP.802.11p-on-APU1D.pdf
* http://gcdc.net/en/teams/rules-and-technology


