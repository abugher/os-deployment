This directory should be the current working directory, for relative paths.

    bin/deploy_image <hostname>

Deploy an OS to an SD card by writing an image file to it and then making modifications.  Refer to files under var/host/ for per-hostname specifications, including hardware and OS.  Refer to files under var/hardware/ and var/os/ for hardware and OS specifications, respectively.

    bin/mkbs <device>

Deploy an OS to a USB storage device by running debootstrap then making modifications, mostly via chroot.
