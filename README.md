Using avrdude with the Raspberry Pi
===================================

Since the Raspberry Pi lacks a DTR pin that makes it oh-so-easy to upload your hex files into
the avr, we need this hack to make it just as easy.  When you wire up your atmega chip, be sure
to connect one of the digital gpio pins to the reset pin, and then you'll be able to use avrdude
as if your serial cable actually had a dtr pin.

Instructions:
-------------

Clone the repo to the users homedir, rename the original avrdude to avrdude-original, symlink the autoreset files from your /usr/bin directory and symlink avrdude-autoreset to become avrdude.

    git clone https://github.com/pb66/avrdude-rpi.git ~/avrdude-rpi
    sudo mv /usr/bin/avrdude /usr/bin/avrdude-original
    sudo ln ~/avrdude-rpi/autoreset /usr/bin
    sudo ln ~/avrdude-rpi/avrdude-autoreset /usr/bin
    sudo ln -s /usr/bin/avrdude-autoreset /usr/bin/avrdude

Modify the autoreset script to use the pin that you wired up to the reset pin.  See the line in
autoreset where we do "pin = 4" and change the 4 to your gpio pin number. (RFM2Pi boards use gpio 4)

Now when you run avrdude from anywhere (including via arduino's normal UI) it will flag dtr when
it is about to upload hex data.

http://www.deanmao.com/2012/08/12/fixing-the-dtr-pin/

Make sure Python is installed

    sudo apt-get update
    sudo apt-get install python-dev python-rpi.gpio
    
####Arduino IDE

If using with Arduino IDE on Raspberry Pi a symlink is also required to /dev/ttyAMA0 from the IDE's default target serial port of /dev/ttyS0 (on Linux). 
This symlink will need to be recreated at each boot so a line should added to the rc.local file

    sudo nano /etc/rc.local
    
and at the end of that file just before the "exit 0" line add the line

    sudo ln -s /dev/ttyAMA0 /dev/ttyS0
    
####RFM2Pi requirements

If using the Arduino IDE rather than avrdude from command line there needs to be an additional board type set up.

    sudo nano /usr/share/arduino/hardware/arduino/boards.txt
    
and insert the following section of text (probally nearer to the top for a higher menu position in IDE)

    #############################################################

    atmega328_384_8.name=RFM2Pi v2 (ATmega328 int 8MHz)

    atmega328_384_8.upload.protocol=arduino
    atmega328_384_8.upload.maximum_size=30720
    atmega328_384_8.upload.speed=38400
    
    atmega328_384_8.bootloader.low_fuses=0xE2
    atmega328_384_8.bootloader.high_fuses=0xDE
    atmega328_384_8.bootloader.extended_fuses=0x05
    atmega328_384_8.bootloader.path=optiboot
    atmega328_384_8.bootloader.file=optiboot_atmega328_384_8.hex
    atmega328_384_8.bootloader.unlock_bits=0x3F
    atmega328_384_8.bootloader.lock_bits=0x0F
    
    atmega328_384_8.build.mcu=atmega328p
    atmega328_384_8.build.f_cpu=8000000L
    atmega328_384_8.build.core=arduino
    atmega328_384_8.build.variant=standard
    
For the bootloader files edit Makefile in the optiboot folder

    sudo nano /usr/share/arduino/hardware/arduino/bootloaders/optiboot/Makefile

and insert these lines below the standard "atmega328" section 
    
    # Standard atmega328, only at 38,400 baud for closer clock accuracy AND using 8Mhz internal RC oscillator
    #
    atmega328_384_8: TARGET = atmega328
    atmega328_384_8: MCU_TARGET = atmega328p
    atmega328_384_8: CFLAGS += '-DLED_START_FLASHES=3' '-DBAUD_RATE=38400'
    atmega328_384_8: AVR_FREQ = 8000000L
    atmega328_384_8: LDSECTIONS  = -Wl,--section-start=.text=0x7e00 -Wl,--section-start=.version=0x7ffe
    atmega328_384_8: $(PROGRAM)_atmega328_384_8.hex
    atmega328_384_8: $(PROGRAM)_atmega328_384_8.lst
    
    atmega328_384_8_isp: atmega328
    atmega328_384_8_isp: TARGET = atmega328
    atmega328_384_8_isp: MCU_TARGET = atmega328p
    # 512 byte boot, SPIEN
    atmega328_384_8_isp: HFUSE = DE
    # Int. RC Osc. 8MHz, slowly rising power-65ms
    atmega328_384_8_isp: LFUSE = E2
    # 2.7V brownout
    atmega328_384_8_isp: EFUSE = 05
    atmega328_384_8_isp: isp

Save the file and then "make" the additional files using

    cd /usr/share/arduino/hardware/arduino/bootloaders/optiboot
    sudo make atmega328_384_8
    cd -

http://forum.arduino.cc/index.php?topic=124879.0

###More info

[OEM forum discussion](http://openenergymonitor.org/emon/node/6121)

[RFM2Pi v2 Wiki](http://wiki.openenergymonitor.org/index.php?title=RFM12Pi_V2)

[RFM2Pi software repo](https://github.com/openenergymonitor/RFM2Pi)

[RFM2Pi hardware repo](https://github.com/openenergymonitor/Hardware/tree/master/RFM2Pi)


