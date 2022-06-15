# 运维命令

不做特殊说明，都是CentOS操作系统下的命令。

## Linux文件权限

![](./img/linux-file-permission.jpg) 

## 修改目录/文件权限

```bash
chmod 777 dir_name // 777表示Owner, Group, Other对该文件或者目录都拥有可读、可写和可执行的权限
```



## 防火墙

**注意**：修改了防火墙配置后，需要重启防火墙

### 开放端口

```bash
firewall-cmd --permanent --add-port=80/tcp
```

### 重启防火墙

```bash
firewall-cmd --reload
```

