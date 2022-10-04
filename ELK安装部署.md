

创建用户，在部署ELK服务器上添加用户ELK

```shell
[root@node04 /]# useradd elk
[root@node04 ~]# passwd elk
#添加密码：elk
```

添加安装目录权限，以及创建存储数据和日志目录并授予权限

```shell
[root@node04 /]# mkdir /apps/elk
[root@node04 /]# chown -R elk /apps/elk
[root@node04 /]# mkdir /data/elk
[root@node04 /]# chown -R elk /data/elk
[root@node04 /]# mkdir /logs/elk
[root@node04 /]# chown -R elk /logs/elk
```

为elk用户添加sudo权限，使用visudo，然后为elk用户添加权限

```shell
[root@node04 ~]# visudo
#添加如下内容
elk     ALL=(ALL)       ALL
```

切换用户到elk，下载elasticsearch，并解压到指定目录

```shell
[root@node04 ~]# su elk
[elk@node04 soft]$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.15.0-linux-x86_64.tar.gz
[elk@node04 soft]$ tar zxvf elasticsearch-7.15.0-linux-x86_64.tar.gz -C /apps/
```

配置ES

```shell
[elk@node04 config]$ mkdir /data/elk/es
[elk@node04 config]$ mkdir /logs/elk/es
[elk@node04 elk]$ cd /apps/elk/elasticsearch-7.15.0/config/

```

```yaml
# 集群名字
cluster.name: es
# 集群中当前的节点
node.name: node04
# 数据目录
path.data: /data/elk/es
# 日志目录
path.logs: /logs/elk/es
# 当前主机的ip地址
network.host: 192.168.1.14
http.port: 9200
# 集群上的节点信息
discovery.zen.ping.unicast.hosts: ["node04","node05","node06"]
# linux安装es的一个bug解决的配置
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
# 是否支持跨域
http.cors.enabled: true
# *表示支持所有域名
http.cors.allow-origin: "*"
```

将配置好的ES复制到其他节点服务器

```shell
[elk@node04 elk]$ scp -r elasticsearch-7.15.0 node05:$PWD
[elk@node04 elk]$ scp -r elasticsearch-7.15.0 node06:$PWD
```

修改其他节点配置，node05节点修改

```shell
[root@node05 elk]# cd /apps/elk/elasticsearch-7.15.0/config/
```

```yaml
# 集群名字
cluster.name: es
# 集群中当前的节点
node.name: node05
# 数据目录
path.data: /data/elk/es
# 日志目录
path.logs: /logs/elk/es
# 当前主机的ip地址
network.host: 192.168.1.15
http.port: 9200
# 集群上的节点信息
discovery.zen.ping.unicast.hosts: ["node04","node05","node06"]
# linux安装es的一个bug解决的配置
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
# 是否支持跨域
http.cors.enabled: true
# *表示支持所有域名
http.cors.allow-origin: "*"
```

修改其他节点配置，node06节点修改

```shell
[root@node05 elk]# cd /apps/elk/elasticsearch-7.15.0/config/
```

```yaml
# 集群名字
cluster.name: es
# 集群中当前的节点
node.name: node06
# 数据目录
path.data: /data/elk/es
# 日志目录
path.logs: /logs/elk/es
# 当前主机的ip地址
network.host: 192.168.1.16
http.port: 9200
# 集群上的节点信息
discovery.zen.ping.unicast.hosts: ["node04","node05","node06"]
# linux安装es的一个bug解决的配置
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
# 是否支持跨域
http.cors.enabled: true
# *表示支持所有域名
http.cors.allow-origin: "*"
```

修改集群每个节点JVM内存大小

```shell
[elk@node06 config]$ vim jvm.options
```

```
-Xms2g
-Xmx2g
```

