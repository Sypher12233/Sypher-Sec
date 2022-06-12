# Pickle Rick Room From Try Hack ME #

> Given IP = 10.10.231.165
> Found Username:R1ckRul3s

username found from page source!

---

## Running Nmap scan ##

```ZSH
sypher@kali:~ $ sudo nmap -sV -sS -T4 10.10.231.165
[sudo] password for sypher: 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-26 14:35 EDT
Nmap scan report for 10.10.231.165
Host is up (0.58s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.27 seconds
```

---

## Also running gobuster scan ##

* In gobuster scan we didn't get much; only two pages 1) assets 2) server-status

---

## Moving on to *Nikto* ##

running scan `nikto -host <ip>`
