```c

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

```c
	//打印时间 毫秒
	struct timeval tv;
	gettimeofday(&tv, NULL);
	printf("main second: %ld\n", tv.tv_sec); // 秒
	printf("main millisecond: %ld\n", tv.tv_sec * 1000 + tv.tv_usec / 1000); // 毫秒
	printf("main microsecond: %ld\n", tv.tv_sec * 1000000 + tv.tv_usec);	 // 徽秒
	time_t mainRunTime;
	struct tm *pmainRunTime;
	mainRunTime = time(NULL);
	pmainRunTime = localtime(&mainRunTime);
	printf("main Run Time: [%04d-%02d-%02d %02d:%02d:%02d:%03d] \n", pmainRunTime->tm_year + 1900, pmainRunTime->tm_mon + 1,
	pmainRunTime->tm_mday, pmainRunTime->tm_hour, pmainRunTime->tm_min, pmainRunTime->tm_sec);
```
