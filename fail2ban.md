---
layout: single
title: "Automatically Detect Intrusions with Fail2Ban"
permalink: /securing-an-apache-server/fail2ban/
toc: true
toc_sticky: true
---

`fail2ban` is software that detects access to your servers and if they are discovered to be engaging in suspicious or malicious access, will automatically ban them, and secure the server. These are topics we will cover.

1. How to setup a **Fail2Ban jail** (like `apache-cgitraversal`).

2. How to create a **custom filter (`apache-access-traversal`)** that correctly detects malicious access log entries.

3. How to **test filters** using `fail2ban-regex` to confirm they match actual log lines.

4. Why a jail triggered but **didn't result in a ban**.

5. How to **make bans permanent** (`bantime = -1`).

6. Understanding the meaning of **Fail2Ban log entries** like `Restore Ban`.

7. Why certain **IP addresses were not banned** automatically.

---

## How to setup a **Fail2Ban jail**™

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

## Recap: What You Now Have

- **Custom jail** (`apache-cgitraversal`) monitoring `/var/log/apache2/error.log`

- **Regex** that matches directory traversal probes using `/cgi-bin/.../bin/sh`

- **Ban duration** set to **1 year** (`bantime = 31536000`)

- **Trigger** on the **first match** (`maxretry = 1`)

---

# Check Fail2ban logs to see attempted intrusions

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

## Blocking IP addresses using a firewall

Next, as ChatGPT suggests, you want to check that these IP addresses have been blocked at the firewall level. Your `fail2ban` jail should have executed `ufw` to block these IP addresses already, but it does not hurt to check. In the next section, we will show you how to use `ufw` - which stands for "Uncomplicated Fire Wall".
