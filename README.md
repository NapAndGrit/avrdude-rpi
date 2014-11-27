Using avrdude with the Raspberry Pi
===================================

Since the Raspberry Pi lacks a DTR pin that makes it oh-so-easy to upload your hex files into
the avr, we need this hack to make it just as easy.  When you wire up your atmega chip, be sure
to connect one of the digital gpio pins to the reset pin, and then you'll be able to use avrdude
as if your serial cable actually had a dtr pin.

Instructions:
-------------

Clone the repo to the users homedir, rename the original avrdude to avrdude-original, symlink the autoreset file from your /usr/bin directory and symlink avrdude-autoreset to become avrdude.

    git clone https://github.com/pb66/avrdude-rpi.git ~/avrdude-rpi
    sudo mv /usr/bin/avrdude /usr/bin/avrdude-original
    sudo ln -s ~/avrdude-rpi/autoreset /usr/bin
    sudo ln -s ~/avrdude-rpi/avrdude-autoreset /usr/bin/avrdude

Modify the autoreset script to use the pin that you wired up to the reset pin.  See the line in
autoreset where we do "pin = 4" and change the 4 to your gpio pin number. (RFM2Pi boards use gpio 4)

Now when you run avrdude from anywhere (including via arduino's normal UI) it will flag dtr when
it is about to upload hex data.

http://www.deanmao.com/2012/08/12/fixing-the-dtr-pin/

Make sure Python is installed

    sudo apt-get update
    sudo apt-get install python-dev python-rpi.gpio

If using with Arduino IDE on Raspberry Pi a symlink is also required to /dev/ttyAMA0 from the IDE's default target serial port of /dev/ttyS0 (on Linux). 
This symlink will need to be recreated at each boot so a line should added to the rc.local file

    sudo nano /etc/rc.local
    
and at the end of that file just before the "exit 0" line add the line

    sudo ln -s /dev/ttyAMA0 /dev/ttyS0
    
the user needs to be added to the "tty" group 
