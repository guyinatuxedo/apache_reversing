## Module Loading

So this document briefly covers how Apache will load modules. A module is effectively a compiled piece of code that Apache can load, which adds additional functionallity. Apache is based on a model where a lot of the functionallity it uses is offloaded to modules, which is loaded in. 

So in the file `modules/core/mod_so.c`, there is the function `load_module` which will load the module into memory. It is called from this call stack:

```
gef➤  bt
#0  0x00005555555e069b in load_module ()
#1  0x00005555555c4ff0 in invoke_cmd ()
#2  0x00005555555c6fd8 in execute_now ()
#3  0x00005555555c5e0a in ap_build_config_sub ()
#4  0x00005555555c668b in ap_build_config ()
#5  0x00005555555c75ec in ap_process_resource_config ()
#6  0x00005555555c89d9 in ap_read_config ()
#7  0x000055555558d475 in main ()
gef➤  
```

Now this function will call `dso_load`, which will then call `apr_dso_load`. The `apr_dso_load` isn't actually in the source code for apache, it is in a different `so` file. This is in the apache portable runtime (apr) module, which is responsible for handling some enviornment specific things. At the end of the day, this function is responsible for actually loading more runnable code into memory, which the process for doing that is different for different enviornments (like windows/linux/etc). In the linux enviornment I ran it in, it effectively will just call `dlopen` to open the shared object, which we can see via disassembling the function:

```
gef➤  disas apr_dso_load
Dump of assembler code for function apr_dso_load:
   0x00007ffff7ef3410 <+0>:	endbr64 
   0x00007ffff7ef3414 <+4>:	push   r13
   0x00007ffff7ef3416 <+6>:	push   r12
   0x00007ffff7ef3418 <+8>:	mov    r12,rdi
   0x00007ffff7ef341b <+11>:	mov    rdi,rsi
   0x00007ffff7ef341e <+14>:	mov    esi,0x102
   0x00007ffff7ef3423 <+19>:	push   rbp
   0x00007ffff7ef3424 <+20>:	mov    rbp,rdx
   0x00007ffff7ef3427 <+23>:	push   rbx
   0x00007ffff7ef3428 <+24>:	sub    rsp,0x8
   0x00007ffff7ef342c <+28>:	call   0x7ffff7ee8300 <dlopen@plt>
   0x00007ffff7ef3431 <+33>:	mov    esi,0x18
   0x00007ffff7ef3436 <+38>:	mov    rdi,rbp
   0x00007ffff7ef3439 <+41>:	mov    r13,rax
   0x00007ffff7ef343c <+44>:	call   0x7ffff7ef9940 <apr_palloc>
   0x00007ffff7ef3441 <+49>:	pxor   xmm0,xmm0
   0x00007ffff7ef3445 <+53>:	mov    QWORD PTR [rax+0x10],0x0
   0x00007ffff7ef344d <+61>:	mov    rbx,rax
   0x00007ffff7ef3450 <+64>:	movups XMMWORD PTR [rax],xmm0
   0x00007ffff7ef3453 <+67>:	mov    QWORD PTR [r12],rax
   0x00007ffff7ef3457 <+71>:	test   r13,r13
   0x00007ffff7ef345a <+74>:	je     0x7ffff7ef3498 <apr_dso_load+136>
   0x00007ffff7ef345c <+76>:	mov    QWORD PTR [rax+0x8],r13
   0x00007ffff7ef3460 <+80>:	mov    rsi,QWORD PTR [r12]
   0x00007ffff7ef3464 <+84>:	mov    rdi,rbp
   0x00007ffff7ef3467 <+87>:	lea    rdx,[rip+0xffffffffffffff02]        # 0x7ffff7ef3370
   0x00007ffff7ef346e <+94>:	lea    rcx,[rip+0x7f9b]        # 0x7ffff7efb410 <apr_pool_cleanup_null>
   0x00007ffff7ef3475 <+101>:	mov    QWORD PTR [rsi],rbp
   0x00007ffff7ef3478 <+104>:	mov    QWORD PTR [rsi+0x10],0x0
   0x00007ffff7ef3480 <+112>:	call   0x7ffff7efb0d0 <apr_pool_cleanup_register>
   0x00007ffff7ef3485 <+117>:	add    rsp,0x8
   0x00007ffff7ef3489 <+121>:	xor    eax,eax
   0x00007ffff7ef348b <+123>:	pop    rbx
   0x00007ffff7ef348c <+124>:	pop    rbp
   0x00007ffff7ef348d <+125>:	pop    r12
   0x00007ffff7ef348f <+127>:	pop    r13
   0x00007ffff7ef3491 <+129>:	ret    
   0x00007ffff7ef3492 <+130>:	nop    WORD PTR [rax+rax*1+0x0]
   0x00007ffff7ef3498 <+136>:	call   0x7ffff7ee8850 <dlerror@plt>
   0x00007ffff7ef349d <+141>:	mov    QWORD PTR [rbx+0x10],rax
   0x00007ffff7ef34a1 <+145>:	add    rsp,0x8
   0x00007ffff7ef34a5 <+149>:	mov    eax,0x4e33
   0x00007ffff7ef34aa <+154>:	pop    rbx
   0x00007ffff7ef34ab <+155>:	pop    rbp
   0x00007ffff7ef34ac <+156>:	pop    r12
   0x00007ffff7ef34ae <+158>:	pop    r13
   0x00007ffff7ef34b0 <+160>:	ret    
End of assembler dump.
```