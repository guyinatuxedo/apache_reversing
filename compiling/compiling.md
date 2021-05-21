# Compiling

This document will cover compiling and setting up apache.

So you will want to download the latest version of apache from `http://httpd.apache.org/download.cgi#apache24`. This is the current version at the time this document is created:
```
$	wget https://downloads.apache.org//httpd/httpd-2.4.46.tar.gz
$	tar -zxvf httpd-2.4.46.tar.gz
$	cd httpd-2.4.46/
```

So we will make the directory which apache will be stored in:

```
$	mkdir /Hackery/apache
```

Now we will compile apache. The `prefix` specifies where we want the generated files to be when we compile apache:

```
$	./configure --prefix=/Hackery/apache
```

Next we will compile apache:

```
$	make
```

And install it:

```
$	make install
```

You might need to edit the conf file:

```
$	vim /Hackery/apache/conf/httpd.conf
```

I had to add a line to specify the full qualified domain name:

```
ServerName "scouts.com"
```

Now the binaries for apache will be in this directory:

```
$	cd /Hackery/apache/bin/
$	ls
ab         checkgid   envvars-std   htdbm     httpd       rotatelogs
apachectl  dbmmanage  fcgistarter   htdigest  httxt2dbm
apxs       envvars    htcacheclean  htpasswd  logresolve
```

You can run it like this:
```
$	sudo gdb ./httpd 
GNU gdb (Ubuntu 9.1-0ubuntu1) 9.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
GEF for linux ready, type `gef' to start, `gef config' to configure
87 commands loaded for GDB 9.1 using Python engine 3.8
[*] 5 commands could not be loaded, run `gef missing` to know why.
GEF for linux ready, type `gef' to start, `gef config' to configure
87 commands loaded for GDB 9.1 using Python engine 3.8
[*] 5 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./httpd...
(No debugging symbols found in ./httpd)
gef➤  set follow-fork-mode child
gef➤  r
Starting program: /Hackery/apache/bin/httpd 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[Attaching after Thread 0x7ffff7c4ac40 (LWP 3051) fork to child process 3055]
[New inferior 2 (process 3055)]
[Detaching after fork from parent process 3051]
[Inferior 1 (process 3051) detached]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[Attaching after Thread 0x7ffff7c4ac40 (LWP 3055) fork to child process 3056]
[New inferior 3 (process 3056)]
[Detaching after fork from parent process 3055]
[Inferior 2 (process 3055) detached]
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff7aae700 (LWP 3113)]
[New Thread 0x7ffff72ad700 (LWP 3114)]
[New Thread 0x7ffff6aac700 (LWP 3115)]
[New Thread 0x7ffff62ab700 (LWP 3116)]
[New Thread 0x7ffff5aaa700 (LWP 3117)]
[New Thread 0x7ffff52a9700 (LWP 3118)]
[New Thread 0x7ffff4aa8700 (LWP 3119)]
[New Thread 0x7fffdffff700 (LWP 3120)]
[New Thread 0x7fffdf7fe700 (LWP 3121)]
[New Thread 0x7fffdeffd700 (LWP 3122)]
[New Thread 0x7fffde7fc700 (LWP 3123)]
[New Thread 0x7fffddffb700 (LWP 3124)]
[New Thread 0x7fffdd7fa700 (LWP 3125)]
[New Thread 0x7fffdcff9700 (LWP 3126)]
[New Thread 0x7fffbbfff700 (LWP 3127)]
[New Thread 0x7fffbb7fe700 (LWP 3128)]
[New Thread 0x7fffbaffd700 (LWP 3129)]
[New Thread 0x7fffba7fc700 (LWP 3130)]
[New Thread 0x7fffb9ffb700 (LWP 3131)]
[New Thread 0x7fffb97fa700 (LWP 3132)]
[New Thread 0x7fffb8ff9700 (LWP 3133)]
[New Thread 0x7fff97fff700 (LWP 3134)]
[New Thread 0x7fff977fe700 (LWP 3135)]
[New Thread 0x7fff96ffd700 (LWP 3136)]
[New Thread 0x7fff967fc700 (LWP 3137)]
[New Thread 0x7fff95ffb700 (LWP 3138)]
[New Thread 0x7fff957fa700 (LWP 3139)]
[Thread 0x7ffff7aae700 (LWP 3113) exited]
```

And when we test it, we see that apache is working:
```
$	wget 127.0.0.1
--2021-05-20 19:35:17--  http://127.0.0.1/
Connecting to 127.0.0.1:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 45 [text/html]
Saving to: ‘index.html.1’

index.html.1        100%[===================>]      45  --.-KB/s    in 0s      

2021-05-20 19:35:17 (9.80 MB/s) - ‘index.html.1’ saved [45/45]
```