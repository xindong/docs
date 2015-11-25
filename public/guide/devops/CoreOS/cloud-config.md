使用Cloud-Config配置CoreOS服务
====

CoreOS集成了coreos-cloudinit程序，通过读取用户文件实现系统初始化配置。
> - cloud-config简单定义：可以被coreos-cloudinit读取并且执行的文件
> - CoreOS的cloud-config文档：https://coreos.com/os/docs/latest/cloud-config.html
> - cloud-config使用`YAML`文件格式
> - cloud-config内容的首行必须是`#cloud-config`（推荐）或者是`#!`（高级）
> - 若是以`#cloud-config`开头，则必须使用以下键：
    - coreos
    - ssh_authorized_keys
    - hostname
    - users
    - write_files
    - manage_etc_hosts
  
#### cloud-config配置示例
> - 主机：本地虚拟机
> - 系统环境：CoreOS Stable 最新版本
> - 系统用户：core
> - 登录模式：虚拟机控制台

  * 系统用户登录，创建文件cloud-config.yml并编辑如下内容
  
        #cloud-config
        
        # 修改主机名称
        hostname: coreos.example.com
        # 将主机名称在/etc/hosts里解析为127.0.0.1
        manage_etc_hosts: localhost
        # 此处的ssh_authorized_keys只对core用户生效
        ssh_authorized_keys:
          - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDgTNRG1JXsf4Ol/yHCp+xuv69IYrSQJ0Dj/Wjnz7eho86mu+9gmyPZb5LZ5jXaIFlBoZrSVdwI2Z4Kbaf1nj5Ykhpqr4oJg7VzXOulPTmebB6CYsYuERJCcQpbsQkhKebNWJUzXYaVEhMR5UOfaHTm4XaENYKeYNqlQwqaBjYxF+qrqTHtlGFmkGioVnrcRLTqA8AmTtv43rVSPKNVpIsxnrboaVlUrEz2jx/sZDd4yg+jasxqN5L9RBwtiHm38k1/SxvjiGAuq95W/lQTQODpoumheHJvvYp/YDCMkWl8ke7z9+HkAoJ6bxcz9UssjPaAUfueHgyYqC4svB1UMr6b test@example.com
        users:
          # 创建新用户test，系统自动创建组test
          - name: test
            # 用户密码填写哈希值而非明文
            passwd: $6$0Er.fC/M$gPfnSEEbeqxnw98pIU1rHyqk1gTQw6maLiqLsRM8Yrl1E/vO6/V3RHEpJX9SiRTvl1nJCcwXUJBFsGtfsTlT51
            # 修改test用户的ssh_authorized_keys
            ssh_authorized_keys:
              - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDgTNRG1JXsf4Ol/yHCp+xuv69IYrSQJ0Dj/Wjnz7eho86mu+9gmyPZb5LZ5jXaIFlBoZrSVdwI2Z4Kbaf1nj5Ykhpqr4oJg7VzXOulPTmebB6CYsYuERJCcQpbsQkhKebNWJUzXYaVEhMR5UOfaHTm4XaENYKeYNqlQwqaBjYxF+qrqTHtlGFmkGioVnrcRLTqA8AmTtv43rVSPKNVpIsxnrboaVlUrEz2jx/sZDd4yg+jasxqN5L9RBwtiHm38k1/SxvjiGAuq95W/lQTQODpoumheHJvvYp/YDCMkWl8ke7z9+HkAoJ6bxcz9UssjPaAUfueHgyYqC4svB1UMr6b test@example.com
            # 将test用户加入组sudo/docker
            groups:
              - sudo
              - docker
            shell: /bin/bash
        # 更新/etc/ssh/sshd_config
        write_files:
          - path: /etc/ssh/sshd_config
            permissions: 0600
            owner: root
            content: |
              # Use most defaults for sshd configuration.
              UsePrivilegeSeparation sandbox
              Subsystem sftp internal-sftp
              ClientAliveInterval 180
              UseDNS no
              PermitRootLogin no
              ChallengeResponseAuthentication no
              PasswordAuthentication no
        # 更新coreos系统服务
        coreos:
          # 禁用系统自动更新后重启，建议设置
          update:
            reboot-strategy: off
          # 更新systemd units
          units:
            # 之前更新了/etc/ssh/sshd_config，启动服务生效；若服务已启动则使用`reload`
            - name: sshd.service
              command: start
            # 设置系统时区
            - name: localtime.service
              command: start
              content: |
                [Unit]
                Description=Set Local TimeZone
                
                [Service]
                ExecStart=/usr/bin/timedatectl set-timezone Asia/Shanghai
                RemainAfterExits=yes
                Type=oneshot
            # 设置网络，假定本地网卡名称是eth0
            - name: 10-eth0.network
              content: |
                [Match]
                Name=eth0
    
                [Network]
                Address=192.168.0.100/24
                Gateway=192.168.0.1
                DNS=8.8.8.8
                DNS=8.8.4.4
  
  * 检查cloud-config.yml文件格式
    
        $ sudo coreos-cloudinit -from-file=cloud-config.yml -validate
        
  * 读取cloud-config.yml并生效
  
        $ sudo coreos-cloudinit -from-file=cloud-config.yml
