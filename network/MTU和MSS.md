# MTU 和 MSS
## MTU
MTU 全称是Maximum Transmission Unit，即最大传输单元。MTU是**数据链路层**中约定的数据载荷部分最大长度，数据超过它时就需要分片。
## MSS
MSS的英文全称叫Max Segment Size，是TCP最大段大小。

MSS 是在**三次握手**的时候，在两端主机之间被计算得出，两端主机在发出建立连接的请求时，会在**TCP首部**中写入**MSS选项**，告诉对方自己的接口能够适应的MSS的大小，然后在两者之间选择一个较小的值投入使用。

**MSS的值是在三次握手的SYN报文中商量出来的，MSS选项也只能出现在SYN报文中。**
## 总结
* MTU是**数据链路层**概念，MSS是**传输层**概念。
* MSS = MTU - IP header头大小 - TCP 头大小
* [因为IP分片丢失需要重传所有分片](/network/IP分片.md)，所以TCP为了IP层不用（依据MTU）分片主动将数据包切割为MSS大小。

## 参考
https://mp.weixin.qq.com/s/ZMV2izeYkBIqjPhsv_-wdw  
网络是怎样连接的  
图解TCP/IP  
https://xiaolincoding.com/network/3_tcp/tcp_interview.html#%E6%97%A2%E7%84%B6-ip-%E5%B1%82%E4%BC%9A%E5%88%86%E7%89%87-%E4%B8%BA%E4%BB%80%E4%B9%88-tcp-%E5%B1%82%E8%BF%98%E9%9C%80%E8%A6%81-mss-%E5%91%A2