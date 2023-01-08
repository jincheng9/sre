# rsync: No buffer space available (105)报错问题

## 问题现象

有时候Windows机器sync的时候会报如下错误：

```bash
rsync: pipe failed in do_recv: No buffer space available (105)
rysnc error: error in IPC code (code 14) at main.c(763) [Receiver=3.0.8]
rsync: connection unexpectedly closed (144 bytes received so far) [sender]
rsync error: error in rsync protocol data stream (code 12) at io.c(226) [sender=3.1.2]
```

出现这个问题的原因是端口资源被耗尽。

```bat
netsh int ipv4 show dynamicportrange tcp
```

以上命令可以查看机器设置的起始端口和支持的端口数量。

## 解决方案1

以管理员身份运行命令窗口，使用如下命令修改端口数

```bat
netsh int ipv4 set dynamicport tcp start=2000 num=63000
```

start为起始端口号，num为端口数量

然后重新rsync即可。

## 解决方案2

重启机器，然后重新rsync即可。



## 总结

不过还是需要找到为啥rsync服务器的端口资源会被耗尽，否则还是可能出现该问题。

