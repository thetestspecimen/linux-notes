# Git

Useful information for using git via the commandline

## Global Configuration

There are some settings that you may wish to use across all repositories. If so
you can set them globally. You can also use these settings without the "--global"
to set them for a specific repository

### Minimal global settings

```bash
git config --global user.name "your-username"
git config --global user.email "your@email.com"
```

### Set git editor

It may be required to add a body to the commit message, in which case you need a text editor,
as it is a bit awkward on the commandline.

The default editor is likely Vim, which is great if you like Vim, but should you want to use
another text editor then you can change it with the following examples.

For nano:

```bash
git config --global core.editor "nano"
```

For Sublime text:

```bash
git config --global core.editor "subl --wait"
```

For VS Code:

```bash
git config --global core.editor "code --wait"
```

The --wait for Sublime and VS Code is important as it allows you to type text and will wait for save/close event.

### Git GPG signing key

If you want to sign your commits or tags with you gpg key then you will need
to tell git which key to use, and you also have the option of specifying it's 
use on every commit.

```bash
git config --global user.signingkey C88001C355A25170
git config --global commit.gpgsign true
```

Please change the key above to your own.

## GPG setup for git with Yubikey

### Importing the public key (if required)

If you have setup the yubikey to link to a URL with your public key, then you can run:

```bash
gpg --card-edit
```

this will open a gpg prompt where you can fetch the public key

```bash
gpg>
gpg>fetch
```

The public key is now imported. 

Alternatively, you can import the public key manually with:

```bash
gpg --import public.asc
```

Changing the name of the file to your public key file name.

### Finding your signing key ID

You can view the public keys available with:

```bash
gpg -k
```

This will list the public keys. Find the one for your UID, and note the keys 
denoted as "sub". The one with a [S] at the end is your signing key. Take note of 
where it is in the list of sub keys.

Then run:

```bash
gpg --card-status
```

This will list your keys on the card. The sub keys are denoted "ssb" in the same
order as the previous command. Your key is the letters and numbers after "rsa4096/".

## Clone a remote repository and commit a file

>Please note that as of 1st October 2020 GitHub started using 'main' instead
>of 'master' as the primary/default branch name. I have therefore followed this
>convention below. However, if you want to use 'master' just replace 'main' with
>'master' in what follows.

Go to the main folder where you want the repository cloned

```bash
git clone git@github.com:thetestspecimen/linux-notes.git
cd linux-notes
touch a-new-file.md
git add a-new-file.md
git commit -m "Add a-new-file"
git branch -M main
git push -u origin main
```

>Flag notes:
>'commit -m' allows a commit message to be supplies. In this case "Add a-new-file".
>'branch -M' renames the current branch (which is typically 'master') to main.
>'push -u' adds an upstream tracking reference, and allows use of git pull without arguments.

## Add git to an existing project and commit to remote (Github)

```bash
cd existing_folder
git init
git remote add origin git@github.com:thetestspecimen/linux-notes.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

## Change remote origin for project

```bash
cd existing_repo
git remote rename origin old-origin
git remote add origin git@github.com:thetestspecimen/linux-notes.git
git push -u origin --all
git push -u origin --tags
```

## Update a local repository from remote

```bash
git pull origin main
```

## Update branch from main

```bash
git checkout main
git pull
git checkout target_feature_branch
git merge main
git push -u origin target_feature_branch
```
## Merge branch into main

```bash
git checkout main
git pull origin main
git merge target_feature_branch
git push -u origin main
git branch -d target_feature_branch
git push origin :target_feature_branch
```

## Signed commits

If you wish to sign a commit then you need to use the -S (Note: the "S" is capitalised!)

```bash
git commit -S -m "First commit"
```

### The "-a" command

If you add "-a"  to the commit command it will include any updated / deleted / changed files that were committed previously, automatically.

```bash
git commit -a -S -m "First commit"
```

### Check if the commit was signed

After the commit is made it is possible to check if it was successful with the following command:

```bash
git log --show-signature -1
```

## Longer commits

There may be times when you need more than just a subject line to describe the commit.
In which case you need a message body.

To acheive this you need to do the commit step in a slightly different way.

Basically, you ommit the "-m" in the commit command like this:

```bash
git commit -a -S
```

Obviously, you could omit the "-a -S" if you like.

This will open your default text editor. You can then write a subject line, leave a blank line
and then write your message body.

Please take note of the message style section, which follows this section for more details as to 
an acceptable layout for the message

## Commit style

As expected, there is a common convension for commit style.

### Subject line

The first line of the commit message should be: 

1. 50 characters or less (although any length is allowed it is not advised)
2. Capitalise the first word
3. Do not end the line with a full stop
4. Use the imperitive (e.g. "Fix the car" not "Fixed the car", "Add new font" not "Added new font" etc.)

Another way of understanding how to word your subject line that is commonly mentioned is to complete the
following sentence:

If applied, this commit will...

e.g. If applied, this commit will **add the ability to use multiple fonts**

### Body

You can add a body to the commit. This is used to explain *why* you made this commit.

Use the same imperitive as the subject when you state you did something, but you 
can also use more descriptive explanations.

For example:

```bash
The error() and exception() methods are never called. Delete them.

Global variables 'oneTime' and 'twoTime' can be made local. Get rid of those variables, and replace with local variables.
```

Always wrap the body text at as close to 72 characters as you can.


## Contributing to GitHub Project

### Get the repo locally

1. Log into GitHub
2. Find the GitHub project you want to contribute to
3. Fork the project to your own github account (using the fork button on GitHub) 
4. Copy the link from the "Clone or Download" button in **YOUR** GitHub for the project we just cloned
5. cd into a selected folder on your computer (in my case GitHub in the home directory)
6. Clone the origin in the commandline: ```git clone https://github.com/your-user-name/linux-notes.git```
7. Move into the new project directory: ```cd linux-notes```
8. Add the upstream (i.e. the original project, not your copy) ```git remote add upstream https://github.com/thetestspecimen/linux-notes.git```
9. Get any upstream changes ```git fetch upstream```
10. Apply any upstream changes ```git merge upstream/main```
11. Push upstream changes to origin ```git push origin main```

### Create a new branch to work on 

1. create new branch: ```git checkout -b new-notes```
2. Make changes to files that you need to change
3. use ```git status``` to see a summary of what has changed
4. add changes to git: ```git add path/to/file-to-be-added-or-changed.py```
5. commit the changes: ```git commit -m "What has changed message"``` (always write commits in present tense)
6. push changes to github: ```git push origin new-notes``` (i.e. the branch name)


### Create pull request

If you are happy all is up to date (see next section). Then submit the pull request.

1. Submit using compare and pull request on github
8. Add necessary details and click "Create pull request"

This will then send a pull request to to the original developer, and if they wish they can merge your branch.
Otherwise they may request updates.

### Updating your branch from main

It is a good idea to keep the branch up to speed with main so there are no conflicts

1. Change to the feature branch ```git checkout new-notes```
2. Pull in the changes from remote ```git fetch upstream```
3. Merge the changes ```git merge upstream/main```
4. If conflicts occur [resolve them](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line)
5. Once changes done, add the new files ```git add -u```
6. Commit changes ```git commit```
7. Then push changes ```git push origin new-notes``` 

### Cleaning up after a successful pull request

To keep up to date with original repository (after branch has been added to original repository main branch):

1) ```git checkout main```
2) ```git remote add upstream https://github.com/thetestspecimen/linux-notes.git```
3) ```git fetch upstream```
4) ```git rebase upstream/main``` (this pulls in any changes from upstream)
5) ```git push origin main```

Now we need to remove branch we created earlier as it is not contained in main:

1) ```git branch -d new-notes```
2) ```git push origin --delete new-notes```

### References for contribution

https://codeburst.io/a-step-by-step-guide-to-making-your-first-github-contribution-5302260a2940
https://gist.github.com/MarcDiethelm/7303312

https://pandas.pydata.org/pandas-docs/stable/development/contributing.html
