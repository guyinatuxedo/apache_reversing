fcgi_request struct is????
look at create_fcgi_request

```
/*
 * fcgi_request holds the state of a particular FastCGI request.
 */
typedef struct {
#ifdef WIN32
    SOCKET fd;
#else
    int fd;                         /* connection to FastCGI server */
#endif
    int gotHeader;                  /* TRUE if reading content bytes */
    unsigned char packetType;       /* type of packet */
    int dataLen;                    /* length of data bytes */
    int paddingLen;                 /* record padding after content */
    fcgi_server *fs;                /* FastCGI server info */
    const char *fs_path;         /* fcgi_server path */
    Buffer *serverInputBuffer;   /* input buffer from FastCgi server */
    Buffer *serverOutputBuffer;  /* output buffer to FastCgi server */
    Buffer *clientInputBuffer;   /* client input buffer */
    Buffer *clientOutputBuffer;  /* client output buffer */
    table *authHeaders;          /* headers received from an auth fs */
    int auth_compat;             /* whether the auth request is spec compat */
    table *saved_subprocess_env; /* subprocess_env before auth handling */
    int expectingClientContent;     /* >0 => more content, <=0 => no more */
    array_header *header;
    char *fs_stderr;
    int fs_stderr_len;
    int parseHeader;                /* TRUE iff parsing response headers */
    request_rec *r;
    int readingEndRequestBody;
    FCGI_EndRequestBody endRequestBody;
    Buffer *erBufPtr;
    int exitStatus;
    int exitStatusSet;
    unsigned int requestId;
    int eofSent;
    int role;                       /* FastCGI Role: Authorizer or Responder */
    int dynamic;                    /* whether or not this is a dynamic app */
    struct timeval startTime;       /* dynamic app's connect() attempt start time */
    struct timeval queueTime;       /* dynamic app's connect() complete time */
    struct timeval completeTime;    /* dynamic app's connection close() time */
    int keepReadingFromFcgiApp;     /* still more to read from fcgi app? */
    const char *user;               /* user used to invoke app (suexec) */
    const char *group;              /* group used to invoke app (suexec) */
#ifdef WIN32
    BOOL using_npipe_io;             /* named pipe io */
#endif
    int nph;
} fcgi_request;
```