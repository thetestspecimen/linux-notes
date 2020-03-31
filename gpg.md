# GPG or OpenPGP

Information on how to use, generate and interact with PGP keys.

## Confusion

First of all I think it would be wise to briefly mention the difference (or similarity) between GPG, PGP and OpenPGP.

Simply put they are all one and the same, although this is not *strictly* true:

- PGP is the original protocol and is now owned by Symantec
- OpenPGP is an open source version of the original PGP standard
- GPG (or GnuPG) is GNU Privacy Guard, and is different implementation of OpenPGP commonly used in Linux

However, **functionally** they are identical.

In some places you will see PGP, in others GPG...the best thing to do is assume they are interchangable. The long and short is 
don't worry about it, and just view them as one and the same.

I will refer to it as GPG throughout, as I think this will reduce confusion, especially when dealing with linux.

## What is GPG?

GPG is basically just a set of cryptographic keys. The keys can then be used for:

- Encryption of files, documents, emails etc.
- Digitally signing files, documents, emails etc.
- Authentication - for example logging into a server with SSH
- Certification - the "signing" of another users key

In terms of day to day use, the following are probably the most useful:

- SSH access to a remote server
- Git commit signing
- Email signing and encryption (for secure communication)

## Private vs Pubic Keys

You may have read about private and public keys before, but what is the difference?

Simply put:

- the private key is **ONLY** kept by you, and should **NEVER** be revealed to anyone!
- the public key can be distributed anywhere and everywhere without a care in the world

### How the system works

First I think it is important to understand how the private/public key system works with some examples.

### Signing

When you "sign" something with gpg, you basically use your private key to "sign" the file, commit or email etc. 
Remember, only you can do this, as only you have the private key.

You could send someone else your public key, and they can use that public key to confirm that it was indeed you that
signed the file, commit or email etc.

This is because the public key is directly linked to your private key, but not in a way that could ever reveal your private key
(that is the clever part).

As a concrete example:

You could upload your public key to your github account, and then sign commits with your private key. Github will then be able to 
verify it was **definitely** you that signed the commit. As only you have the correct private key!

### Authentication

This is similar to signing. For example to ssh into a server:

- public key on the server
- private key used to sign in

### Encryption

Encryption works in a similar, but slightly different way.

If I want to send an encrypted file to you, that only you can decrypt. Then I must encrypt the file with **your** public key.
Once I do that, the only person able to decrypt the file is the one with the matching private key for the public key used for the
encryption (i.e. you). Not even me as the original "encryptor" can decrypt the file!

### Certification

Certification is essetially "signing" someone elses public key. 

This contributes to the trustworthyness of the public key: **the web of trust**

I am not going to go into this here, but it is worth looking up, as it is the core of the gpg methodology.

## Creating a GPG key

The following section will show you how you can create a GPG key in a linux system.

The aim will be to create a master key with three sub keys (all are private keys):

- master: certification
- sub: encryption
- sub: signing
- sub: authentication

We will also output a public key.