# 说明
```
sh$ ./gu  -- 生成 ./deploy
sh$ ./deploy --env dev|prod_luexu
sh$ ./deploy --env --opt DEBUG...
```


# How To Use ? 怎么用？
```
sh$ cd Conf/Dockerfiles
sh$ sudo ./docker-build -H                                  # help 帮助
|[
Usage:
  docker-build [$options]
  docker-build
    $dir  : build the alpha|beta|stable version Dockerfile under this directory
      e.g.
          *       : build all Dockerfiles under current directory
          $dir/* : build all Dockerfiles under this dir
    -a    : Build All, includes alpha, beta and stable
    -b    : Build Beta+, inclues beta and stable
    -f    : fast, do not clean
    -H    : help
]|

sh$ sudo ./docker-build ${centos*|mysql*|php*|redis*}       # build any of these Docerfile 编译其中规则
sh$ sudo ./docker-build *                                   # build all Dockerfiles under current dir
sh$ sudo ./docker-build /dir/*                              # build all Dockerfiles under /dir/
```
# 说明
配置文件一律在容器内，只有重要数据，才挂在到主机
# Docker 常见问题
```
刚刚#Docker# 容器内部无法解析 DNS了，可以ping 通 IP，但是ping不通百度。明显是DNS问题。试了很多解决方案，都没有成功。
不过可以直接在 docker run 的时候，添加 在里面添加  docker_OPTS='--dns 223.5.5.5 --dns 223.6.6.6'  （用的阿里云DNS，可以自己换） 国外的（docker_OPTS='--dns 8.8.8.8 --dns 4.4.4.4'）

不过最后一个：
sh$ systemctl show docker | grep EnvironmentFile
|[
EnvironmentFile=/etc/sysconfig/docker (ignore_errors=yes)
EnvironmentFile=/etc/sysconfig/docker-storage (ignore_errors=yes)
EnvironmentFile=/etc/sysconfig/docker-network (ignore_errors=yes)
]|

如果不存在的话，重新设下docker开机启动 systemctl enable docker，重启下docker

vi /etc/sysconfig/docker  ，在里面添加  docker_OPTS='--dns 223.5.5.5 --dns 223.6.6.6'  （用的阿里云DNS，可以自己换） 国外的（docker_OPTS='--dns 8.8.8.8 --dns 4.4.4.4'  ），然后重启docker，就可以了…

```


# Ports
* [0, 1023]         System Reserved
* [1024, 9999]      Softwares Reserved
* 1xxxx             Cluster 1
* 2xxxx             Cluster 2
* 3xxxx             Cluster 3
* 4xxxx             Cluster 4
* 5xxxx             Cluster 5
* [60000, 65535]    Temporary Ports

* cassandra 9160:9160
* cassandra slaves 49100..49199
* git 9418:9418 10022:22
* gitlab 39418:9418 -p 39480:80 -p 39422:22
* go 30080:80 38080:8080
* kafka 994:994 465:465 9092:9092
* mosquitto 1883:1883 8883:8883
* mosquitto-websockets 11883:1883 18883:8883 11880:11880 18880:18880
* mysql 3306:3306
* mysqlslave 43300..43399
* nginx 80:80 8080:8080 48080:48080 40080:40080 443:443
* nginx clusters 48000..48099
* php/swoole 9000:9000 -p 9001:9001 -p 9003:9003 -p 9501:9501 -p 9502:9502 -p 9503:9503
* php clusters 49000..49099
* postgresql 5432:5432
* postgresql slaves 45400..45499
* redis 6379:6379
* redis slaves/clusters 46300..46399

# Redis Container
