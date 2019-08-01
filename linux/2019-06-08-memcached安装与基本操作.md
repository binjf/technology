# memcached 安装与基本操作

Memcached 服务器

| 服务器名             | IP           | 安装目录             |
| -------------------- | ------------ | -------------------- |
| memcached-data-prod1 | 172.18.xx.xx | /usr/local/memcached |

## 安装与启动

1 解压、检查依赖

```
tar -zxvf memcached-1.5.15.tar.gz 
cd memcached-1.5.15
./configure --prefix=/usr/local/memcached
```

2 可能会缺少依赖，安装依赖如下

```
yum -y install libevent
yum -y install libevent-devel
./configure --prefix=/usr/local/memcached
```

3 编译、安装

```
make && make test
make install
```

4 启动

```
/usr/local/memcached/bin/memcached -p 11211 -m 64m -d -u kduser
```

5 查看，可以安装 `libmemcached` 来查看状态

```
ps -ef|grep memcached

yum -y install libmemcached
memstat --servers=127.0.0.1:11211
```

6 关闭端口 11211

```
netstat -ntlp
kill -9 xxxx
```

