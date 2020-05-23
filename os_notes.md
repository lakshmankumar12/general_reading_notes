# Linux Boot Process

article: https://opensource.com/article/17/2/linux-boot-and-startup

## steps

* BIOS POST
* Boot loader (GRUB2)
* Kernel initialization
* Start systemd, the parent of all processes.

## BIOS

* Done by hardware, independant of OS.

BIOS POST checks the basic operability of the hardware and then it issues a
BIOS interrupt, INT 13H, which locates the boot sectors on any attached
bootable devices. The first boot sector it finds that contains a valid boot
record is loaded into RAM and control is then transferred to the code that was
loaded from the boot sector.

The boot sector is really the first stage of the boot loader. There are three
boot loaders used by most Linux distributions, GRUB, GRUB2, and LILO. GRUB2 is
the newest and is used much more frequently these days than the other older
options.

## GRUB2

* Grand Unified Bootloader2
* can boot many versions of Linux and other free operating systems; it can also
  chain load the boot record of proprietary operating systems.
* configured using the `/boot/grub2/grub.conf`

### Stage-1

The bootstrap code, i.e., GRUB2 stage 1, is very small because it must fit
into the first 512-byte sector on the hard drive along with the partition
table.

The total amount of space allocated for the actual bootstrap code in a classic
generic MBR is 446 bytes. The 446 Byte file for stage 1 is named `boot.img` and
does not contain the partition table which is added to the boot record
separately.

The sole purpose of stage 1 is to locate and load stage 1.5

### Stage-1.5

In order to accomplish this, stage 1.5 of GRUB must be located in the space
between the boot record itself and the first partition on the drive.
This space was historically unused.

The first partition on the hard drive begins at sector 63 and with the MBR in
sector 0, that leaves 62 512-byte sectors—31,744 bytes—in which to store the
`core.img` file which is stage 1.5 of GRUB. The core.img file is 25,389 Bytes so
there is plenty of space available between the MBR and the first disk partition
in which to store it.

stage-1.5, can understand a few file-systems - extN's, FAT, NTFS.

So, the stage-2 can be on partitions with those file-systems. However, it can't
read logical volumes.

The standard location is `/boot/grub2`

The /boot directory must be located on a filesystem that is supported by GRUB

### Stage-2

GRUB2 does not have an image file like stages 1 and 2. Instead, it consists
mostly of runtime kernel modules that are loaded as needed from the
`/boot/grub2/i386-pc` directory.

The function of GRUB2 stage 2 is to locate and load a Linux kernel into RAM and
turn control of the computer over to the kernel.

The kernel and its associated files are located in the /boot directory

The kernel files are identifiable as they are all named starting with vmlinuz

## Kernel

All of the kernels are in a self-extracting, compressed format to save space.
The kernels are located in the /boot directory, along with an initial RAM disk
image, and device maps of the hard drives.


