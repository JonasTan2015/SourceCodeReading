# Implement Distributed Locking service with Redis


This is an implementation of distributed locking with Redis.

## Requirements
* Return true if successfully acquire lock, otherwise return false
* If fail to acquire lock, wait a moment and keep trying. When suceeed, return true. After trying for centain amount of time with no success, return false
* Dead lock free
* Cannot release a lock already acquired by other threads

## Java Implementation
### Seting a lock
#### Requirements
1. Make sure the key does not exists in Redis already
2. Set key-value as a lock. 
3. This lock shall expire after certain amount of time
4. All the previous steps have to be atomic

After version 2.6, Redis supports the following SET command, which is atomic
```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
// EX: expiration time in seconds
// PX:  expiration time in mini-seconds
// NX: set key when not exists
// XX: set key only when it already exists
```

### Unlocking

####Steps
1. Check whether a lock exists in Redis
2. Make sure the lock is already acquired by that thread
3. Remove the key-value pair

####Requirements
1. Unlock only the one belongs to that thread.
2. All the above steps should be atomic.

Redis supports Lua scripts. We could make it atomic through a Lua script
```
jedis.select(dbIndex);
String key = KEY_PRE + key;
String command = "if redis.call('get',KEYS[1])==ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
if (1L.equals(jedis.eval(command, Collections.singletonList(key), Collections.singletonList(value)))) {
    return true;
}
```

