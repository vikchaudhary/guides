---
layout: single
title: "Securing an Apache Server"
permalink: /securing-an-apache-server/
toc: true
toc_sticky: true
font_size: 14px
---

Securing your web server on Apache is essential to protect your website and server from potential attacks. In this article, we will cover the following topics:

1. Discovering server intrusions
2. Automatically detecting intrusions
3. Blocking bad actors


# Discovering server intrusions by bad actors

The goal of this section is verifying whether your servers have been hacked. We will cover:

1. Searching Apache logs for **intrusion attempts**.

2. How to use Grep/Awk to parse logs for **suspicious patterns**. 

3. Identifying **attack signatures** such as such as CGI, directory traversal and RCE.

---

## Searching Apache logs for intrusion attempts

Once you suspect that your server has been hacked into, immediately verify the intrusion attempts by analysing your website's log files, such as Apache logs. Apache logs are in various places. 

On Ubuntu servers, the default Apache access log path is  `/var/log/apache`.  This directory will usually have `access.log`, `error.log`, `other_vhosts_access.log` and previous versions of some of these logs, usually rotated daily. 

In addition, each website running on your server will have its own logs, usually in directories like `/var/www/html/<site-directory>/logs`. 



Let's take a look at one of them:

`sudo tail -f /var/log/apache2/error.log`

The output is typically hard to understand unless you're a Unix expert.

> [Thu Jul 03 00:00:02.246058 2025] [mpm_prefork:notice] [pid 83742] AH00163: Apache/2.4.58 (Ubuntu) OpenSSL/3.0.13 configured -- resuming normal operations 
> 
> [Thu Jul 03 08:17:50.103918 2025] [core:error] [pid 130436] [client 115.76.223.110:39176] AH10244: invalid URI path (/cgi-bin/.%2e/.%2e/bin/sh) 
> 
> [Thu Jul 03 08:18:03.034236 2025] [core:error] [pid 130632] [client 115.76.223.110:43228] AH10244: invalid URI path (/cgi-bin/%%32%65%.../bin/sh) 

Using ChatGPT to parse the content will ease the task of identifying intrusions, which is what I recently did, confirming that a bad actor from IP address `115.76.223.110` had gained access to a server, and how they did it. 

## Securing a server from intrusions

Once you see IP addresses that are, say, attempting to `ssh` into your servers, and these IPs are not from your own local machines, then you should immediately take steps to block these IPs manually, as long as the list is small. Later, as this guide shows you, you will use intelligent automated methods to detect and block malicious attacks from IP addresses, at scale.

### Blocking an IP address

To immediately ban an IP address, you can block malicious IPs using `ufw` and Apache config. Later, we will use `fail2ban` to setup intrusion rules, alerts can be set up using log monitoring tools like `logwatch`, `logrotate+mail`, or custom scripts.

#### ✅ **Option 1: Block IP with UFW (firewall)**

`sudo ufw deny from 115.76.223.110 `

`sudo ufw reload`

#### ✅ **Option 2: Block in Apache Config**

Add to your Apache config or `.htaccess`:

```
<RequireAll>   
    Require all granted   
    Require not ip 115.76.223.110 
</RequireAll>
```

The next section will show you how to automatically detect intrusions.

# Automatically detect and block intrusions

`fail2ban` is software that detects access to your servers and if they are discovered to be engaging in suspicious or malicious access, will automatically ban them, and secure the server. These are topics we will cover.

1. How to setup a **Fail2Ban jail** (like `apache-cgitraversal`).

2. How to create a **custom filter (`apache-access-traversal`)** that correctly detects malicious access log entries.

3. How to **test filters** using `fail2ban-regex` to confirm they match actual log lines.

4. Why a jail triggered but **didn't result in a ban**.

5. How to **make bans permanent** (`bantime = -1`).

6. Understanding the meaning of **Fail2Ban log entries** like `Restore Ban`.

7. Why certain **IP addresses were not banned** automatically.

---

## How to setup a fail2Ban jail

### Install fail2ban for automatic blocking

Install and configure `fail2ban`:

`sudo apt install fail2ban` 

`sudo systemctl enable fail2ban`

#### ✅ Step 1: Create the custom filter file

Create a new filter at:

`sudo nano /etc/fail2ban/filter.d/apache-cgitraversal.conf`

Paste this content:

```
[Definition] 
failregex = \[client <HOST>:\d+\] AH10244: invalid URI path \(.*cgi-bin.*bin/sh.*
ignoreregex =
```

This looks for and matches lines in the Apache error log where:

- Matches `[client <HOST>:PORT]`

- Matches `AH10244: invalid URI path`

- Looks for `cgi-bin.*bin/sh`

---

#### ✅ Step 2: Define the jail

Edit or create:

`sudo nano /etc/fail2ban/jail.local`

Add this:

```
[apache-cgitraversal] 
    enabled  = true 
    port     = http,https 
    filter   = apache-cgitraversal 
    logpath  = /var/log/apache2/error.log 
    maxretry = 1    # even one match triggers a ban. 
    bantime  = 31536000    # ban for 1 year.
```

Then restart `fail2ban`:

`sudo systemctl restart fail2ban`

#### ✅ Step 3: Test the new fail2ban jail

You can simulate an attack by manually adding a dummy matching line to your Apache error log, then triggering `fail2ban` to re-scan it.

1. **Insert a dummy malicious log entry:**

```bash
echo '[Wed Jul 03 23:59:59 2025] [core:error] [pid 99999] \
[client 1.2.3.4:12345] AH10244: \
invalid URI path (/cgi-bin/../../../../bin/sh)' \
| sudo tee -a /var/log/apache2/error.log`
```

2. **Force Fail2Ban to recheck logs:**

`sudo fail2ban-client reload`

3. **Check if the IP is banned:**

`sudo fail2ban-client status apache-cgitraversal`

You should see:

- A banned IP count of `1`

- `1.2.3.4` in the list

Then, unban the IP (for cleanup)

`sudo fail2ban-client set apache-cgitraversal unbanip 1.2.3.4`

To view currently banned IPs:

`sudo fail2ban-client status apache-cgitraversal`

The output looks like:

`Status for the jail: apache-cgitraversal |- Filter |  |- Currently failed: 1 |  |- Total failed:     3 |_  - Banned IP list:   1.2.3.4 5.6.7.8`

## Recap

You now have a working `fail2ban` jail with:

- **Custom jail** (`apache-cgitraversal`) monitoring `/var/log/apache2/error.log`

- **Regex** that matches directory traversal probes using `/cgi-bin/.../bin/sh`

- **Ban duration** set to **1 year** (`bantime = 31536000`)

- **Trigger** on the **first match** (`maxretry = 1`)

---

## Check Fail2ban logs to see attempted intrusions

After a day, you can check Fail2ban logs, and see what intrusions were attempted. It is very likely that, especially if you have a website hosted on that server, you will see a lot of malicious intrusion attempts.

Use the `fail2ban` log files, and If you use log rotation (which is likely), check rotated logs:

`sudo zgrep 'Ban' /var/log/fail2ban.log.*.gz`

This shows all the IP addresses that were banned:

```
/var/log/fail2ban.log.2.gz:2025-08-03 19:39:40,790 fail2ban.actions
        [757]: NOTICE  [apache-access-traversal] Restore Ban 204.48.26.17

/var/log/fail2ban.log.2.gz:2025-08-04 13:20:48,564 fail2ban.actions
        [757]: NOTICE  [apache-access-traversal] Ban 137.131.43.224

/var/log/fail2ban.log.2.gz:2025-08-05 04:28:14,713 fail2ban.actions
        [757]: NOTICE  [apache-access-traversal] Ban 1.55.131.187

/var/log/fail2ban.log.2.gz:2025-08-07 08:51:06,810 fail2ban.actions
        [757]: NOTICE  [apache-access-traversal] Ban 194.62.248.69

/var/log/fail2ban.log.2.gz:2025-08-08 15:03:13,044 fail2ban.actions
        [757]: NOTICE  [apache-access-traversal] Ban 45.56.104.138
```

At this point, I recommend you use ChatGPT to help you understand attempted intrusions. Upload the `fail2ban.log` file or files, and use a prompt such as:

`My rotated logs show this output - help me understand what this means. extract a full list of unique IPs banned during this period with timestamps, and show me the repeat offenders.`

ChatGPT will give you a list like this:

![Screenshot 2025-08-20 at 8.41.15 PM.png](/Users/vik/Dropbox%20(Personal)/Screenshots/Screenshot%202025-08-20%20at%208.41.15%20PM.png)

### Blocking IP addresses using a firewall

Next, as ChatGPT suggests, you want to check that these IP addresses have been blocked at the firewall level. Your `fail2ban` jail should have executed `ufw` to block these IP addresses already, but it does not hurt to check. In the next section, we will show you how to use `ufw` - which stands for "Uncomplicated Fire Wall".

# Using a firewall (UFW)

1. How to **ban a list of IPs** via `ufw`.

2. Why **banned IPs from Fail2Ban were not in `ufw`**.

3. How to use **Fail2Ban with `ufw` as the action**.

4. How to **manually block repeat offenders** using `ufw`.

5. How to **match Fail2Ban bans with `ufw` rules**.

6. Script to **identify IPs banned by Fail2Ban but not `ufw`** and block them.

Note: we assume you have installed `ufw`, the Uncomplicated Fire Wall software. If you have not, then now is a good time to do it, and as an exercise, you can just ask ChatGPT for instructions on installing ufw.

## Ban the list of IPs that are repeat offenders

In the previous section, ChatGPT generated a list of IP addresses that were attempted to hack into your server often. Download this list, and then send it to ChatGPT:

`Generate ufw commands to block all IP addresses in this file`

Here’s a script that will read the file and ban each IP using `ufw`:

```bash
#!/bin/bash  

# Path to your CSV file FILE="/mnt/data/Repeated_Offender_IPs.csv"  

# Skip header and loop through IP addresses 
tail -n +2 "$FILE" | cut -d',' -f1 | while read ip; do
     if [[ -n "$ip" ]]; then
         echo "Banning $ip..."
         sudo ufw deny from "$ip"
    fi done  

# Reload ufw to apply changes 
sudo ufw reload
```

Save this file as `block_repeat_offenders.sh` and then run it

`chmod +x block_repeated_offenders.sh`
`./block_repeated_offenders.sh`

To see the list of IPs that are now blocked, run:
`sudo ufw status verbose`

You will see a list like this:

```bash
Anywhere                   DENY IN     167.99.133.232            
Anywhere                   DENY IN     193.32.162.157            
Anywhere                   DENY IN     167.172.172.72            
Anywhere                   DENY IN     185.93.89.118             
Anywhere                   DENY IN     185.6.2.126               
Anywhere                   DENY IN     68.183.49.235             
Anywhere                   DENY IN     195.178.110.125           
Anywhere                   DENY IN     82.223.10.156             
Anywhere                   DENY IN     103.140.127.215           
Anywhere                   DENY IN     104.244.77.50             
Anywhere                   DENY IN     5.202.75.5                
Anywhere                   DENY IN     92.118.39.92              
Anywhere                   DENY IN     195.178.110.108           
Anywhere                   DENY IN     92.118.39.95              
Anywhere                   DENY IN     107.173.61.177            
Anywhere                   DENY IN     139.59.226.171            
Anywhere                   DENY IN     211.197.21.157            
Anywhere                   DENY IN     161.132.50.174            
Anywhere                   DENY IN     178.250.191.146
```

---

## Why did Fail2Ban not automatically block IP addresses?

Fail2Ban did **not** automatically ban those IPs likely because they did **not match a jail's criteria**—i.e., they didn’t trigger enough logged failures, or the right log patterns weren’t matched. Common reasons:

1. **No matching log entries**: The IPs didn’t trigger the filters in your enabled jails (e.g., `apache-auth`, `apache-badbots`, etc.).

2. **Too few attempts**: Many jails require multiple failures (e.g., `maxretry = 3`) before banning.

3. **Wrong log paths or formats**: Fail2Ban may not be reading the correct log file or expected pattern.

4. **Jail not enabled**: The jail needed to catch those attacks may not be enabled (e.g., `apache-cgitraversal`, `ssh`).

5. **Whitelist/conflict**: The IPs may be in a `ignoreip` list or manually unbanned.

Debugging all these possibilities is a time-consuming exercise. Rather than give a single solution to these situations, I recommend a problem-solving exercise with ChatGPT or the LLM of your choice to figure out if `fail2ban` needs to be configured correctly to work with `ufw`. 

In the meanwhile, you have some practical steps to take.

### Manually ban IPs that should have been banned

So there were IP addresses that should have been banned by `fail2ban`, but were not banned in `ufw`? You can start with a simple ChatGPT query such as:

```
Find me all IP addresses that were banned in fail2ban, but were not 
banned in ufw
```

Here are the commands to **manually find IPs banned by Fail2Ban but not by UFW**:

---

#### ✅ 1. **Extract banned IPs from Fail2Ban log**

`grep 'Ban ' /var/log/fail2ban.log | awk '{print $NF}' | sort -u > /tmp/f2b_ips.txt`

---

#### ✅ 2. **Extract denied IPs from UFW**

`sudo ufw status | grep DENY | awk '{print $NF}' | sort -u > /tmp/ufw_ips.txt`

---

#### ✅ 3. **Find IPs banned by Fail2Ban but missing in UFW**

`comm -23 /tmp/f2b_ips.txt /tmp/ufw_ips.txt`

> `comm -23` shows lines in `f2b_ips.txt` **not** in `ufw_ips.txt`.

You can ask ChatGPT for a single script to all the above, as well as to ban the resulting IP addresses, and you will get something like this below. Save the file, make it executable, and run it.

```bash
#!/bin/bash

# Extract Fail2Ban banned IPs
sudo grep 'Ban ' /var/log/fail2ban.log | awk '{print $NF}' | sort -u > /tmp/f2b_ips.txt

# Extract UFW denied IPs
sudo ufw status | grep DENY | awk '{print $NF}' | sort -u > /tmp/ufw_ips.txt

# Find IPs banned by Fail2Ban but not in UFW
missing=$(comm -23 /tmp/f2b_ips.txt /tmp/ufw_ips.txt)

# Block each missing IP in UFW
echo "Blocking IPs missing from UFW:"
for ip in $missing; do
    echo "Blocking $ip"
    sudo ufw deny from "$ip"
done
```


