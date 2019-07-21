
Q: 我正在写一个unix server程序，不是daemon，经常需要在命令行上重启它，绝大多数时候工作正常，但是某些时候会报告"bind: address in use"，于是重启失败。
A: Andrew Gierth
server程序总是应该在调用bind()之前设置SO_REUSEADDR套接字选项。至于TIME_WAIT状态，你无法避免，那是TCP协议的一部分。
Q: 如何避免等待60秒之后才能重启服务
A: Erik Max Francis
使用setsockopt，比如

```
int option = 1;
if (setsockopt ( masterSocket, SOL_SOCKET, SO_REUSEADDR, &option, sizeof(option) ) < 0)
{
   die( "setsockopt" );
}
```

Q: 编写 TCP/SOCK_STREAM 服务程序时，SO_REUSEADDR到底什么意思？
A: 这个套接字选项通知内核，如果端口忙，但TCP状态位于 TIME_WAIT ，可以重用端口。如果端口忙，而TCP状态位于其他状态，重用端口时依旧得到一个错误信息，指明"地址已经使用中"。如果你的服务程序停止后想立即重启，而新套接字依旧使用同一端口，此时 SO_REUSEADDR 选项非常有用。必须意识到，此时任何非期望数据到达，都可能导致服务程序反应混乱，不过这只是一种可能，事实上很不可能。
一个套接字由相关五元组构成，协议、本地地址、本地端口、远程地址、远程端口。SO_REUSEADDR 仅仅表示可以重用本地本地地址、本地端口，整个相关五元组还是唯一确定的。所以，重启后的服务程序有可能收到非期望数据。必须慎重使用 SO_REUSEADDR 选项。
Q: 在客户机/服务器编程中(TCP/SOCK_STREAM)，如何理解TCP自动机 TIME_WAIT 状态？
A: W. Richard Stevens <1999年逝世，享年49岁>
下面我来解释一下 TIME_WAIT 状态，这些在<>中2.6节解释很清楚了。

```
MSL(最大分段生存期)指明TCP报文在Internet上最长生存时间，每个具体的TCP实现都必须选择一个确定的MSL值。RFC 1122建议是2分钟，但BSD传统实现采用了30秒。
TIME_WAIT 状态最大保持时间是2 * MSL，也就是1-4分钟。
IP头部有一个TTL，最大值255。尽管TTL的单位不是秒(根本和时间无关)，我们仍需假设，TTL为255的TCP报文在Internet上生存时间不能超过MSL。
TCP报文在传送过程中可能因为路由故障被迫缓冲延迟、选择非最优路径等等，结果发送方TCP机制开始超时重传。前一个TCP报文可以称为"漫游TCP重复报文"，后一个TCP报文可以称为"超时重传TCP重复报文"，作为面向连接的可靠协议，TCP实现必须正确处理这种重复报文，因为二者可能最终都到达。
一个通常的TCP连接终止可以用图描述如下：
client server
FIN M
close -----------------> (被动关闭)
ACK M+1
<-----------------
FIN N
<----------------- close
ACK N+1
----------------->
为什么需要 TIME_WAIT 状态？
假设最终的ACK丢失，server将重发FIN，client必须维护TCP状态信息以便可以重发最终的ACK，否则会发送RST，结果server认为发生错误。TCP实现必须可靠地终止连接的两个方向(全双工关闭)，client必须进入 TIME_WAIT 状态，因为client可能面临重发最终ACK的情形。
{
  先调用close()的一方会进入TIME_WAIT状态
}
此外，考虑一种情况，TCP实现可能面临先后两个同样的相关五元组。如果前一个连接处在 TIME_WAIT 状态，而允许另一个拥有相同相关五元组的连接出现，可能处理TCP报文时，两个连接互相干扰。使用 SO_REUSEADDR 选项就需要考虑这种情况。
为什么 TIME_WAIT 状态需要保持 2MSL 这么长的时间？
如果 TIME_WAIT 状态保持时间不足够长(比如小于2MSL)，第一个连接就正常终止了。第二个拥有相同相关五元组的连接出现，而第一个连接的重复报文到达，干扰了第二个连接。TCP实现必须防止某个连接的重复报文在连接终止后出现，所以让TIME_WAIT状态保持时间足够长(2MSL)，连接相应方向上的TCP报文要么完全响应完毕，要么被丢弃。建立第二个连接的时候，不会混淆。
```

A: 小四
在Solaris 7下有内核参数对应 TIME_WAIT 状态保持时间
# ndd -get /dev/tcp tcp_time_wait_interval 240000
# ndd -set /dev/tcp tcp_time_wait_interval 1000
缺省设置是240000ms，也就是4分钟。如果用ndd修改这个值，最小只能设置到1000ms，也就是1秒。显然内核做了限制，需要Kernel Hacking。
# echo "tcp_param_arr/W 0t0" | adb -kw /dev/ksyms /dev/memphysmem 3b72
tcp_param_arr: 0x3e8 = 0x0
# ndd -set /dev/tcp tcp_time_wait_interval 0
我不知道这样做有什么灾难性后果，参看<>的声明。

Q: TIME_WAIT 状态保持时间为0会有什么灾难性后果？在普遍的现实应用中，好象也
就是服务器不稳定点，不见得有什么灾难性后果吧？
D: rain@bbs.whnet.edu.cn
Linux 内核源码 /usr/src/linux/include/net/tcp.h 中
#define TCP_TIMEWAIT_LEN (60*HZ)
最好不要改为0，改成1。端口分配是从上一次分配的端口号+1开始分配的，所以一般不会有什么问题。端口分配算法在tcp_ipv4.c中tcp_v4_get_port中。
来自：http://blog.sina.com.cn/s/blog_53a2ecbf010095db.html
setsockopt中参数之SO_REUSEADDR的意义
1、一般来说，一个端口释放后会等待两分钟之后才能再被使用，SO_REUSEADDR是让端口释放后立即就可以被再次使用。
    SO_REUSEADDR用于对TCP套接字处于TIME_WAIT状态下的socket，才可以重复绑定使用。server程序总是应该在调用bind()之前设置SO_REUSEADDR套接字选项。TCP，先调用close()的一方会进入TIME_WAIT状态
2、SO_REUSEADDR和SO_REUSEPORT
SO_REUSEADDR提供如下四个功能：
    SO_REUSEADDR允许启动一个监听服务器并捆绑其众所周知端口，即使以前建立的将此端口用做他们的本地端口的连接仍存在。这通常是重启监听服务器时出现，若不设置此选项，则bind时将出错。
    SO_REUSEADDR允许在同一端口上启动同一服务器的多个实例，只要每个实例捆绑一个不同的本地IP地址即可。对于TCP，我们根本不可能启动捆绑相同IP地址和相同端口号的多个服务器。
    SO_REUSEADDR允许单个进程捆绑同一端口到多个套接口上，只要每个捆绑指定不同的本地IP地址即可。这一般不用于TCP服务器。
    SO_REUSEADDR允许完全重复的捆绑：当一个IP地址和端口绑定到某个套接口上时，还允许此IP地址和端口捆绑到另一个套接口上。一般来说，这个特性仅在支持多播的系统上才有，而且只对UDP套接口而言（TCP不支持多播）。
SO_REUSEPORT选项有如下语义：
    此选项允许完全重复捆绑，但仅在想捆绑相同IP地址和端口的套接口都指定了此套接口选项才行。
    如果被捆绑的IP地址是一个多播地址，则SO_REUSEADDR和SO_REUSEPORT等效。
使用这两个套接口选项的建议：
    在所有TCP服务器中，在调用bind之前设置SO_REUSEADDR套接口选项；
当编写一个同一时刻在同一主机上可运行多次的多播应用程序时，设置SO_REUSEADDR选项，并将本组的多播地址作为本地IP地址捆绑。
    if (setsockopt(fd, SOL_SOCKET, SO_REUSEADDR,
   (const void *)&nOptval , sizeof(int)) < 0) ...
附
    Q:编写 TCP/SOCK_STREAM 服务程序时，SO_REUSEADDR到底什么意思？
    A:这个套接字选项通知内核，如果端口忙，但TCP状态位于 TIME_WAIT ，可以重用端口。如果端口忙，而TCP状态位于其他状态，重用端口时依旧得到一个错误信息，指明"地址已经使用中"。如果你的服务程序停止后想立即重启，而新套接字依旧使用同一端口，此时SO_REUSEADDR 选项非常有用。必须意识到，此时任何非期望数据到达，都可能导致服务程序反应混乱，不过这只是一种可能，事实上很不可能。
    一个套接字由相关五元组构成，协议、本地地址、本地端口、远程地址、远程端口。SO_REUSEADDR 仅仅表示可以重用本地本地地址、本地端口，整个相关五元组还是唯一确定的。所以，重启后的服务程序有可能收到非期望数据。必须慎重使用SO_REUSEADDR 选项。【2】
【1】 http://topic.csdn.net/u/20090103/16/a0414edb-b289-4c72-84da-39e155e8f4be.html
【2】
以下博客对这个问题进行了对答式的解答：
http://blog.sina.com.cn/s/blog_53a2ecbf010095db.html
【3】 http://www.sudu.cn/info/html/edu/20050101/296180.html