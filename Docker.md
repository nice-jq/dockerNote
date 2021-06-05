# Docker 笔记

## docker 命令

```powershell
$ docker run -it --rm tomcat:9.0 ##测试用，用完即删，docker ps -a 也搜索记录
```

### 启动容器 --tomcat为例

```powershell
$ docker run -d -p 8859:8080 --name tomcat01 tomcat ## -d 后台运行 ，-p 暴露端口号 ，
## 8859--外部端口，8080--tomcat服务端口，--name -- 给容器取名自 tomcat -- 镜像名
```

### 查看内存

```powershell
$ docker stats ##查看CUP的使用情况
```

![截图](b7358f5d775c8f263db660817d84e291.png)

### 限制内存  添加参数   -e

```powershell
$ docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
```

![截图](7884194756bcab8f9e09e8e2aa203e20.png)

### 使用kibanna连接es

网络原理图

![截图](8c499c20fa7f32c58cec445d55919f26.png)

## 可视化工具

- ### portainer
  
  docker图形化管理工具

```powershell
$ docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
# -v 挂载 --privileged  授予访问权限
```

外网访问portainer

![截图](833ef76956c5479eacb0e2b31f32f6f2.png)

设置密码后登录的界面

![截图](9e7b78a8b83c3aa04dcd8ce4b52e0180.png)

<br/>

- ### Rancher(CI/CD)

<br/>

<br/>

## docker分层思想--联合文件系统

![截图](cfc2be500fdcdc34c8ab66bfcf83a211.png)

### commit提交自己镜像

```powershell
$ docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

![截图](e2909915ad4b74b76b2db2c6be4986df.png)

![截图](4293f51b6057243c74eaff5a69885227.png)

如果想保存当前容器的状态，可以使用commit提交这个容器，获得一个镜像，就像vm虚拟机的快照一样。

## 容器数据卷

![截图](724ae6395a22d6414234a009ea99d170.png)

![截图](88b4678a563677809bc6f4a7eba26f9d.png)

![截图](df7a3dacb0d62efb3bef660f7066c653.png)

### 使用数据卷

#### 方式1：直接使用命令来挂载 -v

```powershell
$ docker run -it -v 主机目录：容器目录 #和-p（暴露端口） 相似
```

##### 容器内部目录

![截图](ad827b1186640a156c105692e5b4c2dc.png)

##### 主机目录

![截图](6442ff145cfce212049f709cbb66c82a.png)

![截图](15c90119bf296ac6fd7c13dbf941676a.png)

同步过程，双向绑定。在主机挂载目录中新建文件会同步到容器目录中，反之亦然

停止容器后，在主机上修改文件，再次启动容器，容器内部的数据依旧是同步的

##### 实战挂载mysql的数据存储目录和配置文件目录

```powershell
#启动mysql容器需要添加密码  官方文档
$ docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
# -e添加参数 环境配置
#启动MySQL容器
$ docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
```

##### 第三方工具连接mysql

![截图](f18ced77ad72ec703dc9c40a6a62a16e.png)

![截图](80e728ca7c942fde16490b56ad049f66.png)

![截图](aad3a2473611cb2f9ddc2897c750c0c2.png)

### 具名挂载和匿名挂载

```powershell
#匿名挂载
-v 容器内路径
$ docker run -d -P --name nginx01 -v /etc/nginx nginx 

#查看卷
$ docker volume ls
```

![截图](de40d65c976aae450c37371979b1b888.png)

![截图](4b5f5c8f284abc9818d56f906ac569c9.png)

```shell
#具名挂载
[root@iZwz98g1q9ntiadiije2feZ ~]# docker run -d -P --name nginx02 -v juming-voluma:/etc/nginx nginx  # -P 随机指定端口映射 -v 卷名：容器内路径
0bd616888883fd3f3822cb100dcd6d0d3cddda31bd97d732a7f197049114a708
[root@iZwz98g1q9ntiadiije2feZ ~]# docker volume ls
DRIVER    VOLUME NAME
local     juming-voluma
```

查看这个卷的路径

```shell
$  docker volume inspect juming-voluma
```

![image-20210605151206006](C:\Users\huang\AppData\Roaming\Typora\typora-user-images\image-20210605151206006.png)

![image-20210605151413586](C:\Users\huang\AppData\Roaming\Typora\typora-user-images\image-20210605151413586.png)

```shell
#具名挂载、匿名挂载、指定路径挂载区分
-v 容器内路径  			#匿名挂载
-v 卷名：容器内路径	  	  #具名挂载
-v 宿主机路径：容器内路径  #指定路径挂载
```

![image-20210605152421482](C:\Users\huang\AppData\Roaming\Typora\typora-user-images\image-20210605152421482.png)

### DockerFile

DockerFile就是用来构建docker镜像的构建文件

