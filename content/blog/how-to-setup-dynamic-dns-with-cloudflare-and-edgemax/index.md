---
title: "How to Setup Dynamic DNS with Cloudflare and Edgemax"
date: "2023-11-21"
description: "Cloudflare's API and Ubiquiti's EdgeOS walk into a bar..."
tags: ['technology', 'networking']
---

I'm a big proponent of building a home network of whatever size or complexity. For technical and non-technical people it can help you understand the fundamentals of how the internet works. One small part of those fundamentals is giving yourself the ability to connect to your house from anywhere in the world. You can do that with a public IP address but what happens when your modem reboots and that IP address changes?

This is where a tool like Dynamic DNS can be very helpful. On top of an easier to remember DNS hostname it also automatically updates itself to whatever new public IP address you get. There are a lot of different ways to use Dynamic DNS. However, for my own setup, I have my domain hosted with Cloudflare and my hardware is a Ubiquiti Networks EdgeMax EdgeRouter Pro running EdgeOS `v2.0.9-hotfix.7`. 

The steps for setting this up are outlined in the [UniFi Help Center](https://help.ui.com/hc/en-us/articles/204976324-EdgeMAX-Custom-Dynamic-DNS-with-Cloudflare). But further context is needed in many cases.

Once connected to your router, enter configuration mode with `configure` and apply the dynamic DNS hostname. Specify the appropriate interface that has the public IP address. In my case it was `eth1`.

```
set service dns dynamic interface eth1 service custom-cloudflare host-name subdomain.domain.com
```

Specify the credentials used by the Edgerouter to connect to Cloudflare's API. The `login` parameter is the email address used to access the Cloudflare dashboard. The `password` value is the *Global API Key* found on the [API Tokens page](https://dash.cloudflare.com/profile/api-tokens) under *My Profile*.

```
set service dns dynamic interface eth1 service custom-cloudflare login ryan@ryan.com
set service dns dynamic interface eth1 service custom-cloudflare password xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Specify the protocol the Edgerouter will be using in order to format messages correctly.

```
set service dns dynamic interface eth1 service custom-cloudflare protocol cloudflare
```

Define a domain for the Cloudflare zone. If using `subdomain.domain.com` this would be `domain.com`.

```
set service dns dynamic interface eth1 service custom-cloudflare options zone=domain.com
```

Before committing and saving this new configuration make sure the necessary `A` record on the Cloudflare portal have been added. This DNS record would have the type of `A` and the name would be `subdomain` and the content doesn't necessarily matter but I put a private IP address to avoid any conflicts. The value will just get automatically updated later anyway. Additionally, the proxy status should be `DNS only` with an `Auto` TTL. The changes can then be committed and saved to the configuration.

```
commit
save
exit
```

If everything is working you will see the `A` record IP address update itself after a few minutes. Further verification can be obtained from the Edgerouter, which will share the status and latest timestamp.

```
ryan@router:~$ show dns dynamic status
interface    : eth1
ip address   : x.x.x.x
host-name    : subdomain.domain.com
last update  : Tue Nov 21 12:36:03 2023
update-status: good
```
