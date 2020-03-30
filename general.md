# General Helpful Commands

This section is meant for any commands that are useful but do not warrant their own file.

## Securely delete files

	sudo shred -zvu -n 5 file.txt

Notes:

- -n: number of overwrites of the file (the more the better, but it takes more time)
- -zvu:
	- z: add a final overwrite with zeros to hide shredding
	- v: show verbose information about shredding progress
	- u: truncate and remove file after overwriting

## Problems with kernel crashing on boot

	sudo dnf reinstall dracut
	sudo dnf reinstall kernel-core kernel-modules
	sudo dracut -v -f

## Stress test cpu

	sudo dnf install stress
	sudo stress --cpu 8 --timeout 20

Notes: 

- --cpu: spawn 8 workers spinning on sqrt()
- --timeout: timeout after 20 seconds

## See various sensors including cpu

### Install

	sudo dnf install lm_sensors

### Configuration

This will run a step by step setup to find all the sensors on your computer:

	sudo sensors-detect

### Commands

View an output of all the sensors:

	sensors

View a live stream of the output of all the sensors:

	watch sensors