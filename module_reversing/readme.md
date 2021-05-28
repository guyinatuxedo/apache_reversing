# Module Reversing

So this document covers various Apache module functionallity.

## Abstract Module Functionallity

So this section covers some of the overlapping module functionallity. So starting off in the source code for all of the modules is a Macro which establishes the module called `AP_DECLARE_MODULE`. Here is an example from `mod_alias.c`:

```
AP_DECLARE_MODULE(alias) =
{
    STANDARD20_MODULE_STUFF,
    create_alias_dir_config,       /* dir config creater */
    merge_alias_dir_config,        /* dir merger --- default is to override */
    create_alias_config,           /* server config */
    merge_alias_config,            /* merge server configs */
    alias_cmds,                    /* command apr_table_t */
    register_hooks                 /* register hooks */
};
```

#### STANDARD_MODULE_STUFF

So the first parameter is the `STANDARD_MODULE_STUFF` parameter, which is required by all module declaaration. It is used to help setup the backend for the module. There are different version numbers which correspond to different versions of apache, here it is `20`.

#### dir config create

#### dir config merge

#### alias config create

#### alias config merge

#### cmd registration

So the sixth parameter is `alias_cmds`. This is a function, which can be see in the source code:

```
static const command_rec alias_cmds[] =
{
    AP_INIT_TAKE12("Alias", add_alias, NULL, RSRC_CONF | ACCESS_CONF,
                  "a fakename and a realname, or a realname in a Location"),
    AP_INIT_TAKE12("ScriptAlias", add_alias, "cgi-script", RSRC_CONF | ACCESS_CONF,
                  "a fakename and a realname, or a realname in a Location"),
    AP_INIT_TAKE123("Redirect", add_redirect, (void *) HTTP_MOVED_TEMPORARILY,
                   OR_FILEINFO,
                   "an optional status, then document to be redirected and "
                   "destination URL"),
    AP_INIT_TAKE2("AliasMatch", add_alias_regex, NULL, RSRC_CONF,
                  "a regular expression and a filename"),
    AP_INIT_TAKE2("ScriptAliasMatch", add_alias_regex, "cgi-script", RSRC_CONF,
                  "a regular expression and a filename"),
    AP_INIT_TAKE23("RedirectMatch", add_redirect_regex,
                   (void *) HTTP_MOVED_TEMPORARILY, OR_FILEINFO,
                   "an optional status, then a regular expression and "
                   "destination URL"),
    AP_INIT_TAKE2("RedirectTemp", add_redirect2,
                  (void *) HTTP_MOVED_TEMPORARILY, OR_FILEINFO,
                  "a document to be redirected, then the destination URL"),
    AP_INIT_TAKE2("RedirectPermanent", add_redirect2,
                  (void *) HTTP_MOVED_PERMANENTLY, OR_FILEINFO,
                  "a document to be redirected, then the destination URL"),
    {NULL}
};
```

Basically, these functions are what add configuration options for this module. For instance, here we see options such as `"Alias"` and `"ScriptAlias"`. We see these options in the `httpd.conf` file:

```
ScriptAlias /cgi-bin/ "/Hackery/apache/cgi-bin/"
Alias /y "/Hackery/apache/htdocs/x"
```

So, the `alias_cmds` macro effectivly just calls a bunch of functions that are `AP_INIT_TAKE` followed by some numbers. A `AP_INIT_TAKE` function call will establish a new option in the conf file. The numbers after the `AP_INIT_TAKE` specify the number of arguments for the option. The `AP_INIT_TAKE12` function will take either `1` or `2` arguments. The `AP_INIT_TAKE2` function specifies the option will take `2` arguments. The `AP_INIT_TAKE23` function specifies the option will take `2` or `3` arguments.

The `AP_INIT_TAKE` functiont akes five arguments:
```
directive   -  This specifies the string to identify this option in the conf file.
func        -  This specifies the function responsible for configuring the option.
data_ptr    -  Often null, this is a ptr that is passed as part of the `cmd` struc to the function in argument 2
where       -  This specifies the scope of where this option is allowed in the conf file
help        -  A help message for this option.
```

For this line here:
```
    AP_INIT_TAKE12("Alias", add_alias, NULL, RSRC_CONF | ACCESS_CONF,
                  "a fakename and a realname, or a realname in a Location")
```

These are the arguments:
```
directive   -  "Alias"
func        -  add_alias
data_ptr    -  NULL
where       -  RSRC_CONF | ACCESS_CONF
help        -  "a fakename and a realname, or a realname in a Location"
```

#### hook registration

So the hook registration function is the final argument. It specifies a function which will register the hooks, here is an example from `mod_alias`:

```
static void register_hooks(apr_pool_t *p)
{
    static const char * const aszSucc[]={ "mod_userdir.c",
                                          "mod_vhost_alias.c",NULL };

    ap_hook_translate_name(translate_alias_redir,NULL,aszSucc,APR_HOOK_MIDDLE);
    ap_hook_fixups(fixup_redir,NULL,NULL,APR_HOOK_MIDDLE);
}
```

So there are different types of hooks that can be registered. These can be specified with different types of `ap_hook` functions. Here we see `ap_hook_translate_name` (which specifies the hook should execute whenever a url needs to be converted to a filename) and `ap_hook_fixups` (which specifies the hook should execute right before content is generated). There are a ton of others that you can see here `https://httpd.apache.org/docs/2.4/developer/modules.html` (or just google for them).

There are four arguments to a `ap_hook` function. The first is the function which is to be executed when the hook is called. The other three arguments specify when the hook will execute. The fourth argument specifies when in the request handling procedure, this hook will be executed. `APR_HOOK_FIRST` will run before `APR_HOOK_MIDDLE` which will run before `APR_HOOK_LAST`. The second argument is a list of functions which will be ran before this one (predecessors), and the third argument is a list of functions which will be ran after this one (successors) to provide additional control over the order which module hooks are executed.

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