# Apache Module Config Creation

So this document will describe the process Apache uses to create configurations for modules. Now the functions used for creating the configurations (for servers and directories both, since those are the two types of configuration contexts for modules), are supplied by the modules themselves. Apache will effectively just indirectly call them. This will take place in the `load_module` function, after apache has already loaded in the module to memory. There is a function called at the end of `load_module` (which we can see in `modules/core/mod_so.c`) which is responsible for creating the config:

```
    /*
     * Finally we need to run the configuration process for the module
     */
    ap_single_module_configure(cmd->pool, cmd->server, modp);

```

Now the `ap_single_module_configure` is the function (in `server/config.h`) which will inderectly call the module specified functions, for creating the server/dir configs (if specifed by the module, which we see the if then check in the below source for both server and dir configs). Looking at the source code, we see this:

```
AP_DECLARE(void) ap_single_module_configure(apr_pool_t *p, server_rec *s,
                                            module *m)
{
    if (m->create_server_config)
        ap_set_module_config(s->module_config, m,
                             (*m->create_server_config)(p, s));

    if (m->create_dir_config)
        ap_set_module_config(s->lookup_defaults, m,
                             (*m->create_dir_config)(p, NULL));
}
```

Now looking at the `ap_set_module_config` declarations in `include/http_config.h`, it becomes clear that they are just macros for indirectly calling function:

```
/**
 * Generic accessors for other modules to set their own module-specific
 * data
 * @param cv The vector in which the modules configuration is stored.
 *        usually r->per_dir_config or s->module_config
 * @param m The module to set the data for.
 * @param val The module-specific data to set
 */
AP_DECLARE(void) ap_set_module_config(ap_conf_vector_t *cv, const module *m,
                                      void *val);

	.	.	.


#define ap_get_module_config(v,m)       \
    (((void **)(v))[(m)->module_index])
#define ap_set_module_config(v,m,val)   \
    ((((void **)(v))[(m)->module_index]) = (val))
```

So tl;dr, module configurations are generated as part of the module loading processes. The module specified configuration generation functions are indirectly called by `ap_single_module_configure`, which is called in `load_module`.