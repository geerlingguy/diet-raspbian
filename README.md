# Diet Raspbian - Minimal Raspbian with Ansible

The `diet-raspbian` project aims to trim the fat of the default Raspbian image. I've looked into using other people's minimal Raspbian images, but they're usually out of date by days, weeks, or months, and they aren't built in an open/simple way.

Therefore `diet-raspbian` uses Ansible to take a system built with the official Raspbian image, and strip it of extraneous bits like default IDEs, languages, Wolfram, a window manager, etc.

## Usage

Everything should be done on your local host machine—nothing needs to be done on the Raspberry Pi itself!

  1. Install [Ansible](http://docs.ansible.com/intro_installation.html).
  2. Build a microSD card with the [official Raspbian image](http://www.raspberrypi.org/downloads/), and boot your Pi:
    1. Boot the Pi with the fresh Rasbpian install.
    2. Log in over the network: `$ ssh pi@[IP-ADDRESS]`
    3. Run `$ sudo raspi-config` and change the `pi` account password (using the `Change User Password` option), but *don't expand the filesystem to fill the card's free space*, or change any other settings! For easier configuration, you should also copy your SSH key to the `pi` account's `.ssh/authorized_keys` file at this time.
    4. Log out from your Pi.
  3. Edit the `inventory` file and set the IP address to the address of your running Pi.
  4. Ensure you can log into the Pi via SSH using a key (so you don't have to enter a password).
  5. Run the following command: `$ ansible-playbook -i inventory diet.yml`.

After 10-20 minutes, the space consumed by Raspbian should go from ~2.5 GB to ~700 MB (or lower, depending on how far along this project has come!). If you'd like to create a new image for cloning purposes, run the command `ansible all -i inventory -a "shutdown -h now" -s` to shut down your Pi, then go to step 1 in the next section below.

> IMPORTANT: The `diet.yml` playbook is meant to be run *prior* to any other Raspberry Pi configuration; it changes locale settings, general configuration, etc. (see `vars/main.yml`). This is meant to be run on a freshly-imaged Raspbian microSD/SD card.

## Creating a new diet-raspbian disk image for cloning

Once you've run the `diet.yml` playbook on your Pi, and save a decent amount of space, you may want to create a new `diet-raspbian.img` disk image that you can use to clone (or re-clone) to your microSD cards, so you don't have to run the `diet.yml` playbook in the future, or if you want to start fresh again on your existing Pi.

**Create an image from your now-trim Raspbian install**

  1. With your Raspberry Pi powered off, remove the microSD card and put it into your Mac's card reader
  2. Resize the ext4 partition (OPTIONAL, but will save a couple GB of space and ~10 minutes per cloned microSD card):
    1. Boot up an Ubuntu VM using VirtualBox, VMWare Fusion, or Parallels Desktop
    2. Attach the USB device that has the microSD card attached to the VM
    3. Make sure gparted is installed (e.g. `$ sudo apt-get install -y gparted`)
    4. Start the gparted GUI: `$ sudo gparted`
    5. Select the microSD card (e.g. `/dev/sdb`) from 'Devices' in the GParted menu.
    6. Right click on the `ext4` (should be 2.99 GB or so) and `boot` volumes and unmount them
    7. Right click on the the `ext4` volume and resize it to a smaller value (e.g. `850 MB`)
    8. Click the 'Apply' button (green checkbox) to apply the changes (this will take ~10 minutes)
    9. Eject the card from the Ubuntu VM so you can use it from the Mac again.
  3. Locate the card: `$ diskutil list` (should be something like `/dev/disk2`)
  4. Make a compressed image of the card using `dd`:
    1. With `pv`: `$ sudo dd if=/dev/disk2 bs=1m count=1536 | pv | gzip > ~/Desktop/diet-raspbian.img.gz`
    2. Without `pv`: `$ sudo dd if=/dev/disk2 bs=1m count=1536 | gzip > ~/Desktop/diet-raspbian.img.gz`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

> The `count=1536` above will create an image that is 1.5 GB. If it needs to be larger to contain all the partitions on the microSD card, you'll need to increase the size here.

At this point, you should have a disk image you can write to new SD cards, or use to overwrite your existing SD card.

**Write the image back to another microSD card**

  1. Put the new card into your Mac's card reader
  2. Locate the card: `$ diskutil list` (should be something like `/dev/disk2`)
  3. Unmount any mounted partitions on the card: `$ diskutil unmountDisk /dev/disk2`
  3. Write the image to the new card:
    1. With `pv`: `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | pv | sudo dd of=/dev/disk2 bs=1m`
    2. Without `pv`: `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | sudo dd of=/dev/disk2 bs=1m`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

**Expand the image to fill the new microSD card (on the Pi)**

  1. Put the freshly-minted card into your Pi
  2. Wait for the Pi to boot (fingers crossed!)
  3. Log in (either locally or via SSH)
  4. Disable swap and remove the swap file temporarily: `sudo swapoff -a && sudo rm -f /var/swap`
  4. Run `$ sudo raspi-config`, and select the first option ('Expand Filesystem')
  5. Reboot the Raspberry Pi

## Author Information

Created in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
