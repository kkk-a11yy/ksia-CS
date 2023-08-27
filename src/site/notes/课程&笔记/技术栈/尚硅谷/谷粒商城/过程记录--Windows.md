---
{"dg-publish":true,"permalink":"/and////windows/","tags":["gardenEntry"]}
---

# windows10系统下的操作
下面这部分是win10系统下的搭建，由于原来电脑配置不够，遂换了个M1pro:16GB+1T,重新搭建环境
     "[过程记录--M1 PRO](课程&笔记/技术栈/尚硅谷/谷粒商城/过程记录--M1%20PRO.md)"
# 1.  环境搭建
## 安装 vagrant :
	    官网  cmd命令终端输入：`vagrant` 测试是否下载成功 ，`vagranr init centos/7`  创建vagrantfile（一般在C:\Users\KKK\下，请注意C盘的空余空间要留出来多一点，本人因为这个问题后续下载不成功，删除虚拟机，用分区助手重新划分C盘，空出来48G空间后又重新来了一遍）， `vagrant up` ，下载完毕后启动virtual box 查看新建的虚拟机
	    ![](https://i.imgur.com/nGqShAf.png)
##  与虚拟机连接：
	    终端 `ctrl+c` 退出 , `vagrant ssh ` 连接账户，可以在终端直接敲linux命令：`ls` , `exit` , `whoami` , 停止虚拟机可以在virtual box 停止或开启，也可以在终端使用 `vagrant up`(必须确保在有vagrant file 的目录下使用，"C:\Users\KKK")， 可以使用 `vagrant reload` 重启虚拟机
## 虚拟机网络配置：
	    `ipconfig` 显示的这里是几![](https://i.imgur.com/uXtrT3l.png)
	    ，这里是`169.254.228.175`， 在 vagrantfile里的对应修改为`169.254.228.几` ,这里是4 ，  并保存![](https://i.imgur.com/oGJx9fO.png)，修改后重启+连接虚拟机 `vagrant reload` ,  `vagrant ssh` ， 查看虚拟机ip `ip addr`, 检查主机与虚拟机是否能ping通：`C:\Users\KKK>ping 169.254.228.4` , `[vagrant@bogon ~]$ ping 192.168.1.8`  .  注意：如果换网络了，比如vpn换节点，要重新启动虚拟机
## 安装docker，配置阿里云镜像下载dockerhub中的镜像
	   去docker hub安装镜像，"[Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/)",本人用了魔法，but 这里还是配个阿里云镜像加速
###  卸载旧版本docker：
```    r
  sudo yum remove docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine  
```
### set up the repository
 1. install packages:   
	 ```r
	 sudo yum install -y yum-utils 
	 sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
	 ```
2. 告诉linux docker 去哪里装最新版本docker：
	  ``` r
	  sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin 
	  ```
3. 启动docker
	 ```  r
	 sudo systemctl start docker 
	 ```
	  1. 启动后其他命令：`docker -v` , `sudo docker images` , `sudo docker run hello-world `
	  2. 设置docker 为linux开机自启动 ：`sudo systemctl enable docker`
		1. 配置docker阿里云镜像加速器centos："[容器镜像服务 (aliyun.com)](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)" (本人已魔法，这步可以不做)
             ``` 
			 sudo mkdir -p /etc/docker
			 
			 sudo tee /etc/docker/daemon.json <<-'EOF'  
			 {
				 "registry-mirrors": ["https://u0euiox4.mirror.aliyuncs.com"]
			}
			 EOF
			 
			 sudo systemctl daemon-reload   //重启docker线程
			 
			 sudo systemctl restart docker   //重启docker服务
			  
			  ```
## docker 安装mysql
 1.  如果把虚拟机关闭，重新启动的过程可以是：在VM VirtualBox 启动，然后 `cmd`终端 `vagrant ssh` 启动
 2. DockerHub 中的mysql5.7版本的镜像
	  `docker pull mysql:5.7`
	  1. 权限不足问题：permission denied ：
		 ```  r
		[vagrant@bogon ~]$ docker pull mysql:5.7
		permission denied while trying to connect to the Docker daemon socket at          unix:///var/run/docker.sock: Post    "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/create?fromImage=mysql&tag=5.7": dial unix /var/run/docker.sock: connect: permission denied
			```
		  解决1.："[docker hub pull 没权限 - 掘金 (juejin.cn)](https://juejin.cn/post/7237806553842319420)"
		 ``` r
		 sudo chmod 666 /var/run/docker.sock
		 ```
		  解决2：`sudo docker pull mysql:5.7`
		  解决3：`su root` 切换到root用户，password ：由 vagrant创建的虚拟机，密码也默认是`vagrant`
	  2. docker网络问题：
		 ``` r
		 [vagrant@bogon ~]$ docker pull mysql:5.7
		 Error response from daemon: Get "https://registry-1.docker.io/v2/": dial tcp: lookup registry-1.docker.io on 10.0.2.3:53: server misbehaving
		 ```
		  解决："[Docker 网络问题：服务器行为异常 - 堆栈溢出 (stackoverflow.com)](https://stackoverflow.com/questions/28332845/docker-network-issue-server-misbehaving)"  重启:`vagrant reload`
	3. 虚拟机磁盘空间不足问题：
		  清理镜像： `sudo docker system prune -a --volumes --force
		  清理下载文件： `sudo rm -rf ~/Downloads/*
		  清理日志文件：`sudo rm -rf /var/log/*
		  清理不必要的日志文件：`sudo rm -rf /var/log/*.log`
		  清理临时文件：`sudo rm -rf /tmp/*
		  列出已有的镜像：`docker images`
		  删除指定镜像，替换`<IMAGE_ID>` :  `docker rmi <IMAGE_ID>
		  清理yum缓存：`sudo yum clean all`
		  
	4. 用户权限问题：（chatgpt给的解决办法）：
		 ``` r
		 出现了"permission denied"错误。这是因为当前用户（vagrant）没有足够的权限来访问Docker引擎的UNIX套接字。
		 解决这个问题的方法是将用户添加到`docker`用户组中，以便具有足够的权限来执行Docker命令。您可以按照以下步骤操作：
		 1. 确保您的用户（vagrant）已经在`docker`用户组中。运行以下命令检查：
			  `groups vagrant`
			 检查结果中是否包含`docker`。如果没有，需要执行下一步。
		
		 2. 将用户添加到`docker`用户组。运行以下命令：
			 `sudo usermod -aG docker vagrant`
			  这将将用户（vagrant）添加到`docker`用户组。
		
		 3. 注销并重新登录以使更改生效。或者，您可以运行以下命令来重新加载组成员身份：
		`newgrp docker`
		
		  4. 确保重新登录或重新加载组后，再次尝试执行`docker system prune`命令。您应该不再遇到"permission denied"错误。
		  
		请注意，添加用户到`docker`用户组允许其执行Docker命令，这可能涉及一些安全风险。请仅将可信用户添加到`docker`用户组，并谨慎管理Docker的访问权限。
		 ```
3. linux查看磁盘：
	1. 查看当前的磁盘和分区信息: `sudo fdisk -l
	2. 重新扫描磁盘设备: `sudo partprobe /dev/sda`
	3. 扩展分区：`sudo growpart /dev/sda 1`
## 安装完mysql5.7

1.   查看现有的镜像 ：`sudo docker images`
2.  启动mysql:
	 ``` r
	 sudo docker run -p 3306:3306 --name mysql \      
	-v /mydata/mysql/log:/var/log/mysql \
	-v /mydata/mysql/data:/var/lib/mysql \
	-v /mydata/mysql/conf:/etc/mysql/conf.d \
	-e MYSQL_ROOT_PASSWORD=root \
	-d mysql:5.7
	 
	  -p : 是Linux中的MySQL容器里面装的MySQL的端口号3306和Linux的端口号映射
	  -name: 给当前容器起名叫mysql
	  -v:  目录挂载：每个命令的冒号前面是linux的，后面是mysql的，这几步是做一个端口映射和log、lib、etc这几个文件的挂载挂载，挂载后去linux的对应文件夹下修改，对应的mysql下的文件也就改了
	  -d: 初始化root用户密码
	  注意：防火墙关闭：`systemctl stop firewalld `、挂在目录可以先不加、加的话要看三个目录是否存在，且里面不能有东西
	  启动不了的，购买的云服务器，去安全组开放一下端口，试试 `run mysql  --priviledged=true`
	  高版本mysql要额外设置，连接不上的搜索：mysql 运行外网主机访问
	 ```
3. 查看装在运行中的容器：`docker ps`
	1. 问题：`docker ps` 没有信息：
		1. 查看所有容器：`docker ps -a`  mysql容器炸了
		   `CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS                      PORTS     NAMES
		   `a6438c53845d   mysql:5.7   "docker-entrypoint.s…"   38 minutes ago   Exited (1) 38 minutes ago             mysql  `
		2. 查看日志分析原因：`docker logs --tail 10 -tf a643`
		   `mysqld: Can't read dir of '/etc/mysql/conf.d/'`
		   解决1(成功)：bilibili弹幕说的要把老师给的 `-v /mydata/mysql/conf:/etc/mysql \`  变成 `-v /mydata/mysql/conf:/etc/mysql/conf.d \`
		   
		   解决2---bing：
			``` r
			The error message “Can’t read dir of ‘/etc/mysql/conf.d/’” is usually caused by the fact that the directory /etc/mysql/conf.d/ does not exist or is not readable by the MySQL user. 
			You can try creating the directory and setting the correct permissions for it:
			sudo mkdir -p /etc/mysql/conf.d/
			
			And then set the correct permissions:
			sudo chown -R mysql:mysql /etc/mysql/conf.d/
			//我这里是root用户，所以为：
			[root@bogon vagrant]# sudo chown -R root:root /etc/mysql/conf.d/
			 
			You can also check if the directory exists by running:
			ls -la /etc/mysql/
			
			```
			
	2. 其他命令：
		1. 停止容器：`docker stop mysql`
		2. 删除容器：`docker rm mysql`
	3. linux连接mysql: 
	   打开SQLyog ,新建连接，虚拟机ip: `169.254.228.4` , mysql 密码：root  ![](https://i.imgur.com/gE0Qx4g.png)
		1. 进入容器内部：`docker exec -it  7e3f  /bin/bash `   (`docker ps` 查看容器id，可以只写前几位，或者容器名字)
		   ``` r
		   //查看目录结构，这是一个完整的linux目录结构，说明mysql 容器是一个完整的Linux
		   bash-4.2# ls /
		   bin   docker-entrypoint-initdb.d  home   media  proc  sbin  tmp
		   boot  entrypoint.sh               lib    mnt    root  srv   usr
		   dev   etc                         lib64  opt    run   sys   var
		  // 和MySQL相关的目录都在这
		   bash-4.2# whereis mysql
		   mysql: /usr/bin/mysql /usr/lib/mysql /usr/lib64/mysql /etc/mysql /usr/share/mysql
		   
		   bash-4.2# exit
		   ```
	   4.  修改linux下mysql 的配置文件：
		  ``` r
		[root@bogon vagrant]# cd /mydata/
		[root@bogon mydata]# cd  mysql/
		[root@bogon mysql]# ls
		conf  data  log
		[root@bogon mysql]# cd conf/
		//目前conf文件夹下是空的，创建并修改my.conf文件
		[root@bogon conf]# vi my.conf
		// i 进入insert模式，复制粘贴下面这一段
		[client]
		default-character-set=utf8
		[mysql]
		default-character-set=utf8
		[mysqld]
		init_connect='SET collation_connection = utf8_unicode_ci'
		init_connect='SET NAMES utf8'
		character-set-server=utf8
		collation-server=utf8_unicode_ci
		skip-character-set-client-handshake
		skip-name-resolve
		
		esc退出insert模式后 `:wq` 保存退出
		然后重启MySQL：
		[root@bogon conf]# docker ps
		CONTAINER ID   IMAGE       COMMAND                  CREATED             STATUS             PORTS                                                  NAMES
		7e3f5c1ce585   mysql:5.7   "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysql
		[root@bogon conf]# docker restart mysql
		mysql
		//进入容器内部
		[root@bogon conf]# docker exec -it mysql /bin/bash
		bash-4.2#
		
		这里显示失败的，rm <旧文件名>，重新来一遍
		文件名错了的：mv  <旧文件名>  <新文件名>
		-v /mydata/mysql/conf:/etc/mysql/conf.d \
		容器内部的/etc/mysql/ 和conf 关联
		来容器内部瞅一眼：
		bash-4.2# cd /etc/mysql/
		bash-4.2# ls
		conf.d  mysql.conf.d
		
		//不在/etc/mysql/ 文件夹下，在/etc/mysql/ conf.d/  文件夹下
		bash-4.2# cd conf.d/
		bash-4.2# ls
		my.conf
		bash-4.2# cat my.conf
		[client]
		default-character-set=utf8
		[mysql]
		default-character-set=utf8
		[mysqld]
		init_connect='SET collation_connection = utf8_unicode_ci'
		init_connect='SET NAMES utf8'
		character-set-server=utf8
		collation-server=utf8_unicode_ci
		skip-character-set-client-handshake
		skip-name-resolve
		
		到这mysql 可以使用了
		设置启动docker时运行MySQL
		 [root@hadoop-104 ~]# docker update mysql --restart=always
		```
#  安装Redis：
1. 下载最新版本：`docker pull redis`，老师的是5.0.5
	1. using default错误是网络问题没法下载
	   ``` r
	   mkdir -p /mydata/redis/conf  
	   touch /mydata/redis/conf/redis.conf  
	   
	   //redis服务器要设置密码或者换端口，不然容易被拿去挖矿
	   //6411是Linux的端口，映射redis的6379端口
	   docker run -p 6411:6379 --name redis -v /mydata/redis/data:/data -v/mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
	
	   闪退的：  
	   docker run  --priviledged=true  -p 6379:6379 --name redis -v /mydata/redis/data:/data -v/mydata/redis/conf/redis.conf:/etc/redis/redis.conf -d redis redis-server /etc/redis/redis.conf
	  
	   //如果命令写错了就 `docker stop  `  `docker rm  `  移除容器重新来
	   ```
2. 进入docker的Redis镜像的客户端: `docker exec -it redis redis-cli`
3. 数据持久化：因为redis数据存在内存中，旧版本的redis`docker restart redis` 之后可能存的数据就没了，新版本的默认持久化
   这里我就没有修改redis持久化 
   【Java项目《谷粒商城》】 【06:15】 https://www.bilibili.com/video/BV1np4y1C7Yf/?p=11&share_source=copy_web&vd_source=00aeec1f61d822ffad2b51fadfebd38a&t=375
``` r
//redis的挂载目录
[root@bogon conf]# pwd
 /mydata/redis/conf
 
//查看Linux的docker里面装的redis版本
[root@bogon conf]# docker exec redis redis-server --version
Redis server v=7.0.11 sha=00000000:0 malloc=jemalloc-5.2.1 bits=64 build=6f01df9fd4e71bde

```  
4. 下载 "[Releases · qishibo/AnotherRedisDesktopManager (github.com)](https://github.com/qishibo/AnotherRedisDesktopManager/releases)"  
  测试连接，连不上就关掉linux防火墙 `systemctl stop firewalld` ![](https://i.imgur.com/CeGeGB3.png)
  
# 开发工具&环境安装配置

1. 要求jdk版本在1.8及以上，下载：[Java Downloads | Oracle](https://www.oracle.com/java/technologies/downloads/#java8-windows)
``` r
//我的系统中安装的 Java 运行时环境（Java Runtime Environment，JRE）的版本。
C:\Users\KKK>java -version
java version "16.0.1" 2021-04-20
Java(TM) SE Runtime Environment (build 16.0.1+9-24)
Java HotSpot(TM) 64-Bit Server VM (build 16.0.1+9-24, mixed mode, sharing)

//我的Apache Maven 构建工具使用的 Java 版本
C:\Users\KKK>mvn -version
Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d)
Maven home: D:\environment\apache-maven-3.8.1\bin\..
Java version: 14, vendor: Oracle Corporation, runtime: D:\environment\JAVA\jdk14
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```  
2. maven配置阿里云镜像：
3. IDEA的setting--配置maven,我用的是3.8.1，老师用的3.6.1：![](https://i.imgur.com/tDDujCq.png)
	1. 插件安装：lombok、MyBatisX
4. VS Code 插件：Auto Close Tag 、Auto Rename Tag、ESLint(不行就扔)、open in browser、~~HTML Snippets~~ 、 JavaScript (ES6) code snippets、Live Server、Vetur
5. 安装git ,生成SSH，添加到github的SSH Key，我的密钥位置在："C:\Users\KKK\.ssh"  ![](https://i.imgur.com/DYfk4v6.png)
	``` r
	//生成密钥
	$ ssh-keygen -t rsa -C "ksiafor@gmail.com"
	
	
	// 查看完整密钥
	$ cat ~/.ssh/id_rsa.pub
	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCrgrQsjxheOZ8aVhop4wyBkowWz9YGG31wIfpfMWoJK4aUCgFkDuMhPWAcN0ozCY7Xg/X0f4Cv6ZAycEHPzYF317mnnsaqLNnin4VKsNmBgc4mdRelfXx9b2sdOvHOJTQthJhc6Doe42DR8Y1iQEwRtgsT0dy2l635BEsq3NyNDS7IdQT5PDa4uXiHcBK3WP7EUawcHiWj/RcWgf+de0XT4UsXVf1U65XOi8A6GE0VMUB3Zvr1kAmOh3myqz5grsqj+YropzYI4gzhq/BYwosWbMuQ1UzFp1n26xKwEhIJVqTqeB+2dcSC0Miyw9z4cbOHmx3fMypM/pfVFJbPE1zLD/yJwGMxBKk6/PaLsf39KguWOvEHrBNA/smrTi33Vpk5cPzx+GaZrth0nmOt0/GS4ZIJLYwJ6JrYslUhI4jONg2eSfQkXeMLg52hjSjZvW8w99i9a61+Z95P0Lpo6LNgwEeWACWThkbZWWVpS/CHAh6HfnE7LyNzDVrapDIyM60= ksiafor@gmail.com
   ```  
# 在github创建仓库
{ #5daf8b}


  ![](https://i.imgur.com/zhnf5y0.png)
  仓库地址："https://github.com/kkk-a11yy/Andy-mall.git" ,
  把项目克隆下来：![](https://i.imgur.com/Gcn8B39.png)
  注意spring版本，老师的是2.1.8，找不到2.1.8版本的，先用新版本创建，然后手动更改，我的最新版是3.1.1
  新建微服务模块：用spring initializr ![](https://i.imgur.com/sd0gg4U.png)
   选择微服务必要的
  ![](https://i.imgur.com/LCPgH5g.png)

