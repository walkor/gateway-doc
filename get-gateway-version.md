# 如何知道GatewayWorker版本

打开GatewayWorker/Gateway.php，在Gateway类内部VERSION常量标记了当前GatewayWorker的版本，例如下面GatewayWorker版本号为2.0.2。

如果没有找到VERSION常量，说明是1.0版本。

GatewayWorker/Gateway.php

```php
<?php
/**
 * This file is part of workerman.
 *
 * Licensed under The MIT License
 * For full copyright and license information, please see the MIT-LICENSE.txt
 * Redistributions of files must retain the above copyright notice.
 *
 * @author walkor<walkor@workerman.net>
 * @copyright walkor<walkor@workerman.net>
 * @link http://www.workerman.net/
 * @license http://www.opensource.org/licenses/mit-license.php MIT License
 */
namespace GatewayWorker;

use Workerman\Connection\TcpConnection;

use \Workerman\Worker;
use \Workerman\Lib\Timer;
use \Workerman\Autoloader;
use \Workerman\Connection\AsyncTcpConnection;
use \GatewayWorker\Protocols\GatewayProtocol;

/**
 *
 * Gateway，基于Worker开发
 * 用于转发客户端的数据给Worker处理，以及转发Worker的数据给客户端
 *
 * @author walkor<walkor@workerman.net>
 *
 */
class Gateway extends Worker
{

    /**
     * ######版本######
     * @var string
     */
    const VERSION = '2.0.2';


    ....

```
