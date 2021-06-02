# mod_uwsgi_proxy

So uswgi is a network protocol. This module aims to act as a proxy for that protocol. This module has a dependency of `mod_proxy`


Looking at the module declaration, we see this:

```
module AP_MODULE_DECLARE_DATA proxy_uwsgi_module = {
    STANDARD20_MODULE_STUFF,
    NULL,                       /* create per-directory config structure */
    NULL,                       /* merge per-directory config structures */
    NULL,                       /* create per-server config structure */
    NULL,                       /* merge per-server config structures */
    NULL,                       /* command table */
    register_hooks              /* register hooks */
};
```

Looking at this module, we see that it itself has no module configuration capabilities itself. That is because all of the configurations is managed entirely by `mod_proxy`.

Looking at the `register_hooks` function, we see this:
```
static void register_hooks(apr_pool_t * p)
{
    proxy_hook_scheme_handler(uwsgi_handler, NULL, NULL, APR_HOOK_FIRST);
    proxy_hook_canon_handler(uwsgi_canon, NULL, NULL, APR_HOOK_FIRST);
}
```

So there are only two functions present, `proxy_hook_scheme_handler` and `proxy_hook_canon_handler`. 