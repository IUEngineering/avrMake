# avrMake
A make script to compile source code for avr microcontrollers and program avr microcontrollers


# Installation
To make use of these make files some dependencies must be installed first.

* AVR 8-Bit toolchain
* avrdude

## Linux

### AVR 8-Bit toolchain
The AVR 8-Bit toolchain for linux can be downloaded from [microchips website](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers). 
The downloaded file is a tar.gz file containing the AVR 8-Bit toolchain. </br>
Extract the downloaded tar.gz file and place `avr8-gnu-toolchain-linux_x86_64/` in the  `/opt/` directory. </br>
Now add `/opt/avr8-gnu-toolchain-linux_x86_64/bin` to the system `$PATH` variable.

To check if the toolchain is installed correct run:
```console
foo@bar: ~ $ avr-gcc --version
```
This should output the following:
```console
avr-gcc (GCC) <VERSION>
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
where `<VERSION>` is the version of avr-gcc.

### avrdude
Install avrdude with:
```console
foo@bar: ~ $ sudo apt install avrdude
```
To check if avrdude is installed correctly run:
```console
foo@bar: ~ $ avrdude -v
```
This should output the following
```console
avrdude: Version <VERSION>
         Copyright (c) 2000-2005 Brian Dean, http://www.bdmicro.com/
         Copyright (c) 2007-2014 Joerg Wunsch

         System wide configuration file is "/etc/avrdude.conf"
         User configuration file is "/home/tychoj/.avrduderc"
         User configuration file does not exist or is not a regular file, skipping


avrdude: no programmer has been specified on the command line or the config file
         Specify a programmer using the -c option and try again
```
where `<VERSION>` is the version of avrdude.

## Mac
Download the AVR 8-Bit toolchain for mac from [microchips website](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers)

Extract the downloaded tar.gz file and add `yourPath/avr8-gnu-toolchain-darwin_x86_64/bin` to the system `$PATH`

Download avrdude [from gnu](http://download.savannah.gnu.org/releases/avrdude/), extract the tar.gz and add the bin directory to the `$path`.



## Windows
For windows there is another dependency: msys2.

To install msys go to the [msys2 websit](https://www.msys2.org/) and download the installer and use the installer to install msys2.

Start the msys2 shell and run the following commands:
```console
user@pcName MINGW64 ~
$ pacman -Syu

user@pcName MINGW64 ~
$ pacman -S --needed base-devel mingw-w64-x86_64-toolchain
```

Install the AVR 8-Bit toolchain in the msys2 shell
```console
user@pcName MINGW64 ~
$ pacman -S mingw-w64-x86_64-avr-toolchain
```

Install avrdude in the msys2 shell
```console
user@pcName MINGW64 ~
$ pacman -S mingw-w64-x86_64-avrdude
```


# Usage
To make use of this make file system the project should have the following structure:
```

├── make.avr.mk
├── Makefile
├── libs
│   ├── a
│   ├── b
│   └── c
└── src
    ├── blink.c
    └── subdir
```
The libs folder is not necessary but can be used to differentiate between libraries that are written to be used within multiple projects and libraries specifically written for the current application.

The main advantage is when using git submodules for libraries that are written to be used for multiple projects is that now all the git submodules can be placed at one location.

As the second step in setting up the project change `[your project name]` the your own project name (this can be anything but there should be no spaces in the project name) 
```makefile
# Project name
PROJECTNAME = [your project name]
```

Add the name of the AVR microcontroller at `[avr microcontroller]`. See the avr-libc [user manual](https://www.nongnu.org/avr-libc/user-manual/using_tools.html) if you don't know what the correct name is of your AVR microcontroller. The name can be found under the `-mmcu` flag and is the `MCU name` column. 
```makefile
# The AVR microcontroller 
MICROCONTROLLER = [avr microcontroller]
```

To build the project go to the project directory with the terminal in linux/mac and on windows go to the project directory with the minggw64 shell. The `make all` command will compile and link the source code.
```console
foo@bar: ~/pathToProject/ $ make all
```

The `make test` command will test the connection between the programmer and the microcontroller 
```console
foo@bar: ~/pathToProject/ $ make test
```

The `make flash` command will compile and link the source code and write the compiled hex-file to the flash memory of the microcontroller.
```console
foo@bar: ~/pathToProject/ $ make flash
```

The `make clean` command will delete all generated .hex, .elf, and .o files.
```console
foo@bar: ~/pathToProject/ $ make clean
```

The `make distclean` command will delete all generated .hex, .elf and .o files as wel as the `$(BINFOLDER)` and the `$(OBJFOLDER)` folders.
```console
foo@bar: ~/pathToProject/ $ make distclean
```

For more information run `make help`
```console
foo@bar: ~/pathToProject/ $ make help
```

# Known problems

Some known problems are described here. Some of these problems are not a problem with this tool but with the dependencies of this tool.

## ATtiny
Some ATtiny chips are not (yet) known by libc and require the downloading of extra files. These files can be found at [Microchips website](http://packs.download.atmel.com/).
Download the `Atmel ATtiny Series Device Support` file is a zip file eventhough it has the file extension `.atpack`.

Make a temporary folder in which the `Atmel.ATtiny_DFP.2.0.368.atpack` zip archive can be extracted. and run the following commands 
(These are the commands on linux for when `avr8-gnu-toolchain-linux_x86_64/` is installed in the `/opt/` directory)
```console
foo@bar: ~/Downloads/tmp/ $ sudo cp include/avr/iotn?*1[2467].h /opt/avr8-gnu-toolchain-linux_x86_64/avr/include/avr/

foo@bar: ~/Downloads/tmp/ $ sudo cp gcc/dev/attiny?*1[2467]/avrxmega3/*.{o,a} /opt/avr8-gnu-toolchain-linux_x86_64/avr/lib/avrxmega3/

foo@bar: ~/Downloads/tmp/ $ sudo cp gcc/dev/attiny?*1[2467]/avrxmega3/short-calls/*.{o,a} /opt/avr8-gnu-toolchain-linux_x86_64/avr/lib/avrxmega3/short-calls/
```
At last `/opt/avr8-gnu-toolchain-linux_x86_64/avr/include/avr/io.h` add the following:
```c
#elif defined (__AVR_ATtiny212__)
#  include <avr/iotn212.h>
#elif defined (__AVR_ATtiny412__)
#  include <avr/iotn412.h>
#elif defined (__AVR_ATtiny214__)
#  include <avr/iotn214.h>
#elif defined (__AVR_ATtiny414__)
#  include <avr/iotn414.h>
#elif defined (__AVR_ATtiny814__)
#  include <avr/iotn814.h>
#elif defined (__AVR_ATtiny1614__)
#  include <avr/iotn1614.h>
#elif defined (__AVR_ATtiny3214__)
#  include <avr/iotn3214.h>
#elif defined (__AVR_ATtiny416__)
#  include <avr/iotn416.h>
#elif defined (__AVR_ATtiny816__)
#  include <avr/iotn816.h>
#elif defined (__AVR_ATtiny1616__)
#  include <avr/iotn1616.h>
#elif defined (__AVR_ATtiny3216__)
#  include <avr/iotn3216.h>
#elif defined (__AVR_ATtiny417__)
#  include <avr/iotn417.h>
#elif defined (__AVR_ATtiny817__)
#  include <avr/iotn817.h>
#elif defined (__AVR_ATtiny1617__)
#  include <avr/iotn1617.h>
#elif defined (__AVR_ATtiny3217__)
#  include <avr/iotn3217.h>
```

A big thank you goes to LeoNerd who wrote [this article](http://leonerds-code.blogspot.com/2019/06/building-for-new-attiny-1-series-chips.html) on how to add ATtiny support.
