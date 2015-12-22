构建私有Docker Registry (Deprecated)
====
Docker Registry可以构建私有站点对Docker Images存储和管理
> - 系统环境：CentOS7.1 x86_64
> - 软件包：docker.x86_64，docker-registry.x86_64，nginx.x86_64

#### 注意事项
> - nginx.x86_64使用的是epel源
> - docker registry api的版本是v1，基于Python实现，已不推荐使用，详见https://github.com/docker/docker-registry.git
  
#### 构建Docker Registry服务

  * 安装docker-registry.x86_64
  
        # yum install -y docker-registry.x86_64
  
  * 服务配置文件路径是/etc/sysconfig/docker-registry
  
        # The Docker registry configuration file
        # DOCKER_REGISTRY_CONFIG=/etc/docker-registry.yml

        # The configuration to use from DOCKER_REGISTRY_CONFIG file
        SETTINGS_FLAVOR=local

        # Address to bind the registry to
        REGISTRY_ADDRESS=0.0.0.0

        # Port to bind the registry to
        REGISTRY_PORT=5000

        # Number of workers to handle the connections
        GUNICORN_WORKERS=2
  
  * 存储选项配置文件是/etc/docker-registry.yml，可以选择把用户提交的容器镜像存放在支持的后端存储上。
    支持的存储类型很多，可以编辑/etc/sysconfig/docker-registry里的SETTINGS_FLAVOR选择使用特定的存储类型，默认是本地存储。
    
  * 启动docker-registry
  
        # systemctl start docker-registry
        # systecmtl enable docker-registry
  
  * 提交镜像
        
        # yum install -y docker.x86_64
        # systemctl start docker
        # docker pull centos:latest
        # docker tag centos:latest localhost:5000/centos:latest
        # docker push localhost:5000/centos:latest
        
#### 构建Docker Registry的安全验证代理

  * 安装nginx.x86_64
  
        # yum install -y nginx.x86_64
        
  * 上传认证域名的SSL证书到/etc/nginx/ssl/目录 (可选, 推荐)
  
  * 添加/etc/nginx/docker-registry.conf
  
        proxy_pass                       http://docker-registry;
        proxy_set_header  Host           $http_host;   # required for docker client's sake
        proxy_set_header  X-Real-IP      $remote_addr; # pass on real client's IP
        proxy_set_header  Authorization  ""; # see https://github.com/dotcloud/docker-registry/issues/170
        proxy_read_timeout               900;
      
  * 添加/etc/nginx/conf.d/registry-server.conf
          
        upstream docker-registry {
          server localhost:5000;
        }
        
        # uncomment if you want a 301 redirect for users attempting to connect
        # on port 80
        # NOTE: docker client will still fail. This is just for convenience
        # server {
        #   listen *:80;
        #   server_name my.docker.registry.com;
        #   return 301 https://$server_name$request_uri;
        # }
        
        server {
          listen 443;
          server_name registry.example.com;
        
          ssl on;
          ssl_certificate /etc/nginx/ssl/server.crt;
          ssl_certificate_key /etc/nginx/ssl/server.key;
        
          client_max_body_size 0; # disable any limits to avoid HTTP 413 for large image uploads
        
          # required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
          chunked_transfer_encoding on;
        
          location / {
            auth_basic            "Restricted";
            auth_basic_user_file  docker-registry.htpasswd;
            include               docker-registry.conf;
          }
        
          location /_ping {
            auth_basic off;
            include               docker-registry.conf;
          }
        
          location /v1/_ping {
            auth_basic off;
            include               docker-registry.conf;
          }
        }
        
  * 添加HTTP用户验证文件
  
        # yum install -y httpd-tools
        # htpasswd -c /etc/nginx/docker-registry.htpasswd testuser
        
  * 提交镜像
        
        # echo "127.0.0.1 registry.example.com" > /etc/hosts
        # docker login https://registry.example.com
        # docker pull centos:latest
        # docker tag centos:latest registry.example.com/centos:latest
        # docker push registry.example.com/centos:latest
