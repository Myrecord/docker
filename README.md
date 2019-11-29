#### 一. docker概念与使用
   [1. 容器与虚拟机有什么区别?](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [2. docker架构如何工作？](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [3. 镜像与容器](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [4. 私有仓库](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [5. 数据持久化](https://github.com/Myrecord/Docker/blob/master/README.md)
   
   [6. Dockerfile与docker commit](https://github.com/Myrecord/Docker/blob/master/README.md)
  
   [7. 网络模型](https://github.com/Myrecord/Docker/blob/master/README.md)
    
   [8. 管理容器资源](https://github.com/Myrecord/Docker/blob/master/README.md)
#### 二. docker-compose
----

#### 一. docker概念与使用
##### 1. 容器与虚拟机有什么区别？
![1.jpg](https://github.com/Myrecord/Docker/blob/master/1.jpg)
* 虚拟机是运行在宿主机操作系统之上，在宿主机操作系统上模拟宿主机的硬件、操作系统、内核用来进行隔离。继承在一台宿主机的多个虚拟机之间，内核、系统、用户空间互相不受影响，哪怕其中一台虚拟机挂掉也不会影响其他虚拟机的运行，可以做到彻底的隔离，但对于资源的开销比较大。

* 容器是在虚拟机的结构中去除虚拟机的内核层(系统)，共享宿主机的硬件资源，在宿主机的操上模拟多个用户空间，给进程提供运行环境，保护进程不被其他进程干扰，所以容器是基于用户空间进行隔离。用户空间如何隔离？每个用户空间中都有独立的UTS(主机名称)、Mount(文件系统)、IPS(通信消息队列)、PID(进程)、USER(用户)、NETWORK(网络)，这6种命名空间统称为**Namespaces**,Linux内核在3.8版本中加入了USER命名空间，所以想要使用Docker内核最低3.10

* 相比虚拟机容器隔离并不是很彻底，因为容器之间共享一个内核空间、硬件资源。假设一个容器使用的CPU为100%就会影响其他的容器。但Linux内核提供对资源限制分配的功能叫做Cgroups。它可以对系统资源进行限制分配、比如一个容器可以使用多少CPU核心、多少内存等。

##### 2. docker架构如何工作？
![2.jpg](https://github.com/Myrecord/Docker/blob/master/2.jpg)
* Docker是C/S架构模式运行,支持三种模式监听:ipv4、ipv6、Unix socket文件(默认),Docker在后台启动Docker daemon守护进程监听本地的socket文件,Docker client执行命令启动容器时Docker server默认使用https从仓库中获取，如果本地不存在则从dokcer hub中获取镜像文件,镜像文件是分层构建,底层的镜像文件为只读模式,最终将多层镜像文件汇总成一个读写层提供给容器运行并存放到本地。

##### 3. 镜像与容器
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
    -d       # 将容器放入到后台
    -t       # 创建一个tty终端
    -i       # 打开标准输入
    --name   # 自定义容器名称
    --rm     # 容器运行完后删除容器
    --hostname	 # 修改容器hostname
    --add-host	 # 添加host文件解析记录<Domin:IP>
    --dns	 # 添加dns
    -p    # 指定容器端口映射到宿主机端口<Host_port>:<Container_port>|| <Host_ip:Host_port>:<Container_port>
    -P    # 在docker中使用EXPOSE指定的端口，使用该参数会随机在宿主机中开发一个端口作为映射
    
  docker start|stop|restart <name|id>  #停止关闭重启容器
  docker exec -ti nginx:latest /bin/bash # 打开bash进入一个正在运行容器 
  docker ps｜ps -a        # 查看运行的容器或所有容器
  docler logs｜logs -f      # 查看日志或实时查看
  docker rm <name｜id> # 删除容器
  docker history <name | id> #查看容器内的历史记录
  docker port <container_name> #查看容器暴露的端口信息
  
    
   ```

##### 4. 私有仓库
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
##### 5. 数据持久化
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



##### 6.Dockerfile与docker commit
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
* Dockerfile：通过Dockerfile内部的指令用户可自定义镜像应该如何构建，dockerfile与shell脚本类似，编辑好后使用`docker build`指令来创建。
  * **指令必须为大写，并且第一行开头必须为FROM**
  * **每添加一个指令就会在镜像中生成一层，所以相似的内容尽量合并为一层减少镜像的臃肿**
  * **尽可能通过环境变量的方式在运行容器时传递参数即可，保证镜像原生**
  
  常用Dokcerfile指令：  
  ```
     FROM: 指定选择基础镜像，例如：FROM alpine:latest
    LABLE: 用于表述镜像信息，指定作者
     COPY: 将宿主机中的文件复制到image中
      ADD: 类似与COPY，但它可以使用URL方式下载软件包，如果软件包在本地并且是一个压缩包，它会将包添加到镜像中并进行解压
      ENV: 定义变量
      RUN: 运行容器内的shell命令，取决于选择什么作为基础镜像
   EXPOSE: 声明该镜像要暴露的端口，使用—P时EXPOSE定义的端口才会生效
  WORKDIR: 切换工作目录，如果使用它来切换目录，随后所有上层都会在此目录中
   VOLUME: 指定需要绑定的数据目录
  HEALTHCHECK: 健康状态检查， --interval: 检测间隔默认30s --timeout: 超时时间默认30s --retries: 失败次数默认3次
               初始状态：starting， 成功状态：healthy，失败状态：unhealthy
  CMD: 指定容器内默认启动的命令，例如：CMD ["nginx","-g","daemon off;"] 参数必须是双引号
  ENTRYPOINT: 类似CMD指令，区别在于它可以传递参数，在它之后都将被当作参数传递。若有CMD指令，CMD指令内容将被当作参数传递给它
              它可以使用-e来传递参数
  USER: 切换用户身份，切换后其他上层都将使用此用户，但要求用户必须预先创建
  ```

* 例nginx反向代理：

 **目录结构**：
```
nginx_proxy/
├── Dockerfile
└── nginx_proxy.sh
```
**Dockerfile**：
```
FROM alpine:latest		#选择alpine做基础

ENV NGINX_VERSION=1.10.3 NGINX_SRC=/usr/local/src NGINX_COMPILE=/usr/local/nginx  #定义变量

RUN CONFIG="\   #将下载包和编译全部放在一层，--virtual NAME使用虚拟编译环境，一些GCC的东西并不需要安装，编译完后掉减少镜像大小
    --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_stub_status_module" \ 
    && sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
    && apk add --no-cache curl pcre pcre-dev \
    && apk add --no-cache --virtual mypacks gcc g++ make openssl-dev zlib-dev openssl-dev \
    && mkdir -p $NGINX_SRC \
    && wget http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -P $NGINX_SRC/ \
    && tar zxvf $NGINX_SRC/nginx-$NGINX_VERSION.tar.gz -C $NGINX_SRC \
    && cd $NGINX_SRC/nginx-$NGINX_VERSION \
    && ./configure $CONFIG \
    && sed -i 's/-Werror//'g $NGINX_SRC/nginx-$NGINX_VERSION/objs/Makefile \
    && make \
    && make install \
    && apk del mypacks \
    && rm -rf $NGINX_SRC/*

COPY nginx_proxy.sh /bin/

EXPOSE 80  #若指定-P它才会起作用

HEALTHCHECK --interval=10s --timeout=3s --retries=5 CMD curl -q http://localhost/ || exit 1

CMD ["/usr/local/nginx/sbin/nginx","-g","daemon off;"] 

ENTRYPOINT ["/bin/nginx_proxy.sh"] #执行脚本并使用-e修改变量，随后CMD里都被当作参数传递给它
```
**nginx_proxy.sh**:
```
#!/bin/sh


cat > /usr/local/nginx/conf/nginx.conf << EOF 
user  ${NGINX_USER:-root};
worker_processes  ${NGINX_CPU:-4};

#error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  ${WORKER_CONNECTIONS:-65535};
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '\$remote_addr - \$remote_user [\$time_local] "\$request" '
    #                  '\$status \$body_bytes_sent "\$http_referer" '
    #                  '"\$http_user_agent" "\$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

upstream ${PROXY_NAME:-proxy} {
    server ${PROXY_IP:-127.0.0.1}:${PROXY_PORT:-8080} weight=1;
}

server {
    listen ${PORT:-80};
    server_name ${NAME:-127.0.0.1};
    location / {
	 proxy_pass http://${PROXY_NAME:-proxy};
	 proxy_set_header Host \$host;
         proxy_set_header X-Real-IP \$remote_addr;
         proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    }
 }
}
EOF

exec "$@"
```
**build Dockerfile**:
```
docker build -t nginx_proxy:v1 .  -t:指定tag名称
```
**运行容器**
```
docker run -d --name ngproxy -e PROXY_NAME=test PROXY_IP=1.1.1.1 PROXY_PORT=3321 nginx_proxy:v1

ENTRYPOINT使用-e传递参数，Dcokerfile会先执行这个脚本，ENTRYPOINT它默认会调用shell -c的来解释，
脚本最后exec $@,exec启动一个新的进程，$@是shell中表示所有参数，
在dockerfile中CMD指定的内容将被当作参数传递给了$@,所有相当于 exec nginx -g daemon off，
最后在容器内第一个进程是nginx

每次在运行这个容器时，使用-e参数就可以很方便传递参数到脚本所定义的变量了。

/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process /usr/local/nginx/sbin/nginx -g daemon off;
    8 root      0:00 nginx: worker process
    9 root      0:00 nginx: worker process
   10 root      0:00 nginx: worker process
   11 root      0:00 nginx: worker process
 9191 root      0:00 /bin/sh
 9203 root      0:00 ps

```
**获取镜像**：
```
 docker pull printsmile/nginx_proxy
```

##### 7. 网络模型
使用`docker network ls`列出docker中的网络。
```
NETWORK ID          NAME                DRIVER              SCOPE
5d6f7f55eb12        bridge              bridge              local
218c6ad306d6        host                host                local
555ae4f90af5        none                null                local
```
* Bridge：桥接网络，成对出现docker中使用的默认网络,在宿主机中生成docker0的网桥，在容器内部生成另一半
*   Host：开放网络，容器继承宿主机的网络，和宿主机共用一个Network命名空间
*   None：封闭网络，不设置任何网络，容器内部只有一个lo接口
* Container：联盟网络，两个容器使用共享一个Network命令空间容器通过lo地址可互相访问

**Host与Container只是Network命名空间共享，其他命名空间都处于隔离，通过使用`--network`选择网络模型：**

**Bridge**：
```
docker run -ti --rm  busybox:latest #默认网络是bridge所以不需要指定--network

#ifconfig 
eth0      Link encap:Ethernet  HWaddr 02:42:AC:12:00:04  #在容器内另一半虚拟网卡
          inet addr:172.18.0.4  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```
**Host**：
```
docker run -ti --rm --network host  busybox:latest #与宿主机共享一个Network命名空间

#ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:E3:20:F7:10  
          inet addr:172.18.0.1  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:123025 errors:0 dropped:0 overruns:0 frame:0
          TX packets:198618 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:5350540 (5.1 MiB)  TX bytes:1460011979 (1.3 GiB)

eth0      Link encap:Ethernet  HWaddr 00:16:3E:08:4C:CD  
          inet addr:172.17.112.50  Bcast:172.17.127.255  Mask:255.255.240.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:696605611 errors:0 dropped:0 overruns:0 frame:0
          TX packets:359099329 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1046188926127 (974.3 GiB)  TX bytes:27747472629 (25.8 GiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:926290 errors:0 dropped:0 overruns:0 frame:0
          TX packets:926290 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:70808985 (67.5 MiB)  TX bytes:70808985 (67.5 MiB)
```
**Container**：    
```
docker run -ti --rm --name test1  busybox:latest    #需要创建两个容器第二个容器网络指向第一个，
docker run -ti --name test2 --network container:test1 busybox:latest 

#ifconfig					    #两个容器的网络地址一样
eth0      Link encap:Ethernet  HWaddr 02:42:AC:12:00:04  
          inet addr:172.18.0.4  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
**None**:
```
docker run -ti --rm --network none  busybox:latest 

#ifconfig
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```
也可以自定义网络使用`docker network create [OPTIONS] NETWORK`：
```
docker network create -d bridge --subnet "192.168.0.0/24" --gateway "192.168.0.1" mynet

#docker network ls 
NETWORK ID          NAME                DRIVER              SCOPE
c820d4370c3f        bridge              bridge              local
d9d3ed304e67        harbor_harbor       bridge              local
218c6ad306d6        host                host                local
c3b5f37cacce        mynet               bridge              local
555ae4f90af5        none                null                local
```
##### 8. 管理资源管理
* 默认容器没有任何资源限制，若某个容器出现异常情况就会占用所有宿主机资源，`docker run`时提供参数来限制容器所占用的cpu、mem资源

**常用参数**：
```
--cpus:
--cpuset-cpus:

--m: 限制容器使用最大内存单位：k、b、m、g
--memory-swap: 设置容器swap大小，RAM是正数的前提下，如果swap设置为：
		-1: 使用宿主机中所有的swap空间
		 0: 未设置(unset)
		 unset: swap空间等于 2 * M
		 正数：如果swap与RAM都为正数，swap=swap-ram，如果两个值相等swap则无效
--oom-kill-disable:禁用系统出现OOM时不杀掉容器

```


