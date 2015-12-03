### DDoS攻击现状

DDoS由于成本低、实施容易等特点，经过这么多年的发展，DDoS的产业链条已经发展的十分成熟了。
甚至公开的ddos服务商，例如 vdos-s 和 str3ssed 价格已经低至 50元/日 - 30元/小时。
DDoS黑色产业发展的空前壮大，攻击资源开始蔓延，依托于DDoS的攻击发生的非常频繁。
当托管机房被DDoS攻击时，只能屏蔽IP，随之而来的关联与该IP的服务也就完全停止。
对于游戏来说，如果在架构和防御策略上没有准备好，就只能停服。
而游戏停服带来的损失，不仅是损失几小时或者1天的收入。
用户失去信心而流失对游戏声誉造成的损失也是无法估量的。

这也是为什么我们要求游戏服务端开始使用[统一的Gateway](https://github.com/xindong/docs/blob/master/public/game_review/backend.md)架构的主要原因。

### 防御策略

#### 方案一：所有现有依赖IP的服务，均改为使用dns（或httpdns）等地址查询方式，不依赖固定IP的方式开始连接

只有不依赖IP，而通过地址查询来建立连接，才能够在被攻击导致IP无法正常提供服务时，灵活快速进行切换。
如果使用httpdns，还可以将迁移过程更快的反应到用户。

#### 方案二：平行扩展，使用大量IP，被攻击时快速切换

例如[统一的Gateway](https://github.com/xindong/docs/blob/master/public/game_review/backend.md)要求架构支持网关平行扩展。
我们就可以在架设时同时部署多个甚至是大量网关。当任意网关IP被攻击，我们就通过dns和地址查询服务，快速平滑切换入口至备用IP。

#### 方案三：使用高防IP

目前各大云服务商，都提供了高防IP来抵抗DDoS攻击。
我们也会将备用网关部署在这些高防IP背后。并当攻击量过大时，切换至高防背后的网关。
而高防IP服务通常价格昂贵，并且限制了端口数。因此我们[统一的Gateway](https://github.com/xindong/docs/blob/master/public/game_review/backend.md)要求架构支持只保留一个IP和端口也可以提供服务


