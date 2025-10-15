---
title: SSH Key Tutorial
layout: page
permalink: /tutorials/ssh.html
parent: Tutorials and Resources
nav_enabled: true
nav_order: 2
---
# SSH Key Tutorial 
This lesson is adapted from the *[Connecting to the remote HPC System](https://www.hpc-carpentry.org/hpc-shell/01-connecting/index.html)* lesson of HPC Carpentry.

## What is an SSH Key?
SSH keys are a method for authentication for command line access to remote computing systems. 
After creating an SSH key on a given device, you likely won't need to 
create another one.

SSH keys can also be used for authentication when transferring files from shared servers or for accessing version control systems like GitHub. 

In this section you will create a pair of SSH keys, a private key which you keep on your own computer and a public key which is placed on the remote HPC system that you will log in to.

### A Note on OS Compatibility
If you installed Git Bash using my [recommended instructions]({% link tutorials/windows-git-guide.md %}, you should have access all the ssh key generation commands that your UNIX classmates have through Git Bash. 
You should not need to install any additional software.

If something doesn't work as intended, please raise your hand!

## Checking for Existing Keys
Those of you who have used platforms like GitHub before may have
created an SSH key without realizing you had done so.

SSH keys are their associated configurations are typically stored 
in a hidden folder called `.ssh` in your your home directory.

Hidden folders are hidden from GUIs like File Explorer and Finder by default. 
To see them you must either change how your GUI displays files or use the special `-a` flag with commands like `ls`.

```bash
ls -aF ~
```

You'll probably see a bunch of hidden files related to different
applications, but for now, see if a .ssh folder is there. Don't worry if you don't have one, as the SSH key creation process will take care of that.
```output
.ssh/
...
```

If you do have a hidden `.ssh` folder, check its contents as follows.
```bash
ls ~/.ssh/
```

If you've created an SSH key before, you'll have a list of text files that look something like this.
ed25519 refers to cryptographic algorithm for generating SSH key pairs, so `id_ed25519` is the default name for SSH keys generated using that tool.
```output
config      id_ed25519.pub  known_hosts.old  
id_ed25519  known_hosts     
```
If you have two files named `id_ed25519.pub` and `id_ed25519` respectively, wait. Skip to the `Add your SSH Key to Talapas` step.
If you do *not* have any files with `.pub` suffix (no public/private key pair) continue to the key generation step.

## No SSH Key Pair? Create One
From your terminal run the following command: 
```bash
ssh-keygen -t ed25519 -C "[YOURDUCKID]@uoregon.edu"
```
This creates a new SSH key with the ed25519 algorithm, using your UOregon (presumably professional) email as a label.

```ouput
> Generating public/private ALGORITHM key pair.
> Enter a file in which to save the key: [Press enter]
```

When you're prompted, type **Enter** to accept the default file location. 
The default location is the `~/.ssh` directory we just checked.

**Do not do this step if you already have a key in this location to avoid overwriting it.**

You will then prompted to enter an optional passphrase for your SSH key.
```output
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```
To leave your SSH key without a passphrase, type **Enter** twice more.

## Add Your SSH Key to Talapas

At this point, all of you should be ready to connect to Talapas. Type the following command into your terminal application.

```bash
ssh [yourDuckID]@login2.talapas.uoregon.edu
```
If this is your first time connecting to Talapas through SSH, you will be prompted with a long, unwieldy message that ends with this line.

```ouput
The authenticity of host 'login2.talapas.uoregon.edu (128.223.192.241)' can't be established.
ED25519 key fingerprint is ...
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` (lowercase) and press **Enter** to dismiss the message. You will not prompted with it again unless you change devices or create a new SSH key.

In the future, feel free to use any of the 4 login nodes or to choose the load balancer at `login.talapas.uoregon.edu`. For today, let's
all log on to the same node: `login2.talapas.uoregon.edu`.

![example Talapas welcome message](../images/talapas-welcome.png) *An example Talapas welcome message*

Congratulations, you're now on Talapas.

## Reduce Those Duo Prompts: Copying your SSH (Public Key) to Talapas

Once you've logged on to a Talapas login node through the terminal for the first time, type `exit` to exit.

To reduce the number of Duo 2FA prompts, you can take the following steps:

1. Connect to the UO VPN **uovpn** not **uoprod** before connecting to Talapas.
![UO VPN Screenshot](../images/vpn.JPG)
2. Connect for the first time via `ssh`, say `yes` to the prompt.
3. Add your SSH public key to the `~/.ssh/authorized_keys` file on Talapas.

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub [yourDuckID]@login2.talapas.uoregon.edu
```

```output
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/[YOURUSERNAME]/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys

Number of key(s) added: 1
```

Next, check that your key has been added to the `authorized_keys` file on Talapas.

```bash
ssh [yourDuckID]@login2.talapas.uoregon.edu
cat ~/.ssh/authorized_keys
```
You should see something like this if your key has been successfully added.

```output
ssh-ed25519 ... yourname@uoregon.edu
```

## Talapas Connection Troubleshooting Guide
1. Are you on the VPN? Are you on the `uovpn`? (Do not use the `prod` vpn to connect to Talapas.)
2. Have you connected to this **particular** login node with this SSH key before? You will have to authenticate by password the first time irregardless.
3. Have you cleared your browser cache for the last 24 hours? (OnDemand only)