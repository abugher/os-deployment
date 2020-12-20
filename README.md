# Usage

This directory should be the current working directory, for relative paths.

## deploy\_image

    bin/deploy_image <hostname>

Deploy an OS to an SD card by writing an image file to it and then making modifications.  Refer to files under vars/host/ for per-hostname specifications, including hardware and OS.  Refer to files under vars/hardware/ and vars/os/ for hardware and OS specifications, respectively.

## mkbs

    bin/mkbs <device>

Deploy an OS to a USB storage device by running debootstrap then making modifications, mostly via chroot.

# Requirements

## deploy\_image

| COMMAND     | PACKAGE |
| :------     | :------ |
| openssl     | openssl |
| mkpasswd    | whois   |
| pv          | pv      |
