# gateway worker 分离部署

## 什么是Gateway Worker分离部署
GatewayWorker有三种进程，Gateway进程负责网络IO，BusinessWorker进程负责业务处理，Register进程负责协调Gateway与BusinessWorker之间建立TCP长连接通讯。我们可以把Gateway BusinessWorker Register分开部署在不同的服务器上，当业务进程BusinessWorker出现瓶颈时，单独增加BusinessWorker服务器提升系统负载能力。同理，如果Gateway进程出现瓶颈，则增加Gateway服务器。而Register服务一个集群只需要部署一台服务器，Register服务只有在进程启动的时候协调Gateway与BusinessWorker建立TCP连接，集群运行起来后通讯量极低，不会成为系统瓶颈。

# 部署示例

以Applications/Todpole为例，假如需要部署三台服务器提供高可用服务。瓶颈在BusinessWorker进程，则可使用1台作为gateway服务器，另外两台做BusinessWorker服务器。（如果瓶颈在gateway进程（一般是带宽瓶颈），则可以2台gateway机器，1台BusinessWorker机器，部署方法类似）。Register服务可以部署在任意一台服务器上。


## gateway worker 分离部署扩容步骤
1、由于一个集群只需要一台服务器运行Register服务，这里选择192.168.0.1，端口是1236（端口为start_register.php中监听的端口），其它服务器中start_register.php中的代码可以注释掉。

2、将进程切分，将Gateway进程部署在一台机器上(假设内网ip为192.168.0.1)，这台服务器也运行着集群的Register服务，而BusinessWorker部署在另外两台机器上（内网ip为192.168.0.2/3）。

3、由于192.168.0.1这台机器只部署Gateway进程和Register进程，所以将该服务器上初始化BusinessWorker实例的地方注释或者删掉，避免运行BusinessWorker进程，例如

这里打开文件Applications/Todpole/start_businessworker.php，注释掉bussinessWorker初始化

```php
...
// bussinessWorker
//$worker = new BusinessWorker();
//$worker->name = 'TodpoleBusinessWorker';
//$worker->count = 4;
...
```

4、配置Gateway服务器(192.168.0.1)上start_gateway.php中的```lanIp=192.168.0.1```与本机ip一致，配置registerAddress为```192.168.0.1:1236```，start_gateway.php文件最终类似下面配置

文件Applications/Todpole/start_gateway.php
```php
<?php
use \Workerman\Worker;
use \GatewayWorker\Gateway;

// gateway
$gateway = new Gateway("Websocket://0.0.0.0:8282");
$gateway->name = 'TodpoleGateway';
$gateway->count = 4;
// ==== 注意这里配置的是本机内网ip ====
$gateway->lanIp = '192.168.0.1';
// ==== 注意这里配置的是192.168.0.1:1236 ====
$gateway->registerAddress = '192.168.0.1:1236';
$gateway->startPort = 2000;
$gateway->pingInterval = 10;
$gateway->pingData = '{"type":"ping"}';

...
```

5、由于192.168.0.2/3 两台服务器只部署BusinessWorker进程，所以将这两台服务器上的Gateway进程初始化文件注释掉或者删掉。

这里打开Applications/Todpole/start_gateway.php，注释掉gateway初始化部分

```php
<?php
use \Workerman\Worker;
use \GatewayWorker\Gateway;

// gateway
//$gateway = new Gateway("Websocket://0.0.0.0:8282");
//$gateway->name = 'TodpoleGateway';
//$gateway->count = 4;
//$gateway->lanIp = '192.168.0.1';
//$gateway->registerAddress = '192.168.0.1:1236';
//$gateway->startPort = 2000;
//$gateway->pingInterval = 10;
//$gateway->pingData = '{"type":"ping"}';

```

6、打开192.168.0.2/3两台服务器的start_businessworker.php，配置registerAddress为 192.168.0.1:1236

7、逐台启动

*至此，GatewayWorker分布式部署完毕。*

## 一些问题及解答

### 为什么将Gateway与BusinesWorker分别部署在不同的服务器上？
首先说明的是不一定非要将Gateway BusinessWorker分开部署，但是推荐分开部署，原因如下：

1、由于Gateway只负责网络IO，只要服务器带宽够用，绝大多数情况下Gateway服务器不会成为瓶颈，所以在很长时间我们只需要一台或者少数几台Gateway服务器即可。由于我们不想BusinessWorker影响到Gateway，所以将Gateway和BusinessWorker分开部署

2、BusinessWorker主要负责业务逻辑。当请求量增大时，由于可能BusinessWorker业务比较复杂，负载可能会明显升高，这时我们只要单纯增加BusinessWorker服务器即可，Gateway服务器则一般不需要变动，也就是不用通知客户端Gateway的ip列表有所变动

3、当系统BusinessWorker负载较低，需要下线服务器时，我们只需要下线BusinessWorker服务器即可，无需变动GateWay服务器，也就不会导致客户端链接因为服务器下线而断开。


### 当BusinessWorker服务器集群负载较低时，需要下线一些机器怎么实施?
只需要停止BusinessWorker的服务，运行```php start.php  stop```，然后下线即可。Gateway服务器会自动感知有BusinessWorker服务器下线，不会再将请求转发给下线的机器，整个下线过程中不影响服务质量。

### 当Gateway服务器集群负载较低时，需要下线一些机器怎么实施?
*首先还是要说明下Gateway服务器一般情况下不会成为系统瓶颈，所以一般你很长时间内Gateway服务器数量是一个稳定的值，一般一台即可*

下线Gateway服务器，首先停止服务，运行```php start.php stop```，此时会导致该服务器上已有的客户端链接断开，然后下线服务器即可。此时BusinessWorker会感知到有Gateway服务器下线，会自动断开与Gateway进程的联系。
