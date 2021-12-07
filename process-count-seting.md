# 开启多少进程
Gateway进程使用的非阻塞式IO通讯，属于CPU密集型业务，Gateway进程数设置成与CPU核数相性能最好。即```start_gateway.php```中```$gateway->count= cpu```核数;

BusinessWorker进程中根据业务是否有阻塞式IO设置进程数为CPU核数的1倍-3倍即可。 即```start_businessworker.php```中```$worker->count = cpu```核数的1-3倍;

参见：[Workerman手册-进程数设置](https://doc.workerman.net/faq/processes-count.html)