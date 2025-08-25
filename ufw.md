---
layout: single
title: "Use Uncomplicated Firewall (UFW) to Block Bad Actors"
permalink: /guides/securing-an-apache-server/ufw/
toc: true
toc_sticky: true
---

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

### 

---

#
