# Redis的持久化策略

[toc]



## RDB





Redis关于RDB策略的一些默认配置：

```
################################ SNAPSHOTTING  ################################
#
# Save the DB on disk:
#
#   save <seconds> <changes>
#
#   Will save the DB if both the given number of seconds and the given
#   number of write operations against the DB occurred.
#
#   In the example below the behavior will be to save:
#   after 900 sec (15 min) if at least 1 key changed
#   after 300 sec (5 min) if at least 10 keys changed
#   after 60 sec if at least 10000 keys changed
#
#   Note: you can disable saving completely by commenting out all "save" lines.
#
#   It is also possible to remove all the previously configured save
#   points by adding a save directive with a single empty string argument
#   like in the following example:
#
#   save ""

save 900 1
save 300 10
save 60 10000

# 默认情况下，如果启用了RDB快照（至少有一个保存点），并且最近一次后台保存失败，Redis将停止接受写操作。Redis采用这种不好的方式使用户意识到，数据没有正确地持久化在磁盘上，否则有可能没有人注意到数据持久化失败。
# 如果后台保存过程再次开始工作，Redis将自动允许再次写入。
# 然而，如果你已经设置了你对Redis服务器和持久化的适当监控，你可能想禁用这个功能，这样即使磁盘有问题，Redis也会继续照常工作、 权限等问题，Redis也能照常工作。
stop-writes-on-bgsave-error yes

# 转储.rdb数据库时是否使用LZF压缩字符串对象？
# 默认情况下，压缩是启用的，因为它在绝大多数情况都有优势。
# 如果你想在保存过程中节省一些CPU，把它设置为'no'。
rdbcompression yes

# 是否开启CRC64校验。如果开启，则文件的可靠性更高；如果关闭，则可以提高约10%的性能。
rdbchecksum yes

# rdb文件名
dbfilename dump.rdb

# Remove RDB files used by replication in instances without persistence
# enabled. By default this option is disabled, however there are environments
# where for regulations or other security concerns, RDB files persisted on
# disk by masters in order to feed replicas, or stored on disk by replicas
# in order to load them for the initial synchronization, should be deleted
# ASAP. Note that this option ONLY WORKS in instances that have both AOF
# and RDB persistence disabled, otherwise is completely ignored.
#
# An alternative (and sometimes better) way to obtain the same effect is
# to use diskless replication on both master and replicas instances. However
# in the case of replicas, diskless is not always an option.
rdb-del-sync-files no

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# rbd文件的目录
dir /var/lib/redis
```

在`redis.conf`中还有一些关于RDB的配置。



save和bgsave：

bgsave：主进程会fork一个子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入RDB文件。虽说主进程fork子进程的时候也有阻塞，但是和save的阻塞相比，完全不值一提！

## AOF

