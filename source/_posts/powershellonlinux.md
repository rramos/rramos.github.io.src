---
title: PowerShell on Linux
tags:
  - PowerShell
  - Linux
  - Windows
date: 2017-10-06 23:11:39
---


This article will present the required steps to install [powershell](https://en.wikipedia.org/wiki/PowerShell) on Linux.

# Objective

The objective is to operate from Linux [Windows Azure Pack](https://www.microsoft.com/en-us/cloud-platform/windows-azure-pack).

# Procedure

* Import the public repository GPG keys

```sh
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
```

* Register the Microsoft Ubuntu repository

```sh
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list | sudo tee /etc/apt/sources.list.d/microsoft.list
```

* Update the list of products

```sh
sudo apt-get update
```

* Install PowerShell

```sh
sudo apt-get install -y powershell
```

* Start PowerShell

```sh
powershell
```

# Add Azure module

Install Azure module in powershell running with `sudo`

```
Install-Module AzureRM
Install-Module Azure
```

Check module location

```sh
(gmo -l Azure*).path
```


# Uninstall

```sh
sudo apt-get remove powershell
```

# Conclusion

After some hours i've given up making this thing work on Linux. The time it takes doesn't payoff, and even if i could make it work, i would still need to make Vagrant work with it for some VM's provisioning in a private cloud. It would take me less time to do it by hand :|

From a standard testing perspective it's nice to have this thing on Linux, because you could centralize the management, but at the moment of this article it doesn't seem to payoff the time one would need to setup this. 

If you have lot's of Windows machines or Azure infrastructure to manage just get a Windows PC, or wait for Microsoft to fix this, as there are lot's of reports regarding this issue. But seriously i don't see an effort from Microft to make this work properly, what's the gain, rigth !?.

Cheers,
RR


# References

* https://github.com/PowerShell/PowerShell
* https://github.com/bgelens/WAPTenantPublicAPI
* http://www.tech-coffee.net/windows-azure-pack-powershell-tenant-api/
* https://blogs.msdn.microsoft.com/powershell/2016/08/18/powershell-on-linux-and-open-source-2/
* https://blogs.technet.microsoft.com/privatecloud/2013/12/10/windows-azure-pack-reconfigure-portal-names-ports-and-use-trusted-certificates/
