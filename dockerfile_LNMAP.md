# 基于centos6系统下使用dockerfile创建LNMAP

| 字母| 代表 |
|---|---|
| L | Centos6.9 |
| N | nginx |
| A | apache2.2 |
| P | php5.6 |


# 一、克隆项目
```
# yum install -y git
# git clone https://github.com/funet8/centos6_LANP_dockerfile.git
```

# 二、安装docker

```
# wget https://raw.githubusercontent.com/funet8/centos6_LANP_dockerfile/master/shell/CentOS6_7_intall_docker.sh
# sh CentOS6_7_intall_docker.sh
查看docker版本
# docker -v
Docker version 1.13.1, build 94f4240/1.13.1
```

# 三、创建主镜像（centos6）
```
# cd centos6_LANP_dockerfile/centos6_v2/
# sh build_docker_centos6.sh
或者：
# docker build -t  funet8/centos:6.9 .
```

# 四、构建centos6_MariaDB镜像
```
# cd centos6_LANP_dockerfile/centos6_Yum_MariaDB
构建镜像
# docker build -t  funet8/centos6mariadb .
```

## 启动MariaDB容器
```
# docker run -itd --name centos6MariaDB  funet8/centos6mariadb

# docker run -itd --name centos6MariaDBv1 \
--restart always \
-p 61950:3306 \
-v /data/docker/mysql_conf/my.cnf:/etc/my.conf  \
-v /data/docker/mysql_conf/mysql_slowQuery.log:/var/log/mysql/mysql_slowQuery.log \
-v /data/docker/mysql_docker:/var/lib/mysql \
funet8/centos6mariadb
```

## 进入容器并且创建用户
进入mysql中创建远程登录用户，并且删除默认root用户
```
mysql>CREATE USER 'dbuser_lxx'@'%' IDENTIFIED BY 'Yxa7dvKh94JhYY303bb';
mysql>GRANT  all privileges ON * . * TO 'dbuser_lxx'@'%' IDENTIFIED BY 'Yxa7dvKh94JhYY303bb';
mysql>GRANT ALL PRIVILEGES ON * . * TO 'dbuser_lxx'@'%' WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;
mysql>flush privileges;
```

# 五、创建基于Centos6下的LNAP
```
# cd centos6_LANP_dockerfile/centos6_lnap/
创建目录
# sh create_dir.sh
构建镜像:
# docker build -t  funet8/centos_lnap:6.9.1 .
```
## 启动容器：
```
docker run -itd --name centos6lnap --restart always -p 80:80 -p 443:443 -v /data/:/data/ -v /data/conf/nginx.conf:/etc/nginx/nginx.conf -v /data/conf/httpd.conf:/etc/httpd/conf/httpd.conf -v /data/conf/php.ini:/etc/php.ini  funet8/centos_lnap:6.9.1
```

-----------分割线----------------------------------------


# 管理容器
## 进入容器
```
docker logs centos6lnap
docker exec -it centos6lnap /bin/bash
```
## 删除容器和镜像
```
docker rm -f centos6lnap
docker rmi funet8/centos_lnap:6.9.1
```

## 容器权限问题
```
###权限问题的总结

#在宿主上查看www用户的ID

## cat /etc/passwd |grep www

#www:x:1001:1001::/home/www:/sbin/nologin

#进入docker虚拟机

## usermod -u 1001 www

## groupmod -g 1001 www

#将所需要的目录更改权限

#chown www.www -R /data/web/dir/
```


## docker网络问题解决
```
pkill docker 
iptables -t nat -F 
ifconfig docker0 down 
brctl delbr docker0 
docker -d 
service docker restart
或者
/bin/systemctl restart  docker.service
/bin/systemctl restart iptables

-bash: brctl: command not found
解决：
yum install bridge-utils
```


