# Yubico Yubikey

This file details useful commands and routines with regard to a Yubikey security key

## Setting up GPG

If re-setting up on a new system with a yubikey that you want to use for SSH and PGP signing (for example with git), then the following
should get you started.

### Setup automatic gpg-agent config

The first thing to do is to add the following to the end of your ```.bash-rc``` file in your home folder:

	# Start gpg-agent if it's not running
	if ! pidof gpg-agent > /dev/null; then
	    gpg-agent --homedir $HOME/.gnupg --daemon --sh --enable-ssh-support > $HOME/.gnupg/env
	fi
	if [ -f "$HOME/.gnupg/env" ]; then
	    source $HOME/.gnupg/env
	fi
	gpg-connect-agent updatestartuptty /bye > /dev/null 2>&1

The above automates the setup of gpg-agent so that the Yubikey is recognised for PGP and SSH operations when plugged in. An alternative
to the above is to run the following in each new terminal session:

	sudo killall gpg-agent
	sudo killall ssh-agent
	eval $( gpg-agent --daemon --enable-ssh-support )

...which is a bit tedious.

### Initialise GPG

Now open a terminal, plug in the yubikey, and run:

	gpg --card-edit

This will create the .gnupg folder in your home directory and fetch the keys from the card.

It is possible the above will fail, which I am currently seeing with a new Fedora 33 install. If that is the case I have found that it
is some sort of conflict with pcscd. A restart seems to sort it out:

	sudo service pcscd restart

The restart is only necessary once after a reboot, and can be avoided altogether if you only plug in the yubikey after Fedora has booted.
Hopefully this error will be sorted in the future as it wasn't an issue with Fedora 32.

### Add some config files

You now need to create two files in the .gnupg folder. The first should be named ```gpg.conf```, and have the following contents:

	# Try to use the GnuPG-Agent. With this option, GnuPG first tries to connect to
	# the agent before it asks for a passphrase.

	use-agent

The second file should be ```gpg-agent.conf``` and should have the following contents:

	enable-ssh-support

As you can see that above are some basic functionality settings.

Everything is now ready to go.

## SSH Connection to server from a local computer

A pgp key can be stored on a Yubikey, and this can also double as an SSH key. In my case I use Fedora as my desktop and an Ubuntu server.

### Store SSH key on server

Firstly you need to store the public SSH key on the server that you wish to connect to:

1. Copy the contents of your SSH public key file
2. Paste the contents into the file ~/.ssh/authorized_keys (if either the directory or file don't exist, then create them)
3. Change the folder permission ```sudo find ~/.ssh -type d -exec sudo chmod 700 {} \;``` (folders have only read / write / execute for the folders)
4. Change file permissions ```sudo find ~/.ssh -type f -exec sudo chmod 600 {} \;``` (files have only read / write for the user)
5. Change the folder and file user/group ```sudo chown -R testspecimen:testspecimen``` (where it says "testspecimen" replace with your username)

### Connecting to the server

The connection is now very simple:

	ssh website.com

You may see something like the following the first time you connect:

	The authenticity of host 'yourwebsite.com (214.66.230.184)' can't be established.
	ECDSA key fingerprint is SHA256:fKLT/itf3P/CkFWgt7SbpyKVNhhmc/5LL2H6+zBpgDJ.
	Are you sure you want to continue connecting (yes/no/[fingerprint])?

As long as you are sure you connected to the right place you can proceed. Type 'yes'.

You should then be either automagically logged in, or if you have a pin setup you will be prompted to enter it.

Done. 


## Changing Yubikeys

In order to swap between which YubiKey I want to use, I do the following:

	sudo killall gpg-agent

You then need to delete the previous keys. The keys are stored in the following folder: 

	~/.gnupg/private-keys-v1.d/

If you only have one set of private keys then you can just delete all the contents of the above folder, which will
likely be three files ending in ".key" (there are three as there is one for signing, encryption and authentication)
	
However, if you have other keys you do not want to delete, you need to find the correct keys to delete. 

The file names of the ".key" files are the "keygrip", so you need to list the key information with the keygrip:

	gpg --list-secret-keys --with-keygrip

You then pick the corresponding keygrips for the keys you want to delete, and delete the key files.

Now plug in the new YubiKey

	gpg --card-edit

This makes sure the card is visible and working. It also notifies gpg which keys are available for current card, and will add ".key"
files to the following folder:

	~/.gnupg/private-keys-v1.d/

Now the alternate card should be usable. If it's not, unplug the YubiKey and repeat the steps above, it should work the second time.

The command:

	gpg-connect-agent updatestartuptty /bye

could also help mitigate some issues.