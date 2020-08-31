# <center>Nginx学习</center>

### Nginx简介

- 反向代理
- 负载均衡
- 动静分离
- 集群
- 运行原理

占用内存少，并发能力强；支持高达50000个并发连接数；专为性能优化开发；

#### 反向代理与正向代理

### Nginx常用命令

```shell
cd /usr/local/nginx/sbin
./nginx -v #查看版本号
./nginx -s stop #停止
./nginx #启动
./nginx -s reload #重加载
```

配置文件：

```shell
view /usr/local/nginx/conf/nginx.conf
```

- 全局块：影响整体运行的配置
- events块：网络连接配置，例如最大连接数
- http块：http块、server块

### keepAlived工具配置

