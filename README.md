# Diet Raspbian - Minimal Raspbian with Ansible

The `diet-raspbian` project aims to trim the fat of the default Raspbian image. I've looked into using other people's minimal Raspbian images, but they're usually out of date by days, weeks, or months, and they aren't built in an open/simple way.

Therefore `diet-raspbian` uses Ansible to take a system built with the official Raspbian image, and strip it of extraneous bits like default IDEs, languages, Wolfram, a window manager, etc.

## Usage

  1. Install [Ansible](http://docs.ansible.com/intro_installation.html).
  2. Build a microSD card with the [official Raspbian image](http://www.raspberrypi.org/downloads/), and boot your Pi:
    1. Boot the Pi with the fresh Rasbpian install.
    2. Log in over the network: `$ ssh pi@[IP-ADDRESS]`
    3. Run `$ sudo raspi-config` and set initial options, but *don't expand the filesystem to fill the card's free space*!
  3. Edit the `inventory` file and set the IP address to the address of your running Pi.
  4. Ensure you can log into the Pi via SSH using a key (so you don't have to enter a password).
  5. Run the following command: `$ ansible-playbook -i inventory diet.yml`.
  6. Immediately following the playbook's completion, before rebooting the Pi, follow the steps below to 'Create a new diet-raspbian disk image' if you want to create the smallest image (without a swapfile).

After 10-20 minutes, the space consumed by Raspbian should go from ~2.5 GB to ~1.5 GB (or lower, depending on how far along this project has come!). Restart your Raspberry Pi for full effect.

> IMPORTANT: The `diet.yml` playbook is meant to be run *prior* to any other Raspberry Pi configuration; it changes locale settings, general configuration, etc. (see `vars/main.yml`). This is meant to be run on a freshly-imaged Raspbian microSD/SD card.

## Creating a new diet-raspbian disk image for cloning

Once you've run the `diet.yml` playbook on your Pi, and save a decent amount of space, you may want to create a new `diet-raspbian.img` disk image that you can use to clone (or re-clone) to your microSD cards, so you don't have to run the `diet.yml` playbook in the future, or if you want to start fresh again on your existing Pi.

**Create an image from your now-trim Raspbian install**

  1. Write zeros to the free space on the microSD card (this will take ~10 minutes):
    1. `$ sudo dd if=/dev/zero of=/zerofile bs=1024` (until it errors out on filling the drive)
    2. `$ sudo rm -f /zerofile` (to delete the file)
  2. Shut down your Raspberry Pi (`$ sudo shutdown now`) and pop out the microSD card
  3. Pop the microSD card into your Mac's card reader
  4. Resize the ext4 partition (OPTIONAL, but will save a couple GB of space and ~10 minutes per cloned microSD card):
    1. Boot up an Ubuntu VM using VirtualBox, VMWare Fusion, or Parallels Desktop
    2. Attach the USB device that has the microSD card attached to the VM
    3. Make sure gparted is installed (e.g. `$ sudo apt-get install -y gparted`)
    4. Start the gparted GUI: `$ sudo gparted`
    5. Select the microSD card (e.g. `/dev/sde`).
    6. Right click on the `ext4` volume (should be 2.99 GB or so) and unmount it
    7. Resize the `ext4` volume to a smaller value (e.g. `797.00MB`)
    8. Click the 'Apply' button to apply the changes (this will take ~10 minutes)
    9. Eject the card from the Ubuntu VM so you can use it from the Mac again.
  5. Locate the card: `$ diskutil list` (should be something like `/dev/disk3`)
  6. Make an image of the card using `dd`:
    1. Compressed image (with `pv`): `$ sudo dd if=/dev/disk3 bs=1m count=1280 | pv | gzip > ~/Desktop/diet-raspbian.img.gz`
    2. Compressed image (without `pv`): `$ sudo dd if=/dev/disk3 bs=1m count=1280 | gzip > ~/Desktop/diet-raspbian.img.gz`
    3. Uncompressed image: `$ sudo dd if=/dev/disk3 of=~/Desktop/diet-raspbian.img bs=1m count=1280`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

> The `count=1280` above will create an image that is 1.25 GB. If it needs to be larger to contain all the partitions on the microSD card, you'll need to increase the size here. The settings here should work with a `count` of `1280`.

At this point, you should have a disk image you can write to new SD cards, or use to overwrite your existing SD card.

**Write the image back to another microSD card**

  1. Pop the new card into your Mac's card reader
  2. Locate the card: `$ diskutil list` (should be something like `/dev/disk3`)
  3. Unmount any mounted partitions on the card: `$ diskutil unmountDisk /dev/disk3`
  3. Write the image to the new card:
    1. Compressed image (with `pv`): `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | pv | sudo dd of=/dev/disk3 bs=1m`
    2. Compressed image (without `pv`): `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | sudo dd of=/dev/disk3 bs=1m`
    3. Uncompressed image: `$ sudo dd if=~/Desktop/diet-raspbian.img of=/dev/disk3 bs=1m`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

**Expand the image to fill the new microSD card (on the Pi)**

  1. Pop the freshly-minted card into your Pi
  2. Wait for the Pi to boot (fingers crossed!)
  3. Log in (either locally or via SSH) and run `$ sudo raspi-config`
  4. 

## Author Information

Created in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
