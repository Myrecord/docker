   [一. 容器与虚拟机有什么区别?](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [二. docker架构如何工作？](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [三. 镜像与容器](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [四. 私有仓库](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [五. 数据持久化](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [六. Dockerfile与docker commti](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [七. 网络模型](https://github.com/Myrecord/Docker/blob/master/README.md)
    
   [八. 管理容器资源](https://github.com/Myrecord/Docker/blob/master/README.md)
   
----
##### 一. 容器与虚拟机有什么区别？
![1.jpg](https://github.com/Myrecord/Docker/blob/master/1.jpg)
* 虚拟机是运行在宿主机操作系统之上，在宿主机操作系统上模拟宿主机的硬件、操作系统、内核用来进行隔离。继承在一台宿主机的多个虚拟机之间，内核、系统、用户空间互相不受影响，哪怕其中一台虚拟机挂掉也不会影响其他虚拟机的运行，可以做到彻底的隔离，但对于资源的开销比较大。

* 容器是在虚拟机的结构中去除虚拟机的内核层(系统)，共享宿主机的硬件资源，在宿主机的操上模拟多个用户空间，给进程提供运行环境，保护进程不被其他进程干扰，所以容器是基于用户空间进行隔离。用户空间如何隔离？每个用户空间中都有独立的UTS(主机名称)、Mount(文件系统)、IPS(通信消息队列)、PID(进程)、USER(用户)、NETWORK(网络)，这6种命名空间统称为**Namespaces**,Linux内核在3.8版本中加入了USER命名空间，所以想要使用Docker内核最低3.10

* 相比虚拟机容器隔离并不是很彻底，因为容器之间共享一个内核空间、硬件资源。假设一个容器使用的CPU为100%就会影响其他的容器。但Linux内核提供对资源限制分配的功能叫做Cgroups。它可以对系统资源进行限制分配、比如一个容器可以使用多少CPU核心、多少内存等。

##### 二. docker架构如何工作？
![2.jpg](https://github.com/Myrecord/Docker/blob/master/2.jpg)
* Docker是C/S架构模式运行,支持三种模式监听:ipv4、ipv6、Unix socket文件(默认),Docker在后台启动Docker daemon守护进程监听本地的socket文件,Docker client执行命令启动容器时Docker server默认使用https从仓库中获取，如果本地不存在则从dokcer hub中获取镜像文件,镜像文件是分层构建,底层的镜像文件为只读模式,最终将多层镜像文件汇总成一个读写层提供给容器运行并存放到本地。

##### 三. 镜像与容器
![3.jpg](https://github.com/Myrecord/Docker/blob/master/3.jpg)
* **镜像**: 包含完整的操作系统，在最低层为bootfs、其次rootfs(系统)都为只读模式,在此之上每添加一层为可写层，最后通过联合挂载的方式构建一个读写层供容器运行，用户在容器内修改、添加等操作都在最上层完成。早期docker版本使用的**Aufs(高级多层统一文件系统)** 新版本中docker默认使用文件系统为**overla2(联合挂载)** 层级可以复用、追加。

* docker中大部分命令都与shell类似，在获取镜像时如果不指定版本，默认则拉去latest版本。
   ```
   docker pull nginx            #获取镜像
   docker images                #查看镜像
   docker image rm nginx:latest #删除镜像
   docker search nginx          #搜索镜像
   ```
* docker对镜像或容器操作使用名称或者**ID前4位**即可。

* **容器**: 容器就好比进程一样，但不同的是容器有自己独立的命名空间与系统隔离，应用程序是装载在容器内的。官方建议一个容器只运行一个进程，而容器内的第一个进程ID应该为应用程序的进程ID(busybox默认的进程是sh，当sh执行后容器就会退出，官方提供的nginx镜像默认在容器内启动第一个ID为nginx的ID)，使用`docker inspect [NAME|ID]`查看容器和镜像的详细信息。
   ```
   nginx:
        "Cmd": [
                "nginx",                   #nginx容器默认启动的指令
                "-g",
                "daemon off;"
            ]
   busybox:
        "Cmd": [
                "sh"                       #busybox容器默认启动的指令
            ]     
   
   docker run busybox:latest    #运行busybox容器后因为sh结束容器就会结束
   docker run -d nginx:latest   #运行nginx容器守护进程为nginx的进程   
   ```
* docker run常用参数与指令：
   ```
    -d             # 将容器放入到后台
    -t             # 创建一个tty终端
    -i             # 打开标准输入
    --name         # 自定义容器名称
    --rm           # 容器运行完后删除容器
    
  docker start|stop|restart <name|id>  #停止关闭重启容器
  docker exec -ti nginx:latest /bin/bash # 打开bash进入一个正在运行容器 
  docker ps｜ps -a        # 查看运行的容器或所有容器
  docler logs｜logs -f      # 查看日志或实时查看
  docker rm <name｜id> # 删除容器
  docker history <name | id> #查看容器内的历史记录
  
    
   ```

##### 四. 私有仓库
* 仓库是镜像的集合，docker官方提供一个本地的私有仓库**docker-registry**，实际中工作中很多镜像都需要定制，推送到本地仓库来维护。如果没有ssl认证，修改/etc/docker/daemon.json，之后通过`docker info`查看。外有还有一些开源的仓库[**harbor**](https://github.com/goharbor/harbor)，harbor通过web界面管理docker镜像并且还包含用户权限设置、搜索镜像等功能。
```
{
  "registry-mirrors": ["https://test.registry.com"]， #修改默认获取镜像仓库地址
  "insecure-registries": ["test.registry.com"] # 取消ssl认证
{
```
* 如果私有仓库设置有用户认证使用。注：**默认如果后面不写地址docker默认登录docker hub**。
```
docker login -u <username> -p <password> <registry_address>  #登录仓库
docker logout   #退出仓库
```

* 一个仓库包含不同版本的镜像,例如：`printsmile/nginx:1.10.1`、`printsmile/nginx:1.9` 前面代表仓库名称后面是nginx的版本信息，另外一种方式`test.image.com/printmsile/nginx.:1.10.1`，通过第三方仓库pull，push时需要指定服务器地址，而test.image.com就服务器地址，修改镜像名称推送到私有仓库，通过使用`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]` 
例如：
```
   docker tag nginx:latest test.registry.com/printsmile/nginx:1.10.1 #修改nginx:latest名称
   docker push test.registry.com/printsmile/nginx:1.10.1 #推送到私有仓库
```
* 除了私有仓库来存储镜像，docker还提供倒入倒出的方式：
```
   docker save nginx:latest -o nginx.tar #倒出镜像
   docker load -i nginx.tar  #倒入镜像
```
##### 五. 数据持久化
* 很多有状态的服务需要保存数据比如Mysql、Redis，默认容器产生的数据会随着容器删除而被删除，docker中提供对数据持久保存的方式，容器中的目录可与宿主机中的目录进行绑定，在宿主机对数据修改或者删除时容器也会随之而改变，另外容器被删除后数据将会保留。
* 在创建容器时指定 **-v**参数，可以设置容器内数据目录的权限，多个容器也可以同时共享挂载一个数据盘，复用。使用`docker inspect <name|ID>`查看容器Mounts字段。

共享数据盘：
```
    -v：挂载本地目录或文件与容器绑定。(-v可指定多次，前面指定本地目录，后面为容器内的目录)
    
 docker run -d --name web -v /data/www/html:/data nginx:latest 
    "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/www/html",
                "Destination": "/data",            
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ]
        
 docker run -ti --name test1 -v /data/www/html:/data busybox:latest
    "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/www/html",           #两个容器使用宿主机上同一个目录共享数据
                "Destination": "/data",                  
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
```
复制数据卷--volumes-from：
   ```
      docker run -ti --name test2 --volumes-from web busybox:latest   #复制一个已存在的容器数据卷
      "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/www/html",
                "Destination": "/data",
                "Mode": "",
                "RW": false,
                "Propagation": "rprivate"
            }
        ],

   ```
数据卷权限：
   ```
      docker run -ti --name test1 -v /data/www/html:/data:ro busybox:latest  #容器内的/data目录为只读模式
      在容器内删除数据就会报错： rm: can't remove 'aa': Read-only file system
      "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/www/html",
                "Destination": "/data",
                "Mode": "ro",                #这里的mode为ro只读
                "RW": false,
                "Propagation": "rprivate"
            }
        ],

   ```



##### 六.Dockerfile与docker commit
* docker commit：基于容器制作镜像，它会将当前容器重新打包为新的镜像，比如当容器内修改了内容、或者要保存容器当前的状态.(**但使用docker commit会保留上一个容器中所有的历史记录**)
```
    docker commit：
               -a：指定作者
               -c：调用Dockerfile中的指令
               -m：指定修改信息
               -p：暂停容器
               
    echo "Hello Docker" > /usr/share/nginx/html/index.html  #在容器内修改nginx页面显示的内容
    docker commit -a "test" -m "修改index文件" nginx:latest new_nginx #生成新的镜像
 ```
 ```
    docker history new_nginx  #查看历史记录
    
    IMAGE            CREATED          CREATED BY                                      SIZE   COMMENT
    8a40b5598543   3 minutes ago      nginx -g daemon off;                            120B   修改index文件**
    540a289bab6c     3 weeks ago      /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
    <missing>        3 weeks ago      /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B                  
    <missing>        3 weeks ago      /bin/sh -c #(nop)  EXPOSE 80                    0B                  
    <missing>        3 weeks ago      /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B                 
    <missing>        3 weeks ago      /bin/sh -c set -x     && addgroup --system -…   57MB                
    <missing>        3 weeks ago      /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B                  
    <missing>        3 weeks ago      /bin/sh -c #(nop)  ENV NJS_VERSION=0.3.6        0B                  
    <missing>        3 weeks ago      /bin/sh -c #(nop)  ENV NGINX_VERSION=1.17.5     0B                  
    <missing>        4 weeks ago      /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
    <missing>        4 weeks ago      /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
    <missing>        4 weeks ago      /bin/sh -c #(nop) ADD file:74b2987cacab5a6b0…   69.2MB   
 ```
* Dockerfile：通过Dockerfile内部的指令用户可自定义镜像应该如何构建，通过环境变量的方式传递给容器运行显然比较灵活。
  * 



##### 七. docker中的网络模型
* docker支持多种网络模式使用docker info命令可以查看，默认有三种bridge、host、none如果不指定，使用bridge作为默认的网络，在安装完docker后会创建一个docker0的网桥，docker0不仅是一个虚拟网卡在容器内部也充当交换机。容器创建后，会自动创建**一对网卡**，一端在容器内部，一端在物理机中，并且生成在物理机中的一端虚拟网卡接口都被插在docker0网桥中，通过使用brctl show命令查看。
* bridge：容器之间通信网络接口都连接到docker0网桥中，要想外部访问容器，就需要进行DNAT模式，docker会在iptables中自动创建转发规则，这种模型显然降低带宽的质量，但在测试环境比较适合。
* host：共享宿主机的网络，容器之间访问将不在使用docker0，而是与宿主机使用
同一个IP地址，在容器内开放的端口相当于开发宿主机的端口
* none：不设置任何网络模型，如果启动容器不需要网络可指定，容器内部只有一个lo接口。
* 另外一种网络模型容器之间也可以共享网络，两个容器使用同一个网络命名空间容器通过lo地址可互相访问，但文件系统、主机名称并不共享。

