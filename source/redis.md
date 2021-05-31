title: redis
date: 2019-05-12 10:10:10 
author: Xavier
tags: 
    - redis
type: article
---

# redis

## docker 运行 redis

$ docker run -itd --name redis-test -p 6379:6379 redis
$ docker exec -it redis-test /bin/bash
