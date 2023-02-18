# 运维手册

* [Linux常用命令](#Linux常用命令)

* [网络运维](./network/readme.md)
* [存储运维](./storage/readme.md)
* [阿里云运维](./alicloud/readme.md)
* [其它杂项](./other/readme.md)

# Linux常用命令

如果不做特殊说明，涉及shell命令的都是CentOS操作系统的命令。

Linux命令大全：https://www.runoob.com/linux/linux-command-manual.html

## Linux文件/目录权限

![](./img/linux-file-permission.jpg) 

`x`: 对于目录而言，要有`x`可执行权限，才能`cd`进入到该目录

777表示Owner, Group, Other对该文件或者目录都拥有可读、可写和可执行的权限



## 修改目录所属用户和用户组

```bash
chown -R user dir_name
chown -R user:group dir_name
```

## 修改目录的权限

```bash
chmod -R 755 dir_name
```

-R表示递归处理指定目录及其子目录下的所有文件，对目录使用-R，单独修改某个文件的所属用户/用户组以及修改某个文件的权限，不需要加-R。

参考：https://www.runoob.com/linux/linux-command-manual.html



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



## 添加ssh public key

比如你作为服务器管理员，要允许访问者通过test这个用户名，以ssh key的方式登录。

创建用户参考上面的[用户管理](#用户管理)。

那需要在`/home/test/.ssh/`目录下创建authorized_keys这个文件，把访问者的public key加入到authorized_keys文件里。

```bash
echo xx >> /home/test/.ssh/authorized_keys
```

**备注**：

* echo后面如果用的是 >> 会往文件末尾追加，不会覆盖，追加内容会换行。

* echo后面如果用的是 > 会覆盖文件内容，而不是往后追加内容。

* 如果.ssh目录下没有authorized_keys，执行上面的echo命令后，会自动创建authorized_keys文件。

* 添加ssh publick key到~/.ssh/authorized_keys文件后，不用重启sshd，用户就可以通过public key登录到服务器了。修改了sshd的配置文件ssd_config，才需要重启sshd，也就是ssh server。

* 有时候，把ssh public key加到用户目录下的~/.ssh/authorized_keys后，登录会提示用户密钥未注册之类的，需要设置.ssh目录和authorized_keys目录权限。

  ```bash
  chmod 700 ~/.ssh
  
  chmod 600 ~/.ssh/authorized_keys
  ```

  同时确保该用户是.ssh目录和authorized_keys文件的owner。
  
  ```bash
  chown -R test .ssh
  ```
  

## rsync文件传输

通过ssh key的方式传输文件的准备工作：

1. 源机器上用户的.ssh目录下有id_rsa文件，该文件是私钥。
2. 目标机器的.ssh/authorized_keys文件里放上源机器的ssh key公钥。
3. 在源机器上执行如下命令即可，表示把源机器上的images目录下的xxx.tar文件上传到目标机器192.168.42.55的/root/images目录下。

```bash
rsync images/xxx.tar root@192.168.42.55:/root/images
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

图省事可以直接用XFTP客户端来操作即可。



## 查看磁盘空间的占用

看当前目录下各个文件和文件夹的空间占用

```bash
du -hs * | sort -hr | head 
```



## 查看Linux操作系统版本

命令：`lsb_release -a`。

```bash
# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.9.2009 (Core)
Release:	7.9.2009
Codename:	Core
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

## 查看当前nginx进程使用的conf配置文件路径

```bash
# ./sbin/nginx -t
nginx: the configuration file /usr/local/openresty/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/openresty/nginx/conf/nginx.conf test is successful
```

**注意**：执行`nginx -t`命令的Linux用户，如果和nginx进程启动的用户不一样，那执行`nginx -t`命令后，会修改nginx目录下比如proxy_temp, client_body_temp等目录的所属用户为nobody，导致nginx worker进程访问proxy_temp和client_body_temp等目录的时候可能会出现权限问题，导致访问失败。要使用nginx worker进程启动用户去执行`nginx -t`命令。

参考：https://blog.csdn.net/memory6364/article/details/86155978



## Nginx启动参数

Nginx 的参数包括：

* -c <path_to_config>：使用指定的配置文件而不是 conf 目录下的 nginx.conf。
* -t：测试配置文件是否正确，。
* -v：显示 nginx 版本号。
* -V：显示 nginx 的版本号，编译环境信息以及编译时的参数。

```bash
./sbin/nginx -c ./conf/nginx.conf
```



## 添加路由

https://blog.csdn.net/wangzhen209/article/details/77748107



## Xshell and Xftp

* [Xshell如何记住用户秘钥，不用每次登录时选择private key文件](./network/xshell_public_key.md)

