# 运维手册

* [网络运维](./network/readme.md)
* [存储运维](./storage/readme.md)
* [Linux常用命令](#Linux常用命令)

# Linux常用命令

如果不做特殊说明，涉及shell命令的都是CentOS操作系统的命令。

## Linux文件/目录权限

![](./img/linux-file-permission.jpg) 

`x`: 对于目录而言，要有`x`可执行权限，才能`cd`进入到该目录



## 修改目录/文件权限

```bash
chmod 777 dir_name // 777表示Owner, Group, Other对该文件或者目录都拥有可读、可写和可执行的权限
```



## 用户管理

```bash
# 添加用户
useradd user_name
# 设置密码
passwd user_name
# 查看用户信息
id user_name
# 查看当前用户
whoami
# 切换用户su: switch user
su user_name
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



## 文件/目录上传和下载

### 上传本地文件或目录到远程服务器

```bash
scp -P port /path/to/file username@serverip:remote_dir
scp -P port -r local_dir username@serverip:remote_dir
```

### 从远程服务器下载文件或目录到本地

```bash
scp username@serverip:/path/to/file local_dir
scp -r username@serverip:remote_dir local_dir
```



## 查看磁盘空间的占用

看当前目录下各个文件和文件夹的空间占用

```bash
du -hs * | sort -hr | head 
```



## OpenResty/Nginx

### 查看OpenResty版本

```bash
$ resty -v
$ which resty // 可以看resty命令在/usr/bin目录下
```

### Nginx安装位置

主目录：`/usr/local/openresty/nginx`

可执行文件目录：`/usr/local/openresty/nginx/sbin`

配置文件目录：`/usr/local/openresty/nginx/conf`

## 隐藏openresty版本

在nginx.conf的http段添加如下配置：

```bash
server_token off;
```

然后在命令行终端上执行如下命令，检查配置是否成功(在conf目录下操作的，所以用../sbin/nginx，这个根据实际所在目录调整即可)

```bash
../sbin/nginx -t
```

检查通过后，执行如下命令重新加载conf配置文件

```bash
./nginx -s reload
```



## 添加路由

https://blog.csdn.net/wangzhen209/article/details/77748107
