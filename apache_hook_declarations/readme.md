## Hook Type Declarations

So the function which will actually call a hook, is typically named `ap_run_<hook type>`. For instance, the`ap_run_translate_name` function will call translate name hooks.

Now these functions are declared through the use of macros, so you won't see a function declaration for `ap_run_translate_name` that you can grep from. These functions are made through the use of macros, which we see in `request.c`:

```
APR_HOOK_STRUCT(
    APR_HOOK_LINK(translate_name)
    APR_HOOK_LINK(map_to_storage)
    APR_HOOK_LINK(check_user_id)
    APR_HOOK_LINK(fixups)
    APR_HOOK_LINK(type_checker)
    APR_HOOK_LINK(access_checker)
    APR_HOOK_LINK(access_checker_ex)
    APR_HOOK_LINK(auth_checker)
    APR_HOOK_LINK(insert_filter)
    APR_HOOK_LINK(create_request)
    APR_HOOK_LINK(post_perdir_config)
    APR_HOOK_LINK(dirwalk_stat)
    APR_HOOK_LINK(force_authn)
)

AP_IMPLEMENT_HOOK_RUN_FIRST(int,translate_name,
                            (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,map_to_storage,
                            (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,check_user_id,
                            (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_RUN_ALL(int,fixups,
                          (request_rec *r), (r), OK, DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,type_checker,
                            (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_RUN_ALL(int,access_checker,
                          (request_rec *r), (r), OK, DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,access_checker_ex,
                          (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,auth_checker,
                            (request_rec *r), (r), DECLINED)
AP_IMPLEMENT_HOOK_VOID(insert_filter, (request_rec *r), (r))
AP_IMPLEMENT_HOOK_RUN_ALL(int, create_request,
                          (request_rec *r), (r), OK, DECLINED)
AP_IMPLEMENT_HOOK_RUN_ALL(int, post_perdir_config,
                          (request_rec *r), (r), OK, DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(apr_status_t,dirwalk_stat,
                            (apr_finfo_t *finfo, request_rec *r, apr_int32_t wanted),
                            (finfo, r, wanted), AP_DECLINED)
AP_IMPLEMENT_HOOK_RUN_FIRST(int,force_authn,
                            (request_rec *r), (r), DECLINED)
```

Now we see that the functions are established through the use of macros like `AP_IMPLEMENT_HOOK_RUN_FIRST` and `AP_IMPLEMENT_HOOK_RUN_ALL`. With a simple grep, we see that these macros are established in `include/ap_hooks.h`:

```
#define AP_IMPLEMENT_HOOK_RUN_ALL(ret,name,args_decl,args_use,ok,decline) \
        APR_IMPLEMENT_EXTERNAL_HOOK_RUN_ALL(ap,AP,ret,name,args_decl, \
                                            args_use,ok,decline)


.	.	.


#define AP_IMPLEMENT_HOOK_RUN_FIRST(ret,name,args_decl,args_use,decline) \
        APR_IMPLEMENT_EXTERNAL_HOOK_RUN_FIRST(ap,AP,ret,name,args_decl, \
                                              args_use,decline)
```

So we can see that these functions effectively map to Apache Portable Runtime functions. Now the purpose of the Apache Portal Runtime is to handle enviornment specific functionallity that apache will need to accomplish. This can include things such as loading compiled modules, or calling hooks, since how that functions on Windows is different than Linux, which is different than other enviornments.

Here is a list of all of the "macro mappings" that we see use. This is not all of the available hook types:
```
AP_IMPLEMENT_HOOK_RUN_FIRST 	:	APR_IMPLEMENT_EXTERNAL_HOOK_RUN_FIRST
AP_IMPLEMENT_HOOK_RUN_ALL 		:	APR_IMPLEMENT_OPTIONAL_HOOK_RUN_ALL
AP_IMPLEMENT_HOOK_VOID 			:	APR_IMPLEMENT_EXTERNAL_HOOK_VOID
```