```

    /* connect server get host by name    */
    if ( hostname )
    {
        if( (hostnp = gethostbyname(hostname) ) == NULL )
        {
            printf("get host by name failure: %s\n", strerror(h_errno)) ;
            return -1 ;

        }
        printf("hostname %s\n", hostnp->h_name);
        ip = inet_ntoa( * (struct in_addr *)hostnp->h_addr );
        printf("addr:%s\n",ip) ;
```
