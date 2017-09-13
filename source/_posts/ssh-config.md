---
title: Optimize your SSH Connections with SSH config File
tags:
  - Linux
  - ssh
date: 2017-09-13 23:13:24
---


If most of your work is done remotely though ssh, and you have to access several environments, there will come a time where you need to organize our connection settings. Which user you need to access server X or which port is configured or the case you work in consulting and have several ssh_keys.

There are some tools that migth help on this, but i'm old school and still stick with plain ssh command.

ssh config file helps quite a lot, here are some tips unknown to some:

## Alias

Let's say you want to take advantage of tab auto-completion when using your connections for a enviroment like

```
├── ClientA
│   ├── server1
│   └── server2
├── ClientB
│   └── server1
├── DEV
│   ├── BackOffice
│   │   ├── server1
│   │   └── server2
│   └── FrontOffice
│       ├── server1
│       ├── server2
│       └── server3
├── LIVE
│   ├── Cluster
│   │   ├── node01
│   │   ├── node02
│   │   └── node03
│   └── Servers
│       ├── server01
│       └── server02
└── QA
    └── server1
```

this would be quicker to do something like `ssh DEV` **TAB** `Back` **TAB** `server1`

that's actually possible with ssh_config alias.

Add the following to `~/.ssh/config` to see this in action

```
Host ClientA.server1
     Hostname localhost
Host ClientA.server2
     Hostname localhost
Host ClientB.server1
     Hostname localhost
Host ClientA.server1
     Hostname localhost
Host DEV.BackOffice.server1
     Hostname localhost
Host DEV.BackOffice.server2
     Hostname localhost
Host DEV.FrontOffice.server1
     Hostname localhost
Host DEV.FrontOffice.server2
     Hostname localhost
Host DEV.FrontOffice.server3
     Hostname localhost
Host LIVE.Cluster.node01
     Hostname localhost
Host LIVE.Cluster.node02
     Hostname localhost
Host LIVE.Cluster.node03
     Hostname localhost
Host LIVE.Servers.server01
     Hostname localhost
Host LIVE.Servers.server02
     Hostname localhost
Host QA.server01
     Hostname localhost
```

Now try out the power of Tab-autocompletion. this is just an example of a type of structure you could use. 

You could also add alias like

```
Host LIVE.Servers.server02 server01.mydomain.com
     Hostname localhost
```

So that both ssh attempts to `LIVE.Servers.server02` and `server01.mydomain.com` would use the same configuration.

## Access customizations

No let's say for accessing `LIVE.Servers.server01` you require account `admin` and ssh listens on port `2228` . one could setup the following

```
Host LIVE.Servers.server01
     Hostname localhost
     Port 2228
     User admin
```

With this configuration one could simple execute `ssh LIVE.Servers.server01` and it will use the configured user and port in the connection.

Or if you have a specific ssh_key for it in `QA`

```
Host QA.server01
     Hostname localhost
     Port 2227
     User qa_user01
     IdentityFile ~/.ssh/id_rsa_qa
```

## Tunnels

one could also setup tunnels directly in ssh_config like

```
Host tunnel
    HostName database.example.com
    IdentityFile ~/.ssh/id_rsa_dev
    LocalForward 9906 127.0.0.1:3306
    User dba    
```

You can simple execute `ssh -f -N tunnel`

Or if you have access to server3 only from server1

```
Host DEV.FrontOffice.server3
     Hostname 10.0.0.1
     Port 22
     User admin
     IdentityFile ~/.ssh/id_rsa_dev
     ProxyCommand ssh -q -W %h:%p DEV.FrontOffice.server1
```

One configuration i normally use in development with containers or virtual machines wich are deprovision with regularity is the following:

```
Host 192.168.77.* mesos-*
   StrictHostKeyChecking no
   User rramos
   IdentityFile ~/.ssh/id_rsa
   UserKnownHostsFile=/dev/null
   LogLevel QUIET
```

This means any ssh connection to local network `192.168.77.*` or hosts with name `mesos-*` wont get registered in KnownHosts.

You could also use this to change your settings for `TCPKeepAlive` or any other specific connections settings you may need the [man](https://linux.die.net/man/5/ssh_config) page as the full list of options.

Cheers,
RR


# References

* https://linux.die.net/man/5/ssh_config