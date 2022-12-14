## 获取服务器的互联网ip

使用以下个命令都可以。

```bash
curl cip.cc
curl ifconfig.me
```

服务器通常有多个网卡，看服务器的互联网流量是通过哪个网卡可以使用如下命令：

```bash
ip r get 8.8.8.8
```

