---
title: Quick Guide for Vagrant and Libvirt
tags:
  - Vagrant
  - Qemu
  - KVM
  - libvirt
  - Mesos
  - Ansible
date: 2017-09-26 23:43:35
---


This article is going to be about a quick way to setup a PoC environment using [libvirt](https://libvirt.org/) ([QEMU](https://www.qemu.org/)) and [vagrant](https://www.vagrantup.com/), and auto configure the enviroment using [ansible](https://www.ansible.com/).

# Why ?

One migth go a bit further in the testing enviroments, instead of dockers using virtualized enviroment. This kind of approcah whould be suitable when you like to try infrastructure services, or simple don't want to use dockers.

If you check Internet Trends we can see docker interest have grown quite a lot on the last 5 years. 

{% raw %}
<script type="text/javascript" src="https://ssl.gstatic.com/trends_nrtr/1154_RC03/embed_loader.js"></script> <script type="text/javascript"> trends.embed.renderExploreWidget("TIMESERIES", {"comparisonItem":[{"keyword":"docker","geo":"","time":"today 5-y"},{"keyword":"Virtual Machines","geo":"","time":"today 5-y"}],"category":13,"property":""}, {"exploreQuery":"cat=13&date=today 5-y,today 5-y&q=docker,Virtual%20Machines","guestPath":"https://trends.google.com:443/trends/embed/"}); </script> 
{% endraw %}

**Still i'm OldSchool and i like my plain-simple-VMs some times :D**

A later article would be about configuring a Mesos Cluster so this one makes sense as a preparation guide for it.

# Requirements

* Make sure you have vagrant installed
* Make sure your machine supports virtualization :)
* Make sure you have QEMU installed
* Make sure you have libvirt up and running
* Make sure you have ansible (Optional)

**Note:** This guide was tested on Ubuntu 16. Use at your own risk

One can test libvirt version suing the command

```
$ virsh version
Compiled against library: libvirt 1.3.1
Using library: libvirt 1.3.1
Using API: QEMU 1.3.1
Running hypervisor: QEMU 2.5.0
```

**NOTE:** Before you start using Vagrant-libvirt, please make sure your libvirt and qemu installation is working correctly and you are able to create qemu or kvm type virtual machines with virsh or virt-manager.

# setup

Install dependencies

``` 
sudo apt-get install qemu libvirt-bin ebtables dnsmasq
sudo apt-get install libxslt-dev libxml2-dev libvirt-dev zlib1g-dev ruby-dev
sudo apt-get install vagrant ruby-libvirt
```

Install vagrant-libvirt plugin:

```
vagrant plugin install vagrant-libvirt
```

Now let's test by creating a startup Box. There are several available [here](https://app.vagrantup.com/boxes/search?provider=libvirt)

Example: (Ubuntu 16)

```
mkdir Vagrant
cd Vagrant
vagrant init marczis/ubuntu_16_04
```

Let's start the VM

```
vagrant up --provider=libvirt
```

After executing the comman you should have some output similar

```
==> default: Successfully added box 'marczis/ubuntu_16_04' (v0.1.2) for 'libvirt'!
==> default: Uploading base box image as volume into libvirt storage...
==> default: Creating image (snapshot of base box volume).
==> default: Creating domain with the following settings...
==> default:  -- Name:              Vagrant_default
==> default:  -- Domain type:       kvm
==> default:  -- Cpus:              1
==> default:  -- Feature:           acpi
==> default:  -- Feature:           apic
==> default:  -- Feature:           pae
==> default:  -- Memory:            512M
==> default:  -- Management MAC:    
==> default:  -- Loader:            
==> default:  -- Base box:          marczis/ubuntu_16_04
==> default:  -- Storage pool:      default
==> default:  -- Image:             /var/lib/libvirt/images/Vagrant_default.img (10G)
==> default:  -- Volume Cache:      default
==> default:  -- Kernel:            
==> default:  -- Initrd:            
==> default:  -- Graphics Type:     vnc
==> default:  -- Graphics Port:     5900
==> default:  -- Graphics IP:       127.0.0.1
==> default:  -- Graphics Password: Not defined
==> default:  -- Video Type:        cirrus
==> default:  -- Video VRAM:        9216
==> default:  -- Sound Type:    
==> default:  -- Keymap:            en-us
==> default:  -- TPM Path:          
==> default:  -- INPUT:             type=mouse, bus=ps2
==> default: Creating shared folders metadata...
==> default: Starting domain.
==> default: Waiting for domain to get an IP address...
==> default: Waiting for SSH to become available...
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /home/rramos/Development/Vagrant/ => /vagrant
==> default: Check for insecure vagrant key: OK (not present)
```

If you execute `virt-manager` you can see the virtual machine specification and adapt the Vagrant file to you needs.


{% asset_img "virt-manager.png" "virt-manager" %}

You can then ssh to the VM using the command `vagrant ssh`. You should have a prompt like:

```
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-77-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
vagrant@ubuntu:~$ 
```

You can add the ssh key to your ssh configuration executing the following command. ( Check my previous  {% post_link ssh-config [article] %} regarding this topic)

```
vagrant ssh-config >> ~/.ssh/config
```


then try the command `ssh default` . Default if the default name of the VM defined in the vagrant file. 

# Let's extend a bit.

The next step is to set Ansible as our provisioning provider for the Vagrant Box. Add the following lines before the end statement in your Vagrantfile to set Ansible as the provisioning provider:

```
config.vm.provision :ansible do |ansible|
  ansible.playbook = "playbook.yml"
end
```

Let's create the playbook.yml file with the following content

```
---
- hosts: all
  become: yes
  become_user: root
  tasks:
    - name: install apache2
      apt: name=apache2 update_cache=yes state=latest
  handlers:
    - name: restart apache2
      service: name=apache2 state=restarted

```

This example playbook will install apache service on the all hosts given by Vagrantfile and start the service.

Then we can run the command to just run the provision stuff

```
vagrant provision
```

Let's add the following configuratoin on our Vagrant file

```
config.vm.network "forwarded_port", guest: 80, host: 8080
```

And execute `vagrant reload`

If one access http://localhost:8080 we are now accessing the VM installed service.

# Cluster Configuration

One could have Vagrant more complex configuration like the following for several VMs.

```
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "echo Hello"

  config.vm.define "web" do |web|
    web.vm.box = "apache"
  end

  config.vm.define "db" do |db|
    db.vm.box = "mysql"
  end
end
```

# Conclusion

In this article is described a simple way to integrate Vangrant with libvirt for quickly spinning VM's and running ansible playbooks on them.

This could be usefull for testing ansible playbooks before shipping them to LIVE enviroments or testing purposes. 

Also to quickly configure a more complex infrastructure, where you don't wont to use dockers.

On the next article i will write a step-by-step guide to spin up a Mesos Cluster using this approach, so this article could be used as prep-guide.

Cheers,
RR

# References

* https://libvirt.org/
* https://www.qemu.org/
* https://www.vagrantup.com/
* https://www.ansible.com/
* https://github.com/vagrant-libvirt/vagrant-libvirt
* https://app.vagrantup.com/boxes/search?provider=libvirt
* https://fedoramagazine.org/using-ansible-provision-vagrant-boxes/
* https://www.vagrantup.com/docs/multi-machine/
