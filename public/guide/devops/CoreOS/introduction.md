CoreOS简介
====

#### 什么是CoreOS
> - CoreOS is a minimal operating system that supports popular container systems out of the box. The operating system is designed to be operated in clusters. For example, it is engineered to be easy to boot via PXE and on most cloud providers.

> - CoreOS produces, maintains and utilizes open source software for Linux containers and distributed systems. Projects are designed to be composable and complement each other in order to run container-ready infrastructure.

#### 为什么选择CoreOS部署容器
> - CoreOS是个轻量级的操作系统，整合了多种开源软件来构建容器基础架构，或者说它就是一个为了运行容器而定制的操作系统

> - 相比普通的Linux发型版本，因为用途明确，所以对系统进行了大量的定制和裁剪，保证了容器运行环境的稳定和高效

> - 集成了分布式管理系统，可以轻松构建服务集群

> - 更新周期稳定，保证容器环境可以支持新的功能

> - 系统使用双分区交替更新，更新后的系统分区如果意外损坏，可以用未更新的分区启动，降低了系统更新故障导致无法启动系统的风险
