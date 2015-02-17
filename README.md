# Diet Raspbian - Minimal Raspbian with Ansible

The `diet-raspbian` project aims to trim the fat of the default Raspbian image. I've looked into using other people's minimal Raspbian images, but they're usually out of date by days, weeks, or months, and they aren't built in an open/simple way.

Therefore `diet-raspbian` uses Ansible to take a system built with the official Raspbian image, and strip it of extraneous bits like default IDEs, languages, Wolfram, a window manager, etc.

## Usage

  1. Install [Ansible](http://docs.ansible.com/intro_installation.html).
  2. Edit the `inventory` file and set the IP address to the address of your running Pi.
  3. Ensure you can log into the Pi via SSH using a key (so you don't have to enter a password).
  4. Run the following command: `$ ansible-playbook -i inventory diet.yml`.

After 10-20 minutes, the space consumed by Raspbian should go from ~2.5 GB to ~1.5 GB (or lower, depending on how far along this project has come!). Restart your Raspberry Pi for full effect.

> IMPORTANT: This is meant to be run *prior* to any other Raspberry Pi configuration; it changes locale settings, general configuration, etc. (see `vars/main.yml`). This is meant to be run on a freshly-imaged Raspbian microSD/SD card.

## Author Information

Created in 2015 by [Jeff Geerling](http://jeffgeerling.com/), author of [Ansible for DevOps](http://ansiblefordevops.com/).
