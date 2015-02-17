# Diet Raspbian - Minimal Raspbian with Ansible

The `diet-raspbian` project aims to trim the fat of the default Raspbian image. I've looked into using other people's minimal Raspbian images, but they're usually out of date by days, weeks, or months, and they aren't built in an open/simple way.

Therefore `diet-raspbian` uses Ansible to take a system built with the official Raspbian image, and strip it of extraneous bits like default IDEs, languages, Wolfram, a window manager, etc.

## Usage

  1. Install [Ansible](http://docs.ansible.com/intro_installation.html).
  2. Edit the `inventory` file and set the IP address to the address of your running Pi.
  3. Ensure you can log into the Pi via SSH using a key (so you don't have to enter a password).
  4. Run the following command: `$ ansible-playbook -i inventory diet.yml`.
  5. Immediately following the playbook's completion, before rebooting the Pi, follow the steps below to 'Create a new diet-raspbian disk image' if you want to create the smallest image (without a swapfile).

After 10-20 minutes, the space consumed by Raspbian should go from ~2.5 GB to ~1.5 GB (or lower, depending on how far along this project has come!). Restart your Raspberry Pi for full effect.

> IMPORTANT: This is meant to be run *prior* to any other Raspberry Pi configuration; it changes locale settings, general configuration, etc. (see `vars/main.yml`). This is meant to be run on a freshly-imaged Raspbian microSD/SD card.

## Creating a new diet-raspbian disk image for cloning

Once you've run the `diet.yml` playbook on your Pi, and save a decent amount of space, you may want to create a new `diet-raspbian.img` disk image that you can use to clone (or re-clone) to your microSD cards, so you don't have to run the `diet.yml` playbook in the future, or if you want to start fresh again on your existing Pi.

Do the following to create an image from your now-trim Raspbian install:

  1. Use `fdisk`, `parted`, or `gdisk` to resize the partition to just a little more than the data on the disk.
    1. TODO
  2. Shut down your Raspberry Pi (`$ sudo shutdown now`) and pop out the microSD card
  3. Pop the microSD card into your Mac's card reader
  4. Locate the card: `$ diskutil list` (should be something like `/dev/disk3`)
  5. Make an image of the card using `dd`:
    1. Compressed image (with `pv`): `$ sudo dd if=/dev/disk6 bs=1m count=1024 | pv | gzip > ~/Desktop/diet-raspbian.img.gz`
    2. Compressed image (without `pv`): `$ sudo dd if=/dev/disk6 bs=1m count=1024 | gzip > ~/Desktop/diet-raspbian.img.gz`
    3. Uncompressed image: `$ sudo dd if=/dev/disk3 of=~/Desktop/diet-raspbian.img bs=1m count=1024`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

> NOTE: The `count=1024` option in the `dd` command limits the copy to 1 GB, so anything futher than 1GB of the partition will not be copied. Make sure you copy at least as much data as is on the partition (e.g. the used space reported in step 1.iii), with a little extra padding. Use a fresh Raspbian install and/or remove the `count` parameter if you're having issues with a bad image.

At this point, you should have a disk image you can write to new SD cards, or use to overwrite your existing SD card.

To write the image back to another microSD card:

  1. Pop the new card into your Mac's card reader
  2. Locate the card: `$ diskutil list` (should be something like `/dev/disk3`)
  3. Unmount any mounted partitions on the card: `$ diskutil unmountDisk /dev/disk3`
  3. Write the image to the new card:
    1. Compressed image (with `pv`): `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | pv | sudo dd of=/dev/disk3 bs=1m`
    2. Compressed image (without `pv`): `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | sudo dd of=/dev/disk3 bs=1m`
    3. Uncompressed image: `$ sudo dd if=~/Desktop/diet-raspbian.img of=/dev/disk3 bs=1m`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

## Author Information

Created in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
