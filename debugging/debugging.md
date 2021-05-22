# Debugging

So this document covers basic debugging. So the apache server runs with different processes. So we see that there are no `httpd` processes prior to running apache:

```
$	ps -aux | grep httpd
guyinat+    2475  0.0  0.0   9040   724 pts/0    S+   19:29   0:00 grep --color=auto httpd
```

Then after we run apache, we see that several httpd process startup:

```
$	sudo ./httpd 
$	ps -aux | grep httpd
root        2488  0.0  0.0   6224  3744 ?        Ss   19:30   0:00 ./httpd
daemon      2489  0.0  0.0 1997560 4076 ?        Sl   19:30   0:00 ./httpd
daemon      2490  0.0  0.0 1997560 4040 ?        Sl   19:30   0:00 ./httpd
daemon      2491  0.0  0.0 1997560 4012 ?        Sl   19:30   0:00 ./httpd
guyinat+    2574  0.0  0.0   9040   736 pts/0    S+   19:30   0:00 grep --color=auto httpd
```

So there are four seperate `httpd` processes, one running as root, and the other three are running as daemon. From right now, it appears that there is one process that manages the other threads, and the other processes actually handle requests. 

```
gef➤  info stack
#0  __libc_read (nbytes=0x1, buf=0x7ffc5f50215f, fd=0x5) at ../sysdeps/unix/sysv/linux/read.c:26
#1  __libc_read (fd=0x5, buf=0x7ffc5f50215f, nbytes=0x1) at ../sysdeps/unix/sysv/linux/read.c:24
#2  0x000055a7856fb6a3 in ap_mpm_podx_check ()
#3  0x000055a785719825 in child_main ()
#4  0x000055a785719ad1 in make_child ()
#5  0x000055a785719bd1 in startup_children ()
#6  0x000055a78571ac27 in event_run ()
#7  0x000055a7856bc23a in ap_run_mpm ()
#8  0x000055a7856b1d24 in main ()
gef➤  info threads
  Id   Target Id                                Frame 
* 1    Thread 0x7f339bc35c40 (LWP 2489) "httpd" __libc_read (nbytes=0x1, buf=0x7ffc5f50215f, fd=0x5) at ../sysdeps/unix/sysv/linux/read.c:26
  2    Thread 0x7f339b298700 (LWP 2493) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  3    Thread 0x7f339aa97700 (LWP 2494) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  4    Thread 0x7f339a296700 (LWP 2495) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  5    Thread 0x7f3399a95700 (LWP 2496) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  6    Thread 0x7f3399294700 (LWP 2497) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  7    Thread 0x7f3398a93700 (LWP 2498) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  8    Thread 0x7f3383fff700 (LWP 2499) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  9    Thread 0x7f33837fe700 (LWP 2500) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  10   Thread 0x7f3382ffd700 (LWP 2501) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  11   Thread 0x7f33827fc700 (LWP 2502) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  12   Thread 0x7f3381ffb700 (LWP 2503) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  13   Thread 0x7f33817fa700 (LWP 2504) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  14   Thread 0x7f3380ff9700 (LWP 2505) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  15   Thread 0x7f3363fff700 (LWP 2506) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  16   Thread 0x7f33637fe700 (LWP 2507) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  17   Thread 0x7f3362ffd700 (LWP 2508) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  18   Thread 0x7f33627fc700 (LWP 2510) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  19   Thread 0x7f3361ffb700 (LWP 2512) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  20   Thread 0x7f33617fa700 (LWP 2513) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  21   Thread 0x7f3360ff9700 (LWP 2515) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  22   Thread 0x7f333ffff700 (LWP 2518) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  23   Thread 0x7f333f7fe700 (LWP 2519) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  24   Thread 0x7f333effd700 (LWP 2524) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  25   Thread 0x7f333e7fc700 (LWP 2527) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  26   Thread 0x7f333dffb700 (LWP 2529) "httpd" futex_wait_cancelable (private=<optimized out>, expected=0x0, futex_word=0x7f339bae2160) at ../sysdeps/nptl/futex-internal.h:183
  27   Thread 0x7f333d7fa700 (LWP 2531) "httpd" 0x00007f339bdd35ce in epoll_wait (epfd=0x8, events=0x7f339bae25f0, maxevents=0x34, timeout=0xffffffff) at ../sysdeps/unix/sysv/linux/epoll_wait.c:30

```