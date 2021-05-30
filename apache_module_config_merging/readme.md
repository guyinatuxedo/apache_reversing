# Apache Module Config Merging

So this document will cover Apache Config Merging, for both server and directory configs.

## Module Server Config Merging

So this happens as part of the server startup. The `merge_server_configs` function is what is responsible for making the indirect call to merge the configs (as specified by the module declaration). it is called from this callstack for calling the `merge_alias_config` function to merge `mod_alias` virtualhost configs:

```
gef➤  bt
#0  0x00007ffff7b87594 in merge_alias_config () from /Hackery/apache/modules/mod_alias.so
#1  0x00005555555c3cbc in merge_server_configs ()
#2  0x00005555555c83ad in ap_fixup_virtual_hosts ()
#3  0x000055555558d5d5 in main ()
```

To hit that codepath, I just specified the same `Alias` option across two seperate VirtualHosts. 

## Module Directory Config Merging

So this is a bit interesting. It is not called on server startup. Instead it is called by the worker processes, while serviceing requests. The `ap_merge_per_dir_configs` is what is responsible for making the indirect call to merge the configs (as specifed by the module declaration). The `ap_merge_per_dir_configs` seems to be called 10 times in `server/request.c`, along with a few times in some modules. Primarily the functions it appears to be called in in `server/request.c` deal with directory walking (most commonly `ap_directory_walk`). It also appears to be called twice within `ap_directory_walk` from here:

```
gef➤  bt
#0  0x00007f8c0fd0d70f in merge_dir_configs () from /Hackery/apache/modules/mod_dir.so
#1  0x000055834ba5ba48 in ap_merge_per_dir_configs ()
#2  0x000055834ba4fc67 in ap_directory_walk ()
#3  0x000055834ba49534 in core_map_to_storage ()
#4  0x000055834ba4c4f0 in ap_run_map_to_storage ()
#5  0x000055834ba4db7f in ap_process_request_internal ()
#6  0x000055834ba7dc4e in ap_process_async_request ()
#7  0x000055834ba7931e in ap_process_http_async_connection ()
#8  0x000055834ba79554 in ap_process_http_connection ()
#9  0x000055834ba6b82d in ap_run_process_connection ()
#10 0x000055834ba89684 in process_socket ()
#11 0x000055834ba8c328 in worker_thread ()
#12 0x00007f8c10040609 in start_thread (arg=<optimized out>) at pthread_create.c:477
#13 0x00007f8c0ff67293 in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

and here:

```
gef➤  bt
#0  0x00007f8c0fd0d70f in merge_dir_configs () from /Hackery/apache/modules/mod_dir.so
#1  0x000055834ba5ba48 in ap_merge_per_dir_configs ()
#2  0x000055834ba50b4a in ap_directory_walk ()
#3  0x000055834ba49534 in core_map_to_storage ()
#4  0x000055834ba4c4f0 in ap_run_map_to_storage ()
#5  0x000055834ba4db7f in ap_process_request_internal ()
#6  0x000055834ba7dc4e in ap_process_async_request ()
#7  0x000055834ba7931e in ap_process_http_async_connection ()
#8  0x000055834ba79554 in ap_process_http_connection ()
#9  0x000055834ba6b82d in ap_run_process_connection ()
#10 0x000055834ba89684 in process_socket ()
#11 0x000055834ba8c328 in worker_thread ()
#12 0x00007f8c10040609 in start_thread (arg=<optimized out>) at pthread_create.c:477
#13 0x00007f8c0ff67293 in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```
