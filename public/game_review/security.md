### 心动网络游戏安全评议要点

#### 客户端/前端安全

* 关键数据需要具备防内存篡改机制

	手机游戏可参考 心动安全SDK(请向运营索取) 。

* 客户端本地数据需要加密

	包括素材、设置、关卡配置文件等均需具备加密机制。

* 需要具备防破解机制

	可使用第三方工具，例如：
	* [360加固宝](http://jiagu.360.cn/)
	* [腾讯云应用加固](http://jiagu.qcloud.com/)
	* [梆梆安全](http://bangcle.com/) 
	* [爱加密](http://www.ijiami.cn)
	* [娜迦](http://www.nagain.com)
	
	_注意，防破解只是增加反向工程的难度，并不能因此忽视防内存篡改和通讯加密_

#### 服务端/后端安全

* 通讯协议需要具备防重发设计
 
	标准的做法是使用 nonce 设计。参考 [Wiki](https://en.wikipedia.org/wiki/Cryptographic_nonce) [中文](https://zh.wikipedia.org/wiki/Nonce)

* 通讯内容需加密

	加密强度和复杂度要高

* 通讯协议需具备校验通讯内容的机制

	每个数据包附带校验码，防止篡改。


