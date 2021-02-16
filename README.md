# Usage

This directory should be the current working directory, for relative paths.

## mksd

    bin/mksd <hostname>

Deploy an OS to an SD card by writing an image file to it and then making modifications.  Refer to files under vars/host/ for per-hostname specifications, including hardware and OS.  Refer to files under vars/hardware/ and vars/os/ for hardware and OS specifications, respectively.

## mkbs

    bin/mkbs <device>

Deploy an OS to a USB storage device by running debootstrap then making modifications, mostly via chroot.

# Requirements

## mksd

| COMMAND     | PACKAGE |
| :------     | :------ |
| openssl     | openssl |
| mkpasswd    | whois   |
| pv          | pv      |


# Bugs:

I had some fun using file descriptors to establish debugging levels, but the
way I did it really made most of this code look awful.  I need to get rid of that.

Right now there are three related lists used to mask systemd services:
* Files to remove.
* Directories to create.
* Files to symlink to /dev/null.
Two problems, here.
1.  ~~The code should be changed to automatically remove files destined for null-linking, so that the remove and symlink lists can be deduplicated.~~
2.  These lists should be broken down and distributed to the appropriate platform-specific vars files, instead of assuming they all need to be handled all the time. 
