---
title: "A Solution to Postfix's \"unknown group: vmail\" Error"
author: "Mitch Talmadge"
date: 2015-08-27T01:05:00.000-0600
last_modified_at: 2018-05-25T21:12:09.157-0600
categories: ["Mitch Talmadge"]
tags: ["other","networking","linux","email","error"]
description: "My Postfix mail server was giving me yet-another headache. The fix was easy, but took forever to find!"
---

### A Solution to Postfix’s “unknown group: vmail” Error

Recently I migrated from one VPS to another, which involved re\-installing many pieces of software and attempting to repair permissions on many files and directories\. During the process of migrating, I started to notice errors appearing on my mail\.err log file apparently thrown by postfix every couple of minutes, which looked like this:
```
postfix/pipe[4274]: fatal: get_service_attr: unknown group: vmail
```

**For the solution, skip to the bottom\.**

The error at hand was preventing any emails from sending or receiving on my server\. I had no idea what emails I was missing, and it was causing problems\. Among other important emails, I started missing comment notifications on this website, and crash report notifications for my programs\. I needed to find the solution, and I needed to find it fast\.

Unsure of where I went wrong, I began to check the `/etc/group` file and the `/etc/passwd` files to make sure that both the `vmail` group and user existed — and they did\. I tried everything I could think of to find the source of the error, from restarting the server, to reinstalling postfix and dovecot, to deleting the `vmail` user and group and re\-adding them\. I even attempted creating a whole new user and group, reassigning all the permissions, then changing all the `vmail` occurrences in every file I could find to the new group’s name\. Nothing worked, and I was losing hope quickly\.

With Google being no help, I turned to the Unix & Linux section of StackExchange for help\. I posted [a question](http://unix.stackexchange.com/questions/225522/postfix-unknown-group-vmail) and waited, but got no comments or answers after over 24 hours\. My question quickly got buried and I decided that I couldn’t sit around and wait any longer\.

I was really considering starting over and installing everything again \(including Debian\), but I decided to give it all one more look\-over before I pulled the trigger and wiped everything\. While staring blankly at the `/etc/group` file and wondering why the heck nothing was working, I suddenly realized that I had never checked the permissions of the file\.
### Solution:

I ran `ls -l /etc/group` and… **OMG\.** The group file had no world read permissions\. This whole time, postfix has been constantly trying to read the group file to check if the group existed, but was **not allowed** to because somewhere in the process of my VPS migration, I accidentally removed all world permissions from the group file\. Running `chmod 644 /etc/group` and rebooting my server solved all my problems, and emails began to pile in from the nearly 5\-day\-old mail queue\.




