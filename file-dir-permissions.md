# Changing File and Folder Permissions

How to change file and folder permissions on Linux

## File permissions

All files and folders can have three basic permissions set against them:

- Read
- Write
- Execute

and each of the above can be set for:

- Owner
- Group
- Others

If you run ```ls -l``` you will get a list of files and folders and their permissions. For example:

- drwxrw-r-- folder
- -rw-r----- example.txt 

The first letter in the list has 'd'for the folder and '-' for the file. It is either a directory (d) or not (-).

Then we have three groups (owner, group and others) of rwx (read, write and execute).   

For example the folder has:

- Owner can: read, write and execute
- Group can: read and write
- Others can: read only

The file has:

- Owner can: read and write
- Group can: read only
- Others can: do nothing (i.e. no access)

### Permission Number Definitions

I will use numbered formats for the permissions as I find this the simplest way, so 
here is a description of what the numbers mean:

Number | Permission | Output
--- | --- | ---
0 | No permission | ---
1 | Execute | --x
2 | Write | -w-
3 | Execute - Write | -wx
4 | Read | r--
5 | Read - Execute | r-x
6 | Read - Write | rw-
7 | Read - Write - Execute | rwx

So for the folder and file example in the previous section the numbers would be:

- Folder: 764
- File: 640

### Single files or folders

To set a single file or folder is easy:

	sudo chmod 640 file.txt
	sudo chmod 640 /var/www/file.txt
	sudo chmod 750 /var/www/

### Recursive files and folders

It is often the case that you may want to recursively set file permissions.

For example you may want to set all files and folders contained within:

	/var/www/html/

to specific permissions.

You could set all files and folders to the same permission using the "-R" command, which stands for recursive:

	sudo chmod -R 750 /var/www/html

However, a lot of the time you don't want the same permissions for files and folders. The main difference
is the "execute" permission. Folders require "execute" for you to be able to open them, whereas you don't want
a file to be executable without it really needing it.

Therefore, in a lot of circumstances the files will not need the execute permission, but the folder will. Which
results in the files having a number that is one smaller:

If the folders are 750 then the files would be 640 etc.

This is not possible with the command above, so we need an alternative that targets files and folders separately.

There are a few ways of doing this. The first is using xargs (d = directory, f = file):

	sudo find /path/to/base/dir -type d -print0 | sudo xargs -0 chmod 750 
	sudo find /path/to/base/dir -type f -print0 | sudo xargs -0 chmod 640

The second is:

	sudo find /path/to/base/dir -type d -exec sudo chmod 750 {} \;
	sudo find /path/to/base/dir -type f -exec sudo chmod 640 {} \;

So which should you use, xargs or exec method?

In theory xargs is faster and more efficient:

- the exec command finds a file/folder and executes the chmod on each file individually
- the xargs command finds all the files/folders and executes chmod once at the end (therefore more efficient)

...but xargs also has some caveats. You **must** include the "-print0" and
"-0" parts of the command otherwise files and folders with whitespace will cause problems.

Furthermore, xargs can potentially run into issues with long commands (lots of files and folders chmoded in batch at the end).

In summary, if you are not sure use the "-exec" version. it is still quick enough with modern CPUs, but if you have a really
large amount of files/folders use xargs.

## Default file and folder permissions

You may note that when a file or folder is created the user, group and other permissions are automatically set.

It may be the case that you want to change the default for newly created files and / or folders.

### Install acl if required

The acl utility sets Access Control Lists (ACLs) of files and directories.

- Ubuntu: ```sudo apt install acl```
- Fedora: ```sudo dnf install acl```

### setfacl command

We will use the setfacl command in the next few sections. Here are some relevant notes:

- "-R" = Recursive
- "-m" = modify (also -M = modify-file)
- "-d" = default
- "g::" = group
- "o::" = other

with regard to permissions, we use letters rather than the 0-7 number in the previous section.
The reason for this is that we can use capital 'X' to set the execute permission only for folders, which is useful.

### Set GID (Group UID) for main folder
 
	sudo chmod g+s /path/to/folder
 
### Set current permissions for current files and folders (in this case 750 folders and 640 files)
 
	sudo setfacl -R -m g::r-X /path/to/folder 
	sudo setfacl -R -m o::--- /path/to/folder

Note: because of the capital 'X' no execute permission will be set on files for the group, only folders.
 
### Set default permissions for files and folders (in this case 750 folders and 640 files)
 
	sudo setfacl -R -d -m g::r-X /path/to/folder
	sudo setfacl -R -d -m o::--- /path/to/folder

Note: because of the capital 'X' no execute permission will be set on files for the group,  only folders.

### Change the group of current files and folders to www-data (or whatever other group you need)
 
	sudo find /path/to/folder -type d -exec chgrp www-data {} +
	sudo find /path/to/folder -type f -exec chgrp www-data {} +
 
### Set GID for all files and folders
 
	sudo find /path/to/folder -type d -exec chmod g+s {} +
	sudo find /path/to/folder -type f -exec chmod g+s {} +