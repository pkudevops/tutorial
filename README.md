# DevOps �̳�

## ʵ�����

ʵ��DevOps��ʹ��Jenkins��Docker��ɡ�����ҽԺ������ϵͳ�ĸ���Ͳ���


## ��״

���ڵĳ���ҽԺ/������ʹ�ô�ͳ��ֽ�ʵ���ά�������¼�������ڼ�����ѯ��

## ����

ؽ����ӻ��ֶΰ������Ϲ鵵���Ա����ṩ���߷���������ϵͳ���ϣ���������֧������

## ����

����·�ߣ�JEE + ��ϵ���ݿ�

��Դ���ߣ�Jenkins + Docker

## ��Դ�Ĳο�ʵ��

Spring Sample��PetClinic

https://github.com/spring-projects/spring-petclinic 

## ׼������

һ̨�����Ĺ�����������Զ�̵������

SoftLayer������ Ubuntu 14.04 LTS x86_64

1Core, 4G Memory, 25GB, hourly VM

## ׼��������

��װSSH���(Putty)

http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html]

## SSH��¼Զ�����

���²���װ���(git, jdk, Jenkins)

```
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
echo 'deb http://pkg.jenkins-ci.org/debian binary/' >> /etc/apt/sources.list
sudo apt-get update -y
sudo apt-get install git -y
sudo apt-get install openjdk-7-jdk -y
sudo apt-get install jenkins -y
```

## ���ò�����Jenkins

```
echo 'jenkins  ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
service jenkins restart
```

## ���Է���Jenkins����,��װ���

http://pku??.devops.wha.la:8080/pluginManager/available

1. Delivery Pipeline Plugin
2. Git Plugin
3. Clone Workspace SCM Plugin
4. Copy Artifact Plugin

## ��Jenkins������Maven����ʱ

http://pku??.devops.wha.la:8080/configure

## ȡ��ԭʼ���루���Թ���

��¡Spring-petclinic��Ŀ:

- ��ע���˺ŵ�¼github.com
- ����spring-petclinic��Ŀ�������Fork����ť

## �µ�spring-petclinic ��Ŀ

https://github.com/pkudevops/spring-petclinic

Repository URL: https://github.com/pkudevops/spring-petclinic.git

## ��Jenkins���½�Job����spring-petclinic ��Ŀ 

test-petclinic

## ��Jenkins�����build job

build-petclinic

```
mvn package -Dmaven.test.skip=true

```

## ����DevOps Pipeline

��ͼ���½�pipeline, ���ÿ�ʼjob Ϊ test-petclinic

---

## ׼������ʱ

## ��Զ�������װ���docker, docker-compose

```
sudo apt-get install linux-image-extra-$(uname -r)
wget -qO- https://get.docker.com/ | sh

curl -L https://github.com/docker/compose/releases/download/1.2.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```

## ��jenkins����docker�û���

```
sudo groupadd docker
sudo gpasswd -a jenkins docker
sudo service docker restart
sudo service jenkins restart
```

## ������һ��docker����

```
sudo docker run hello-world
```

## ��Docker Hub��ȡ�����ľ���

�ٷ�tomcat�������ҳ https://registry.hub.docker.com/_/tomcat/

�ٷ�mysql �������ҳ https://registry.hub.docker.com/_/mysql/

```
docker pull tomcat
docker pull mysql
```

## ��������petclinicӦ�õ�Dockerӳ��

Dockerfile

```
FROM tomcat

COPY petclinic.war /usr/local/tomcat/webapps/
```

��������

```
cp /var/lib/jenkins/jobs/build-petclinic/workspace/target/petclinic.war .
docker build -t petclinic .

docker images
```

## ����Ӧ��

```
docker run -p 80:8080 -d petclinic
```

## ֹͣӦ��

������
docker stop $(docker ps -a -q)
������

## ��DevOps Pipeline��������

���ú��򴥷�job

�½�2��job

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

## ʹ��docker-compose���������Ӧ��

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
