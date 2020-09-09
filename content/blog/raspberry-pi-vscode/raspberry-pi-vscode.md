title: Installing VSCode in Raspberry Pi
date: 2020-09-09 16:30
modified: 2020-09-09 16:30
category: tutorial
tags: raspberry pi, vscode, linux
slug: raspberry-pi-vscode
author: satyajit-ghana
summary: Simple Tutorial on installing VSCode in Raspberry Pi

# Installing VSCode in Raspberry Pi

## Add the apt repository

```bash
wget https://packagecloud.io/headmelted/codebuilds/gpgkey -O - | sudo apt-key add -
```

## Install VSCode

```bash
curl -L https://raw.githubusercontent.com/headmelted/codebuilds/master/docs/installers/apt.sh | sudo bash
```

![install]({attach}install.png)

## Now it will be added to the Programming menu called `Code-OSS (heartmelted)`

![launch]({attach}vscode-launch.png)

## Open !

![vscode]({attach}vscode.png)

Something that you can do now is enable SSH, and then on your main machine open vscode and then open SSH Remote VSCode session! so now you can code on your main machine and execute on your pi ! isn't it amazing ?
