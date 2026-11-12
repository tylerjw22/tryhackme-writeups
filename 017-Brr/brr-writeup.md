# [Brr] Writeup

**TryHackMe:** [https://tryhackme.com/room/brr]

## Overview

**Category** -- [Web exploitation - Industrial Control Systems]

**Difficulty** -- [Easy]

**Provided Info** -- [A single target machine IP]

**Questions:**

- `What's the flag?`

---

## 1. Nmap Recon
Started with a default nmap scan with the service version detection flag (-sV) on all ports (-p-):
```
nmap -p- -sV [target_ip]
```
Results:

| Port | Service                             | What it does                      |
| ---- | ----------------------------------- | --------------------------------- |
| 22   | OpenSSH 9.6p1                       | Remote shell connection protocol  |
| 80   | Websockify Python/3.12.3            | Python web server                 |
| 5020 | zenginkyo-1                         | (This process is unknown for now) |
| 5901 | VNC (protocol 3.8)                  | Remote desktop service            |
| 8080 | Apache Tomcat/Coyote JSP engine 1.1 | Java web application server       |

The zenginkyo process is suspicious so i checked it out using `netcat` to see if it had a banner that would reveal any information:
```
nc -nv [target_ip] 5020
```

I got no response at all not even a process name, and every time i typed something it quit instantly.
## 2. Logging into ScadaBR
So i continued to check out the website on port 8080 and it led me to a `ScadaBR` login page:
![[017-Brr/images/image0.png]]

ScadaBR is used for industrial monitoring of OT (Operational Technology) equipment and allows users to view and manage data and equipment via the web page. After some researching i discovered that ScadaBR has default credentials that sometimes people forget to change. So i logged in with the username `admin` and the password `admin`, and it worked. After logging in it took me to the dashboard:
![[017-Brr/images/image1.png]]

## 3. Finding the data sources
After clicking around a bit i found the data sources section which lists a data source named "secret" that lists its "Connection" as "plc:5020". This is suspicious and it explains what the "zenginkyo-1" process on port 5020 is (zenginkyo-1 is just Nmaps guess at what the process is based on the port number, the process actually has nothing to do with zenginkyo-1):
![[017-Brr/images/image2.png]]
## 4. Contacting the PLC with the Modbus Protocol
So after assuming that i have to contact the port 5020 and communicate via the Modbus Protocol, i downloaded a CLI tool that could do this called `mbpoll`, and used this command:
```
mbpoll -p 5020 -c 20 [target_ip]
```
What the command flags do:
- `-p 5020` - use port 5020
- `-c 20` - number of values to read set to 20 
So this command is essentially contacting the targets IP on port 5020 and establishing a TCP connection, it then sends a Modbus request that asks for the information contained in the first 20 data registers, then displays those values back to me in the terminal.

Heres what the resulting values were:
![[017-Brr/images/image3.png]]

## 5. Finding the final flag
After entering these decimal digits into a decimal to ascii converter it revealed the final flag:
![[017-Brr/images/image4.png]]

## Conclusion
This was a short but fun CTF challenge which required these skills to complete:
- Reconnaissance with `Nmap`
- Ability to find default credentials and other information on things like ScadaBR and Modbus
- Utilisation of the `mbpoll` CLI tool to contact the PLC via the Modbus protocol
- Ability to identify cryptic decimal output and follow with the utilisation of unit conversion tools

## What I Learned
I learned that sometimes the solution isn't always complicated like brute forcing a login page with `hydra` or using `gobuster` to find hidden directories, but something simple like just trying the default admin credentials. This CTF also showed me not to shy away from tools i haven't used before as they can make the process a lot simpler if i give myself some time to learn them.