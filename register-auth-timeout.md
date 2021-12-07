~~~
Register auth timeout (*.*.*.*).
~~~

~~~
Bad request for Register service.  Request info(IP:*.*.*.*, Request Buffer:**)
~~~

~~~
Register unknown event:** IP: *.*.*.* Buffer:**
~~~

### 出现以上提示的原因

有程序向Register服务发起了socket链接，然而Register服务不允许其它程序链接。  
  

注意：客户端应该链接Gateway端口，不应该链接Register端口。  
  

Register服务端口是用于GatewayWorker内部服务注册用的，任何其它程序不应该链接这个端口，出现这个提示说明有其它外部程序链接到此端口。  
为了避免外网有程序链接到此端口，可以将start\_register.php中的监听地址改为本机内网ip，如果是单机部署(非分布式部署)可以将监听ip设置为127.0.0.1。  
这样可以避免外部程序链接此端口，避免这个提示出现。  
  

如果对方ip为127.0.0.1，则说明是本机发起的socket链接，请检查本机是否有程序链接Register服务。