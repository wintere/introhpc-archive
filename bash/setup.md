---
title: Setup
layout: page
permalink: /bash/setup.html
nav_enabled: true
parent: Introduction to Bash
nav_order: 1
---
# Setup

## Installation

This workshop is designed to teach Bash commands and key filesystem concepts to beginners on their **own devices**. This means that participants may encounter commands (like `man` command or the `--help` flag) that behave differently based on their laptop's operating system. I have marked these instances wherever possible.

Do not worry too much about these distinctions! Lessons after today are conducted on Talapas, a RedHat Linux cluster. Differences among laptops are less relevant when everyone is on the same operating system. 

### Windows
Because Windows is not a UNIX-based operating system, Windows computers do not come with a Bash shell. For teaching purposes, Windows participants will rely on an emulator included with Git called Git Bash. 

To start, download [Git](https://git-scm.com/downloads), install it, and open the Git Bash application from your Start Menu, which emulates a Bash shell on a Windows operating system. 

Need help with the installation steps? I have [included a screen-by-screen guide]({% link tutorials/windows-git-guide.md %})

### MacOS
Open the Terminal application by searching for "Terminal" in Spotlight Search. The default shell [Zsh](https://support.apple.com/en-us/102360) will run all of the Bash commands in this lesson. If you would like to launch a Bash subshell, type the command
`bash` and press <kbd>Enter</kbd>.

### Linux 
Open the Terminal application by searching for "Terminal" in the Applications menu. On popular deployments like Ubuntu, Fedora, and Debian, the default shell is Bash. 

## Download and Extract Files
You need to download and extract a zip folder of activity files to follow along with this workshop. 

Home directory absolute paths vary among operating systems. On Macs, it's typically `/users/joe`, on Windows, it will typically `C:\Users\Joe`, and on Linux, it may look like `/home/joe`. 

1. Download `talapas-bash.zip` by clicking the button below and move the file to your home directory.
2. Unzip/extract the file.
    * **Mac**: Double-click the .zip file in Finder.
    * **Windows**: Right-click the .zip in Windows Explorer, select **Extract All**. 
    * **Linux**: Right-click the .zip in the GUI, select **Extract Here**.
    * **Feeling confident?**: Try `unzip talapas-bash.zip` from your Git Bash or Terminal application, which should open to your home directory.
3. You should now have a folder called **`talapas-bash`** in your home directory.

<br> 
<span class="fs-5">
[Download Zip](../downloads/talapas-bash.zip){: .btn .btn-purple}
</span>