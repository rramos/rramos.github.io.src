---
title: Sublime Install Guide
tags:
  - IDE
  - Install
  - Linux
date: 2017-09-23 09:18:18
---


For me [Sublime](https://www.sublimetext.com/), this is one of the best smart editors in Linux. Here are the quick instructions to install for Ubuntu.

```
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install sublime-text

```


That's it
RR