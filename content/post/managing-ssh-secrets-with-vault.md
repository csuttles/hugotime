+++
author = "Chris Suttles"
categories = ["vault", "secrets", "consul", "Docker", "containers"]
date = 2018-01-03T17:19:48Z
description = ""
draft = false
cover = "/images/2018/01/Screen-Shot-2018-01-03-at-1.14.50-AM.png"
slug = "managing-ssh-secrets-with-vault"
tags = ["vault", "secrets", "consul", "Docker", "containers"]
title = "Managing SSH Secrets with Vault"
aliases = ["/managing-ssh-secrets-with-vault/"]

+++


This post takes a look at using [Hashicorp's Vault](https://www.vaultproject.io/) to manage secrets for SSH authentication.

For this post, I started working with Vault pretty quickly via this `docker-compose` [setup I found via GitHub](https://github.com/tolitius/cault). It's a very quick way to get a Vault instance with a [Consul](https://www.consul.io/) backend. You'd never do this for production, since they are single instances, but for functional testing, it's enough.

# Requirements

What are we trying to accomplish?

Let's verify the ability to store, retrieve, generate and revoke SSH keys.

## Setup, Unseal, and Use Vault

[Refer to this github repo](https://github.com/tolitius/cault) for bootstrapping the containers, and unsealing the vault as well as environment set up, and creating/accessing a credential, and creating a one-time token, and using it for access to a key (secret) stored in Vault.

## SSH Authentication

### Configuring Vault and a Target Host

We'll use the guide at [https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates.html](https://www.vaultproject.io/docs/secrets/ssh/signed-ssh-certificates.html) for reference.

We start by mounting the backend:

{{< highlight markdown >}} bash
vault mount -path=ssh-client-signer ssh
Successfully mounted 'ssh' at 'ssh-client-signer'!
{{< / highlight >}}

Then we create a CA. You can also use an internal CA, but we'll use an auto_generated one (`generate_signing_key=true`) for demo purposes:

{{< highlight markdown >}} bash
vault write ssh-client-signer/config/ca generate_signing_key=true
{{< / highlight >}}

Next we create the trusted keys pem file on the target host (I am using my localhost) so that certificates issued by our CA can be verified: 

{{< highlight markdown >}} bash
vault read -field=public_key ssh-client-signer/config/ca > /etc/ssh/trusted-user-ca-keys.pem
{{< / highlight >}}

Now we update the sshd_config on our target host to accept certificates trusted by this CA (again, I am using localhost for this test):

{{< highlight markdown >}} bash 
# /etc/ssh/sshd_config
# ...
TrustedUserCAKeys /etc/ssh/trusted-user-ca-keys.pem
{{< / highlight >}}

Now we create a role in Vault for signing SSH keys, including an extension to allow pseudo ttys and with a short TTL for demo purposes:

{{< highlight markdown >}} bash 
vault write ssh-client-signer/roles/my-role -<<"EOH"
{
  "allow_user_certificates": true,
  "allowed_users": "*",
  "default_extensions": [
    {
      "permit-pty": ""
    }
  ],
  "key_type": "ca",
  "default_user": "csuttles",
  "ttl": "2m0s"
}
EOH
{{< / highlight >}}

### Configuring the Client

First we add our SSH key to Vault:

{{< highlight markdown >}} bash
vault write ssh-client-signer/sign/my-role \
    public_key=@$HOME/.ssh/id_rsa.pub
{{< / highlight >}}

Now we save the signed public key to disk:

{{< highlight markdown >}} bash
vault write -field=signed_key ssh-client-signer/sign/my-role \
    public_key=@$HOME/.ssh/id_rsa.pub > signed-cert.pub
{{< / highlight >}}

If we inspect the extensions and metadata of the signed key, we can verify the parameters we specified, like the short TTL:

{{< highlight markdown >}} bash
csuttles@csuttles-Mac:[~]: ssh-keygen -Lf ~/.ssh/signed-cert.pub
/Users/csuttles/.ssh/signed-cert.pub:
        Type: ssh-rsa-cert-v01@openssh.com user certificate
        Public key: RSA-CERT SHA256:1d2oFnwq0s6F+QRHr6DudpsVwfxeI9rX5UOaC6y+OLc
        Signing CA: RSA SHA256:mrYCyfzQD1sv7Wn4Afj7sbwuOT4edV0to7NJKdxRUlw
        Key ID: "vault-root-d5dda8167c2ad2ce85f90447afa0ee769b15c1fc5e23dad7e5439a0bacbe38b7"
        Serial: 15839214298381429970
        Valid: from 2018-01-02T14:15:49 to 2018-01-02T14:18:19
        Principals:
                csuttles
        Critical Options: (none)
        Extensions:
                permit-pty


{{< / highlight >}}

We can authenticate to test. Here I am using the signed key, within the expiry we see from the previous command to authenticate via ssh to localhost and run the date command:

{{< highlight markdown >}} bash
csuttles@csuttles-Mac:[~]: ssh -i ~/.ssh/signed-cert.pub -i ~/.ssh/id_rsa localhost date
Tue Jan  2 14:18:18 PST 2018
[Exit: 0] 14:18
{{< / highlight >}}

Once the TTL expires, it is no longer a sufficient authentication method, and I am prompted for my password: 

{{< highlight markdown >}} bash
csuttles@csuttles-Mac:[~]: ssh -i ~/.ssh/signed-cert.pub -i ~/.ssh/id_rsa localhost date
Password:

[Exit: 130] 14:20
csuttles@csuttles-Mac:[~]:
{{< / highlight >}}

After our TTL has expired, we can simply refresh the signed public key and validate that things work again (until our new TTL expires):

{{< highlight markdown >}} bash
csuttles@csuttles-Mac:[~]: vault write -field=signed_key ssh-client-signer/sign/my-role     public_key="$(cat $HOME/.ssh/id_rsa.pub)" > ~/.ssh/signed-cert.pub
[Exit: 0] 14:21
csuttles@csuttles-Mac:[~]: ssh -i ~/.ssh/signed-cert.pub -i ~/.ssh/id_rsa localhost date
Tue Jan  2 14:21:52 PST 2018
[Exit: 0] 14:21
{{< / highlight >}}

### Revoking Credentials

I looked around for information on revoking the certificates used with SSH and Vault, and I don't think it is supported at this time, since I was unable to find docs for how to do so, and [this feature request asking for a CRL to be published when using the SSH backend](https://github.com/hashicorp/vault/issues/3377).

The current best practice (at the time of writing) seems to be not revoking the certificates at all:

> [We generally recommend making certificate lifetimes useful for only a single connection. As a result CRLs are unnecessary. This workflow can be made easier using vault ssh.](https://github.com/hashicorp/vault/issues/3377#issuecomment-332245797)

Since we are not using a CRL, the small TTL of two minutes might not be a bad choice, if the intention is to make the certificate good only for a single connection. This means any certificate we issue using the TTL of 2 minutes would have a max "revocation time" of 2 minutes, and quite likely less. The automatic expiration within 2 minutes seems like it would be faster or equal in speed to manual recovation in most circumstances.

