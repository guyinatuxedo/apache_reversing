## Module Declaration

So this section covers Module declaration. So starting off in the source code for all of the modules is a Macro which establishes the module called `AP_DECLARE_MODULE`. Here is an example from `mod_alias.c`:

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

For here:
```
STANDARD_MODULE_STUFF   =  STANDARD20_MODULE_STUFF
dir_config_create       =  create_alias_dir_config
dir_config_merge        =  merge_alias_dir_config
server_config_create    =  create_alias_config
server_config_merge     =  merge_alias_config
cmd_registration        =  alias_cmds
hook_registration       =  register_hooks
```

Now one thing to note, you don't need to have every argument filled out to have a properly working module. In a lot of instances you'll see, a Module will just have a parameter be `Null`, which is the equivalent of leaving it empty. Here is an example from `mod_dir.c` where the `server_config_create` and `server_config_merge` parameters are emptry, meaning it has no per sevrer configuration functionallity:
```
AP_DECLARE_MODULE(dir) = {
    STANDARD20_MODULE_STUFF,
    create_dir_config,          /* create per-directory config structure */
    merge_dir_configs,          /* merge per-directory config structures */
    NULL,                       /* create per-server config structure */
    NULL,                       /* merge per-server config structures */
    dir_cmds,                   /* command apr_table_t */
    register_hooks              /* register hooks */
};

```

#### STANDARD_MODULE_STUFF

So the first parameter is the `STANDARD_MODULE_STUFF` parameter, which is required by all module declaaration. It is used to help setup the backend for the module. There are different version numbers which correspond to different versions of apache, here it is `20`.

#### dir_config_create

So the second parameter is the `dir_config_create` parameter. This is a function that is responsible for creating a config with an individual directory context. So apache modules have configurations that are specified through things like conf files. These configurations specify certain things about how the modules function. Now these module configurations have two types of contexts, which are per server and per directory. That way you can specify different module functionallity for different directories and servers. Typically when a hook function is called, it will grab the current server configuration (using `ap_get_module_config`), and use that as part of it's decision making process on what to do.

Now the function specified by this parameter, has a standard for the arguments it has. This argument is that it will have two arguments. The first is a ptr to the pool (`apr_pool_t`) that it is a part of, and the second is char ptr to the directory it is a part of.

Here is an example of the `dir_config_create` function from `mod_alias.c`:
```
static void *create_alias_dir_config(apr_pool_t *p, char *d)
{
    alias_dir_conf *a =
    (alias_dir_conf *) apr_pcalloc(p, sizeof(alias_dir_conf));
    a->redirects = apr_array_make(p, 2, sizeof(alias_entry));
    return a;
}
```

#### dir_config_merge

So the third parameter is the `dir_config_merge` parameter. This paramater specifies a function, that is used to merge directory configurations. So apache modules have configurations that are specified through things like conf files. These configurations specify certain things about how the modules function. Now these module configurations have two types of contexts, which are per server and per directory. That way you can specify different module functionallity for different directories and servers.

One thing apache can do to help simplify things, is it will sometimes merge configurations. This is done when Apache deems that directory configurations are identical. let's say you have an exact same configuration for a directory for a parent and child directory. Apache will merge those two configurations together, to help lower computational resources consumed.

Here is an example of the `dir_config_merge` function from `mod_alias.c`:
```
static void *merge_alias_dir_config(apr_pool_t *p, void *basev, void *overridesv)
{
    alias_dir_conf *a =
    (alias_dir_conf *) apr_pcalloc(p, sizeof(alias_dir_conf));
    alias_dir_conf *base = (alias_dir_conf *) basev;
    alias_dir_conf *overrides = (alias_dir_conf *) overridesv;

    a->redirects = apr_array_append(p, overrides->redirects, base->redirects);

    a->alias = (overrides->alias_set == 0) ? base->alias : overrides->alias;
    a->handler = (overrides->alias_set == 0) ? base->handler : overrides->handler;
    a->alias_set = overrides->alias_set || base->alias_set;

    a->redirect = (overrides->redirect_set == 0) ? base->redirect : overrides->redirect;
    a->redirect_status = (overrides->redirect_set == 0) ? base->redirect_status : overrides->redirect_status;
    a->redirect_set = overrides->redirect_set || base->redirect_set;

    return a;
}
```

#### server_config_create

So the fourth parameter is pretty much `dir_config_create`, but for the per server context instead of per directory (read the `dir_config_create` section for much more details).

Here is an example of the `server_config_create` function from `mod_alias.c`:
```
static void *create_alias_config(apr_pool_t *p, server_rec *s)
{
    alias_server_conf *a =
    (alias_server_conf *) apr_pcalloc(p, sizeof(alias_server_conf));

    a->aliases = apr_array_make(p, 20, sizeof(alias_entry));
    a->redirects = apr_array_make(p, 20, sizeof(alias_entry));
    return a;
}
```

#### server_config_merge

So the fourth parameter is pretty much `dir_config_merge`, but for the per server context instead of per directory (read the `dir_config_merge` section for much more details).

Here is an example of the `server_config_merge` function from `mod_alias.c`:
```
static void *merge_alias_config(apr_pool_t *p, void *basev, void *overridesv)
{
    alias_server_conf *a =
    (alias_server_conf *) apr_pcalloc(p, sizeof(alias_server_conf));
    alias_server_conf *base = (alias_server_conf *) basev;
    alias_server_conf *overrides = (alias_server_conf *) overridesv;

    a->aliases = apr_array_append(p, overrides->aliases, base->aliases);
    a->redirects = apr_array_append(p, overrides->redirects, base->redirects);
    return a;
}
```

#### cmd_registration

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

#### hook_registration

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
