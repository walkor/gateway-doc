# 关于client_id

client_id固定为20个字符的字符串，用来全局标记一个socket连接，每个客户端连接都会被分配一个全局唯一的client_id。

client_id不能自定义，由GatewayWorker自动生成。

如果client_id对应的客户端连接断开了，那么这个client_id也就失效了。当这个客户端再次连接到Gateway时，将会获得一个新的client_id。也就是说client_id和客户端的socket连接生命周期是一致的。

client_id一旦被使用过，将不会被再次使用，也就是说client_id是不会重复的，即使分布式部署也不会重复。

只要有client_id，并且对应的客户端在线，就可以调用```Gateway::sendToClient($client_id, $data)```等方法向这个客户端发送数据。