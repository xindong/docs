Dockerfile 基础篇
====

#### 什么是Dockerfile

  * Dockerfile是一种文本格式的文档，由Docker官方提供，目的是为了能够让用户通过调用一连串的指令来自动化构建容器镜像
  
#### 为什么使用Dockerfile

  * 构造容器的指令可以进行编排和调整，在构建过程中对容器内部进行优化可以合理控制容器镜像的体积

  * 构建容器镜像的过程可以在Dockerfile里回溯，方便调试和排查
  
  * 在Dockerfile中调用的指令是可以连续叠加的，Docker引擎会在上一次执行Dockerfile生成的本地容器缓存里叠加新的指令，加快了容器重构的速度

#### 如何定义Dockerfile

  * 每个Dockerfile文件都是为了构建一个特定用途的容器，推荐建立目录层级来存放每个容器的Dockerfile
  
  * Dockerfile支持把主机的目录/文件添加到容器内部，建议把需要放进容器内部的资源统一存放到对应Dockerfile所在的目录中，方便管理
  
  * 从官方docker镜像站点下载最新的linux镜像，发型版本根据自己的实际情况来选择，之后可以定义Dockerfile在此镜像之上构建应用程序的运行环境

#### Dockerfile构建Memcached容器镜像示例

   > - 系统环境：CentOS7.1
   > - 软件包：docker.x86_64
   
  * 安装docker.x86_64，运行docker服务
  
        # yum install -y docker.x86_64
        # systemctl start docker.service
      
  * 下载最新的centos容器镜像
  
        # docker pull centos:latest
        
  * 本地创建目录memcached，构建memcached容器镜像的Dockerfile与其他资源都存放于此目录
 
        # mkdir memcached
        # cd memcached
        
  * 创建名为Dockerfile的文件，并定义如下内容（容器内部安装软件后务必清理系统缓存文件，减少容器体积）
  
        FROM centos:latest
        RUN yum install -y memcached && yum clean all
        EXPOSE 11211
        ENTRYPOINT ["/usr/bin/memcached","-p","11211","-l","0.0.0.0"]
        
  * 构建容器
  
        # docker build -t memcached:latest .

  * 运行容器
        
        # docker run memcached:latest
  
