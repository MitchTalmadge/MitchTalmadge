---
title: "Use SSH and tar to Backup Your Entire Linux Server"
author: "Mitch Talmadge"
date: 2016-06-23T03:20:44.000-0600
last_modified_at: 2019-04-18T02:10:55.287-0600
categories: ["Mitch Talmadge"]
tags: ["networking","other","backup","linux","programming"]
description: "A quick-and-dirty script to backup your entire linux server in one go."
---

### Use SSH and tar to Backup Your Entire Linux Server

Sometimes I just need to make a backup of every single thing that could ever possibly be important on my server\. Rather than selectively backup only the things I need, I made this script that will just backup the whole damn thing\! It even pipes the whole backup to another server\. 👾
```
tar -zcf - --directory / --exclude=mnt --exclude=proc --exclude=tmp --exclude=run --exclude var/spool . | ssh -i YOUR_RSA_KEY_PATH backups@my.backup-server.com "cat > /backup_disk/vps/full-backup-`date '+%d-%B-%Y'`.tar.gz"
```

All of the data is piped into `tar` , `gzip` , then `ssh` \. The data is never permanently saved to the server that is running the script\.
#### Don’t Forget:
- Replace `YOUR_RSA_KEY_PATH` with your own RSA key path, or just use a password\. I always use RSA keys on all of my servers for extended security, and disable password authentication\.
- Replace the `ssh` username and address\.
- Exclude any directories that should not be backed up\.


Let me know if you need any help\!




