# 快速使用
获取 mqantserver：

	git clone https://github.com/liangdas/mqantserver

设置 mqantserver 目录到 GOPATH 后获取相关依赖：

	go get github.com/gorilla/mux
	go get github.com/gorilla/websocket
	go get github.com/streadway/amqp
	go get github.com/golang/protobuf
	go get github.com/golang/net/context
	go get github.com/gogo/protobuf
	go get github.com/opentracing/basictracer-go
	go get github.com/opentracing/opentracing-go
	go get github.com/yireyun/go-queue
	go get github.com/eclipse/paho.mqtt.golang
	go get github.com/liangdas/mqant

编译 mqantserver：

go install server
如果一切顺利，运行 bin/server 你可以获得以下输出：

	[release] mqant 1.0.0 starting up
	[debug  ] RPCClient create success type(Gate) id(127.0.0.1:Gate)
	[debug  ] RPCClient create success type(Login) id(127.0.0.1:Login)
	[debug  ] RPCClient create success type(Chat) id(127.0.0.1:Chat)
	[release] MySelfHost 172.16.8.4
	[release] WS Listen :%!(EXTRA string=0.0.0.0:3653)
	[release] TCP Listen :%!(EXTRA string=0.0.0.0:3563)

敲击 Ctrl + C 关闭游戏服务器，服务器正常关闭输出：

	[debug  ] RPCServer close success id(127.0.0.1:Chat)
	[debug  ] RPCServer close success id(127.0.0.1:Login)
	[debug  ] RPCServer close success id(127.0.0.1:Gate)
	[debug  ] RPCClient close success type(Gate) id(127.0.0.1:Gate)
	[debug  ] RPCClient close success type(Login) id(127.0.0.1:Login)
	[debug  ] RPCClient close success type(Chat) id(127.0.0.1:Chat)
	[release] mqant closing down (signal: interrupt)

# 更改web服务器文件访问本地路径

	src/webapp/module.go 中
	
	static.Handler(http.StripPrefix("/mqant/", http.FileServer(http.Dir("/work/go/mqantserver/bin"))))
	
	/work/go/mqantserver/bin 改为你下载git的对应路径

# 启动网页版本客户端
编译 mqantserver：

go install client

如果一切顺利，运行 bin/client

访问地址为：http://127.0.0.1:8080/mqant/chat/index.html

小球碰撞游戏DEMO访问地址为：http://127.0.0.1:8080/mqant/hitball/index.html

# 启动python版本客户端

执行src/client/mqtt_chat_client.py即可 需要安装paho.mqtt库,请自行百度

# Demo演示说明

	1. 启动服务器
	2. 启动网页客户端	(默认房间名,用户名)
	3. 登陆成功后就可以聊天了

# 分布式跟踪系统功能测试
[Appdash，用Go实现的分布式系统跟踪神器](http://tonybai.com/2015/06/17/appdash-distributed-systems-tracing-in-go/)

客户端访问Chat/HD_JoinChat/{msgid}时后端将会收集访问信息，通过以下地址就可以看到了
[访问地址 http://localhost:7700](http://localhost:7700)

示意图：
![示意图](https://github.com/liangdas/mqant/wiki/images/mqant_tracing.png)

# 项目目录结构

https://github.com/liangdas/mqantserver 仓库中包含了mqant框架,所用到的第三方库,聊天Demo服务端,聊天代码客户端代码

	bin		
		|-conf/server.conf			服务端配置文件
		|-public					web客户端静态文件
		|-hitball					小球碰撞游戏DEMO客户端文件
		|-console                   控制台web静态文件(还未完成)
	src
		|-client
			|-mqtt_chat_client.py 	聊天客户端 Python版本
			|-webclient.go			聊天客户端网页版本
		|-github.com                需要执行 go get 命令拉取
			|-gorilla.websocket		websocket框架
			|-liangdas.mqant			mqant框架代码
			|-streadway.amqp			rabbitmq通信框架
		|-hitball						小球碰撞游戏DEMO客户端源码
		|-server						聊天服务器Demo
			|-gate						网关模块
			|-chat						聊天模块
			|-login						登陆模块
			|-hitball					小球碰撞游戏模块
			|-tracing					分布式跟踪系统服务模块
			|-main.go					服务器启动入口
		|-sourcegraph.com			开源分布式跟踪系统Appdash源码


# 客户端快速测试
如果你需要测试其他语言的mqtt客户端，可以使用mqant提供的测试接口来测试
### tcp mqtt :
	host: mqant.com
	port: 3563
	protocol=mqtt.MQTTv31
	tcp:  tls/TLSv1
	
	如果客户端需要ca证书可以使用下面这个网站提供的
	https://curl.haxx.se/docs/caextract.html

### websocket mqtt :
	host: ws://www.mqant.com:3653/mqant
	protocol=mqtt.MQTTv31
	
### 测试协议

1. 登陆接口

		向服务器publish一条登陆消息
	
		topic:		Login/HD_Login/{msgid}
		
		message:	{"userName": "liangdas", "passWord": "Hello,anyone!"}
	
	如果topic添加了msgid,则服务器会返回一条回复消息

2. 加入聊天室

		向服务器publish一条登陆消息
	
		topic:		Chat/HD_JoinChat/{msgid}
		
		message:	{"roomName": "mqant"}
	
		服务器会广播消息给所有聊天室成员
		
		topic:		Chat/OnJoin
			
		message:	{"users": [“liangdas”]}

3. 发送一条聊天

		向服务器publish一条登陆消息
	
		topic:		Chat/HD_Say/{msgid}
		
		message:	{"roomName": "mqant","from":"liangdas","target":"*","content": "大家好!!"}
	
		服务器会广播消息给所有聊天室成员
		
		topic:		Chat/OnChat
			
		message:	{"roomName": "mqant","from":"liangdas","target":"*","msg":"大家好!!"}