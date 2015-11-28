Ansible Playbooks
====

Playbooks是Ansible的配置/部署/任务编排使用的语言。

> - Ansible modules是负责具体任务的工具，playbooks负责设计和安排如何使用这些工具
> - Ansible playbooks的内容基于YAML语法
> - Ansible playbooks的官方配置用例请参考：https://github.com/ansible/ansible-examples
> - Ansible官方文档请参考：http://docs.ansible.com/ansible/index.html

### Ansible Playbooks配置用例说明
> - 为了更好地理解Ansible playbooks的结构，此处将选用官方配置中的lamp_simple来做些简单的说明
> - 官方用例地址：https://github.com/ansible/ansible-examples/tree/master/lamp_simple

  * lamp_simple的目录层次如下，README.md包含一些简单的说明

        lamp_simple
        ├── LICENSE.md
        ├── README.md
        ├── group_vars
        │   ├── all
        │   └── dbservers
        ├── hosts
        ├── roles
        │   ├── common
        │   │   ├── handlers
        │   │   │   └── main.yml
        │   │   ├── tasks
        │   │   │   └── main.yml
        │   │   └── templates
        │   │       └── ntp.conf.j2
        │   ├── db
        │   │   ├── handlers
        │   │   │   └── main.yml
        │   │   ├── tasks
        │   │   │   └── main.yml
        │   │   └── templates
        │   │       └── my.cnf.j2
        │   └── web
        │       ├── handlers
        │       │   └── main.yml
        │       ├── tasks
        │       │   ├── copy_code.yml
        │       │   ├── install_httpd.yml
        │       │   └── main.yml
        │       └── templates
        │           └── index.php.j2
        └── site.yml
        
  * 文件hosts是inventory file，定义了需要被playbooks配置的节点信息
  
        # []内定义了组名称，一般可以把相同用途的节点放在一个组里
        [webservers]
        # 节点名称，需要能被解析为IP地址；也可以直接填写IP地址
        web3
        
        [dbservers]
        web2
        
  * 目录roles定义了节点需要被配置的角色类型，例如web，db（可以根据服务类型自定义名称）等；每个角色可以包含一份或者多份playbooks，
    此处定义了三种角色类型：common，db，web
  
        roles/
        ├── common
        │   ├── handlers
        │   │   └── main.yml
        │   ├── tasks
        │   │   └── main.yml
        │   └── templates
        │       └── ntp.conf.j2
        ├── db
        │   ├── handlers
        │   │   └── main.yml
        │   ├── tasks
        │   │   └── main.yml
        │   └── templates
        │       └── my.cnf.j2
        └── web
            ├── handlers
            │   └── main.yml
            ├── tasks
            │   ├── copy_code.yml
            │   ├── install_httpd.yml
            │   └── main.yml
            └── templates
                └── index.php.j2
  
  * 每种角色类型的目录下可以定义handlers/tasks/templates/files这4个子目录，.yml的文件就是playbook
    > - tasks目录内的playbook用作多任务编排
    > - handlers目录内的playbook定义的任务一般只有在执行变更动作的时候才被触发（比如tasks目录的playbook内的任务变更了nginx配置，
        可以触发handlers的playbook内重启nginx服务任务的执行）
    > - templates目录内定义了配置模板，在远程主机上会根据配置模板生成配置文件
    > - files目录内定义了配置文件，内容会直接同步到远程主机的配置文件
    
  * roles/common/handlers/main.yml定义了多个可被触发的任务；任务通过module来定义<br/>
    执行`ansible-doc -l`列出所有内置的module<br/>
    执行`ansible-doc [module]`获取module的选项和用法
  
        ---
        # Handler to handle common notifications. Handlers are called by other plays.
        # See http://docs.ansible.com/playbooks_intro.html for more information about handlers.
        
        # 定义任务名称，执行playbook时会显示
        - name: restart ntp
        # 调用`service`module重启ntpd
          service: name=ntpd state=restarted
        
        - name: restart iptables
          service: name=iptables state=restarted
          
  * roles/common/tasks/main.yml定义了多个相互依赖的任务（安装ntp包 -> 配置ntp -> 启动ntpd -> 检查selinux是否运行），
    任务根据文件内容从上往下顺序执行；一个playbook内定义的多个任务实际就是配置一个服务执行的相关步骤
    
  
        ---
        # This playbook contains common plays that will be run on all nodes.
        
        - name: Install ntp
          yum: name=ntp state=present
          # 可以对任务做标记（tags），运行playbook的时候可以加上-t参数执行指定tag的任务
          tags: ntp
        
        - name: Configure ntp file
          template: src=ntp.conf.j2 dest=/etc/ntp.conf
          tags: ntp
          # notify就是触发handlers目录内playbook定义的`restart ntp`任务执行
          notify: restart ntp
        
        - name: Start the ntp service
          service: name=ntpd state=started enabled=yes
          tags: ntp
        
        - name: test to see if selinux is running
          command: getenforce
          # registry把getenforce执行的结果保存到sestatus变量
          register: sestatus
          # 此处定义只有当`setstatus`值是`false`时才会执行变更
          changed_when: false
        
  * 配置模板基于[Jinja2](http://jinja.pocoo.org/)模板引擎解析，可以加上`.j2`后缀作为标识
    > - 模板引擎查找group_vars里的变量文件
    > - group_vars里存放多份变量文件时，如果存在和节点所在组名称相同的文件，在此文件中查找变量替换值
    > - group_vars/all是默认的变量文件
  
        # roles/web/templates/index.php.j2
        <html>
         <head>
          <title>Ansible Application</title>
         </head>
         <body>
         </br>
          # {{ }}内定义了变量
          # `ansible_default_ipv4.address`是远程主机的系统信息，可以通过`setup`modules获取所有系统信息
          # 具体参考官方文档：http://docs.ansible.com/ansible/setup_module.html
          <a href=http://{{ ansible_default_ipv4.address }}/index.html>Homepage</a>
         </br>
        <?php
         Print "Hello, World! I am a web server configured using Ansible and I am : ";
         echo exec('hostname');
         Print  "</BR>";
        echo  "List of Databases: </BR>";
                # Jinja2支持{% %}内定义简单的条件和循环
                # `groups`/`hostvars`是Ansible内置变量
                # `groups['dbservers']`是inventory file里定义的[dbservers]
                # `hostvars[host]`可以查询对应主机的变量值
                {% for host in groups['dbservers'] %}
                        $link = mysqli_connect('{{ hostvars[host].ansible_default_ipv4.address }}', '{{ hostvars[host].dbuser }}', '{{ hostvars[host].upassword }}') or die(mysqli_connect_error($link));
                {% endfor %}
                $res = mysqli_query($link, "SHOW DATABASES;");
                while ($row = mysqli_fetch_assoc($res)) {
                        echo $row['Database'] . "\n";
                }
        ?>
        </body>
        </html>
        
  * playbook支持`- include: another.yml`，可以写多份playbook，然后在main.yml中include其他playbook；
    任务入口的playbook始终是main.yml<br/>
    playbook支持在定义任务时使用变量，执行时在group_vars里查找变量文件<br/>
    playbook支持循环，参考：http://docs.ansible.com/ansible/playbooks_loops.html
    
  * 文件site.yml是全局playbook，定义了节点组和角色类型的对应关系
  
        ---
        # This playbook deploys the whole application stack in this site.
        
        - name: apply common configuration to all nodes
          hosts: all
          # ssh远程执行用户
          remote_user: root
          # 节点组对应的角色
          roles:
            - common
        
        - name: configure and deploy the webservers and application code
          hosts: webservers
          remote_user: root
        
          roles:
            - web
        
        - name: deploy MySQL and configure the databases
          hosts: dbservers
          remote_user: root
        
          roles:
            - db
            
  * 执行playbook
  
        $ cd lamp_simple
        $ ansible-playbook site.yml -i hosts
