# Packet Reading

## Listening

So this section covers how apache actually listens, and scans in a packet into memeory.

So there is a function called `apr_socket_recv`

```
#0  0x00007fa3da3774f0 in apr_socket_recv () from /lib/x86_64-linux-gnu/libapr-1.so.0
#1  0x00007fa3da39ac13 in ?? () from /lib/x86_64-linux-gnu/libaprutil-1.so.0
#2  0x00007fa3da398d44 in apr_brigade_split_line () from /lib/x86_64-linux-gnu/libaprutil-1.so.0
#3  0x0000557d0fb6f9a9 in ap_core_input_filter ()
#4  0x0000557d0fb4ec31 in ap_get_brigade ()
#5  0x00007fa3da064b37 in reqtimeout_filter () from /Hackery/apache/modules/mod_reqtimeout.so
#6  0x0000557d0fb4ec31 in ap_get_brigade ()
#7  0x0000557d0fb51d04 in ap_rgetline_core ()
#8  0x0000557d0fb52746 in read_request_line ()
#9  0x0000557d0fb56e55 in ap_read_request ()
#10 0x0000557d0fb95293 in ap_process_http_async_connection ()
#11 0x0000557d0fb95554 in ap_process_http_connection ()
#12 0x0000557d0fb8782d in ap_run_process_connection ()
#13 0x0000557d0fba5684 in process_socket ()
#14 0x0000557d0fba8328 in worker_thread ()
#15 0x00007fa3da33b609 in start_thread (arg=<optimized out>) at pthread_create.c:477
#16 0x00007fa3da262293 in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```