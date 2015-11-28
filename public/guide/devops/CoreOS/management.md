使用Ansible管理CoreOS
====

> - 官方教程请参考：https://coreos.com/blog/managing-coreos-with-ansible/
> - CoreOS标准的Init程序和配置平台是systemd，任何服务都应当参照systemd的规范部署
> - CoreOS Systemd官方文档请参考：https://coreos.com/using-coreos/systemd/

### Managing CoreOS With Ansible演示
> - 系统环境：CoreOS Stable Current
> - 系统用户：core
> - 目标主机：hostname=coreos.example.com，ipaddr=192.168.0.1，
              disk1=/dev/sda（系统盘），disk2=/dev/sdb（数据盘）
> - 内容的首行注释表示文件的路径

#### CoreOS安装python环境

* 登录目标主机，在系统用户的~/.ssh/authorized_keys里添加用户主机的ssh public key

* 用户主机安装ansible

        $ sudo pip install ansible
        $ ansible --verison

* 用户主机上创建目录`ansible`
  
        $ mkdir ~/ansible

* 添加inventory file `hosts`，内容如下
    
        # ~/ansible/hosts
        
        [coreos]
        coreos.example.com
        
        # 为coreos组中的节点定义Ansbile参数
        [coreos:vars]
        ansible_ssh_user=core
        ansible_python_interpreter="PATH=/home/core/bin:$PATH python"
        
* 创建目录`roles`，并安装`defunctzombie.coreos-bootstrap`

        $ mkdir ~/ansible/roles
        $ ansible-galaxy install defunctzombie.coreos-bootstrap -p ~/ansible/roles
        
* 创建`bootstrap.yml`，内容如下:
        
        ---
        # ~/ansible/bootstrap.yml
        
        - hosts: coreos
          roles:
            - bootstrap
            
* 在目标主机上执行bootstrap playbook，CoreOS上通过pypy提供python环境

        $ ansible-playbook -i ~/ansible/hosts ~/ansible/bootstrap.yml
        
* 目标主机的bootstrap完成后，系统用户目录下新增了bin和pypy目录

* 用户主机上检查目标主机的状态

        $ ansible -i ~/ansible/hosts coreos.example.com -m ping
        $ ansible -i ~/ansible/hosts coreos.example.com -a 'uname -a'
    
#### 加固CoreOS系统
  > - 首次安装后即可使用CoreOS部署容器服务，官方已经优化过系统参数。建议根据实际情况来加固系统安全
  > - 以下演示使用Ansible加固CoreOS的sshd服务（强烈推荐）
  
* 用户主机创建目录`roles/common`，可以把基础系统的相关任务都归类到这个目录的playbook

        $ mkdir -p ~/ansible/roles/common/{tasks,handlers,files}
        
* 创建playbook`roles/common/tasks/sshd.yml`，内容如下

        ---
        # ~/ansible/roles/common/tasks/sshd.yml
        
        - name: Configure sshd file
          # `src`对应的文件或者目录必须位于`roles/common/files`里
          copy: src=sshd_confg dest=/etc/ssh/sshd_config
            owner=root group=root mode=0600
          # `reload-sshd`在handlers里被定义
          notify: reload-sshd
          tags: sshd
        
        - name: Start sshd service
          service: name=sshd state=started enabled=yes
          tags: sshd
          
* 创建playbook`roles/common/tasks/main.yml`，内容如下

        ---
        # ~/ansible/roles/common/tasks/main.yml
        
        # main.yml没有包含实际的任务，只是用作其他playbook的入口  
        - include: sshd.yml
    

* 创建playbook`roles/common/handlers/main.yml`，内容如下
  
        ---
        # ~/ansible/roles/common/handlers/main.yml
        
        - name: reload sshd
          service: name=sshd state=reloaded
          
* 创建配置文件`roles/common/files/sshd_config`，内容如下

        # Use most defaults for sshd configuration.
        UsePrivilegeSeparation sandbox
        Subsystem sftp internal-sftp
        ClientAliveInterval 180
        UseDNS no
        # 禁止root帐号的ssh登录（强烈建议）
        PermitRootLogin no
        # 禁止密码登录（强烈建议）
        ChallengeResponseAuthentication no
        PasswordAuthentication no
        # 只允许core用户的ssh登录（建议，可选）
        AllowUsers core

          
* 用户目录创建playbook`ansible/site.yml`，内容如下

        ---
        # ~/ansible/site.yml
        
        # 将角色common绑定到coreos节点组
        - hosts: coreos
          roles:
            - common

* 执行playbook`ansible/site.yml`，使目标主机上的服务生效（-t表示只运行对应tag的任务；-D表示显示文件内容差异；-C表示模拟运行，不会实际更改目标主机）<br/>
  <font color="green">建议每次执行前都先在参数上加上-DC</font>

        $ ansible-playbook -i ~/ansible/hosts ~/ansible/site.yml -t sshd -DC
        $ ansible-playbook -i ~/ansible/hosts ~/ansible/site.yml -t sshd
        
#### 调整CoreOS系统服务
  > - docker damon默认的socket文件是/var/run/docker.sock，只允许在本地使用Docker Remote API
  > - 以下演示使用Ansible通过systemd开启docker的TCP Socket
  
* CoreOS上默认的Docker Socket在`/usr/lib/systemd/system/docker.socket`中定义，需要调整ListenStream属性

* 用户主机上创建playbook`roles/common/tasks/docker.yml`（docker是CoreOS默认自带的服务，可以把它归类为基础系统服务），内容如下

        ---
        # ~/ansible/roles/common/tasks/docker.yml
  
        # 强烈不推荐直接更改/usr/lib/systemd/system/docker.socket
        # 应该在/etc/systemd/system/docker.socket.d/里定义用户配置
        # systemd unit规范请参考：http://www.freedesktop.org/software/systemd/man/systemd.unit.html
        - name: Create directory /etc/systemd/system/docker.socket.d/
          file: path=/etc/systemd/system/docker.socket.d
            owner=root group=root state=directory
          tags: docker
      
        - name: Configure docker socket in /etc/systemd/system/docker.socket.d/10-ListenStream.conf
          copy: content="[Socket]\nListenStream=0.0.0.0:2375\n"
            dest=/etc/systemd/system/docker.socket.d/10-ListenStream.conf
            owner=root group=root mode=0644
          # systemd规定服务修改配置后必须执行`systemctl daemon-reload`
          notify: systemctl daemon-reload
          tags: docker
  
* 在playbook`roles/common/handlers/main.yml`中添加如下内容

        - name: systemctl daemon-reload
          command: /usr/bin/systemctl daemon-reload
    
* 在playbook`roles/common/tasks/main.yml`中添加`- include: docker.yml`

* 执行playbook`ansible/site.yml`
  
        $ ansible-playbook -i ~/ansible/hosts ~/ansible/site.yml -t docker -DC
        $ ansible-playbook -i ~/ansible/hosts ~/ansible/site.yml
        $ ansible -i ~/ansible/hosts coreos.example.com -m shell -a 'sudo systemctl stop docker.service && sudo systemctl restart docker.socket && sudo systemctl start docker.service'

* 在目标主机上验证Docker TCP Socket

        $ docker -H tcp://192.168.0.1:2375 info

#### CoreOS挂载文件系统
  > - 推荐CoreOS使用LVM卷动态调整分区
  > - 建议安装CoreOS的服务器本地磁盘应该分为两种：系统盘和数据盘，非系统生成的数据应该全部写入数据盘内
  > - 以下演示使用Ansible通过systemd挂载文件系统
  
* 用户主机上创建playbook`roles/common/tasks/mount.yml`，内容如下

        ---
        # ~/ansible/roles/common/tasks/mount.yml
        
        # 将数据盘/dev/sdb加入LVM卷组VG0
        - name: vgcreate vg0 /dev/sdb
          lvg: pvs=/dev/sdb vg=VG0 state=present
          tags: mount
          
        # 创建100G的LVM分区/dev/VG0/LV0
        - name: lvcreate -L 100G -n LV0 VG0
          lvol: lv=LV0 vg=VG0 size=100G state=present
          tags: mount
      
        # 使用xfs格式化LVM分区，如果已经存在文件系统则不操作
        # force=true会强制执行格式化
        - name: mkfs.xfs /dev/VG0/LV0
          filesystem: dev=/dev/VG0/LV0 fstype=xfs
          tags: mount
        
        # 创建systemd mount file
        # /dev/VG0/LV0将被挂载到/srv/www
        # systemd mount的规范请参考：http://www.freedesktop.org/software/systemd/man/systemd.mount.html
        - name: Configure srv-www.mount
          copy: src=srv-www.mount dest=/etc/systemd/system/srv-www.mount
            owner=root group=root mode=0644
          tags: mount
          
        # 启动服务挂载文件系统
        - name: Start srv-www.mount
          service: name=srv-www.mount state=started enabled=yes
          tags: mount
            
* 创建文件`roles/common/files/srv-www.mount`，内容如下

        # ~/ansible/roles/common/files/srv-www.mount
        [Mount]
        What=/dev/VG0/LV0
        Where=/srv-www
        Type=xfs
        Options=noatime,nobarrier
        
        [Install]
        WantedBy=multi-user.target
        
* 在playbook`roles/common/tasks/main.yml`添加`- include: mount.yml`

* 执行playbook`ansible/site.yml`

        $ ansible-playbook -i ~/ansbile/hosts ~/ansible/site.yml -t mount -DC
        $ ansible-playbook -i ~/ansbile/hosts ~/ansible/site.yml -t mount
        
#### 后记
> - 本文因为篇幅有限，目前只提供了部分Ansible配置示例。未来还会不定期更新。
> - 有兴趣的用户可以在官方和社区获得更多的帮助。
