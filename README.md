# Usage

This directory should be the current working directory when launching these scripts, as they may rely on relative paths.  (This is a bug.)

## mksd

    bin/mksd <hostname>

Deploy an OS to an SD card by writing an image file to it and then making modifications.  Refer to files under vars/host/ for per-hostname specifications, including hardware and OS.  Refer to files under vars/hardware/ and vars/os/ for hardware and OS specifications, respectively.

## mkbs

    bin/mkbs <device> <release>

Deploy an OS to a USB storage device by running debootstrap then making modifications, mostly via chroot.

# Requirements

## mksd

| COMMAND     | PACKAGE     |
| :------     | :------     |
| openssl     | openssl     |
| mkpasswd    | whois       |
| pv          | pv          |
| mkfs.f2fs   | f2fs-tools  |
| mkfs.ext4   | e2fsprogs   |

## mkbs
| COMMAND     | PACKAGE     |
| :------     | :------     |
| sgdisk      | gdisk       |
| parted      | parted      |
| debootstrap | debootstrap |

# Bugs:

##

In mksd code, I had some fun using file descriptors to establish debugging
levels, but the way I did it really made most of this code look awful.  I need
to get rid of that.

##

mkbs is slow.  It completes in like a quarter hour on a fast USB stick.  More
like a day on a regular USB stick.  It should probably generate an image on
local storage then write that to the device.  I'm assuming there's some
redundant I/O during debootstrap and such, but maybe it just really takes that
long to write all those bits to disk.  I should generate a stick, extract the
image, write the image back, and compare the operation times, to make sure that
the imaging approach will actually save time.  That's also an extra layer of
complexity and fragility, which I might just rather skip, so consider an
alternative approach.  Make sure the script can target a loop device, then
write a wrapper that just does that, and call that an image generation
mechanism.  Writing images to disks doesn't need scripting.

##

mkbs should acquire a lock on the target device, and refuse to operate
concurrently on the same target.

##

Required packages may not be completely documented.

##

If it runs long enough, mkbs prompts for root password again.  It is supposed
to keep it from expiring.

##

The scripts should work regardless of current working directory.

## 

mksd should generate a new image in a file instead of on a storage device.  It
might do so only if the upstream image is newer.  It might also still write the
resulting image to a storage device, skipping the image generation step if
possible.

##

Network configuration assumes ifupdown.  Netplan is becoming default on
Armbian.  Unconfigured netplan will use DHCP.  That works as long as the MAC
address is already known to the DHCP server, but otherwise means a fresh
deployed host will come up with a wrong IP address.
