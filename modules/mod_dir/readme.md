# mod_dir

So this is one of the more simpler modules. It primarily has two functionalities. Appending tailing newline characters to URL's that point to a directory (example turning `http://127.0.0.1/volbeat` into `http://127.0.0.1/volbeat/` where `volbeat` is a directory), and having the DirectoryIndex (the default document served when a URL requests a directory).

## Module Declaration

So here is the module declaration:
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

Here are the commands specified:

```
static const command_rec dir_cmds[] =
{
    AP_INIT_TAKE1("FallbackResource", ap_set_string_slot,
                  (void*)APR_OFFSETOF(dir_config_rec, dflt),
                  DIR_CMD_PERMS, "Set a default handler"),
    AP_INIT_RAW_ARGS("DirectoryIndex", add_index, NULL, DIR_CMD_PERMS,
                    "a list of file names"),
    AP_INIT_FLAG("DirectorySlash", configure_slash, NULL, DIR_CMD_PERMS,
                 "On or Off"),
    AP_INIT_FLAG("DirectoryCheckHandler", configure_checkhandler, NULL, DIR_CMD_PERMS,
                 "On or Off"),
    AP_INIT_TAKE1("DirectoryIndexRedirect", configure_redirect,
                   NULL, DIR_CMD_PERMS, "On, Off, or a 3xx status code."),

    {NULL}
};
```

Here are what they mean:
* FallbackResource 			-	This specifies the default handler to use, if a URL doesn't map to anything in filesystem.
* DirectoryIndex 			-	This specifies the directory indexes to be used (default page to be served for a directory).
* DirectorySlash 			-	This specifies if `mod_dir` should check for a trailing forward slash for directories.
* DirectoryCheckHandler 	-	This specifies if `mod_dir` should check for a trailing forward slash, if a different handler is being used.
* DirectoryIndexRedirect 	-	This specifies if the redirect to the `DirectoryIndex` page should use an external redirect.


There is a single hook:
```
static void register_hooks(apr_pool_t *p)
{
    ap_hook_fixups(dir_fixups,NULL,NULL,APR_HOOK_LAST);
}
```

## Fuzzing Functionality

* Requesting DirectoryIndex
* Requesting FallbackResource
* Requesting Directory without trailing slash





