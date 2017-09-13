---
title: Setting up Proxy Environments with Chrome
tags:
  - GoogleChrome
  - Customization
  - Internet
  - Proxy
  - Linux
date: 2017-09-13 22:18:10
---


Something that keeps bugging me is the need to access different environments in the browser through proxy tunnels, due to security restrictions or lack of routing between your local network and the environment you are working. Very common on Cloud environments or if you are working has a consultant for several clients at the same time.

So this guide is intended to help on the setup of Google Chrome to use your configured tunnels.

## Requirements

### Configure you local tunnel

You require a tunnel configuration setup. For Linux i recommend setting this up in your `.ssh/config` file. I'll give some examples, if you have interest on expanding you `./ssh/config` check the next article ill go in depth with it. There are also other tools such as [gSTM](https://sourceforge.net/projects/gstm/) tool.

So let's begging to create our reverse tunnel

```
ssh -R 19999:localhost:22 sourceuser@remoteip
```

You could use any other available localport

### Install plugin in Google Chrome

Go into your google chrome settings and search for **Extensions** .
I'm using **Proxy SwitchySharp** But you could use any other.

Go to your Extension Options and let's create a profile

* **Name:** mylocaldomain.dev
* **SOCKS HOST:** localhost
* **Port:** 19999
* **SOCKS v5:** (enable)

No let's create a rule for our local domain in **Switch Rules**

Add the rule for our domain: 

* **Rule Name:** MyLocalDomain
* **URL Pattern:** *.mylocaldomain.dev
* **Pattern Type:** Wildcard
* **ProxyProfile:** mylocaldomain.dev

**NOTE:** Don't forget to active the option **Enable Switch Rules**

Also don't forget to activate the **Auto Switch Mode** in the extension.

And that's it now when you access the browser to domain http://something.mylocaldomain.dev you be using your define ssh tunnel for it.


## Test

Lets do a quick test :)

```
sudo sh -c 'echo "127.0.0.1  test.mylocaldomain.dev"  >> /etc/hosts'
ssh -R 19999:localhost:22 localhost
python -m SimpleHTTPServer 8000
```

Now access on the browser to http://test.mylocaldomain.dev:8000

That's' it, you should now have different profiles configured and autoswitch will do the job.

Cheers,
RR 






