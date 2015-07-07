# Diet Raspbian - Minimal Raspbian with Ansible

**Note**: If you just want to use a premade copy of the Diet Raspbian image, you can download it directly from the [Midwestern Mac Files](http://files.midwesternmac.com/) site (under the 'Raspberry Pi Images' section).

The `diet-raspbian` project aims to trim the fat of the default Raspbian image. I've looked into using other people's minimal Raspbian images, but they're usually out of date by days, weeks, or months, and they aren't built in an open/simple way.

Therefore `diet-raspbian` uses Ansible to take a system built with the official Raspbian image, and strip it of extraneous bits like default IDEs, languages, Wolfram, a window manager, etc. Why would this be helpful? If you're running a Raspberry Pi as a small headless server (e.g. for home automation, a fun robot project, or in a clustered configuration), there's no need for all the extra cruft.

## Usage

Everything should be done on your local host machine—nothing needs to be done on the Raspberry Pi itself!

  1. Install [Ansible](http://docs.ansible.com/intro_installation.html).
  2. Build a microSD card with the [official Raspbian image](http://www.raspberrypi.org/downloads/), and boot your Pi:
    1. Boot the Pi with the fresh Rasbpian install.
    2. Copy your public key for passwordless SSH login (e.g. `ssh-copy-id pi@[IP-ADDRESS]`), and make sure you can login to the Pi without a password (e.g. `ssh pi@[IP-ADDRESS]`).
      1. If you want, you can also SSH into the Pi and run `passwd` to change the `pi` account password from the default, `raspberry`.
      2. *Don't* run `raspi-config` at this time.
  3. Edit the `inventory` file and set the IP address to the address of your running Pi.
  4. Run the following command: `$ ansible-playbook -i inventory diet.yml`.

After 10-20 minutes, the space consumed by Raspbian should go from ~2.5 GB to ~700 MB (or lower, depending on how far along this project has come!). If you'd like to create a new image for cloning purposes, run the command `ansible all -i inventory -a "shutdown -h now" -s` to shut down your Pi, then go to step 1 in the next section below.

> IMPORTANT: The `diet.yml` playbook is meant to be run *prior* to any other Raspberry Pi configuration; it changes locale settings, general configuration, etc. (see `vars/main.yml`). This is meant to be run on a freshly-imaged Raspbian microSD/SD card.

## Creating a new Diet Raspian disk image for cloning

Once you've run the `diet.yml` playbook on your Pi, you can create a new `diet-raspbian.img.gz` compressed disk image that you can use to clone (or re-clone) to your microSD cards, so you don't have to run the `diet.yml` playbook in the future, or if you want to quickly rebuild your existing Pi's OS.

  1. With your Raspberry Pi powered off, remove the microSD card and put it into your Mac's card reader.
  2. Resize the ext4 partition (OPTIONAL, but will save a couple GB of space and ~10 minutes per cloned microSD card):
    1. Boot up an Ubuntu VM using VirtualBox, VMWare Fusion, or Parallels Desktop.
    2. Attach the USB device that has the microSD card attached to the VM.
    3. Make sure gparted is installed: `$ sudo apt-get install -y gparted`
    4. Start the gparted GUI: `$ sudo gparted`
    5. Select the microSD card (e.g. `/dev/sdb`) from 'Devices' in the GParted menu.
    6. Right click on the `ext4` (should be 2.99 GB or so) and `boot` volumes and unmount them.
    7. Right click on the the `ext4` volume and resize it to a smaller value (e.g. `850 MB`).
    8. Click the 'Apply' button (green checkbox) to apply the changes (this will take ~10 minutes).
    9. Eject the card from the Ubuntu VM so you can use it from the Mac again.
  3. Locate the card: `$ diskutil list` (should be something like `/dev/disk2`)
  4. Make a compressed image of the card using `dd`:
    1. With `pv`: `$ sudo dd if=/dev/disk2 bs=1m count=1536 | pv | gzip > ~/Desktop/diet-raspbian.img.gz`
    2. Without `pv`: `$ sudo dd if=/dev/disk2 bs=1m count=1536 | gzip > ~/Desktop/diet-raspbian.img.gz`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

> The `count=1536` above will create an image that is 1.5 GB. If it needs to be larger to contain all the partitions on the microSD card, you'll need to increase the size here.

At this point, you should have a disk image you can write to new SD cards, or use to overwrite your existing SD card.

## Write the Diet Raspbian image back to another microSD card

  1. Put the new card into your Mac's card reader.
  2. Locate the card: `$ diskutil list` (should be something like `/dev/disk2`)
  3. Unmount any mounted partitions on the card: `$ diskutil unmountDisk /dev/disk2`
  3. Write the image to the new card:
    1. With `pv`: `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | pv | sudo dd of=/dev/disk2 bs=1m`
    2. Without `pv`: `$ gzip -dc ~/Desktop/diet-raspbian.img.gz | sudo dd of=/dev/disk2 bs=1m`

> WARNING: Double-check that you're using the right `if` disk and `of` or `gzip` destinations; these values will be different on your system.

### Use the fresh microSD card in your Raspberry Pi

  1. Put the freshly-minted card into your Pi.
  2. Wait for the Pi to boot (fingers crossed!).
  3. Log in (either locally or via SSH).
  4. Disable swap and remove the swap file temporarily: `sudo swapoff -a && sudo rm -f /var/swap`
  4. Run `$ sudo raspi-config`, and select the first option ('Expand Filesystem').
  5. Reboot the Raspberry Pi.

## Author Information

Created in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
