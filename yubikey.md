# Yubico Yubikey

This file details useful commands and routines with regard to a Yubikey security key

## SSH Connection to server from a local computer

A pgp key can be stored on a Yubikey, and this can also double as an SSH key. In my case I use Fedora as my desktop and an Ubuntu server.

### Store SSH key on server

Firstly you need to store the public SSH key on the server that you wish to connect to:

1. Copy the contents of your SSH public key file
2. Paste the contents into the file ~/.ssh/authorized_keys (if either the directory or file don't exist, then create them)
3. Change the folder permission ```sudo find ~/.ssh -type d -exec sudo chmod 700 {} \;``` (folders have only read / write / execute for the folders)
4. Change file permissions ```sudo find ~/.ssh -type f -exec sudo chmod 600 {} \;``` (files have only read / write for the user)
5. Change the folder and file user/group ```sudo chown -R testspecimen:testspecimen``` (where it says "testspecimen" replace with your username)

### Connect from local computer to server

At this point you can plug in the Yubikey. You will need to run the following commands before trying to connect.
This will need to be run in every new terminal instance, and for any program that needs SSH from the Yubikey.
For example Filezilla should always be started from the commandline **after** running the following commands.

	sudo killall gpg-agent
	sudo killall ssh-agent
	eval $( gpg-agent --daemon --enable-ssh-support )

## Changing Yubikeys

In order to swap between which YubiKey I want to use, I do the following:

	sudo killall gpg-agent
	sudo rm -r ~/.gnupg/private-keys-v1.d/

Now plug in the new YubiKey

	gpg --card-edit

This makes sure the card is visible and working. It also notifies gpg which keys are available for current card.

Now the alternate card should be usable. If it's not, unplug the YubiKey and repeat the steps above, it should work the second time.

The command:

	gpg-connect-agent updatestartuptty /bye

could also help mitigate some issues.