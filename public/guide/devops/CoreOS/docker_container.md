### CoreOS集群部署和管理Docker容器

  > - 本文的CoreOS集群基于fleet构建，详细请参考: https://github.com/xindong/docs/blob/master/public/guide/devops/CoreOS/clustering.md
  > - CoreOS集群内可以独立构建Docker Swarm，通过fleet部署的Docker容器支持无缝切换到Docker Swarm进行管理，而通过Docker Swarm部署的容器无法切换到fleet管理
  > - fleet部署/管理容器的模式类似systemd管理本地服务，通过加载unit file管理服务；加载后的unit file内容被存放到etcd存储（对集群内的所有fleet daemon可见），集群内的任一主机都可对其进行管理
  > - unit file内容格式可参考systemd服务启动文件实现，详细请参考:    https://coreos.com/fleet/docs/latest/unit-files-and-scheduling.html
  
#### fleet部署和管理容器示例

  > - CoreOS集群内的每台主机都有一个全局唯一的Hash ID，可通过本机/etc/machine-id单独获取，也可执行fleetctl list-machines -l得到全部ID
  > - 系统环境: CoreOS Stable 最新版本
  > - 系统用户: core
  > - 主机1: hostname=coreos001, ipaddr=192.168.0.1, machineid=df515c3682aa43b4ae2b7d2168e17d38
  > - 主机2: hostname=coreos002, ipaddr=192.168.0.2, machineid=23f81afecc6b4a7597c77776cf66a387
  > - 主机3: hostname=coreos003, ipaddr=192.168.0.3, machineid=87c716357024495eabb713c2954031bb
  > - 在所有主机上启动nginx服务（使用官方nginx镜像）
  
  * 使用系统用户登录主机，创建nginx.service（注意：文件名称务必不能和系统服务名称相同，比如本地定义的sshd.service在启动后会覆盖系统的sshd.service，关于系统服务可执行systemctl list-unit-files | grep .service），内容如下
  
        [Unit]
        Description=Nginx Service in Docker
        After=docker.service
        Requires=docker.service
  
        [Service]
        # this is the username who pulls a docker images
        User=core
        Restart=on-failure
        RestartSec=5
        # kill and remove the remaining container regardless of success or failure
        ExecStartPre=-/usr/bin/docker kill nginx
        ExecStartPre=-/usr/bin/docker rm nginx
        # pull the docker image before we launch a container
        # if this step fails, we will not go on
        ExecStartPre=/usr/bin/docker pull nginx:latest
        # launch a container in which nginx process exists
        ExecStart=/usr/bin/docker run --name=nginx -p 80:80 -p 443:443 nginx:latest
        ExecStop=/usr/bin/docker stop nginx
        
        # specical section of fleet
        [X-Fleet]
        # schedule this container on coreso001
        MachineID=df515c3682aa43b4ae2b7d2168e17d38
      
  * 提交unit file并且查看内容（注意：unit file被提交后无法被修改，只能删除后重新提交）
  
        $ fleetctl submit nginx.service
        $ fleetctl cat nginx.service
        $ fleectl start nginx
        $ fleetctl list-units | grep nginx.service
        UNIT		    MACHINE                 ACTIVE  SUB
        nginx.service   df515c36.../192.168.0.1	active	running
  
  * 查看nginx服务详细日志
      
        $ journalctl -u nginx.service

#### 加载配置和部署代码

  > - 方法一：下载官方的docker image后，在此镜像基础上构建Dockerfile，将用户配置和外部代码复制到镜像内部打包。
    - 优点：真正实现一次打包，多处运行
    - 缺点：每次更新用户配置或者代码都需要重新打包镜像，不支持代码和配置热更新
  > - 方法二：所有用户配置和外部代码都放在主机上，每次运行时将主机上的文件和目录映射到容器内部使用。
    - 优点：容器内部只打包了服务运行环境，可以被重复使用；用户配置和外部代码支持热更新，内容变更后不再需要重新打包镜像
    - 缺点：用户配置和外部代码需要使用版本控制工具管理，需要对文件/目录的用户权限进行限制保证安全
  > - 线上环境推荐用方法二，文件管理/备份相对会比较简单，配置/代码热更新也是一大优势
  > - 若是采用了方法二，容器内部尽量不要直接用root运行服务，而是在镜像内部和主机上创建一个相同的非特权用户（UID/用户名在主机和容器内部一致），主机上需要映射到容器内部的目录owner和group都属于此特权用户，配置和代码尽量以只读模式映射到容器内部。执行docker run
启动容器的时候加上--user指定以某个非特权用户启动进程。
