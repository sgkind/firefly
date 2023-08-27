简介
======


安装
=====
一、下载当前稳定版的Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
二、添加可执行权限
```
sudo chmod +x /usr/local/bin/docker-compose
```
注：
如果安装后执行docker-compose失败，则需要添加软连接
```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
三、测试安装是否成功
```
$ docker-compose --version
docker-compose version 1.24.1, build 4667896b
```

Compose命令说明
========
docker-compose命令的基本格式是
```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

命令选项
* -f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
* -p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名
* --x-networking 使用Docker的可插拔网络后端特性
* --x-network-driver DRIVER 指定网络后端的驱动，默认为bridge
* --verbose 输出更多调试信息
* -v, --version 打印版本并退出

1.build

构建（重新构建）项目中的服务容器
```
docker-compose build [options] [SERVICE...]
```

* --force-rm 删除构建过程中的临时容器
* --no-cache 构建镜像过程中不使用cache
* --pull 始终尝试通过pull来获取更新版本的镜像

2.config

验证Compose文件格式是否正确，若正确则显示配置，若格式错误显示错误原因

3.down

此命令将会停止up命令所启动的容器，并移除网络

4.exec

进入指定的容器

5.help

获得一个命令的帮助

6.images

列出Compose文件中包含的镜像

7.kill

通过发送SIGKILL信号来强制停止服务容器
```
docker-compose kill [options] [SERVICE...]
```

支持通过-s参数来指定发送的信号，例如通过如下指令发送SIGINT信号

8.logs
```
docker-compose logs [options] [SERVICE...]
```

查看服务容器的输出。默认情况下，docker-compose 将对不同的服务输出使用不同的颜色来区分。可以通过 --no-color 来关闭颜色.

9.pause

暂停一个服务容器
```
docker-compose pause [SERVICE...]
```

10.port

打印某个容器端口所映射的公共端口
```
docker-compose port [options] SERVICE PRIVATE_PORT
```
选项：
* --protocol=proto 指定端口协议，tcp(默认值)或udp
* --index=index 如果同一服务存在多个容器，指定命令对象容器的序号（默认为1）

11.ps

列出项目中目前所有容器
```
docker-compose ps [options] [SERVICE...]
```
选项：
* -q 只打印容器的ID信息

12.pull

拉取服务依赖的镜像
```
docker-compose pull [options] [SERVICE...]
```
选项：
* --ignore-pull-failures 忽略拉取镜像过程中的错误

13.push

推送服务依赖的镜像到Docker镜像仓库

14.restart

重启项目中的服务

```
docker-compose restart [options] [SERVICE...]
```
选项：
* -t, --timeout TIMEOUT 指定重启前停止容器的超时(默认为10s)

15.rm

删除所有(停止状态的)服务容器。推荐先执行`docker-compose stop`命令来停止容器
```
docker-compose rm [options] [SERVICE...]
```
选项：
* -f, --force 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项
* -v 删除容器所挂载的数据卷

16.run

在指定服务上执行一个命令
```
docker-compose run [options] [-p PORT...] [-e KEY=VAL...] SERVICE [COMMAND] [ARGS...]
```
选项：
* -d 后台运行容器
* --name NAME 为容器指定一个名字
* --entrypoint CMD 覆盖默认的容器启动指令
* -e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量
* -u, --user="" 指定运行容器的用户名或者uid
* --no-deps 不自动启动关联的服务容器
* --rm 运行命令后自动删除容器，d模式下将忽略
* -p, --publish=[] 映射容器端口到本地主机
* --service-ports 配置服务端口并映射到本地主机
* -T 不分配伪tty, 意味着依赖tty的指令将无法运行

16.scale

设置指定服务运行的容器个数
```
docker-compose scale web=3 db=2
```
将启动3个容器运行web服务，2个容器运送db服务
* -t, --timeout TIMEOUT 停止容器时候的超时(默认为10s)

17.start

启动已经存在的服务容器
```
docker-compose start [SERVICE...]
```

18.stop

停止已经处于运行状态的容器，但不删除它。通过`docker-compose start`可以再次启动这些容器
```
docker-compose stop [options] [SERVICE...]
```
* -t, --timeout TIMEOUT 停止容器时候的超时(默认为10s)

19.top

查看各个服务容器内运行的进程

20.unpause

恢复处于暂停状态中的服务
```
docker-compose unpause [SERVICE...]
```

21.up

尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。
```
docker-compose up [options] [SERVICE...]
```
选项：
* -d 在后台运行服务容器
* --no-color 不使用颜色来区分不同的服务的控制台输出
* --no-deps 不启动服务所链接的容器
* --force-recreate 强制重新创建容器，不能与--no-recreate同时使用
* --no-recreate 如果容器已经存在了，则不重新创建，不能与--force-recreate同时使用
* --no-build不自动构建缺失的服务镜像
* -t, --timeout TIMEOUT 停止容器时候的超时(默认10s)

Docker Compose配置文件
=======================
官方配置文件示例
```
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```

service定义包含应用于为该服务启动的每个容器的配置，就像传递命令行参数一样`docker container create`。同样，网络和卷的定义类似于`docker network create`和`docker volume create`.

配置选项

1.build

build可以指定Dockerfile所在文件夹的路径，在up启动前执行构建镜像的任务。
```
build: /path/to/build/dir
```
也可以是相对路径
```
build: ./dir
```
设定上下文目录，然后以该目录为准指定Dockerfile
```
build:
    content: ../
    dockerfile: path/of/Dockerfile
```
如果context中有指定的路径，并且可以选定Dockerfile和args。那么arg这个标签，就像Dockerfile中的ARG指令，它可以在构建过程中指定环境变量，但是在构建成功后取消
```
version: '3'
services:
    webapp:
        build:
            context: ./dir
            dockerfile: Dockerfile-alternate
            args:
                buildno: 1
```
与ENV不同的是，ARG可以为空值
```
args:
    - buildno
    - password
```
如果要指定image以及build，选项格式为
```
build: ./dir
image: webapp:tag
```
这会在./dir目录生成一个名为webapp和标记为tag的镜像

2.context
```
build:
    context: ./dir
```
context选项可以是Dockerfile的文件路径，也可以是链接到git仓库的url

当提供的值是相对路径时，它被接卸为相对于撰写文件的路径，此目录也是发送到Docker守护进程的context

3.dockerfile
```
build:
    context: .
    dockerfile: Dockerfile-alternate
```
使用此dockerfile文件来构建，必须指定构建路径

4.args

添加构建参数，这些参数是仅在构建过程中可访问的环境变量
首先，在Dockerfile中指定参数：
```
ARG buildno
ARG password
```

然后指定build下的参数，可以传递映射或列表
```
build:
    context: .
    args:
        buildno: 1
        password: secret
```
或
```
build:
    context: .
    args:
        - buildno=1
        - password=secret
```
指定构建参数时可以省略该值，在这种情况下，构建时的值默认构成运行环境中的值（**不理解**）
```
args:
    - buildno
    - password
```

5.cache_from

编写缓存解析镜像列表（**什么意思**）
```
build:
    context: .
    cache_from:
        - alphine:latest
        - corp/web_app:3.14
```

6.labels

使用Docker标签将元数据添加到生成的镜像中，可以使用数组或字典

建议使用反向DNS标记来防止签名与其他软件所使用的签名冲突
```
build:
    context: .
    labels:
        com.example.description: "Accounting webapp"
        com.example.department:  "Finance"
        com.example.label-with-empty-value: ""
```
或
```
build:
    context: .
    labels:
        - "com.example.description=Accounting webapp"
        - "com.example.department=Finance"
        - "com.example.label-with-empty-value"
```

7.shm_size

设置容器 /dev/shm 分区的大小，值为表示字节的整数值或表示字符的字符串

```
build:
    context: .
    shm_size: '2gb'
```
或
```
build:
    context: .
    shm_size: 10000000
```

8.target

根据对应的Dockerfile构建指定stage
```
build:
    context: .
    target: prod
```

9.cap_add、cap_drop

添加或删除容器功能
```
cap_add:
    - ALL
    
cap_drop:
    - NET_ADMIN
    - SYS_ADMIN
```

10.command

覆盖容器启动后默认执行的命令
```
command: bundle exec thin -p 3000
```
该命令也可以是一个列表，方法类似于dockerfile:
```
command: ["bundle", "exec", "thin", "-p", "3000"]
```

11.configs

使用服务configs配置为每个服务赋予相应的访问权限，支持两种不同的语法.

12.cgroup_parent

可以为容器选择一个可选的父cgroup
```
cgroup_parent: m-executor-abcd
```

13.container_name

为自定义的容器指定一个名称，而不是使用默认的名称
```
container_name: my-web-container
```

14.credential_spec

为托管服务账号配置凭据规范，此选项仅适用于Windows容器服务

15.deploy

指定与部署和运行服务相关的配置

16.devices

设置映射列表，与Docker客户端的--device参数类似:
```
devices:
    - "/dev/ttyUSB0:/dev/ttyUSB0"
```

17.depends_on

此选项解决了启动顺序的问题

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。

指定服务之间的依赖关系，有两种效果
* docker-compose up以依赖顺序启动服务
* docker-compose up SERVICE自动包含SERVICE的依赖性

```
version: '3'
services:
    web:
        build: .
        depends_on:
            - db
            - redis
    redis:
        image: redis
    db:
        image: postgres
```

18.dns

自定义DNS服务器，与--dns具有一样的用途，可以是单个值或列表
```
dns: 8.8.8.8
dns:
    - 8.8.8.8
    - 9.9.9.9
```

19.dns-search

自定义DNS搜索域，可以是单个值或列表
```
dns_search: example.com
dns_search:
    - dc1.example.com
    - dc2.example.com
```

20.tmpfs

挂载临时文件目录到容器内部，与 run 的参数一样效果，可以是单个值或列表
```
tmpfs: /run
tmpfs:
    - /run
    - /tmp
```

21.entrypoint

在 Dockerfile 中有一个指令叫做 ENTRYPOINT 指令，用于指定接入点。在 docker-compose.yml 中可以定义接入点，覆盖 Dockerfile 中的定义：
```
entrypoint: /code/entrypint.sh
```
entrypoint也可以是一个列表
```
entrypoint:
    - php
    - d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

22.env_file

从文件中添加环境变量。可以是单个值或是列表 
如果已经用 docker-compose -f FILE 指定了 Compose 文件，那么 env_file 路径值为相对于该文件所在的目录

但environment环境中设置的变量会覆盖这些值，无论这些值未定义还是为None
```
env_file: .env
```
或
```
env_file:
    - ./common.env
    - ./apps/web.env
    - /opt/secrets.env
```
环境配置文件env_file中的声明每行都是以VAR=VAL格式，其中#开头的被解析为注释而被忽略

23.environment

添加环境变量，可以使用数组或字典。与上面的 env_file 选项完全不同，反而和 arg 有几分类似，这个标签的作用是设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置，这是与 arg 最大的不同。 \

一般 arg 标签的变量仅用在构建过程中。而 environment 和 Dockerfile 中的 ENV 指令一样会把变量一直保存在镜像、容器中，类似 docker run -e 的效果

```
environment
    RACK_ENV: development
    SHOW: 'true'
    SESSION_SECRET:
```
或
```
environment:
    - RACK_ENV=development
    - SHOW=true
    - SESSION_SECRET
```

23.expose

暴露端口，但不映射到宿主机，只被连接的服务访问。这个标签与 Dockerfile 中的 EXPOSE 指令一样，用于指定暴露的端口，但是只是作为一种参考，实际上 docker-compose.yml 的端口映射还得 ports 这样的标签

```
expose:
    - "3000"
    - "8000"
```

24.external_links

链接到 docker-compose.yml 外部的容器，甚至 并非 Compose 项目文件管理的容器。参数格式跟 links 类似

> 在使用Docker过程中，会有许多单独使用 docker run 启动的容器的情况，为了使 Compose 能够连接这些不在docker-compose.yml 配置文件中定义的容器，那么就需要一个特殊的标签，就是 external_links，它可以让Compose 项目里面的容器连接到那些项目配置外部的容器（前提是外部容器中必须至少有一个容器是连接到与项目内的服务的同一个网络里面）。

```
external_links:
    - redis_1
    - project_db_1: mysql
    - project_db_1: postgresql
```

25.extra_hosts

添加主机名的标签，就是往 /etc/hosts 文件中添加一些记录，与 Docker 客户端 中的 --add-host 类似
```
extra_hosts:
    - "somehost: 162.242.195.82"
    - "outherhost: 50.31.209.229"
```

26.healthcheck

用于检查测试服务使用的容器是否正常

```
healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost"]
    interval: 1m30s
    timeout: 10s
    retries: 3
    start_period_ 40s
```
test 必须是字符串或列表，如果它是一个列表，第一项必须是 NONE，CMD 或 CMD-SHELL ；如果它是一个字符串，则相当于指定CMD-SHELL 后跟该字符串，如下：
```
test: ["CMD", "curl", "-f", "http://localhost"]

test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]

test: curl -f https://localhost || exit 1
```

如果需要禁用镜像的所有检查项目，可以使用 disable:true,相当于 test:["NONE"]

```
healthcheck:
    disable: true
```

27.image

从指定的镜像中启动容器，可以是存储仓库、标签以及镜像 ID

```
image: redis
image: ubuntu:14.04
image: example-registry.com:4000/postgresql
```

28.isolation

指定容器的隔离技术。Windows上支持的值有：default、process和hyperv，linux上仅支持default。

29.links

链接到其它服务的中的容器，可以指定服务名称也可以指定链接别名（SERVICE：ALIAS)，与 Docker 客户端的 --link 有一样效果，会连接到其它服务中的容器

```
web:
    links:
        - db
        - db:database
        - redis
```

使用的别名将会自动在服务容器中的/etc/hosts里创建，例如：
> 172.12.2.186  db</br> 172.12.2.186  database</br> 172.12.2.187  redis</br>

相应的环境变量也将被创建

30.logging

配置日志服务
```
logging:
    driver: syslog
    options:
        syslog-address: "tcp://192.168.0.42:123"
```
该driver值是指定服务器的日志驱动程序，默认值为json-file，与--log-driver选项一样，可选值为json-file,syslog,none。对于可选值，可以使用options指定日志记录中的日志记录选项
```
driver: "syslog"
options:
    syslog-address: "tcp://192.168.0.42:123"
```

31.network_mode

网络模式，用法类似于Docker客户端的--net选项，格式为：service:[service name]
```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

32.networks

加入指定网络
```
services:
    some-service:
        networks:
            - some-network
            - other-network
```

33.aliases

同一网络上的其他容器可以使用服务器名称或别名来连接到其他服务的容器
```
services:
    some-service:
        networks:
            some-network:
                aliases:
                    - alias1
                    - alias3
            other-network:
                aliases:
                    - alias2
```
例:
```
version: '2'

services:
    web:
        build: ./web
        networks:
            - new
            
    worker:
        build: ./worker
        networks:
            - legacy
            
    db:
        image: mysql
        networks:
            new:
                aliases:
                    - database
            legacy:
                aliases:
                    - mysql
networks:
    new:
    legacy:
```
相同的服务可以在不同的网络有不同的别名

34.ipv4_address、ipv6_address

为服务的容器指定一个静态IP地址
```
version: '2.1'

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
      - subnet: 172.16.238.0/24
      - subnet: 2001:3984:3989::/64
```

35.PID

```
pid: "host"
```
将PID模式设置为主机PID模式，可以打开容器与主机操作系统之间的共享PID地址空间。使用此标志启动的容器可以访问和操作宿主机的其他容器，反之亦然。

36.ports

映射端口

**SHORT语法**

可以使用HOST:CONTAINER的方式指定端口，也可以指定容器端口（选择临时主机端口），宿主机会随机映射端口.
```
ports:
    - "3000"
    - "3000-3005"
    - "8000:8000"
    - "9090-9091:8080-8081"
    - "49100:22"
    - "127.0.0.1:8001:8001"
    - "127.0.0.1:5000-5010:5000-5010"
    - "6060:6060/udp"
```

**LONG语法**

LONG语法支持SHORT语法不支持的附加字段
* target: 容器内的端口
* published: 公开的端口
* protocol: 端口协议(tcp/udp)
* mode: 通过host用在每个节点还是哪个发布的主机端口或使用ingresss用于集群模式端口进行平衡负载。
```
ports:
    - target: 80
    - published: 8080
    - protocol: tcp
    - mode: host
```

37.secrets

通过secrets为每个服务授予相应的访问权限

**SHORT语法**
```
verion: "3.1"
services:
    redis:
        image: redis:latest
        deploy:
            replicas: 1
        secrets:
            - my_secret
            - my_other_secret
secrets:
    my_secret:
        file: ./my_secret.txt
    my_other_secret:
        external: true
```

**LONG语法**

LONG语法可以添加其他选项
* source: secret名称
* target: 在服务任务容器中需要装载在/run/secrets/中的文件名称，如果source未定义，那么默认为此值
* uid&gid: 在服务的任务容器中拥有该文件的 UID 或 GID。如果未指定，两者都默认为 0
* mode: 以八进制表示法将文件装载到服务的的任务容器中/run/secrets/的权限，例如，0444代表可读。
```
version: "3.1"
services:
    redis:
        image: redis:latest
        deploy:
            replicas: 1
        secrets:
            - source: my_secret
              target: redis_secret
              uid: '103'
              gid: '103'
              mode: 0444
secrets:
    my_secret:
        file: ./my_secret.txt
    my_other_secret:
        external: true
```

38.security_opt

为每个容器覆盖默认的标签。简单说来就是管理全部服务的标签，比如设置全部服务的 user 标签值为 USER （**不理解**)

```
security_opt:
    - label:user:USER
    - label:role:ROLE
```

39.stop_grace_period

在发送SIGKILL之前指定stop_signal，如果试图停止容器（如果它没有处理SIGTERM或只当的任何停止信号），则需要等待的时间

```
stop_grace_period: 1s
stop_grace_period: 1m30s
```

40.stop_signal

设置另一个信号来停止容器。在默认情况下使用SIGTERM来停止容器。设置另一个信号可以使用stop_signal标签
```
stop_signal: SIGUSR1
```

41.sysctls

在容器中设置的内核参数，可以为数组或字典
```
sysctls:
    net.core.somaxconn: 1024
    net.ipv4.tcp_syncookies: 0
    
sysctls:
    - net.core.somaxconn=1024
    - net.ipv4.tcp_syncookies=0
```

42.ulimits

覆盖容器的默认限制，可以单一地将限制值设为一个整数，也可以将soft/hard限制指定为映射
```
ulimits:
    nproc: 65535
    nofile:
        soft: 20000
        hard: 40000
```

43.userns_mode

Disables the user namespace for this service, if Docker daemon is configured with user namespaces.
```
userns_mode: "host"
```

44.volumes

挂载一个目录或者一个已存在的数据卷容器，可以直接使用HOST:CONTAINER这样的格式，或者使用HOST:CONTAINER:ro这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。
```
version: "3.2"
services:
    web:
        image: nginx:alphine
        volumes:
            - type:volume
              source: mydata
              volume:
                  nocopy: true
            - type: bind
              source: ./static
              target: /opt/app/static
    db:
        image: postgres:latest
        volumes:
            - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
            - "dbdata:/var/lib/postgresql/data"
volumes:
    mydata:
    dbdata:
```

数据卷的格式可以是下面多种形式:
```
volumes:
    # 只是指定一个路径，Docker会自动创建一个数据卷(这个路径是容器内部的)
    - /var/lib/mysql
    
    # 使用绝对路径挂载数据卷
    - /opt/data:/var/lib/mysql
    
    # 以Compose配置文件为中心的相对路径作为数据卷挂载到容器
    - ./cache:/tmp/cache
    
    # 使用用户的相对路径
    - ~/configs:/etc/configs/:ro
    
    # 已经存在的命令的数据卷
    - datavolume:/var/lib/mysql
```
如果不适用宿主机的路径，可以指定一个volume-driver
```
volume_driver: mydriver
```

**SHORT语法**

可以选择在主机(HOST:CONTAINER)或访问模式(HOST:CONTAINER:ro)上指定路径

可以在主机上挂载相对路径，该路径相对于正在使用的Compose配置文件的目录进行扩展。相对路径应始终以.或..开头
```
volumes:
    # Just specify a path and let the Engine create a volume
    - /var/lib/mysql
    
    # Specify an absolute path mapping
    - /opt/data:/var/lib/mysql
    
    # Path on the host, relative to the Compose file
    - ./cache:/tmp/cache
    
    # User-relative path
    - ~/configs:/etc/configs/:ro
    
    # Named volume
    - datavolume:/var/lib/mysql
```

**LONG语法**
* type: 安装类型，可以为volume、bind或tmpfs
* source: 安装源，主机上用于绑定安装的路径或定义在顶级volumes密钥中卷的名称不适用于tmpfs类型安装
* target: 卷安装在容器中的路径
* read_only: 标志将卷设置为只读
* bind: 配置额外的绑定选项
* propagation: 用于绑定的传播模式
* volume: 配置额外的卷选项
* nocopy: 创建卷时禁止从容器复制数据的标志
* tmpfs: 配置额外的tmpfs选项
* size: tmpfs的大小，以字节为单位
```
version: "3.2"
services:
    web:
        image: nginx:alpine
        ports:
            - "80:80"
        volumes:
            - type: volume
              source: mydata
              target: /data
              volume:
                  nocopy: true
            - type: bind
              source: ./static
              target: /opt/app/static
networks:
    webnet:
    
volumes:
    mydata:
```

45.volumes_from
从其他容器或者服务挂载数据卷，可选的参数是ro或rw，前者表示可读，后者表示对数据卷是可读可写的（默认情况为可读可写的）
```
volumes_from:
    - service_name
    - service_name:ro
    - container:container_name
    - container:container_name:rw
```

46.用于服务、集群以及堆栈文件的卷

在使用服务，群集和 docker-stack.yml文件时，请记住支持服务的任务（容器）可以部署在群集中的任何节点上，并且每次更新服务时都可能是不同的节点。

在缺少指定源的命名卷的情况下，Docker为支持服务的每个任务创建一个匿名卷。关联的容器被移除后，匿名卷不会保留。

如果希望数据持久存在，请使用可识别多主机的命名卷和卷驱动程序，以便可以从任何节点访问数据。或者，对该服务设置约束，以便将其任务部署在具有该卷的节点上。

下面一个例子，Docker Labs中votingapp示例的docker-stack.yml文件中定义了一个称为 db 的服务。它被配置为一个命名卷来保存群体上的数据，并且仅限于在节点上运行。下面是来自该文件的部分内容：db postgres manager
```
version: "3"
services:
    db:
        image: postgres:9.4
        volumes:
            - db-data:/var/lib/postgresql/data
        networks:
            - backend
        deploy:
            placement:
                constraints: [node.role == manager]
```

47.restart

默认值为 no ，即在任何情况下都不会重新启动容器；当值为 always 时，容器总是重新启动；当值为 on-failure 时，当出现 on-failure 报错容器退出时，容器重新启动。
```
restart: "no"
restart: "always"
restart: "on-failure"
restart: "unless-stopped"
```

48.其他选项

关于标签：cpu_shares、cpu_quota、 cpuse、domainname、hostname、 ipc、 mac_address、privileged、 read_only、 shm_size、stdin_open、tty、 user、 working_dir

上面这些都是一个单值的标签，类似于使用 docker run 的效果
```
cpu_shares: 73
cpu_quota: 50000
cpuset: 0,1

user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged:true

read_only: true
shm_size:64M
stdin_open:true
tty:true
```

49.持续时间单位

某些配置选项如check的子选项interval以及timeout的设置格式支持的单位有us、ms、s、m以及h

50.指定字节值

某些选项如build的子选项shm_size，支持的单位是b,k,m以及g，或kb，mb和gb。

51.extends

这个标签可以扩展另一个服务，扩展内容可以是来自在当前文件，也可以是来自其他文件，相同服务的情况下，后来者会有选择地覆盖原有配置
```
extends:
    file: common.yml
    service: webapp
```

其他
======
1.docker-compose中运行ubuntu
```
ubuntu:
    image: ubuntu:latest
    restart: always
    tty: true
    command: '/bin/bash'
```