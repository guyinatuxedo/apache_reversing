# Mod Alias

So this document briefly describes the functionallity of `mod_alias`. Now the `mod_alias` module is primarily responsible for aliasing things. For instance we can alias the page `/x` to `/y`, so we can reach the pafe `/x` by accessing page `/y`. This would be done by adding the following line to `apache.conf` to the `<IfModule alias_module>` section:

```
Alias /y "/Hackery/apache/htdocs/x"
```

We can see that the `mod_alias` module is loaded via this conf line:
```
LoadModule alias_module modules/mod_alias.so
```

And also, if we set a breakpoint at `load_module` in the process that sets up apache, we see that it is one of the modules that is loaded (the breakpoint is hit once per module, and there are a lot of modules).

Now looking at the `mod_alias.c`, we see the function `register_hooks`. Now how my current understanding of how apache modules work is this. Modules will register hooks. These hooks are functions which will execute at certain times durring the http request processing procedure. There are defined spots in the request processing procedure, such as the start/middle/end (google for the exact spots). You can set a function to execute at a particular time. Here we see two hooks register, which are functions for `translate_alias_redir` and `fixup_redir`:

```
static void register_hooks(apr_pool_t *p)
{
    static const char * const aszSucc[]={ "mod_userdir.c",
                                          "mod_vhost_alias.c",NULL };

    ap_hook_translate_name(translate_alias_redir,NULL,aszSucc,APR_HOOK_MIDDLE);
    ap_hook_fixups(fixup_redir,NULL,NULL,APR_HOOK_MIDDLE);
}
```

So when we look at the `translate_alias_redir` function, we see that it is called from this call stack. Interesting enough, this call stack appears to be from the same as the request processing one we saw earlier. This makes sense since it is supposed to be a hook that is executed as part of that code path:

```
gef➤  bt
#0  0x00007f0f7047fa33 in translate_alias_redir () from /Hackery/apache/modules/mod_alias.so
#1  0x000055c84486c389 in ap_run_translate_name ()
#2  0x000055c84486db39 in ap_process_request_internal ()
#3  0x000055c84489dc4e in ap_process_async_request ()
#4  0x000055c84489931e in ap_process_http_async_connection ()
#5  0x000055c844899554 in ap_process_http_connection ()
#6  0x000055c84488b82d in ap_run_process_connection ()
#7  0x000055c8448a9684 in process_socket ()
#8  0x000055c8448ac328 in worker_thread ()
#9  0x00007f0f707b8609 in start_thread (arg=<optimized out>) at pthread_create.c:477
#10 0x00007f0f706df293 in clone () at ../sysdeps/unix/sysv/linux/x86_64/clone.S:95
```

Now looking at the `translate_alias_redir` function in the `mod_alias` file, we see this:

```
static int translate_alias_redir(request_rec *r)
{
    ap_conf_vector_t *sconf = r->server->module_config;
    alias_server_conf *serverconf = ap_get_module_config(sconf, &alias_module);
    char *ret;
    int status;

    if (r->uri[0] != '/' && r->uri[0] != '\0') {
        return DECLINED;
    }

    if ((ret = try_redirect(r, &status)) != NULL
            || (ret = try_alias_list(r, serverconf->redirects, 1, &status))
                    != NULL) {
        if (ret == PREGSUB_ERROR)
            return HTTP_INTERNAL_SERVER_ERROR;
        if (ap_is_HTTP_REDIRECT(status)) {
            if (ret[0] == '/') {
                char *orig_target = ret;

                ret = ap_construct_url(r->pool, ret, r);
                ap_log_rerror(APLOG_MARK, APLOG_DEBUG, 0, r, APLOGNO(00673)
                              "incomplete redirection target of '%s' for "
                              "URI '%s' modified to '%s'",
                              orig_target, r->uri, ret);
            }
            if (!ap_is_url(ret)) {
                status = HTTP_INTERNAL_SERVER_ERROR;
                ap_log_rerror(APLOG_MARK, APLOG_ERR, 0, r, APLOGNO(00674)
                              "cannot redirect '%s' to '%s'; "
                              "target is not a valid absoluteURI or abs_path",
                              r->uri, ret);
            }
            else {
                /* append requested query only, if the config didn't
                 * supply its own.
                 */
                if (r->args && !ap_strchr(ret, '?')) {
                    ret = apr_pstrcat(r->pool, ret, "?", r->args, NULL);
                }
                apr_table_setn(r->headers_out, "Location", ret);
            }
        }
        return status;
    }

    if ((ret = try_alias(r)) != NULL
            || (ret = try_alias_list(r, serverconf->aliases, 0, &status))
                    != NULL) {
        r->filename = ret;
        return OK;
    }

    return DECLINED;
}
```

Now breaking this down, this function appears to do the following. First there is a check to see if the first byte of the iri. Then proceeding that, calls are made to `try_redirect`, `try_alias_list`, and `try_alias`. Effectively what these functions are checking for, is with the current server config, if there are any aliases (or things like that) that apply to the current request. And if there are, then handle the redirection.

In the case I setup, what ends up happening is that is that all of the function calls, up to the second `try_alias_list` call return null. The `try_alias_list` function call returns the char pointer to the file mapped to the alias, which then the request has it's filename set to that. Looking at the `try_alias_list` function, we see that it effectively just iterates through the list of aliases it's been given, looking for an alias that matches the one in the request.

## Commands

So looking at the command list, we see there are `8` of them:
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

## Fuzzing Functionality

* Send request to aliased (and different resource with aliasMatch)
* Send request to redirected (and different resource with redirectMatch)
* Send request to scriptAliased (and different resource with scriptAliased)