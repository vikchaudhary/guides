---
title: Discovering Intrusions by Bad Actors
parent: Securing an Apache Server
nav_order: 2
---

## Discovering Intrusions by Bad Actors

The goal of this section is verifying whether your servers have been hacked. We will cover:

1. Searching Apache logs for **intrusion attempts**.

2. How to use Grep/Awk to parse logs for **suspicious patterns**. 

3. Identifying **attack signatures** such as such as CGI, directory traversal and RCE.

---

### Searching Apache logs for intrusion attempts

Once you suspect that your server has been hacked into, immediately verify the intrusion attempts by analysing your website's log files, such as Apache logs. Apache logs are in various places. 

On Ubuntu servers, the default Apache access log path is  `/var/log/apache`.  This directory will usually have `access.log`, `error.log`, `other_vhosts_access.log` and previous versions of some of these logs, usually rotated daily. 

In addition, each website running on your server will have its own logs, usually in directories like `/var/www/html/<site-directory>/logs`. 



Let's take a look at one of them:

`sudo tail -f /var/log/apache2/error.log`

The output is typically hard to understand unless you're a Unix expert.

> [Thu Jul 03 00:00:02.246058 2025] [mpm_prefork:notice] [pid 83742] AH00163: Apache/2.4.58 (Ubuntu) OpenSSL/3.0.13 configured -- resuming normal operations 
> 
> [Thu Jul 03 08:17:50.103918 2025] [core:error] [pid 130436] [client 115.76.223.110:39176] AH10244: invalid URI path (/cgi-bin/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/.%2e/bin/sh) 
> 
> [Thu Jul 03 08:18:03.034236 2025] [core:error] [pid 130632] [client 115.76.223.110:43228] AH10244: invalid URI path (/cgi-bin/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/bin/sh) 

Using ChatGPT to parse the content will ease the task of identifying intrusions, which is what I recently did, confirming that a bad actor from IP address `115.76.223.110` had gained access to a server, and how they did it. 

### Securing a server from intrusions

Once you see IP addresses that are, say, attempting to `ssh` into your servers, and these IPs are not from your own local machines, then you should immediately take steps to block these IPs manually, as long as the list is small. Later, as this guide shows you, you will use intelligent automated methods to detect and block malicious attacks from IP addresses, at scale.

#### Blocking an IP address

To immediately ban an IP address, you can block malicious IPs using `ufw` and Apache config. Later, we will use `fail2ban` to setup intrusion rules, alerts can be set up using log monitoring tools like `logwatch`, `logrotate+mail`, or custom scripts.

##### ✅ **Option 1: Block IP with UFW (firewall)**

`sudo ufw deny from 115.76.223.110 `

`sudo ufw reload`

##### ✅ **Option 2: Block in Apache Config**

Add to your Apache config or `.htaccess`:

```
<RequireAll>   
    Require all granted   
    Require not ip 115.76.223.110 
</RequireAll>
```

The next section will show you how to automatically detect intrusions.
