# SSH-CA

This is a reference for implementing an SSH Certificate Authority.

## Run

### Requirements

* Vagrant
* An existing pubkey (named `id_rsa.pub`)

### Steps

1. Bring an existing pubkey into this directory
2. Run: `vagrant up`
3. Copy the resultant `id_rsa-cert.pub` into $HOME/.ssh on your machine
4. Find the port of the `ssh-host` server, run: `vagrant port`
5. SSH into the box as root (assuming port 2200 from command above), run: `ssh root@localhost -p 2200`
