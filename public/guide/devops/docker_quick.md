# 3步开始使用容器开发

心动提供一个基础镜像。启动这个镜像并配置后即可 ssh 登陆 root 账号

[TOC]

## 安装开发环境客户端

	* Mac OSX 下载地址 [Docker Toolbox](https://www.docker.com/toolbox)
	* centos 下运行 `yum install -y docker` `service docker start`

* 启动开发环境

	Mac 可以通过Docker Toolbox中的工具启动 Docker 终端
	\*nix 则可以直接使用 docker 命令 
	
		`docker run -t -i -p 5022:22 -v ~/.ssh/authorized_keys:/home/centos/.ssh/authorized_keys tomasen/centos `
		
		_其中：5022 是可以自定义的任意端口，authorized_keys 是ssh的公钥 _
				 

* 通过 ssh 进入操作系统
	
	此时可以通过下面命令登入
		`ssh -v root@192.168.99.100 -p5022` 
	
	其中 192.168.99.100 是 Docker 所在的 IP (Mac 可以在Kitematic界面中的Port项下看到，\*nix则可以使用本机IP)
	
* 你已经获得了一个容器环境，ssh进入系统后可以使用 yum 安装开发工具，例如：

	`yum groupinstall -y "Development Tools"`
	`yum groupinstall -y "Additional Development"`
	

## 制作上线的镜像

* 在前述容器环境中部署应用后

	修改 supervisord.conf 配置 将业务进程加入启动项

* 使用 `docker ps` 找到当前容器ID（CONTAINER ID）
	| `docker commit <CONTAINER ID>`  | 生成镜像 |
	| `docker images` | 查看镜像列表 |
	| `docker push <IMAGE ID>` | 将镜像提交至 registry |

