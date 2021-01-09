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
