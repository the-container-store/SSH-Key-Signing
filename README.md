# SSH-CA

This is a reference for implementing an SSH Certificate Authority.

## Run

### Requirements

* Vagrant
* Vault

### Steps

1. Grab the CA pubkey from vault: `vault read -field=public_key ssh-client-signer/config/ca > trusted-user-ca-keys.pem`
2. Run: `vagrant up`
3. Sign your pubkey:
```
vault write -field=signed_key ssh-client-signer/sign/my-role \
    public_key=@$HOME/.ssh/id_rsa.pub \
    valid_principals="zone-webservers" > $HOME/.ssh/id_rsa-cert.pub
```
4. Find the port of the server:
```
$ vagrant port

The forwarded ports for the machine are listed below. Please note that
these values may differ from values configured in the Vagrantfile if the
provider supports automatic port collision detection and resolution.

    22 (guest) => 2222 (host)
```
5. SSH into the box as ops, run: `ssh ops@localhost -p 2222`
