### 介绍
为了改善web应用相应时延，google发布了通过修改TCP协议利用三次握手时进行数据交换的TFO(TCP fast open，RFC 7413)。

### 原理
```
第一次完整连接 + 短时间内 = 二次连接大概率不出问题
通过减少一次RTT提升短连接效率
```

### 内核启用
TFO功能在Linux 3.7 内核中开始集成，因此RHEL7/CentOS7是支持的，但默认没有开启，使用以下方式开启：

```
echo 3 > /proc/sys/net/ipv4/tcp_fastopen
#3的意思是开启TFO客户端和服务器端
#1表示开启客户端，2表示开启服务器端
```

### 实验
```
我们在服务器端开启TFO，并配置nginx支持TFO。 
客户端开启TFO，升级curl到7.61版本。然后使用curl访问HTTP页面进行测试。 
客户端如下

# curl -s -o/dev/null --tcp-fastopen http://10.140.10.16/
使用ip tcp_metrics show可以看到cookie
# ip tcp_metrics show | grep "fo_cookie"
10.140.10.16 age 41.955sec tw_ts 282422045/42sec ago rtt 250us rttvar 250us cwnd 10 metric_5 2380 metric_6 1190 fo_mss 1460 fo_cookie 1640a20f99195995
服务器端抓包如下，可以看到发出的cookie，1640a20f99195995。

20:17:10.533466 IP 10.140.12.45.28722 > 10.140.10.16.80: Flags [S], seq 1532602092, win 29200, options [mss 1460,sackOK,TS val 982198124 ecr 0,nop,wscale 9,tfo cookiereq,nop,nop], length 0
20:17:10.533518 IP 10.140.10.16.80 > 10.140.12.45.28722: Flags [S.], seq 108109466, ack 1532602093, win 28960, options [mss 1460,sackOK,TS val 282422044 ecr 982198124,nop,wscale 9,tfo cookie 1640a20f99195995,nop,nop], length 0

使用以下命令可以查看TFO连接的统计信息

# grep '^TcpExt:' /proc/net/netstat | cut -d ' ' -f 87-92 | column -t
TCPOFOMerge TCPChallengeACK TCPSYNChallenge TCPFastOpenActive TCPFastOpenActiveFail TCPFastOpenPassive
9306 29958 2457 0 0 11
```


### 参考
http://www.ipcpu.com/2018/05/tfotcp-fast-open/

http://martinbj2008.github.io/2016/11/23/what-is-tfo/ 
https://zhuanlan.zhihu.com/p/36239657 
http://abcdxyzk.github.io/blog/2018/07/30/kernel-tcp_metric/ 
https://tools.ietf.org/html/rfc7413
