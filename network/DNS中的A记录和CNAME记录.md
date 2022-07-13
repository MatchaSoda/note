# DNS中的A记录和CNAME记录
### DNS
DNS即域名解析，因为IP地址不方便记忆便有了域名，而从域名转换到IP地址的过程就是域名解析。

DNS中的A记录是将域名解析成IP，CNAME是将域名解析成另外一个域名。
### A记录
A记录，即Address记录，直接将域名或主机名指向某个IP，如：
* 域名 www.aa.com → 1.1.1.1
* 主机名 DD → 2.2.2.2
### CNAME记录
CNAME记录，也叫别名记录，如：
* www.xx.com → www.aa.com → 1.1.1.1
* www.yy.com → www.aa.com → 1.1.1.1
* www.zz.com → www.aa.com → 1.1.1.1

xx、yy和zz都称为别名，他们都指向了aa，当我们想更换IP地址的时候不必更换每个别名的IP，只需要修改aa的地址即可。
### CNAME的应用
现在常用于[CDN](./CDN.md)网络加速上，如：
* www.dd.com → www.dd.cdn.com

当用户访问www.dd.com时，本地DNS系统会将域名的解析权交给CNAME指向的CDN专用DNS服务器。  