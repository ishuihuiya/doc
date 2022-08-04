# DNS和BIND



## 什么是DNS

DNS服务这张是在很多企业中，属于一个基础服务，在互联网上也好，在局域网中访问远程主机也好，虽然说通过IP地址是可以访问远程主机的，但是通过IP地址访问这个有很多困难。第一就是不好记忆，第二就是如果ip变更就无法成功访问。

将便于记忆的域名解析成ip地址这个解析服务就是需要靠DNS服务器来完成。



## FQDN   

FQND 全称域名=主机名+域名。 



## DNS域名

* 根域
* 顶级域
  * 组织域  .com  .gov .edu .net .org...
  * 国家域 .cn .jp ...
* 二级域

每个DNS服务器上会有数据库，相关的.



## 缓存DNS服务器

安装`bind`

```shell
sudo yum intall bind -y
```

编辑配置文件

```shell
options {
        listen-on port 53 { localhost; };
#       listen-on port 53 { 127.0.0.1; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };
...
# localhost; 
# any;
```

重启服务

```shell
rndc reload 
```





## 搭建主DNS服务器

zone:   一个区域对应的数据库文件

```shell
zone "." IN {
        type hint;  # hint 表示根服务器  主DNS服务器写master
        file "named.ca"; # 这个数据存放的文件， 存放了互联网的13个根DNS服务器
};
```

### SOA记录

SOA起始授权记录，在DNS数据库存放的第一条记录。内容就是当前域的描述信息。

格式：

```shell
name [TTL] IN  rr_type  value 
# name 
# ttl 时间，有效时长，生命期。 缓存的有效期 秒为单位。
# in internet记录， 固定写法
# rr_type  正向解析，反向解析 ipv4  ipv6  域名到ipv4 jqui1A
#     域名到ipv6就是AAAAS
# value 域名对应的ip
```

### 模板文件 `/var/named/named.localhost`

```shell
# 第一条是soa记录，然后才是ip域名的解析
# ttl 时间1天， 也就是缓存信息有效期为1天
$TTL 1D
# 一共五项 name ttl in rr_type value 
# @	表示当前域，根据文件名
# IN
# SOA
# rname.invalid.  主DNS服务器

# ID： 推送
# 1H： 推送失败重新尝试
# 1W: 一周无法推送就从DNS服务器失效
# 3H： 不存在的请求重复访问，3H内不会在去查询，直接缓存返回不存在。
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1

# soa记录
@  86400 IN SOA  master.dnstest.com. 

	NS mater
master.dnstest.com A IP


# 域名ip解析的原本方式：
ftp 86400 IN A 1.1.1.1
# 通过省略
ftp  A 1.1.1.1
```

DNS配置文件添加新增域名的配置文件`/etc/named.rfc1912.zones`

```shell
zone "dnstest.com" {
	type master;
	file "dnstest.com.zone";
}
```

### redc

```shell
# 语法检查，只检查配置文件
named-checkconf
# 检查数据库
named-checkzone  y域名  域名文件地址

# 清空缓存
rndc flush 


rndc reload
```

### 范域名解析

`\*` 通配



## 反向解析

反向区域 `arpa`

```shell
zone "37.168.192.in-addr.arpa"{
	type master;
	file "123";
};


# 
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1

6   PTR  webserver.dnstest.com.

```

## 从DNS服务器

### 定义从dns

```shell
nano /etc/named.rfc1912.zones

zone "kaladns.com" IN {
        # 指定该节点为从DNS服务器
        type slave;
        # 指定DNS主服务器
        masters {172.16.96.129;};
        # 指定区域数据文件存放地址
        file "slaves/dnstest.com.zone.slave";
};

```

### 主DNS指定从DNS

```shell
  GNU nano 5.6.1                                     kaladns.com.zone                                             
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        5       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
       NS      slave2
slave2  A       172.16.96.131

# 添加NS记录，并将NS记录纸箱从DNS服务器， 并将serial数值调大，然后重启服务，
```

### 主DNS设置允许同步的机器，降低安全风险

```shell
nano /etc/named.conf 

allow-transfer {从DNSip;};
```

### 从DNS 设置安全

```shell
nano /etc/named.conf 
allow-transfer {none;};
```



## 子域委派和转发

