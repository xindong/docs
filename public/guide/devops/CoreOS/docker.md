### CoreOS集群部署和管理Docker容器

  > - 本文的CoreOS集群基于fleet构建，详细请参考: https://github.com/xindong/docs/blob/master/public/guide/devops/CoreOS/clustering.md
  > - CoreOS集群内可以独立构建Docker Swarm，通过fleet部署的Docker容器支持无缝切换到Docker Swarm进行管理，而通过Docker Swarm部署的容器无法切换到fleet管理
  > - fleet部署/管理容器的模式类似systemd管理本地服务，通过加载.service文件管理服务；加载后的.service内容被存放到etcd存储（对集群内的所有fleet daemon可见），集群内的任一主机都可对其进行管理
  > - .service内容格式可参考systemd服务启动文件实现，具体请参考: 
  
#### fleet部署和管理容器示例

  > - CoreOS集群内的每台主机都有一个全局唯一的Hash ID，可通过本机/etc/machine-id单独获取，也可执行fleetctl list-machines -l得到全部ID
  > - 系统环境: CoreOS Stable 最新版本
  > - 系统用户: core
  > - 主机1: hostname=coreos001, ipaddr=192.168.0.1, machineid=df515c3682aa43b4ae2b7d2168e17d38
  > - 主机2: hostname=coreos002, ipaddr=192.168.0.2, machineid=23f81afecc6b4a7597c77776cf66a387
  > - 主机3: hostname=coreos003, ipaddr=192.168.0.3, machineid=87c716357024495eabb713c2954031bb
  > - 在所有主机上启动nginx服务（使用官方nginx镜像）
  
  * 使用系统用户登录主机，创建nginx.service，内容如下
