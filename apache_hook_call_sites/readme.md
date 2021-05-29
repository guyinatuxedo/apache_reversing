# Apache Module Hook Call Sites

So this document will cover the different spots in Apache that will call function hooks from modules, since those are the points which module functionallity can actually execute from. In base apache, there are 13 different hook types, with thirteen different functions that can run those hook types:

*[ap_run_translate_name](#ap_run_translate_name)
*[ap_run_map_to_storage](#ap_run_map_to_storage)
*[ap_run_check_user_id](#ap_run_check_user_id)
*[ap_run_fixups](#ap_run_fixups)
*[ap_run_type_checker](#ap_run_type_checker)
*[ap_run_access_checker](#ap_run_access_checker)
*[ap_run_access_checker_ex](#ap_run_access_checker_ex)
*[ap_run_auth_checker](#ap_run_auth_checker)
*[ap_run_insert_filter](#ap_run_insert_filter)
*[ap_run_create_request](#ap_run_create_request)
*[ap_run_post_perdir_config](#ap_run_post_perdir_config)
*[ap_run_dirwalk_stat](#ap_run_dirwalk_stat)
*[ap_run_force_authn](#ap_run_force_authn)


Now the `ap_process_request_internal` function is where the majority of the hook call functions are from. Next we will list exactly where these functions which call module hooks, are called from.

## ap_run_translate_name

Called From:
```
server/request.c 	Function: ap_process_request_internal
```

## ap_run_map_to_storage

Called From:
```
server/request.c 	Function: ap_process_request_internal
```

## ap_run_check_user_id

Called From:
```
server/request.c 				Function: ap_process_request_internal
```

## ap_run_fixups

Called From:
```
server/request.c 				Function: ap_process_request_internal
```

## ap_run_type_checker

Called From:
```
server/request.c 				Function: ap_process_request_internal
```

## ap_run_access_checker

Called From:
```
server/request.c 				Function: ap_process_request_internal
server/request.c 				Function: ap_some_authn_required
```

## ap_run_access_checker_ex

Called From:
```
server/request.c 				Function: ap_process_request_internal
server/request.c 				Function: ap_some_authn_required
```

## ap_run_auth_checker

Called From:
```
server/request.c 				Function: ap_process_request_internal
```

## ap_run_insert_filter

Called From:
```
modules/cache/mod_cache.c 		Function: cache_quick_handler
modules/aaa/mod_auth_form.c 	Function: authenticate_form_authn
server/config.c 				Function: ap_invoke_handler
```

## ap_run_create_request

Called From:
```
server/request.c 				Function: make_sub_request
server/protocol.c 				Function: ap_read_request
modules/http/http_request.c 	Function: internal_internal_redirect
modules/http2/h2_request.c 		Function: my_ap_create_request
```

## ap_run_post_perdir_config

Called From:
```
server/request.c 				Function: ap_process_request_internal
```

## ap_run_dirwalk_stat

Called From:
```
server/request.c 				Function: ap_directory_walk
```

## ap_run_force_authn

Called From:
```
server/request.c 				Function: ap_directory_walk
```
