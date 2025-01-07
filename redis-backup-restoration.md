######################

# Case 

1 - old redis in k8s cluster #1

2 - fresh install of redis in Cluster #2


- Target backup data in old cluster and restore in new cluster


# Info:
- No auth configured
- command redis-cli BGSAVE - Save the DB in background.
- docker image https://hub.docker.com/r/bitnami/redis
 


# Backup
1. Goto k8s cluster #1
2. Goto redis pod and create redis dump

>>>
     $ kubectl  exec -ti redis-0 -- bash
     $ redis-cli
     $> BGSAVE
     $> exit
>>>

3. Copy dump from pod to local
>>>
     $ kubectl cp redis-0:/bitnami/redis/data/dump.rdb $(pwd)/dump.rdb
>>>
4. Done with backup


---

# Restore 
In cluster #2, new redis

1. Copy redis dump file to new redis pods
>>>
    $ kubectl cp ./dump.rdb redis-0:/bitnami/redis/data/dump.rdb-1
>>>  

2. Modify k8s resource deployment or statefullset  and add new ENV Var
>>>
    $ kubectl edit deployments.apps redis
>>>
    containers:
      - env:
        - name: REDIS_AOF_ENABLED
          value: "no"
        - name: REDIS_DISABLE_COMMANDS
          value: FLUSHDB,FLUSHALL
>>>
- edit and save resource
- pod should restart

3. Restore redis dump
>>>
    $ kubectl  exec -ti redis-0 -- bash
    $ cd /bitnami/redis/data/
    $ chmod 777 dump.rdb-1
    $ cp dump.rdb-1 dump.rdb
    $ redis-cli
    $> bgrewriteaof     # start restoring
    $> info             # find status of 'aof_rewrite_in_progress'
    $> keys *           # find new keys
    $> exit
>>>

4. Modify k8s resource deployment or statefullset and delete ENV Var - REDIS_AOF_ENABLED,REDIS_DISABLE_COMMANDS
>>>
    $ kubectl edit deployments.apps redis
>>>    

5. Wait for new pod and goto into and check if data restored
>>>
    $ kubectl  exec -ti redis-0 -- bash
    $ redis-cli
    $> keys *           # find new keys
>>>
 
