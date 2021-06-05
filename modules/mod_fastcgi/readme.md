# mod_fastcgi



## Setup

So to set this up, we first need to compile it. To do it, first clone the git repo:
```
$	git clone https://github.com/FastCGI-Archives/mod_fastcgi.git
$	cd mod_fastcgi
```

Next we will need to set the Makefile. There are several different versions. We will be using the one for Apache2:
```
$	cp Makefile.AP2 Makefile
```

Next we will compile the binary. Since we installed apache in a different directory than the typical one, we will need to set the `top_dir` to the directory we installed apache to:
```
$	make top_dir=/Hackery/apache
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c mod_fastcgi.c && touch mod_fastcgi.slo
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c fcgi_pm.c && touch fcgi_pm.slo
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c fcgi_util.c && touch fcgi_util.slo
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c fcgi_protocol.c && touch fcgi_protocol.slo
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c fcgi_buf.c && touch fcgi_buf.slo
/usr/share/apr-1.0/build/libtool --silent --mode=compile x86_64-linux-gnu-gcc  -pthread      -DLINUX -D_REENTRANT -D_GNU_SOURCE     -I/Hackery/apache/include -I. -I/usr/include/apr-1.0 -I/usr/include -prefer-pic -c fcgi_config.c && touch fcgi_config.slo
/usr/share/apr-1.0/build/libtool --silent --mode=link x86_64-linux-gnu-gcc  -pthread           -o mod_fastcgi.la -rpath /Hackery/apache/modules -module -avoid-version mod_fastcgi.lo fcgi_pm.lo fcgi_util.lo fcgi_protocol.lo fcgi_buf.lo fcgi_config.lo
```

And then install it:
```
$	make install top_dir=/Hackery/apache
make[1]: Entering directory '/Hackery/mod_fastcgi'
/usr/share/apr-1.0/build/libtool --silent --mode=install install mod_fastcgi.la /Hackery/apache/modules/
make[1]: Leaving directory '/Hackery/mod_fastcgi'
```

We see the module has been created:
```
$	file /Hackery/apache/modules/mod_fastcgi.so 
/Hackery/apache/modules/mod_fastcgi.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=d957756eb70a78b3651d07fbf8142db7dfde01b8, not stripped
```

Next we just need to edit the apache conf file:

```
$	vim /Hackery/apache/conf/httpd.conf
```

And add this line, to specify to load the `mod_fastcgi` module:

```
LoadModule fastcgi_module modules/mod_fastcgi.so
```

Then when we startup apache, we see that `mod_fastcgi.so` is loaded into memory.

```
gef➤  vmmap


.	.	.

0x00007f7d30a4b000 0x00007f7d30a50000 0x0000000000000000 r-- /Hackery/apache/modules/mod_fastcgi.so
0x00007f7d30a50000 0x00007f7d30a61000 0x0000000000005000 r-x /Hackery/apache/modules/mod_fastcgi.so
0x00007f7d30a61000 0x00007f7d30a65000 0x0000000000016000 r-- /Hackery/apache/modules/mod_fastcgi.so
0x00007f7d30a65000 0x00007f7d30a66000 0x0000000000019000 r-- /Hackery/apache/modules/mod_fastcgi.so
0x00007f7d30a66000 0x00007f7d30a67000 0x000000000001a000 rw- /Hackery/apache/modules/mod_fastcgi.so
```

## Reversing

So starting off, this is the module declaration:

```
module AP_MODULE_DECLARE_DATA fastcgi_module =
{
    STANDARD20_MODULE_STUFF,
    fcgi_config_create_dir_config,  /* per-directory config creator */
    NULL,                           /* dir config merger */
    NULL,                           /* server config creator */
    NULL,                           /* server config merger */
    fastcgi_cmds,                   /* command table */
    register_hooks,                 /* set up other request processing hooks */
};
```

The commands:
```
static const command_rec fastcgi_cmds[] = 
{
    AP_INIT_RAW_ARGS("AppClass",      fcgi_config_new_static_server, NULL, RSRC_CONF, NULL),
    AP_INIT_RAW_ARGS("FastCgiServer", fcgi_config_new_static_server, NULL, RSRC_CONF, NULL),

    AP_INIT_RAW_ARGS("ExternalAppClass",      fcgi_config_new_external_server, NULL, RSRC_CONF, NULL),
    AP_INIT_RAW_ARGS("FastCgiExternalServer", fcgi_config_new_external_server, NULL, RSRC_CONF, NULL),

    AP_INIT_TAKE1("FastCgiIpcDir", fcgi_config_set_socket_dir, NULL, RSRC_CONF, NULL),

    AP_INIT_TAKE1("FastCgiSuexec",  fcgi_config_set_wrapper, NULL, RSRC_CONF, NULL),
    AP_INIT_TAKE1("FastCgiWrapper", fcgi_config_set_wrapper, NULL, RSRC_CONF, NULL),

    AP_INIT_RAW_ARGS("FCGIConfig",    fcgi_config_set_config, NULL, RSRC_CONF, NULL),
    AP_INIT_RAW_ARGS("FastCgiConfig", fcgi_config_set_config, NULL, RSRC_CONF, NULL),

    AP_INIT_TAKE12("FastCgiAuthenticator", fcgi_config_new_auth_server,
        (void *)FCGI_AUTH_TYPE_AUTHENTICATOR, ACCESS_CONF,
        "a fastcgi-script path (absolute or relative to ServerRoot) followed by an optional -compat"),
    AP_INIT_FLAG("FastCgiAuthenticatorAuthoritative", fcgi_config_set_authoritative_slot,
        (void *)XtOffsetOf(fcgi_dir_config, authenticator_options), ACCESS_CONF,
        "Set to 'off' to allow authentication to be passed along to lower modules upon failure"),

    AP_INIT_TAKE12("FastCgiAuthorizer", fcgi_config_new_auth_server,
        (void *)FCGI_AUTH_TYPE_AUTHORIZER, ACCESS_CONF,
        "a fastcgi-script path (absolute or relative to ServerRoot) followed by an optional -compat"),
    AP_INIT_FLAG("FastCgiAuthorizerAuthoritative", fcgi_config_set_authoritative_slot,
        (void *)XtOffsetOf(fcgi_dir_config, authorizer_options), ACCESS_CONF,
        "Set to 'off' to allow authorization to be passed along to lower modules upon failure"),

    AP_INIT_TAKE12("FastCgiAccessChecker", fcgi_config_new_auth_server,
        (void *)FCGI_AUTH_TYPE_ACCESS_CHECKER, ACCESS_CONF,
        "a fastcgi-script path (absolute or relative to ServerRoot) followed by an optional -compat"),
    AP_INIT_FLAG("FastCgiAccessCheckerAuthoritative", fcgi_config_set_authoritative_slot,
        (void *)XtOffsetOf(fcgi_dir_config, access_checker_options), ACCESS_CONF,
        "Set to 'off' to allow access control to be passed along to lower modules upon failure"),
    { NULL }
};
```

The hooks:
```
static void register_hooks(apr_pool_t * p)
{
    /* ap_hook_pre_config(x_pre_config, NULL, NULL, APR_HOOK_MIDDLE); */
    ap_hook_post_config(init_module, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_child_init(fcgi_child_init, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_handler(content_handler, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_check_user_id(check_user_authentication, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_access_checker(check_access, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_auth_checker(check_user_authorization, NULL, NULL, APR_HOOK_MIDDLE);
    ap_hook_fixups(fixups, NULL, NULL, APR_HOOK_MIDDLE); 
}
```

So we see that there are `8` hooks.

## init_module hook

This ia a `post_config` hook which is ran durring startup from this call stack:
```
gef➤  bt
#0  0x00007ffff7c08afc in init_module () from /Hackery/apache/modules/mod_fastcgi.so
#1  0x00005555555c2f29 in ap_run_post_config ()
#2  0x000055555558d86e in main ()
```

Looking at this function, it appears that it is starting a process manager. It will fork a process using `apr_proc_fork`, and start the process manager with `fcgi_pm_main`.

## fcgi_child_init hook

This hook appears to create handlers to `MBOX/TERM/WAKE` event handlers using `CreateEvent`. It also creates a mutex for `MBOX`, and spawns a process manager thread fpr `fcgi_pm_main`. It is a `child_init` hook, that is ran whenever a child process is initialized.

## content_handler hook

This hook is what processes fastcgi-script requests. It handles all requests that are not based around authentication. it is a `handler` type hook.

Now the process of how it handles a request, is primarily composed of these parts. First it creates a new FastCGI request from the request it's been given using `create_fcgi_request`. Then it proceeds to service the request with the `do_work` function. Then if the request calls for redirects, it will handle them with the `post_process_for_redirects` function.

## check_user_authentication hook

So this function is pretty similar to the `content_handler` hook. It handles fastcgi-script requests, that are based around certain types of auth. It is a `check_user_id` type of hook.

## check_access hook

So this function is pretty similar to the `content_handler` hook. It handles fastcgi-script requests, that are based around certain types of auth. It is a `access_checker` type of hook.

## check_user_authorization hook

So this function is pretty similar to the `content_handler` hook. It handles fastcgi-script requests, that are based around certain types of auth. It is a `auth_checker` type of hook.

## fixups hook

So this is a `fixups` style hook. It effectively just checks, does the request have a filename, and is there a server associated with the group and user id of the request. If there is, set the request handler to the request is set to `content_handler` from before.












