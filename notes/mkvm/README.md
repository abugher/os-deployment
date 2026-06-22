# Synopsis

    bin/mkvm <hostname>

Deploy an OS to a VM.  The VM should be reachable by SSH on localhost at the SSH port specified in ansible host variables for the named host.  The SSH host key for the named host should be automatically installed as the SSH host key for the VM.  The root password for the named host should be set on the VM.


# Terminology

The word "host" should be avoided unless referring to the system hosting
guest VMs.  I like the word "host", so watch out for slips.

I use the word "real" a lot.  A better term might be "production".
Since all the production systems are physical and the staging systems
are virtual, "real" seems appropriate enough.


# To Do

* Automate configuration of the staging gateway.  
    * Maybe there should be an ansible role for this.  
    * See notes on the [staging network](#staging-network) for the simple steps needed for this.
* Bootstrap DNS service inside the staging network.
    * See notes on [DNS](#DNS) for current problem and plan.
* Match architecture and OS of real/production hosts.
    * Initial implementation will use plain Debian on the most convenient architecture.
    * Once the initial implementation is working, emulate the real environments.
        * Emulate the correct hardware.
        * Deploy the correct OS.
        * This might require running `mksd` on an image file to create a virtual SD card.
* Achieve [networking goals](#networking-goals).
* Create or modify an ansible role to install requisites for mkvm to run.
    * See [requirements](#requirements).
* Preseed security:
    * Consider safe handling of preseed file.  
        * The preseed configuration includes SSH private key and root password in plaintext.
        * Maybe instead of keeping it, delete it every time, but keep a checksum to detect change.  
    * Find a way to securely use the installer image for production deployment.  
        * The preseed configuration includes SSH private key and root password in plaintext.  
        * It should not be burned to CD or made available via PXE network boot.
        * May not apply if `mksd` continues to be the preferred OS deployment method.
        * This goal seems out of scope for this specific script.
* Information lookup:
    * Replace reference to control-center/stg with a path-to-self lookup.
    * Retrieve information from ansible inventory in a better way.  (Right now it is done by interpreting text and makes assumptions about formatting.  It would be better to ask ansible to show the values of the host variables.  `ansible-inventory` may be able to do so.)
    * Look up network, netmask, gateway, and DNS server from inventory.  (Right now they are static in the preseed template, in qemu invocation, and in network configuration scripting.)
    * Look up network interface name from live configuration.  (Right now it is hardcoded in `mkvmnet`.)
    * Get all network configuration variables from ansible inventory.
    * Look up release, version, architecture, and resource specifications from host variables.  (Also make sure these reflect and will continue to reflect the production hosts.)

* Out of scope:
    * These goals are mostly scoped for the `ansible` and `control-center` repos, but the purpose of `mkvm` is to achieve them.

    * Automate full deployment of staging versions of all important systems.
        * Qualify "important".  
            * Maybe the `servers` hostgroup.
            * Maybe the `ansible-targets` hostgroup, but those are not all fully ansible controlled.
        * This is the big goal; more intermediate steps will probably need to be discovered and completed.
    * Implement ansible testing using this tool.  (Making sure Nagios shows all green is a good start.  If more testing is necessary, it should probably be added to Nagios, anyway.)
    * Implement pre-deployment testing in ansible using VMs deployed this way.  (Same as above?)
    * Differentiate testing all hosts as test VMs together from testing one host as a test VM with access to real hosts.  (Determine whether the second is possible and/or reasonable to implement.)
    * Exclude untestable roles from testing.  Some roles require hardware that may not be feasible to emulate, like a specific printer model.
    * Download (torrent) original installer image automatically.
        * Or SD card image as appropriate.

# Requirements

| COMMAND             | PACKAGE         |
| :------             | :------         |
|7zz                  | 7zip            |
|genisoimage          | genisoimage     |
|cpio                 | cpio            |
|isohybrid            | syslinux-utils  |
|qemu-system-x86\_64  | qemu-system     |
|sshpass              | sshpass         |


# Networking

## Networking Plan

Duplicate the LAN as a staging environment of VMs, using an identical
address space, isolated from the real LAN.

When a staging VM reaches out to neuron, packets should reach the
staging system named neuron, not the real neuron.

The host system should be able to reach the gateway by SSH at staging-gateway.
Configuration for qemu and SSH should facilitate that.  An SSH tunnel through
staging-gateway, exposing the unique SSH port of a target host, should
facilitate reaching other hosts in a similar way, for example staging-neuron,
if required or desired.


## Networking Goals

* Replicate production (real) LAN as a staging LAN in qemu, only connecting VMs to each other.  (192.168.11.0/24)
* Implement an additional subnet for a staging mail server, replicating the production mail server addres.
* Implement a NAT gateway VM using a different subnet to connect to the host system.  (10.0.0.0/8, probably)
* Implement a dedicated ansible master host attached to the staging LAN, reachable by port forwarding through the gateway.
* Ensure that within the staging LAN packets are never sent directly to the production LAN.
* Ensure that within the staging LAN packets for the mail server are sent only to the staging mail server.
* Decide whether internet access via the gateway is beneficial and/or necessary.


## Intermediary Network

qemu can provide one layer of NAT, implementing
an intermediary address space different from the real LAN.  qemu will
provision the first interface of the staging gateway as a member of this
virtual network.  That means the gateway will be able to reach the real LAN
address space via its default route until a route is established for the
staging network.


## Staging Network

qemu will provision the second interface of the
staging gateway as part of a second virtual network.  This interface
needs to be assigned an identical address to that of the real gateway,
and a route for the real LAN network space needs to send all traffic
through this interface.  IP forwarding must also be enabled.  (The link
may also need to be brought "up".)  qemu will provision all other
staging systems with only one interface as parts of this staging
network.  Their preseed files should configure identical addressing to
their real counterparts.


## DNS

This is the current problem.  The staging gateway can be deployed
without the second interface configured, as a member of the intermediary
network only.  It uses the real DNS server on the real LAN.  Once the
staging network is brought up, that is no longer available.  That means
the netinstall process cannot find Debian repo servers, causing
installation to fail.  That is stopping me from deploying a staging
version of the DNS server.  This might be solved just by using an
offline install medium ("disc one"), especially if it can provide enough
packages to bring up the staging DNS server.  Otherwise, it might be
necessary to set up some temporary DNS forwarding, then break that once
the staging DNS server is up ... but I hope not.
