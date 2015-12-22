构建Private Docker Registry 2.0
====
Docker Registry是容器镜像的存储站点，提供上传镜像/下载镜像/搜索镜像等功能
> - 系统环境：CoreOS Stable Current
> - 系统用户：core

#### 注意事项
> - 本教程完全使用容器构建Private Docker Regitry 2.0
> - docker registry 2.0基于Golang实现，目前是推荐版本，详见
https://github.com/docker/distribution
> - docker registry 2.0支持用redis缓存信息

#### 构建Private Docker Registy
  
  * 下载源码目录
    
        $ git clone https://github.com/docker/distribution.git
    
  * 基于源码目录中的Dockerfile打包镜像
      
        $ cd distribution
        $ docker build -t registry:v2 .
      
  * 下载官方redis镜像并启动服务
    
        $ docker pull redis:latest
        $ docker run -d --name=redis redis:latest
  
  * 编辑docker registry 2.0配置文件config.yml，内容如下（请自行替换相关变量）
    
        version: 0.1
        log:
          level: debug
          fields:
            service: registry
            environment: development
          hooks:
            - type: mail
              disabled: true
              levels:
                - panic
              options:
                smtp:
                  addr: <mail.example.com:25>
                  username: <mailuser>
                  password: <mailpass>
                  insecure: true
                from: <noreply@example.com>
                to:
                  - <someone@example.com>
        storage:
            delete:
              enabled: true
            cache:
                blobdescriptor: redis
            filesystem:
                rootdirectory: /var/lib/registry
            maintenance:
                uploadpurging:
                    enabled: false
        http:
            addr: :5000
            debug:
                addr: localhost:5001
            headers:
                X-Content-Type-Options: [nosniff]
        redis:
          addr: redis:6379
          pool:
            maxidle: 16
            maxactive: 64
            idletimeout: 300s
          dialtimeout: 10ms
          readtimeout: 10ms
          writetimeout: 10ms
        notifications:
            endpoints:
                - name: local-5003
                  url: http://localhost:5003/callback
                  headers:
                     Authorization: [Bearer <an example token>]
                  timeout: 1s
                  threshold: 10
                  backoff: 1s
                  disabled: true
                - name: local-8083
                  url: http://localhost:8083/callback
                  timeout: 1s
                  threshold: 10
                  backoff: 1s
                  disabled: true
        health:
          storagedriver:
            enabled: true
            interval: 10s
            threshold: 3
  
  * 启动docker registry 2.0，将镜像文件存储在/data/docker-registy-v2（可以考虑挂载单独分区）
        
        $ docker run -d --name=docker-registry --link redis -v ~/config.yml:/etc/docker/registry/config.yml -v /data/docker-registry-v2:/var/lib/registry registy:v2
        
  * 下载官方nginx镜像
  
        $ docker pull nginx:latest
        
  * 编辑nginx配置文件，SSL证书和htpasswd验证文件请自行创建
  
        $ mkdir -p nginx/conf.d
        $ cat << EOF > nginx/conf.d/docker-registry-v2.conf       
        upstream docker-registry {
          server docker-registry:5000;
        }
        
        ## Set a variable to help us decide if we need to add the
        ## 'Docker-Distribution-Api-Version' header.
        ## The registry always sets this header.
        ## In the case of nginx performing auth, the header will be unset
        ## since nginx is auth-ing before proxying.
        map $upstream_http_docker_distribution_api_version $docker_distribution_api_version {
          'registry/2.0' '';
          default registry/2.0;
        }
        
        server {
          listen 443 ssl;
          server_name registry-v2.example.com;
        
          # SSL
          ssl_certificate /etc/nginx/conf.d/example.crt;
          ssl_certificate_key /etc/nginx/conf.d/example.key;
        
          # Recommendations from https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html
          ssl_protocols TLSv1.1 TLSv1.2;
          ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
          ssl_prefer_server_ciphers on;
          ssl_session_cache shared:SSL:10m;
        
          # disable any limits to avoid HTTP 413 for large image uploads
          client_max_body_size 0;
        
          # required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
          chunked_transfer_encoding on;
        
          location /v2/ {
            # Do not allow connections from docker 1.5 and earlier
            # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
            if (\$http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*\$" ) {
              return 404;
            }
        
            # To add basic authentication to v2 use auth_basic setting.
            #auth_basic "Registry realm";
            #auth_basic_user_file /etc/nginx/conf.d/nginx.htpasswd;
            auth_basic off;
        
            ## If $docker_distribution_api_version is empty, the header will not be added.
            ## See the map directive above where this variable is defined.
            add_header 'Docker-Distribution-Api-Version' $docker_distribution_api_version always;
        
            proxy_pass                          http://docker-registry;
            proxy_set_header  Host              \$http_host;   # required for docker client's sake
            proxy_set_header  X-Real-IP         \$remote_addr; # pass on real client's IP
            proxy_set_header  X-Forwarded-For   \$proxy_add_x_forwarded_for;
            proxy_set_header  X-Forwarded-Proto \$scheme;
            proxy_read_timeout                  900;
          }
        }
        EOF

  * 启动nginx服务
  
        $ docker run -d --name=nginx -p 443:443 -v ~/nginx/conf.d:/etc/nginx/conf.d --link docker-registry nginx:latest
        
  * 验证docker registry 2.0
  
        $ curl -L -v https://registry-v2.example.com/v2/
        
  * docker registry 2.0 API更多用例请参考 https://github.com/docker/distribution/blob/master/docs/spec/api.md
  
        
