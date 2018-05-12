简介
=======

Openwrt ac 控制器(acctl)， 包含服务端和客户端两大块。支持以下特性:

* 同一网段，二层发现，自动配置ip
* 自动二层/三层发送控制信息
* 二层主动探测网络中ap，可接管其他ac控制ap
* 客户端主动链接，并保持长链接


目录
=======

ac: 服务器代码
ap: 客户端代码
lib: 共用代码
include: 共用头文件
scripts: 编译等辅助脚本


ac
================

ac 为多线程，包含以下线程:

* net_recv: 接收二层或者三层数据包
* net_netlisten: 监听tcp链接
* net_dllbrd: 二层广播
* message_travel: 定时处理收到的报文
* 主线程： 用于命令行查看系统状态

- net_dllbrd 线程广播探测报文, 报文中携带ac的地址和监听端口
- ap接收到探测报文后，发起tcp链接
- ac 将ap信息插入aphash
- ap 定时汇报状态
- ac 从数据库中提出配置，与状态比较，并发出命令
- 当tcp链接断开时，认为ap掉线
- 删除aphash记录

启动过程:

* 初始化ac控制器uuid
* 初始化epoll
* 初始化二层socket, 加入epoll
* 启动net_recv线程，收包（此时只接收二层数据包）
* 初始化三层tcp listen线程, 等待tcp 链接
* 初始化二层广播，汇报服务器地址，端口，ac uuid

ap
================

ap 为多线程，包含以下线程:

* net_recv: 接收二层或者三层数据包
* message_travel: 定时处理收到的报文
* 主线程: 状态查询


chap 二层保护
=================

编译时makefile会要求输入password， 客户端和服务端必须密码一致，用于报文确认。random 用于防止重放攻击。

1、发送broadcast报文， 生成broadcast `random0`
2、ap接收到broadcast报文, 生成`random1`, 对`数据+ random0 + password`计算`md5sum1`, 发送ap reg 报文。
3、ac收到ap reg报文，提取`md5sum1`并置原报文位置为0, 对`报文 + random0 + password`计算`md5sum2`, 如果md5sum1 与 md5sum2 不一致，则丢弃。ac 生成reg response 报文，生成`random2`, 对`数据 + random1 + password`计算`md5sum3`
4、ap 收到reg response 报文，提取`md5sum3`并置原报文位置为0, 对`报文 + random1 + password`计算`md5sum3`, 如果md5sum3 与 md5sum4 不一致，则丢弃


Openwrt ac 控制器(acctl)， 包含服务端和客户端两大块。支持以下特性:

    同一网段，二层发现，自动配置ip
    自动二层/三层发送控制信息
    二层主动探测网络中ap，可接管其他ac控制ap
    客户端主动链接，并保持长链接

目录

ac: 服务器代码 ap: 客户端代码 lib: 共用代码 include: 共用头文件 scripts: 编译等辅助脚本
ac

ac 为多线程，包含以下线程:

    net_recv: 接收二层或者三层数据包
    net_netlisten: 监听tcp链接
    net_dllbrd: 二层广播
    message_travel: 定时处理收到的报文

    主线程： 用于命令行查看系统状态

    net_dllbrd 线程广播探测报文, 报文中携带ac的地址和监听端口
    ap接收到探测报文后，发起tcp链接
    ac 将ap信息插入aphash
    ap 定时汇报状态
    ac 从数据库中提出配置，与状态比较，并发出命令
    当tcp链接断开时，认为ap掉线
    删除aphash记录

启动过程:

    初始化ac控制器uuid
    初始化epoll
    初始化二层socket, 加入epoll
    启动net_recv线程，收包（此时只接收二层数据包）
    初始化三层tcp listen线程, 等待tcp 链接
    初始化二层广播，汇报服务器地址，端口，ac uuid

ap

ap 为多线程，包含以下线程:

    net_recv: 接收二层或者三层数据包
    message_travel: 定时处理收到的报文
    主线程: 状态查询

chap 二层保护

编译时makefile会要求输入password， 客户端和服务端必须密码一致，用于报文确认。random 用于防止重放攻击。

1、发送broadcast报文， 生成broadcast random0 2、ap接收到broadcast报文, 生成random1, 对数据+ random0 + password计算md5sum1, 发送ap reg 报文。 3、ac收到ap reg报文，提取md5sum1并置原报文位置为0, 对报文 + random0 + password计算md5sum2, 如果md5sum1 与 md5sum2 不一致，则丢弃。ac 生成reg response 报文，生成random2, 对数据 + random1 + password计算md5sum3 4、ap 收到reg response 报文，提取md5sum3并置原报文位置为0, 对报文 + random1 + password计算md5sum3, 如果md5sum3 与 md5sum4 不一致，则丢弃










介绍cliprobe 用于探测网络中的wifi客户，无需用户链接，进入网络后主动探测。并根据多台ap的反馈信号，计算出设备在场地内的坐标。
核心代码:
ap等路由系统上的报文抓取（未开源）server端根据汇报数据计算client位置
对应web:
一、基于用户做统计1、用户mac地址2、设备类型3、首次到场时间4、最近到场时间5、平均每次在场时间与在场总时间6、指定时间用户活跃排名（自选默认）
二、基于场地做统计1、用户mac地址，用户到场时间，1、场地每天总人数2、平均在店时间3、在场人数在不同时间的曲线（采样时间自选）4、用户活跃区域图
三、报表基于以上统计信息，每天，每周，每月，每季度出统计表
编译执行make只生成client
make server生成服务端
运行客户端
客户端运行在任何支持monitor mode的机器上。不仅仅可以运行在路由器。检查是否支持:iw phy查看Supported interface modes:
./cliprobe USAGE: probe wlan0 mon0 server minsig debuge.g:./cliprobe wlan0 mon0 192.168.10.119 70 0minsig: 最小关心信号强度，注意没有负号
服务端
配置文件: /etc/clipos/serprobe.confap位置配置文件: /etc/clipos/ap.conf
[jianxi@Jianxi cliprobe]$ ./server/serprobe  --helpUSAGE: serprobe [options]  -a, --apconf           ap position config file (default /etc/clipos/ap.conf)  -t, --agetime          ap report age time (default 5s)  -l, --losttime         client lost age time (default 300s)  -i, --dis              sample distance  (default 1m)  -s, --sig              sample distance signal (default 53)  -f, --fac              default factory 2.5 ~ 5 (default 4.5)  -r, --drift            default drift (default 1)  -m, --minsig           min signal get in calculate (default -70)  -d, --daemon           daemon mode  -b, --debug            enable debug will auto disable daemon_mode  --help                 help info正常参数不需要修改， 长期修改请修改配置文件。正式部署时，请启动daemon 模式 serprobe -d
数据库mysql数据库, 数据库配置使用my.cnf, 使用[serprobe] section.
数据库：cliprobe 表: clipos, user
climac： 8字节存储，后6字节为mac地址
macstr: mac地址字符串表示形式
x，y，z: 坐标
time: 时间戳
first_time: 客户端首次出现时间
last_time: 客户端最近一次出现时间
CREATE TABLE IF NOT EXISTS clipos (    climac BIGINT NOT NULL,    macstr CHAR(17) NOT NULL,     x DOUBLE,     y DOUBLE,     z DOUBLE,     time TIMESTAMP);CREATE TABLE IF NOT EXISTS  user (    climac BIGINT NOT NULL,    macstr CHAR(17) NOT NULL,    first_time DATETIME NOT NULL,    last_time DATETIME NOT NULL,    primary key (climac));
存储
8 + 17 + 8 + 8 + 8 + 4 = 53B
活跃信息，每2秒一条记录
程序设计最大用户10000人
每天开业12小时
综上:
12 * 3600 / 2 * 10000 * 53 = 11,448,000,000B  (11,179,687KB, 10,917.7MB, 10.7GB)

