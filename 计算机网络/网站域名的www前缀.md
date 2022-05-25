# 网站域名的www前缀
### www前缀是什么
首先我们要知道，**域名是从后向前读的**。  
例如:

www.google.com

实际为: company公司 - google谷歌 - www(world wide web)万维网  
此处www，实际上就是目前所谓的国际互联网。那么上述域名含义就是“谷歌公司的互联网网站”。

同样smtp.google.com就代表了google的stmp服务，ftp.google.com就代表了ftp服务，此处的www就代表是http服务。  
而现在除了http服务其他的很少用，即便不写www也默认了是要访问http网页，就有很多公司直接使用裸域名了。
### 不带www的坏处
#### DNS不支持在根域名上设置 CNAME 纪录
裸域名只能绑定 DNS 的 A 记录，不能绑定 CNAME 记录。也就是说你不能把裸域设定为另外域名的别名。  
很多时候这对管理不是很方便，特别是使用第三方托管服务的时候。如果第三方迁移服务器导致 IP 地址变更，你必须自己去更改 DNS 的 A 记录。如：
* www.dd.com → www.dd.cdn.com
* dd.com × dd.cdn.com
#### 裸域的 Cookie 的作用范围太大
在根域名上设置的 Cookie 会被发送给该域名下的所有子域名，也就是说所有子域名都可以用根域名的 Cookie，不利于保护隐私和安全。
### 应该怎样
使用带有www的域名，并且将裸域使用301跳转到带www的域名。