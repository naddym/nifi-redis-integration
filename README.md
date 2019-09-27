# Integrating Apache NiFi with Redis
Redis is a key-value store which allows data to be stored and accessed at lightning fast speeds. Integrating NiFi with external cache system like Redis makes in-memory datastore much durable, reliable and fault-tolerant. Though NiFi provides internal cache system called DistributedMapCache (DMC) which can either run on all nodes or a single node (seed node). However, niether this is scalable solution (as node count increases then duplication of cache data increases) nor it offers high avalibility if the seed node goes down (single point of failure). Hence, a mechanism of external system is required to address above shortcomings which is eliminated by integrating nifi with redis.

## Configuration Steps
There are two ways of Integrating NiFi with Redis
- Using docker cli
- Using docker-compose

### Using docker cli
To begin with, we need two docker images (redis image and nifi image)
```
    docker pull apache/nifi
    docker pull redis
 ```



After images are pulled, We need to enable communication between them for which I create a custom network.

Before creating a custom network, I wish to see my available network drivers
```
    docker network ls
 ```

From bove drivers, I choose to create a custom network of bridge driver
```
    docker network create --driver bridge nifi-redis-squad
 ```
Note: For docker-compose method, we don't need as it auto handles docker networking.

I link redis image with custom network created and bind host port to `6379`
```
     docker run --rm -it --name redis -p 6379:6379 --network nifi-redis-squad redis
 ```

Again, I link apache/nifi with custom network created and bind host port to `8322`
```
    docker run --rm -itd --name nifi -p 8322:8080 --network nifi-redis-squad redis
 ```

 Verification Step: In order to check if I'm able to reach redis from nifi container, I do a simple ping test
```
    docker exec -it nifi sh
    ping redis  (exec into the container and then do ping)
 ```

 Great!! I'm able to reach redis. Now, I go to my apache/nifi application running to create a simple dataflow which puts key-value in redis. I drag `PutDistributedMapCache` Processor on canvas and create 'RedisDistributedMapCacheClient' for my `Distributed Cache Service` property.

Next, I configure `RedisConnectionPoolService` Controller Service by navigating from `RedisDistributedMapCacheClient` Controller Serivce. I configured `redis:6379` for `Connection String` Property, `redis` is my hostname which I have chosen as my container-name 
```
Note : RedisConnectionPoolService is nested Controller Service linked from RedisDistributedMapCacheClient
```

### Using second method ( docker-compose )
This method is much simpler than previous method. Thanks to docker-compose which handles all the networking for us. Click on below link to navigate to compose file. 

https://github.com/naddybot/nifi-redis-integration/tree/dev/docker-compose.yml


