### CoreOS集群管理

CoreOS提供了分布式管理系统fleet，配合分布式K-V存储etcd构建集群，可在整个集群内批量部署/管理Docker容器。

fleet提供了可选的RestAPI管理集群和容器。

 > - fleet架构的详细介绍请参考: https://github.com/coreos/fleet/blob/master/Documentation/architecture.md
 > - etcd存储的详细介绍请参考：https://coreos.com/etcd/
 > - etcd集群规模的优化请参考: https://coreos.com/etcd/docs/2.2.1/admin_guide.html
 > - fleet API v1文档: https://github.com/coreos/fleet/blob/master/Documentation/api-v1.md
 
#### 构建集群示例

 > - 系统环境: CoreOS Stable 最新版本
 > - 配置模式: cloud-config
 > - 系统用户: core
 > - 主机1: hostname=coreos001, ipaddr=192.168.0.1
 > - 主机2: hostname=coreos002, ipaddr=192.168.0.2
 > - 主机3: hostname=coreos003, ipaddr=192.168.0.3
 
  * 系统用户登录主机，依次在每台主机上创建文件cloud-config.yml，内容如下 (将变量替换为对应值)
 
        #cloud-config
        coreos:
          etcd2:
            name: <hostname>
            data-dir: /var/lib/etcd2
            listen-peer-urls: http://0.0.0.0:2380
            listen-client-urls: http://0.0.0.0:2379
            initial-advertise-peer-urls: http://<ipaddr>:2380
            advertise-client-urls: http://<ipaddr>:2379
            initial-cluster: coreos-001=http://192.168.0.1:2380,coreos-002=http://192.168.0.2:2380,coreos-003=http://192.168.0.3:2380
          fleet:
            etcd_servers: http://localhost:2379
          units:
            - name: etcd2.service
              command: start
            - name: fleet.service
              command: start
              
  * 在主机上依次执行以下指令使配置生效
  
        # sudo coreos-cloudinit -from-file=cloud-config.yml
        
  * 检查etcd存储状态，fleet需要与etcd通讯，将集群信息存储到etcd，在任一主机上执行以下命令验证
  
        # etcdctl member list
        304287bb6c7ae05e: name=coreos-001 peerURLs=http://192.168.0.1:2380 clientURLs=http://192.168.0.1:2379
        520a72d23b13c39f: name=coreos-002 peerURLs=http://192.168.0.2:2380 clientURLs=http://192.168.0.2:2379
        ca50541499b08632: name=coreos-003 peerURLs=http://192.168.0.3:2380 clientURLs=http://192.168.0.3:2379

        # curl -L http://127.0.0.1:2379/health
        {"health": "true"}

  * 查询coreos集群状态信息，在任一主机上执行以下命令验证

        # fleetctl list-machines -l
        MACHINE					IP		METADATA
        23f81afecc6b4a7597c77776cf66a387	192.168.0.3	-
        87c716357024495eabb713c2954031bb	192.168.0.1	-
        df515c3682aa43b4ae2b7d2168e17d38	192.168.0.2	-
 
#### fleet API v1简单用例 （可选）
  > - fleet API的TCP Socket默认关闭，需要配置服务参数开启
  > - 可在任一/多台主机上启用fleet API TCP Socket
  
  * 基于CoreOS官方推荐的模式开启fleet API v1的TCP Socket
      
        # sudo mkdir /etc/systemd/system/fleet.socket.d
        # sudo echo -e "[Socket]\nListenStream=0.0.0.0:3030" > /etc/systemd/system/fleet.socket.d/30-ListenStream.conf
        # sudo systemctl stop fleet.service
        # sudo systemctl restart fleet.socket
        # sudo systemctl start fleet.service

  * 在开启TCP Socket的主机上访问fleet API v1
  
        # curl -L http://127.0.0.1:3030/fleet/v1/machines
