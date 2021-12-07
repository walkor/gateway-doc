## 提示
GatewayWorker提供的所有接口都是支持分布式调用的，所以业务代码不需要任何更改，直接就可以分布式部署。

# 如何分布式GatewayWorker
GatewayWorker通过Register服务来建立划分集群。同一集群使用相同的Register服务ip和端口，即Gateway 和 businessWorker的注册服务地址(```$gateway->registerAddress``` ```$businessworker->registerAddress```)指向同一台Register服务。

Gateway BusinessWorker Register之间如何工作的，请看[原理](principle.md)。

## 分布式部署的关键步骤

 1、一个集群只需要一台服务器作为Register服务，用于进程启动时协调Gateway与BusinessWorker之间的建立连接通讯，其它服务器可以删掉start_register.php文件或者注释掉里面的代码。（Register服务本身通讯量极低，一般仅在进程启动时通讯，所以Register服务本身不会成为瓶颈，运行过程中即使Register服务服务器暂时挂掉，也不会对外网服务造成影响，所以Register服务一般不需要做高可用）
 2、将Gateway 和 businessWorker的注册服务地址(registerAddress)设置成统一的Register服务地址，也就是步骤1选择的Register服务所在服务器的ip和端口。
 3、设置Gateway启动脚本(一般是start_gateway.php)中的```lanIp```与当前服务器内网ip一致

## 部署示例
假如需要部署三台服务器(192.168.1.1-3)提供高可用服务。

1、选择一台服务器运行统一的Register服务(该服务器同时也运行Gateway进程和BusinessWorker进程)，这里选择192.168.1.1，Register服务的端口假设为1236（实际端口请打开start_register.php查看），其它服务器上的Register服务代码start_register.php可以删除或者注释掉。

2、配置三台服务器start_gateway.php start_businessworker.php中的registerAddress为'192.168.1.1:1236'。

3、分别配置三台服务器start_gateway.php中的```lanIp```为当前服务器的内网ip(192.168.1.1-3)。

4、逐台启动，分布式部署完毕。

**注意事项及说明：**

1、注意多机部署时以下端口不要被服务器防火墙屏蔽（不知道服务器防火墙如何配置的请自行搜索资料学习）：

①、Register服务监听的端口要可以被其它内网服务器访问（外网访问可以屏蔽）；

②、start_gateway.php中如果```$gateway->startPort=2300; $gateway->count=4;```，则2300 2301 2302 2303四个端口需要被设置成能被其它服务器访问，也就是起始端口```$gateway->startPort```到```$gateway->startPort + $gateway->count - 1```这 ```$gateway->count```个端口要设置成能被其它内网服务器访问。

2、如果多机部署服务器不在一个局域网，部署时ip参数可以使用外网ip(注意云服务器一般没有外网网卡，所以阿里云、腾讯云等云服务器一般无法外网部署集群)，对应端口防火墙应该设置成能被外网服务器访问。

3、三台GatewayWorker机器都运行了Gateway进程和Worker进程，客户端连接上任意一台GatewayWorker的Gateway端口即通讯，开发。

4、为了方便前端接入和扩容，可以在Gateway前加一层DNS、LVS等负载均衡策略（不熟悉DNS LVS的请自行搜索资料学习）。

5、如果服务器不够用可以使用同样的方法增加服务器

6、如果需要下线服务器，直接stop对应服务器即可。由于Gateway进程维护着客户端连接，当服务器下线时，对应服务器的客户端会掉线一次。如何做到下线机器不影响用户参考下一节。


