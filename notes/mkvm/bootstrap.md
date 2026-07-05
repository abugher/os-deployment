This file should document the requirements to bootstrap the staging
network as observed during prototyping.

* Gateway needs to proxy ARP.
    * This is codified in the preseed files.
    * I need to understand why this is necessary.  Probably no further
      configuration changes are necessary, but it bothers me that I don't
      understand.  It seems like this shouldn't be necessary, but routing
      through the gateway does not work without it.

* Tunnel through the gateway to reach the staging deployment system, assumed to be (staging-)controller, and the staging DNS system, assumed to be (staging-)neuron.
    * SSH client configuration should direct connections to the unique port on localhost.
    * qemu configuration should forward the unique port for gateway from
      localhost to the right address on the intermediary network.
    * This command line should forward connections to unique ports through
      staging-gateway to staging systems.

    ssh -L 2238:192.168.11.83:22 -L 2235:192.168.11.54:22 root@staging-gateway

* Temporarily reverse tunnel DNS requests (TCP only) from staging-controller
  through the host system to real neuron.
    * Temporary!
    * Use this until staging-neuron can be configured via ansible to serve DNS.
    * Possibly not necessary at all...
        * Necessary only if local DNS is required to bootstrap DNS service.
        * Otherwise the next step could use an internet DNS server.
    * Also forward SSH keys.

    ssh -A -R 53:192.168.11.54:53 root@staging-controller

* Create a second tunnel for DNS to the staging DNS system, assumed to be (staging)-neuron.
    * Same caveats; this might not be necessary.

    ssh -R 53:192.168.11.54:53 root@staging-neuron

* Temporarily configure DNS on staging-controller and staging-neuron.
    * Direct requests to 127.0.0.1, relying on reverse tunnel.
    * Specify TCP only.
    * /etc/resolv.conf:

    nameserver 127.0.0.1
    options use-vc

* Install rsync, ansible, pass, and dependencies, on staging-controller.

    apt update
    apt install -y rsync ansible pass

* Copy GPG key to staging-controller.
    * Forwarding looks like a pain.
        * https://wiki.gnupg.org/AgentForwarding
    * If we're feeling fancy, there could probably be a non-root user for deployment.
        * "ansible-master" or something, maybe.
        * "aaron" might be a sensible choice.
    * Create and use a separate key/identity with access to a small subset of
      passwords.

    # Recording the one time process to generate such a key.
    #
    # Create home directory for new identity.
    mkdir ~/.gnupg-controller
    # Fix permissions to avoid warnings later.
    chmod 0700 ~/.gnupg-controller
    # Generate and store a a pass phrase to use with the new PGP identity.
    dp aaron/staging-controller-gpg-passphrase
    # See the pass phrase, so it can be copied and pasted in a moment.
    pass show aaron/staging-controller-gpg-passphrase

    # Generate the new PGP identity and key.
    #
    # Enter the passphrase when prompted.
    #
    # If "encr" is not specified, expect errors like this when trying to use
    # pass to give access to certain passwords to controller:
    #
    #   gpg: [long key]: skipped: Unusable public key
    #
    gpg --homedir ~/.gnupg-controller --quick-gen-key controller rsa4096 sign,encr never
    # Import new public key into main personal keychain.
    gpg --homedir ~/.gnupg-controller --export controller | gpg --import
    # Sign key.
    new_key_fpr="$(gpg --homedir ~/.gnupg-controller --with-colons -K | awk -F : '/^fpr:/ {print $10}')"
    gpg --command-fd 0 --sign-key "${new_key_fpr}" <<< "$(printf '%s\n%s\n' 'y' 'y')"

    #
    # pass ... This part gets weird.
    #

    my_key_fpr="$(gpg --with-colons -K | awk -F : '/^fpr:/ {print $10}' | head -n 1)"

    # GPG "trust level" is not necessary, so skip this.
    #gpg --quick-set-ownertrust "${new_key_fpr}" full

    # Initialize (reinitialize) some directories to give access to the
    # controller identity.
    # 
    # All keys in an initialized directory get reencrypted to the new key and the
    # personal key.  Notably, the output only seems to indicate they are being
    # encrypted to the old key, but experimentation shows they are accessible
    # with the new key.  pass mv aaron/files controller/files
    #
    # The controller identify does not have and should not need access to all
    # root passwords.  A forwarded SSH key should enable SSH access as root to
    # all staging hosts.
    pass init -p aaron/neuron-mail "${new_key_fpr}" "${my_key_fpr}"
    pass init -p aaron/neuron "${new_key_fpr}" "${my_key_fpr}"
    pass init -p aaron/files "${new_key_fpr}" "${my_key_fpr}"
    pass init -p shared/vpn "${new_key_fpr}" "${my_key_fpr}"

    # Sync the key/identity dedicated for this purpose.
    rsync -rlp --delete ~/.gnupg-controller/ root@staging-controller:.gnupg/


* Sync a copy of the control center to staging-controller.

    rsync --progress -v -rlp --delete ./control-center/ root@staging-controller:control-center/


* PROGRESS POINT -- CONTINUE EDITING HERE


* Sync passwords to staging-controller.
    * Same caveats as gpg above.

    rsync -rlp --delete ~/.password-store/ root@staging-controller:.password-store/

* ... probably sync over that stupid pinentry wrapper referred to by my gpg configuration.

* Deploy some roles.
    * Currently figuring this out.
    * Problem:  Platform roles defined in hostvars are wrong for staging environment.
        * Right now `mkvm` does debian on an amd64 system.
            * devuan != debian
            * armhf != amd64
        * Maybe:  Do correct OS on correct hardware in `mkvm`.
            * Non-trivial development required.
            * The ability to stage everything on an arbitrary platform could have good testing value.
            * I think this is an eventual goal but not the immediate solution.
        * Maybe:  Remove platform roles from dependency tree.
            * Maybe each platform role can have a group, so they only get deployed with `deploy-host`.
                * ... I do want to do `deploy-host` in the staging environment, though.
            * Maybe:  Special "os-upgrade" role deploys platform roles.
                * Only deploy it at OS upgrade time.
                    * Maybe also initial deployment.
                    * Review deployment process to be sure.
                * Not pulled in by `deploy-host` or any other roles.
    * To staging-controller (localhost):  `ansible-master`
    * To staging-neuron (neuron):  `ansible-target`, `dns-internal`
