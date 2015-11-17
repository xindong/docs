### 三步开始使用容器开发

心动提供一个包含 sshd 和 supervisord 基础镜像。启动这个镜像并配置后即可 ssh 登入容器的命令行控制台，像使用一个标准操作系统一样的使用容器。

 > - 安装下载镜像速度较慢时，可以在所有 `docker` 命令中使用参数 <br/>
 	`--registry-mirror=http://reg.docker.xdapp.com:5000` <br/>
 	或者使用 [DaoCloud的加速服务](https://dashboard.daocloud.io/mirror)
 > - 本文中的docker使用和镜像制作方式仅为快速上手设计，并不见得是最佳生产实践方式

#### 开发环境

* 安装开发环境

	* Mac OSX 下载安装 [Docker Toolbox](https://www.docker.com/toolbox)
	* centos 下运行 

			yum install -y docker <br/>
			service docker start

* 启动预定义容器

	* Mac 下可以通过 [Docker Toolbox](https://www.docker.com/toolbox) 中的工具启动 Docker 终端
	* linux 中则可以直接使用 docker 命令 
	
			docker run -t -i -p 5022:22 \
			-v ~/.ssh/authorized_keys:/home/centos/.ssh/authorized_keys \
			tomasen/centos

			_其中：5022 是可以自定义的任意端口，authorized_keys 是开发者（你）的ssh公钥_

* 通过 ssh 进入容器命令行控制台
	
	容器启动后，可以通过下面命令登入

			ssh -v centos@192.168.99.100 -p5022
			_其中 192.168.99.100 是 Docker 所在的 IP (Mac 可以在Kitematic界面中的Port项下看到，\*nix则可以使用本机IP)_
	
	ssh进入控制台界面后，可以使用 yum 安装开发工具等，如常部署开发环境，例如：

			yum groupinstall -y "Development Tools"
			yum groupinstall -y "Additional Development"
	
	如果 docker 启动时就通过 `-v` 命令将本地目录绑定至容器内，就可以在容器外进行代码编辑，同时在容器内编译和调试。例如：
	
			docker run -t -i -p 5022:22 **-v ~/project/src:/src** \
			-v ~/.ssh/authorized_keys:/home/centos/.ssh/authorized_keys \
			tomasen/centos
			
			_可以将宿主设备上的`~/project/src`目录挂在至容器内的`/src`目录开始开发_

#### 制作上线镜像的方案

* 在前述容器环境中编译或部署应用后，再次通过 ssh 进入容器控制台

	修改 supervisord.conf 配置 将业务进程加入启动项

* 使用 `docker ps` 找到当前容器ID（`<CONTAINER ID>`）后，依次执行下面指令，完成提交。

	| 顺序 | 命令 | 说明 | 
	| ---- | ---- | ---- | 
	|1| `docker commit <CONTAINER ID>`  | 生成镜像 |
	|2| `docker images` | 查看镜像列表，找到 <IMAGE ID> |
	|3| `docker push <IMAGE ID>` | 将镜像提交至 registry |

* 参考 Dockerfiles
	[centos + sshd](./Dockerfiles/centos/)
	[nodejs](https://github.com/nodejs/docker-node/blob/master/0.10/Dockerfile)
	[golang](https://github.com/docker-library/golang/blob/master/1.5/Dockerfile)
