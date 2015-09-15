
AWS使用注意事项

## 安全

* 不要遗失秘钥，AWS不可重置登陆密码或秘钥。如果遗失，按下面思路找回数据
  * 将EC2制作成临时AMI和快照数据卷（如有）
  * 创建新的EC2基于该临时AMI并挂载数据卷（如有）

* 每个新的业务类型需要建立新的VPC子网以便安全隔离

## 技巧

* AMZ的操作系统中开启epel的yum repo
  只需要修改 `/etc/yum.repos.d/epel.repo` 文件 : enable=1 即可
* 启用root用户登陆(并不推荐)
  除了修改 /etc/ssh/sshd_config 中 PermitRootLogin yes 之外，需要修改 /root/.ssh/authorized_keys 并删除第一行的文字部分

