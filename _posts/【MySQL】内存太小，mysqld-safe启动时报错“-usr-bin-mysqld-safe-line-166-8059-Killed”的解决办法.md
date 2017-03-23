---
title: '【MySQL】内存太小，mysqld_safe启动时报错“/usr/bin/mysqld_safe: line 166: 8059 Killed”的解决办法'
date: 2015-11-20 16:31:29
tags:
- 方法
- 原创
categories: 
- 方法
---
今天早上服务器出了点问题，MySQL down了，以往的问题log都有提示，今天没有提示，如下
```python
151120 11:58:15 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2363 2015-11-20 11:58:15 0 [Note] /usr/sbin/mysqld (mysqld 5.6.27-log) starting as process 4834 ...
2364 2015-11-20 11:58:15 4834 [Warning] option 'innodb-buffer-pool-size': signed value 2097152 adjusted to 5242880
2365 2015-11-20 11:58:15 4834 [Note] Plugin 'FEDERATED' is disabled.
2366 2015-11-20 11:58:15 4834 [Note] InnoDB: Using atomics to ref count buffer pool pages
2367 2015-11-20 11:58:15 4834 [Note] InnoDB: The InnoDB memory heap is disabled
2368 2015-11-20 11:58:15 4834 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2369 2015-11-20 11:58:15 4834 [Note] InnoDB: Memory barrier is not used
2370 2015-11-20 11:58:15 4834 [Note] InnoDB: Compressed tables use zlib 1.2.3
2371 2015-11-20 11:58:15 4834 [Note] InnoDB: Using Linux native AIO
2372 2015-11-20 11:58:15 4834 [Note] InnoDB: Using CPU crc32 instructions
2373 2015-11-20 11:58:15 4834 [Note] InnoDB: Initializing buffer pool, size = 5.0M
2374 2015-11-20 11:58:15 4834 [Note] InnoDB: Completed initialization of buffer pool
2375 2015-11-20 11:58:16 4834 [Note] InnoDB: Highest supported file format is Barracuda.
2376 151120 11:58:18 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
2377 151120 12:05:47 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2378 151120 12:05:47 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
2379 151120 12:08:17 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2380 151120 12:08:18 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
2381 151120 12:10:01 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2382 151120 12:10:02 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
2383 151120 12:10:39 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2384 151120 12:10:40 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
2385 151120 12:10:49 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2386 151120 12:10:49 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
[root@iZ94y2i49cgZ ~]# mysqld_safe
151120 12:23:21 mysqld_safe Logging to '/var/log/mysqld.log'.
151120 12:23:21 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
/usr/bin/mysqld_safe: line 166: 8059 Killed nohup /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --user=mysql --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock < /dev/null >> /var/log/mysqld.log 2>&1
151120 12:23:21 mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
```
网上查了下，一般大家碰到的问题在/var/log/mysqld.log中有提示，竟然几乎没有和我一样问题的
唯一碰到个[一样的帖子](http://www.itpub.net/forum.php?mod=viewthread&tid=1810586&highlight=) 也是稀里糊涂解决了，他也没讲清楚，不过他提到了虚拟机的内存问题，就在前几天我的数据库刚刚因为占内存太多当机了，所以我觉得就是内存的问题，但是当时我就设置了innodb_buffer_pool_size=5M 已经改的足够小了，还是起不来，没办法，继续折腾吧，后来看到一个详细的配置的方法，[原文](http://serverfault.com/questions/565128/mysql-5-6-on-centos-6-silently-fails-to-start)，如下：
```python
key_buffer=16K
table_open_cache=4
query_cache_limit=256K
query_cache_size=4M
max_allowed_packet=1M
sort_buffer_size=64K
read_buffer_size=256K
thread_stack=64K
innodb_buffer_pool_size = 56M
```
加入到my.cnf中，终于起来了
原文里还说道“It was definitely a memory issue. This became evident once I looked in /var/log/messages. I was trying to run recent versions of Nginx, PHP-FPM, and MySQL, and the default configurations for all 3 of these together were too much for my little droplet that had only 512 MB of memory and no swap space.I tweaked my /etc/my.cnf...”
果然还是服务器内存太小的原因，只能说穷人伤不起，只能在配置上挖掘服务器更多的潜力。
