# SpringBoot+Redis实战

[toc]

人为给值添加层级关系，实际上并没有层级关系。

```
redisTemplate.opsForValue().set("user:age:", "8"); 
```



redis创建6位二进制，并将第5位（从0计数）置1。

```
stringRedisTemplate.opsForValue().setBit("myKey", 5, true);
相当于：127.0.0.1:6379> setbit mykey 5 1

```
