# 网络运维

## Windows系统添加指定IP地址的路由

多网卡情况下，添加路由，指定到目的IP的路由走哪个网卡。

* dest_ip：目的IP地址，要访问的IP地址
* submask_ip: 子网掩码IP地址
* gateway_ip: 网关IP地址

```bash
# route -p add dest_ip mask submask_ip gateway_ip
route -p add 123.124.125.126 mask 255.255.255.0 192.168.10.1
```

修改默认路由地址：https://blog.csdn.net/Season_hangzhou/article/details/18841981
