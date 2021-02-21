# Yubico Yubikey

This file details useful commands and routines with regard to a Yubikey security key

## Setting up GPG

If re-setting up on a new system with a yubikey that you want to use for SSH and PGP signing (for example with git), then the following
should get you started.

### udev settings

It is likely that you will need to add a few files that contain udev settings.

Add the following three files to ```/etc/udev/rules.d/''', and then reboot.

69-yubikey.rules

	ACTION!="add|change", GOTO="yubico_end"

	# Udev rules for letting the console user access the Yubikey USB
	# device node, needed for challenge/response to work correctly.

	# Yubico Yubikey II
	ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0010|0110|0111|0114|0116|0401|0403|0405|0407|0410", \
		ENV{ID_SECURITY_TOKEN}="1"

	LABEL="yubico_end"

70-u2f.rules

	# Copyright (C) 2013-2015 Yubico AB
	#
	# This program is free software; you can redistribute it and/or modify it
	# under the terms of the GNU Lesser General Public License as published by
	# the Free Software Foundation; either version 2.1, or (at your option)
	# any later version.
	#
	# This program is distributed in the hope that it will be useful, but
	# WITHOUT ANY WARRANTY; without even the implied warranty of
	# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU Lesser
	# General Public License for more details.
	#
	# You should have received a copy of the GNU Lesser General Public License
	# along with this program; if not, see <http://www.gnu.org/licenses/>.

	# this udev file should be used with udev 188 and newer
	ACTION!="add|change", GOTO="u2f_end"

	# Yubico YubiKey
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0113|0114|0115|0116|0120|0121|0200|0402|0403|0406|0407|0410", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Happlink (formerly Plug-Up) Security KEY
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="2581", ATTRS{idProduct}=="f1d0", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Neowave Keydo and Keydo AES
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1e0d", ATTRS{idProduct}=="f1d0|f1ae", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# HyperSecu HyperFIDO
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="096e|2ccf", ATTRS{idProduct}=="0880", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Feitian ePass FIDO, BioPass FIDO2
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="096e", ATTRS{idProduct}=="0850|0852|0853|0854|0856|0858|085a|085b|085d", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# JaCarta U2F
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="24dc", ATTRS{idProduct}=="0101|0501", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# U2F Zero
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="8acf", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# VASCO SecureClick
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1a44", ATTRS{idProduct}=="00bb", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Bluink Key
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="2abe", ATTRS{idProduct}=="1002", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Thetis Key
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1ea8", ATTRS{idProduct}=="f025", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Nitrokey FIDO U2F, Nitrokey FIDO2, Safetech SafeKey
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="20a0", ATTRS{idProduct}=="4287|42b1|42b3", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Google Titan U2F
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="18d1", ATTRS{idProduct}=="5026", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Tomu board + chopstx U2F + SoloKeys
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="cdab|a2ca", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# SoloKeys
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="5070|50b0", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Trezor
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="534c", ATTRS{idProduct}=="0001", TAG+="uaccess", GROUP="plugdev", MODE="0660"
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1209", ATTRS{idProduct}=="53c1", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Infineon FIDO
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="058b", ATTRS{idProduct}=="022d", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Ledger Nano S and Nano X
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="2c97", ATTRS{idProduct}=="0001|0004", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Kensington VeriMark
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="06cb", ATTRS{idProduct}=="0088", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# Longmai mFIDO
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="4c4d", ATTRS{idProduct}=="f703", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# eWBM FIDO2 - Goldengate 310, 320, 500, 450
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="311f", ATTRS{idProduct}=="4a1a|4c2a|5c2f|f47c", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# OnlyKey (FIDO2 / U2F)
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1d50", ATTRS{idProduct}=="60fc", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	# GoTrust Idem Key
	KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="1fc9", ATTRS{idProduct}=="f143", TAG+="uaccess", GROUP="plugdev", MODE="0660"

	LABEL="u2f_end"

70-yubikey.rules

	# Udev rules for letting the console user access the Yubikey USB
	# device node, needed for challenge/response to work correctly.

	ACTION=="add|change", SUBSYSTEM=="usb", \
	ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0010|0110|0111|0114|0116|0401|0403|0405|0407|0410", \
	TEST=="/var/run/ConsoleKit/database", \
	RUN+="udev-acl --action=$env{ACTION} --device=$env{DEVNAME}"

### Setup automatic gpg-agent config

#### Fedora only

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

It is possible the above will fail, which I am currently seeing with a new Fedora 33 install. I have found that it
is some sort of conflict with pcscd. A restart seems to sort it out:

	sudo service pcscd restart

The restart is only necessary once after a reboot, and can be avoided altogether if you only plug in the yubikey after Fedora has booted.
Hopefully this error will be sorted in the future as it wasn't an issue with Fedora 32.

### Add some config files

#### Fedora only

You now need to create two files in the .gnupg folder. The first should be named ```gpg.conf```, and have the following contents:

	# Try to use the GnuPG-Agent. With this option, GnuPG first tries to connect to
	# the agent before it asks for a passphrase.

	use-agent

The second file should be ```gpg-agent.conf``` and should have the following contents:

	enable-ssh-support

As you can see that above are some basic functionality settings.

Everything is now ready to go.

#### Manjaro / Arch only

As per the [Arch wiki](https://wiki.archlinux.org/index.php/GnuPG#SSH_agent) the following should be added to ```~/.bashrc```:

	unset SSH_AGENT_PID
	if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
	export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
	fi

	export GPG_TTY=$(tty)
	gpg-connect-agent updatestartuptty /bye >/dev/null

this allows the use of ssh.

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

	The authenticity of host 'website.com (214.66.230.184)' can't be established.
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