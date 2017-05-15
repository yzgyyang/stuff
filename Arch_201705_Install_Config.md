# Arch 201705 Dual-boot Install Config  

Charlie's guide to the installation and configuration of ArchLinux 201705, dual-boot with Windows 10.   
This guide is based on my experiences on my laptop (ThinkPad X1 Carbon 3rd), and these procedures **should not be assumed the same on other machines**.  

# Steps  

## Spare some free space for Arch  

Right click "This PC" then "Manage", "Disk Management". Select any partition and shrink it. I recommend the free space to be over 15GB. **Don't create any new partitions yet, we will leave that for later.**  


## Make a USB installation disk  

Use [unetbootin](https://unetbootin.github.io/) on any platforms to download the system image, wipe the USB disk and make a UEFI startup disk in one click.  
*Be sure to select the correct architecture and system image version.*  

After [unetbootin](https://unetbootin.github.io/) finishes its job, **check if the volume name of the USB appears as something like `ARCH_201706`**. If not, change to it (or you may encounter a problem, which we will cover in the next step).  

## Reboot into ArchLinux on USB

Go into BIOS and turn off some Windows-featured options (fast boot, secure boot, etc.). Insert your USB stick, restart the computer and select USB to be the startup device. Press enter and wait a while until the command line pops up.  

If the ArchLinux image doesn't boot up properly, remove the `quiet` flag to the correspondent bootloader option to get more info.  

*You may encounter a problem (at least I did) when the startup process hangs at an error message:*

```
ERROR: '/dev/disk/by-label' device did not show up after 30 seconds...  
```

*I resolved it by renaming the volume name like `Arch_201706`. I think unetbootin don't know this, but some other tools might do.*  

## Create partitions  

## Set up Internet connection  

## Install base and necessary tools  

## Install bootloader  

{Work in Progress}
