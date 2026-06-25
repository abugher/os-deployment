This file should document the requirements to bootstrap the staging
network as observed during prototyping.

* Gateway needs to proxy ARP.
    * This is codified in the preseed files.
    * I need to understand why this is necessary.  Probably no further
      configuration changes are necessary, but it bothers me that I don't
      understand.  It seems like this shouldn't be necessary, but routing
      through the gateway does not work without it.
* Tunnel through the gateway to reach the first host, assumed to be (staging-)neuron.
    * Forward the unique port for that host, 2235.
    * SSH client configuration should direct connections to staging-neuron to 127.0.0.1:2235.
    * SSH client configuration should direct connections to staging-gateway similarly.
    * qemu configuration should forward the unique port for gateway to the
      right address on the intermediary network.

    ssh -L 2235:192.168.11.54:22 staging-gateway

* Temporarily reverse tunnel DNS requests (TCP only) from staging-neuron
  through the host system to real neuron.
    * Temporary!
    * Use this until staging-neuron can be configured via ansible to serve DNS.
    * Possibly not necessary at all...
        * Necessary only if local DNS is required to bootstrap DNS service.
        * Otherwise the next step could use an internet DNS server.

    ssh -R 53:192.168.11.54:53 staging-neuron

* Temporarily configure DNS on staging-neuron.
    * Direct requests to 127.0.0.1, relying on reverse tunnel.
    * Specify TCP only.
    * /etc/resolv.conf:

    nameserver 127.0.0.1
    options use-vc

* Sync a copy of the control center to staging-neuron.

    rsync --progress -v -rlp --delete ./control-center/ root@staging-neuron:control-center/

* Deploy some roles to localhost.
    * Currently figuring this out.
    * Problem:  Platform roles defined in hostvars are wrong for staging environment.
        * Eventually maybe I'll do this with armbian on an ARM system.
        * Right now its debian on an amd64 system.
    * Problem:  Pass is not available.
        * I could probably just sync in my password database and GPG keys.
            * I hate making copies of my GPG keys.
        * SSH can forward gpg-agent, right?
            * Maybe I can just sync in the password database.
    * Maybe:  `ansible-master`
    * Maybe:  `ansible-target`
    * Definitely:  `dns-internal`
