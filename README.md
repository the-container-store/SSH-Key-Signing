```
 .oooooo..o .oooooo..oooooo   ooooo
d8P'    `Y8d8P'    `Y8`888'   `888'
Y88bo.     Y88bo.      888     888
 `"Y8888o.  `"Y8888o.  888ooooo888
     `"Y88b     `"Y88b 888     888
oo     .d8Poo     .d8P 888     888
8""88888P' 8""88888P' o888o   o888o
oooo    oooo                          .oooooo..o o8o                       o8o
`888   .8P'                          d8P'    `Y8 `"'                       `"'
 888  d8'    .ooooo. oooo    ooo     Y88bo.     oooo  .ooooooooooo. .oo.  oooo ooo. .oo.   .oooooooo
 88888[     d88' `88b `88.  .8'       `"Y8888o. `888 888' `88b `888P"Y88b `888 `888P"Y88b 888' `88b
 888`88b.   888ooo888  `88..8'  88888     `"Y88b 888 888   888  888   888  888  888   888 888   888
 888  `88b. 888    .o   `888'        oo     .d8P 888 `88bod8P'  888   888  888  888   888 `88bod8P'
o888o  o888o`Y8bod8P'    .8'         8""88888P' o888o`8oooooo. o888o o888oo888oo888o o888o`8oooooo.
                     .o..P'                          d"     YD                            d"     YD
                     `Y8P'                           "Y88888P'                            "Y88888P'
```

# Table of Contents
1. [Motivation](#motivation)
2. [Implementation](#implementation)
    * [Requirements](#requirements)
    * [Sign your key](#sign-your-key)
    * [Breaking it down](#breaking-it-down)
3. [Resources](#resources)
4. [Contributors](#contributors)

# Motivation

We're relieving recurring pain with this solution. Before key-signing, we had significant user
sprawl on our GNU/Linux-based virtual machines. When a new user would join the team, we would
manually add their user to these virtual machines as needed. When a new service user was created
in Active Directory, we would manually add the user to these virtual machines as needed. As this
became more frequent, we'd start codifying this in Ansible, but password rotations for each of
these users became very cumbersome. Even if we used private keys to login instead of passwords, we
still had passwords assigned to the users and rotation was still a problem. We tried to shift our
strategy for user management:

* Centralize user management with Active Directory (+ SSSD)

This ended up not working out well for us:

1. Logging in became painfully slow because each server would have to make requests from AD.
2. AD was case insensitive and Linux was case sensitive, so we'd see user account duplicates,
e.g.: `service_user` and `Service_User` with the same `uid`.

This frustration led us to look for another solution. Let's talk about what that solution is and
distill it so you can use it too!

# Implementation

## Requirements

* [Vagrant](https://www.vagrantup.com/)
* [Vault](https://www.vaultproject.io/)

Run this:
```
VAULT_ADDR=https://my-vault-domain vagrant up
```

What comes up from Vagrant is a centos 7.4 box that was provisioned with an `ops` user. The `ops`
user has no password associated with it. The `ops` user also has very limited privileges, but you
can take a look at processes and you have a home directory to do things with. Go ahead and try to
login as the `ops` user. Doesn't work. That's because you don't have a signed key to login with.

## Sign your key

1. To get access to this box, you'll need a private key, let's make one:
```
ssh-keygen -t ecdsa -C my-ops-key -f my-ops-key
```
2. Take note of your accompanying public key:
```
ls -1a | grep ops
my-ops-key
my-ops-key.pub
```
3. Get a vault token
```
vault auth -method=ldap username=my-user
```
4. Get that key signed by the certificate authority (vault in our case)

```
vault write -field=signed_key ssh-client-signer/sign/ops \
    public_key=@my-ops-key.pub > my-ops-key-cert.pub
```
5. Check which port ssh is listening on for our box:
```
vagrant port
The forwarded ports for the machine are listed below. Please note that
these values may differ from values configured in the Vagrantfile if the
provider supports automatic port collision detection and resolution.

    22 (guest) => 2222 (host)
```

6. Login (pass port into the command)
```
ssh -i my-ops-key -i my-ops-key-cert.pub ops@localhost -p 2222
```

## Breaking it down

Now, any certificate authority can sign public keys. Since we were using Vault for other parts of
our infrastructure, and we knew it could also act as a certificate authority, we decided to give it
a shot as part of our implementation. If you are not using vault, you can look at the `step-1` tag
in this repository to see what you would need to spin up an independent certificate authority for
signing public keys with.

We're also taking advantage of our existing Active Directory resources for user management and so
we're integrating Vault with AD. This might seem like the same centralized concept we tried
originally, however it's actually different. Instead of making every server dependent on a
centralized login mechanism, we're only making Vault dependent on that. The servers will accept any
private / public key pair that has been signed by the right authority. So, if you logged into Vault
with the correct credentials via GitHub, LDAP, a standalone Vault Token, it wouldn't care. If your
token has the policies that allow you to request access to these servers, then you can sign your
public key.

When we call `vault write`, we're actually pushing our public key to a role we created in Vault
that specializes in signing public keys specifically for the `ops` user. The role could be as
generic or granular as you want. Our Vault Token also has policy access to access this role. So, we
fetch out the `signed_key` field from the output and assign it to `my-ops-key-cert.pub`. SSHD knows
to look at a key that matches the name of the public key with `-cert` appended. If you're placing
your keys in a standard location (i.e.: `$HOME/.ssh`) then it will automatically try using keypair
without needing you to specify `-i` in your command.

In the end, this implementation has scaled really well for us. We are able to login to servers in
our datacenter and conceivably in the cloud as well since the servers take a decentralized approach
to authentication and authorization.

# Resources

* [Scalable and secure access with SSH](https://code.facebook.com/posts/365787980419535/scalable-and-secure-access-with-ssh/)

This is the blog post that got us thinking about SSH logins in a decentralized way. Highly
recommended reading.

# Contributors

* [Ryan Breidenbach](https://github.com/twoqubed)
* [John Jelinek](https://github.com/johnjelinek)
* [Matt Smith](https://github.com/mpsmith)
