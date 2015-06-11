# DevOps 教程

## 实验设计

实践DevOps，使用Jenkins和Docker完成“宠物医院”线上系统的改造和部署


## 现状

国内的宠物医院/诊所，使用传统的纸质档案维护就诊记录，不便于检索查询。

## 需求

亟需电子化手段帮助线上归档，以便今后提供在线服务及与其他系统整合（例如线上支付）。

## 方案

技术路线：JEE + 关系数据库

开源工具：Jenkins + Docker

## 开源的参考实现

Spring Sample：PetClinic

https://github.com/spring-projects/spring-petclinic 

## 准备工作

一台联网的工作机，连接远程的虚机；

SoftLayer虚机规格 Ubuntu 14.04 LTS x86_64

1Core, 4G Memory, 25GB, hourly VM

## 准备工作机

安装SSH软件(Putty)

http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html]

## SSH登录远程虚机

更新并安装软件(git, jdk, Jenkins)

```
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
echo 'deb http://pkg.jenkins-ci.org/debian binary/' >> /etc/apt/sources.list
sudo apt-get update -y
sudo apt-get install git -y
sudo apt-get install openjdk-7-jdk -y
sudo apt-get install jenkins -y
```

## 配置并重启Jenkins

```
echo 'jenkins  ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
service jenkins restart
```

## 尝试访问Jenkins界面,安装插件

http://pku??.devops.wha.la:8080/pluginManager/available

1. Delivery Pipeline Plugin
2. Git Plugin
3. Clone Workspace SCM Plugin
4. Copy Artifact Plugin

## 在Jenkins中配置Maven运行时

http://pku??.devops.wha.la:8080/configure

## 取得原始代码（可略过）

克隆Spring-petclinic项目:

- 用注册账号登录github.com
- 访问spring-petclinic项目，点击“Fork”按钮

## 新的spring-petclinic 项目

https://github.com/pkudevops/spring-petclinic

Repository URL: https://github.com/pkudevops/spring-petclinic.git

## 在Jenkins中新建Job测试spring-petclinic 项目 

test-petclinic

## 在Jenkins中添加build job

build-petclinic

```
mvn package -Dmaven.test.skip=true

```

## 建立DevOps Pipeline

视图中新建pipeline, 设置开始job 为 test-petclinic

---

## 准备运行时

## 在远程虚机安装软件docker, docker-compose

```
sudo apt-get install linux-image-extra-$(uname -r)
wget -qO- https://get.docker.com/ | sh

curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

## 把jenkins加入docker用户组

```
sudo groupadd docker
sudo gpasswd -a jenkins docker
sudo service docker restart
sudo service jenkins restart
```

## 试运行一个docker容器

```
sudo docker run hello-world
```

## 从Docker Hub获取基本的镜像

官方tomcat镜像的主页 https://registry.hub.docker.com/_/tomcat/

官方mysql 镜像的主页 https://registry.hub.docker.com/_/mysql/

```
docker pull tomcat
docker pull mysql
```

## 构建包含petclinic应用的Docker映像

Dockerfile

```
FROM tomcat

COPY petclinic.war /usr/local/tomcat/webapps/
```

构建命令

```
cp /var/lib/jenkins/jobs/build-petclinic/workspace/target/petclinic.war .
docker build -t petclinic .

docker images
```

## 启动应用

```
docker run -p 80:8080 -d petclinic
```

## 停止应用

···
docker stop $(docker ps -a -q)
···

## 让DevOps Pipeline流动起来

设置后向触发job

新建2个job

- cleandocker
- rundocker

## job "cleandocker"

```
containers=$(docker ps -a -q | wc -l)

if [ $containers != 0 ]; then
  docker stop $(docker ps -a -q)
  docker rm $(docker ps -a -q)
fi

img=$(docker images | grep petclinic | awk -F ' ' '{print $3}')

if [ $img ]; then
  docker rmi $img
fi
```

## job "rundocker"

```
mv target/spring-petclinic-1.0.0-SNAPSHOT.war target/petclinic.war

echo "FROM tomcat" > Dockerfile
echo "COPY target/petclinic.war /usr/local/tomcat/webapps/" >> Dockerfile

docker build -t petclinic .
docker run -p 80:8080 -d petclinic
```

## 使用docker-compose构建多组件应用

https://github.com/pkudevops/docker-petapp

docker-compose.yml

```
db:
  image: mysql
  env_file:
    - ./common.env
  volumes:
    - db:/var/lib/mysql
  ports:
    - "3306"

petclinic:
  build: .
  links:
    - db:mysql
  ports:
    - "80:8080"
```

common.env

```
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=petclinic
MYSQL_USER=user
MYSQL_PASSWORD=passw0rd
```
