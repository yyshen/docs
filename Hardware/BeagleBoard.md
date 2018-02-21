= seL4 on the BeagleBoard =

This page documents booting seL4 on the
\[\[<http://beagleboard.org/beagleboard%7CBeagleboard>\]\], Omap
3530See.

\[\[<http://sel4.systems/pipermail/devel/2014-August/000030.html%7CTim>
Newsham's post\]\] on the mailing list for one user's experience.

== Preparing your SD card == === Prologue === These instructions are for
later versions of the Beagleboard. Before the Xm, U-Boot and MLO were
held in flash.

The first stage boot loader expects to find the TI X-loader in the root
of a FAT filesystem, on the first partition of the SD card with the name
MLO.

MLO expects to find a file named u-boot.bin in the root directory of the
SD card.

== Setting up Minicom == Plug the board in. seL4 userspace currently
does no power management — you will probably need a 5V power source, and
not rely on powering the board over USB

If you do not have minicom installed:

{{{\#!highlight bash numbers=off sudo apt-get install minicom }}} If you
are connecting via a USB serial adapter:

{{{\#!highlight bash numbers=off sudo minicom -s ttyUSB0 }}} And if you
were connecting via a "real" serial port:

{{{\#!highlight bash numbers=off sudo minicom -s ttyS0 }}} In either
case, this will take you to a configuration menu.

> 1.  Choose Serial Port Setup
> 2.  Set A: Serial Device to /dev/ttyUSB0 or /dev/ttyS0 depending on
>     which serial device you want to use.
> 3.  Set F: Hardware Flow Control to No
> 4.  Set speed to 115200 and eight bit no parity.
> 5.  Save setup as ttyUSB0 or ttyS0
> 6.  Exit Minicom

You can now connect to the !BeagleBoard using Minicom:

{{{\#!highlight bash numbers=off minicom ttyUSB0 }}} Or:

{{{\#!highlight bash numbers=off minicom ttyS0 }}} === Permissions ===
If you get permissions errors you need to add yourself to the
appropriate group. Find out which group on your machine has access to
the serial ports (on Debian, it's usually dialout):

{{{\#!highlight bash numbers=off \$ ls -l /dev/ttyUSB0 crw-rw---- 1 root
dialout 188, 0 Aug 11 09:43 /dev/ttyUSB0 }}}

Then add yourself to the right group:

{{{\#!highlight bash numbers=off sudo usermod -G dialout -a
your\_login\_name }}} === U-Boot === Now minicom should connect to what
it thinks is a "modem", and then give you a good old console to work
with. You are now in the bootloader, U-Boot, of the \~BeagleBoard. You
can type commands here and it'll display the results.

Some quick useful commands: ||&lt;tablewidth="600px"&gt;'''Command'''
||'''Description''' || ||help ||display list of commands || ||printenv
||lists defined environment variables || ||mmc init ||initialise MMC (to
read the the SD card) || ||mmcinfo ||display current SD card info ||
||fatls mmc 0 ||display list of files on SD card 0 || ||fatload ||load
script/image into some RAM address to be run || ||run ||run scripts ||
||bootelf ||boot into en ELF image ||

== Running seL4test == Get the sel4test-manifest repo using the
instructions at
\[\[<https://sel4.systems/Info/Hardware/home.pml%7CDownload>\]\]

Then run:

{{{\#!highlight bash numbers=off make beagle\_debug\_xml\_defconfig make
}}} Which after a few minutes should give you:

{{{ \[GEN\_IMAGE\] sel4test-image-arm-omap3 }}} Now, the ELF image we
boot into is the sel4test-image-arm file. Pull out the SD card, put it
into the SD card reader and plug into your computer, then copy that file
into the boot sector, then sync and remove, then plug the SD card back
into the !BeagleBoard. Pretty self-explanatory.

Reset the !BeagleBoard by pressing the "S2" (reset) button.

=== To run the image: === {{{ mmc init mmcinfo fatload mmc 0
\${loadaddr} sel4test-image-arm bootelf \${loadaddr} }}} where loadaddr
is some address, in this example defined as an environment variable.
After this you should start seeing output from seL4test.

You can use:

{{{ fatls mmc 0 }}} to see what is on the SD card.