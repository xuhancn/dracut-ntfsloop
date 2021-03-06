dracut-ntfsloop: Mount image on NTFS partition
==============================================

This dracut module allows you to use a root system located inside
a disk image on a NTFS partition, similar to how Wubi worked.

## Requirements

* dracut (tested with Fedora 23)
* fuse, ntfs-3g, kpartx, losetup
* fuse and loop kernel modules available

The disk image should contain a full partition table. You'll usually create this
by installing your linux distro of choice in qemu.

## Installation

* copy the `90ntfsloop` directory to `/usr/lib/dracut/modules.d`
* regenerate your initrd using `dracut --force`
* add to your kernel command line: `rd.ntfsloop=/PATH/TO/DEVICE:PATH/TO/IMAGE/IN/PARTITION`
  where `/PATH/TO/DEVICE` can be like `/dev/sda1` but also `UUID=...`.

You'll want to do this while your system is still running inside qemu.

## Booting

To boot the final system, your `/boot` partition will need to be accessible
by your boot loader of choice. You can use grub2's ntfs support combined with
its ability to access partitions on a loopback image to make this a breeze.

I installed grub2 onto a flash drive (
`grub2-install --no-floppy /dev/sdX --boot-directory=/mount/point/of/sdX`) and
set up a config file (`grub2/grub.cfg`) similar to this:

    loopback l (hd0,msdos1)/PATH/TO/DISK.img
    configfile (l,msdos1)/grub2/grub.cfg

By the way: Your disk image is still bootable in qemu, and could be made
bootable in VirtualBox/VMWare too by constructing a suitable `vmdk` file.

## Performance

Not great, because ntfs-3g is involved. Could be improved by calculating the image
file offset on the disk and creating the loop device directly, but that is quite
risky, especially when the image file is fragmented.
