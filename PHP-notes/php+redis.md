# php+redis

## 安装

### Windows

> 这一段文章参考[如何在windows安装php redis扩展](http://www.xiabingbao.com/php/2017/08/27/window-php-redis.html)

#### 下载redis程序

在GitHub上下载Windows版，其中早期版本可以在[这里](https://github.com/MicrosoftArchive/redis/releases)找到（推荐安装），而[这里](https://github.com/tporadowski/redis/releases)可以找到更新的版本。

#### 获取redis扩展

由于wampserver默认没有提供redis扩展，需要自己下载，在wampserver启动后，打开本地页面http://localhost，点击左下角的`phpinfo()`查看自己的版本。主要留意这三条数据：

- php version : `7.2.10`
- Architecture : `x64`
- PHP Extension Build : `API20170718,TS,VC15`

redis扩展是有两个文件的: `php_igbinary.dll`和`php_redis.dll`。

##### 选择igbinary

在这个[链接](https://windows.php.net/downloads/pecl/releases/igbinary/)找到各个版本的文件，**根据自己的参数来选择**，php版本是7.0,TS,VC15，cpu是x64，那我们可以选择2.08里面的`php_igbinary-2.0.8-7.2-ts-vc15-x64.zip`

##### 选择redis

在这个[链接](https://windows.php.net/downloads/pecl/releases/redis/)找到各个版本的文件，同样根据自己的参数来选择下载对应的文件。

#### 安装扩展

下载了上面两个文件之后，将他们解压，获取到他们各自的`*.dll`文件，将这个文件放到`\wamp\bin\php\php7.2.10\ext`中。然后，在任务栏点击wampserver图标，指向php，选择`php.ini`，在弹出的txt编辑器里添加：

```
;redis
extension=php_igbinary.dll
extension=php_redis.dll
```

#### 测试

重启wampserver，如果在`phpinfo()`中能看到redis，则说明扩展已经安装成功了。

在Windows本地安装redis的根目录里启动服务

```shell
redis-server.exe redis.windows.conf
# 收到下面的返回则表示启动成功
[11012] 12 Feb 13:07:17.941 # Creating Server TCP listening socket 127.0.0.1:6379: bind: No error
```

在php程序中测试下面的指令

```php
$redis = new Redis();                   //redis对象
$redis->connect("127.0.0.1","6379");    //连接redis服务器
$redis->set("test","Hello World");      //set字符串值
echo $redis->get("test");               //获取值
```

如果能输出`Hello World`则说明redis能正常存取数据了。



### Linux

参考[菜鸟教程](http://www.runoob.com/redis/redis-php.html)



## [redis配置](http://www.redis.net.cn/tutorial/3504.html)

在redis安装目录下的名为`redis.windows.conf`与`redis.windows-service.conf`分别对应客户端和服务器端的配置项。

可以通过`redis-cli.exe`客户端输入命令的方式查看与更改。

```shell
# 使用*获取所有配置项
redis 127.0.0.1:6379> CONFIG GET *
# 或者指定获取某一个配置项
redis 127.0.0.1:6379> CONFIG GET bind
# 给某一项配置设置值
redis 127.0.0.1:6379> CONFIG SET loglevel "notice"
```

配置说明：

1.  Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

    **daemonize no**

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

    **pidfile /var/run/redis.pid**

3. 指定Redis监听端口，默认端口为6379，作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

    **port 6379**

4. 绑定的主机地址

    **bind 127.0.0.1**

5. 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

    **timeout 300**

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

    **loglevel verbose**

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

    **logfile stdout**

8. 设置数据库的数量，默认数据库为0，可以使用SELECT `<dbid>`命令在连接上指定数据库id

    **databases 16**

9. 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合

    **save <seconds> <changes>**

    Redis默认配置文件中提供了三个条件：

    **save 900 1**

    **save 300 10**

    **save 60 10000**

    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

    **rdbcompression yes**

11. RDB文件是否需要CRC64校验, 若为yes,会在生成RDB文件后计算其CRC64并将结果追加至文件尾，同样，Redis启动Load RDB时，也会先计算该文件的CRC64并与dump时的计算结果对比。优点：可以严格保证RDB的完整性及安全性。缺点：会在dump或load时损失10%的性能。如果要最大化Redis的性能，这个配置项应该用no关掉。

     **rdbchecksum yes**

12. 指定本地数据库文件名，默认值为dump.rdb

     **dbfilename dump.rdb**

13. 指定本地数据库存放目录

     **dir ./**

14. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

     **slaveof <masterip> <masterport>**

15. 当master服务设置了密码保护时，slav服务连接master的密码

     **masterauth <master-password>**

16. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH `<password>`命令提供密码，默认关闭

     **requirepass foobared**

17. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

     **maxclients 128**

18. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

     **maxmemory <bytes>**

19. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

     **appendonly no**

20. 指定更新日志文件名，默认为appendonly.aof

     **appendfilename appendonly.aof**

21. 指定更新日志条件，共有3个可选值：     **no**：表示等操作系统进行数据缓存同步到磁盘（快）     **always**：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）     **everysec**：表示每秒同步一次（折衷，默认值）

     **appendfsync everysec**

22. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

     **vm-enabled no**

23. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

     **vm-swap-file /tmp/redis.swap**

24. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

      **vm-max-memory 0**

25. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

      **vm-page-size 32**

26. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

      **vm-pages 134217728**

27 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

     **vm-max-threads 4**

28. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

    **glueoutputbuf yes**

29. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

    **hash-max-zipmap-entries 64**

    **hash-max-zipmap-value 512**

30 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

    **activerehashing yes**

31. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

    **include /path/to/local.conf**



## redis持久化

redis支持两种持久化策略，分别是RDB和AOF。

### RDB

可以直接手动使用命令`save`立即生成一个rdb文件。请注意，**这会阻塞服务器进程**，直至rdb文件创建完毕。

RDB方式会根据用户在配置项里的` save <seconds> <changes>`来进行自动生成rdb文件，里面任意一项条件成立则触发`save`策略。当save条件被触发时，redis会通过子进程异步写盘，所以会增加内存使用。如果项目不需要通过RDB方式进行持久化，将所有save条件注释掉即可。

#### RDB的优势

- RDB是Redis数据的一个非常紧凑的单文件时间点表示。RDB文件非常适合备份。例如，您可能希望在最近24小时内每小时归档您的RDB文件，并且每天保存RDB快照30天。这使您可以在服务器发生意外时轻松恢复不同版本的数据集。
- RDB非常适合数据恢复，可以将单个压缩文件传输到远端数据中心。
- RDB最大限度地提高了Redis的性能，因为Redis父进程会专注于数据库读写操作，而备份操作则会交给子程序进行对磁盘的读写操作。

#### RDB的缺点

- 如果您需要在Redis停止工作时（例如断电后）将数据丢失的可能性降至最低，则RDB并不好。您可以配置生成RDB的不同*保存点*（例如，在对数据集进行至少五分钟和100次写入之后，但您可以有多个保存点）。但是，您通常每五分钟或更长时间创建一个RDB快照，因此如果Redis因任何原因停止工作而没有正确关闭，您会丢失最新的数据分钟。
- RDB经常需要fork才能使用子进程持久存储在磁盘上。如果数据集很大，Fork可能会非常耗时，如果数据集非常大且CPU性能不佳，可能会导致Redis停止服务客户端几毫秒甚至一秒钟。AOF也需要fork，但你可以调整你想要重写日志的频率而不需要对耐久性进行任何权衡。



### AOF

将配置项里的`appendonly`设置为`yes`开启AOF持久化，`appendfsync`项设置执行同步的时机。

#### AOF优势

- 使用AOF Redis更持久：您可以使用不同的fsync策略：根本不使用fsync，每秒执行fsync，每次查询都使用fsync。使用fsync的默认策略，每秒写入性能仍然很好（使用后台线程执行fsync，并且当没有fsync正在进行时，主线程将努力执行写入。）但是您只能丢失一秒的写入。
- AOF日志是仅附加日志，因此如果停电，也没有损坏问题。即使由于某种原因（磁盘已满或其他原因）而令文件损坏，redis-check-aof工具也能够轻松修复它。
- 当Redis太大时，Redis能够在后台自动重写AOF。重写是完全安全的，因为当Redis继续附加到旧文件时，使用创建当前数据集所需的最小操作集生成一个全新的文件，并且一旦第二个文件准备就绪，Redis会切换两个并开始附加到新的那一个。
- AOF以易于理解和解析的格式一个接一个地包含所有操作的日志。您甚至可以轻松导出AOF文件。例如，即使您使用FLUSHALL命令刷新了所有错误，如果在此期间未执行重写日志，您仍然可以保存数据集，只需停止服务器，删除最新命令，然后重新启动Redis。



#### AOF的缺点

- AOF文件通常比同一数据集的等效RDB文件大。
- 根据确切的fsync策略，AOF可能比RDB慢。一般来说，fsync设置为*每秒*性能仍然非常高，并且在fsync禁用的情况下，即使在高负载下也应该与RDB一样快。即使在写入负载很大的情况下，RDB仍能够提供有关最大延迟的更多保证。





## redis指令

[指令文档](http://www.redis.net.cn/tutorial/3501.html) [官方文档](https://redis.io/documentation)



- 获取所有key

  ```redis
  keys *
  ```

- 获取带有key的数据库信息

  ```redis
  info keyspace
  ```

- 获取数据库信息

  ```redis
  config get databases
  ```

- 切换当前数据库（*代表要切换到几号数据库）

  ```redis
  select *
  ```

- 删除**所有**数据库的key

  ```redis
  flushall
  ```

- 查看当前数据库的key数量

  ```redis
  dbsize
  ```



### php里Redis指令

#### 初始化连接

```php
$r = new Redis();
$r->connect(HOST,PORT);
```



#### 键（Key）

##### EXISTS

`EXISTS key`

检测指定key是否存在

**返回值：**若`key`存在，返回`1`，否则返回`0`。

```php
// EXISTS
$redis->set('db',"redis"); //bool(true) 
var_dump($redis->exists('db'));  # key存在 //bool(true) 
$redis->del('db');   # 删除key //int(1)
var_dump($redis->exists('db'))  # key不存在 //bool(false)
```



#### delete/unlink

`DELETE/UNLINK key1 key2...`

移除指定`key`。

若redis版本大于4.0时，可使用`unlink`对`key进行`非阻塞的异步删除。

**返回值：**移除`key`的数量。**unlink由于是异步删除，所以无返回值。**

```php
// DELETE UNLINK
$redis->delete('key1', 'key2'); /* return 2 */
$redis->delete(array('key3', 'key4')); /* return 2 */

/* 如果使用Redis >= 4.0.0 你可以使用unlink */
$redis->unlink('key1', 'key2');
$redis->unlink(Array('key1', 'key2'));
```







#### 字符串（String）

##### GET 

`GET key`

返回对应key所关联的字符串值，如果key不存在则返回`nil`（false），如果key所存储的值不是字符串，则返回一个错误，**`get`只能处理字符串值。**

```php
//GET
var_dump($redis->GET('fake_key')); #(nil) // return bool(false)
```



##### SET

`SET key value`

为指定的`key`设置字符串`value`值，若当前`key`已存在，则覆盖旧值。

**返回值：**总是返回`OK(TRUE)`，因为`SET`不可能失败。

```php
// SET
$redis->SET('apple', 'www.apple.com');#OK  // bool(true)
$redis->GET('apple');//"www.apple.com"

$redis->SET('greet_list', "yooooooooooooooooo");   # 覆盖列表类型 #OK //bool(true)
$redis->TYPE('greet_list');#string // int(1)
```



##### SETNX

`SETNX key value`

为指定的`key`设置字符串`value`值，若当前`key`已存在，**则不做任何动作**。

**返回值：**设置成功，返回`1`。设置失败，返回`0`。

```php
// SETNX
$redis->SETNX('job', "programmer");  # job设置成功 //bool(true)
$redis->SETNX('job', "code-farmer");  # job设置失败 //bool(false)
```



#### 连接（Connection）

##### SELECT

`SELECT index`

切换到指定数据库，数据库索引号用数值型表示，默认是在0号数据库

```php
// SECELT
$redis->SELECT(0); # 成功则返回1，否则无返回信息
```



#### 列表（List）

##### LPUSH

`LPUSH key value [value ...]`

将一个或多个值插入表头为`key`的列表，添加的顺序为在列表左端（头端）加入。

**返回值：**成功则返回列表的长度。若指定的`key`已存在且不是列表，则返回`false`。

```php
// LPUSH
$redis->lPush('key1', 'C'); // returns 1
$redis->lPush('key1', 'B'); // returns 2
$redis->lPush('key1', 'A'); // returns 3
/* list: [ 'A', 'B', 'C' ] */
```





#### 集合（Set）

##### SADD

`SADD key value...`

将值添加到指定`key`中。

**返回值：**若成功则返回这次添加的元素数量，如果值已存在与该集合，则返回`false`。

```php
// SADD
$redis->sAdd('key1' , 'member1'); /* 1, 'key1' => {'member1'} */
$redis->sAdd('key1' , 'member2', 'member3'); /* 2, 'key1' => {'member1', 'member2', 'member3'}*/
$redis->sAdd('key1' , 'member2'); /* 0, 'key1' => {'member1', 'member2', 'member3'}*/
```


