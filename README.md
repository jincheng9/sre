# 运维手册

* [Linux常用命令](#Linux常用命令)

* [网络运维](./network/readme.md)
* [存储运维](./storage/readme.md)
* [阿里云运维](./alicloud/readme.md)
* [git命令](#Git命令)
* [Mac电脑相关](#Mac)
* [MySQL操作](#MySQL操作)
* [Docker操作](#Docker)
* [Windows运维](#Windows)
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

## setfacl精细化权限控制

* 如果你有一个目录，希望给其它用户赋予该目录下**已有目录**和**已有文件**的操作权限

  ```bash
  # 这里-m是修改ACL规则的选项，u:xly:rwX指定了用户（u）xly对指定目录有读（r）和写（w）权限，大写的X意味着如果对象是一个目录或任何当前带有执行（x）权限的文件，则赋予执行权限。
  setfacl -m u:user1:rwX /path/to/directory
  
  # -R选项使命令递归地应用到所有子目录和文件。
  setfacl -Rm u:user1:rwX /path/to/directory
  ```

  如果执行了上面的命令后，新创建的目录和文件，并不会满足上面的权限设置

* 如果你有一个目录，希望给其它用户赋予后续该目录下**新建的目录**和**新建的文件**的操作权限

  ```bash
  # -Rm选项是递归地修改ACL设置。
  # d:u:jxm:r指定为用户（u）jxm设置默认（d）ACL，其中包括读（r）权限。d就是default
  # d: 表示这是一个默认ACL。对于目录，它会影响该目录中所有新创建的文件和子目录的ACL。现有的文件和目录不受影响。
  # 这里的X权限表示只对目录应用执行权限，表示用户可以进入该目录，并不会给文件赋予执行权限
  setfacl -Rm d:u:user1:rwX /path/to/directory
  setfacl -Rm d:u:user2:r /path/to/directory
  ```

* **注意**：执行`setfacl -m u:user1:rwX /path/to/directory`，只修改用户对directory这个子目录的权限，并不会改变用户对/path目录和/path/to目录的权限。操作三部曲：

  ```bash
  setfacl -m u:user1:rX /path
  setfacl -m u:user1:rX /path/to
  setfacl -Rm u:user1:rwX /path/to/directory
  setfacl -Rm d:u:user1:rwX /path/to/directory
  ```
  
  `X`权限是一个特殊标记，仅用于`chmod`或`setfacl`命令的递归 (`-R`) 操作中。它表示只为目录设置执行权限，而不改变已有文件的执行权限。

* 取消权限

  ```bash
  setfacl -x u:user1 /path/to/directory # 移除user1用户对/path/to/directory已有目录和文件的权限
  setfacl -x d:u:user1 /path/to/directory # 移除user1用户对/path/to/directory的默认权限
  setfacl -Rx u:user1 /path/to/directory # 递归移除user1用户对/path/to/directory目录及其子目录和文件的权限
  setfacl -Rx d:u:user1 /path/to/directory # 递归移除user1用户对/path/to/directory目录及其子目录和文件的默认权限
  # 移除文件或目录上的所有访问控制列表（ACL）条目，将其恢复到标准的文件权限系统
  setfacl -b /path/to/directory 
  ```


* **注意**：拷贝的文件和目录不会继承setfacl设置好的权限，要手动给拷贝的目录和文件设置权限。

## 用户管理

```bash
# 添加用户，使用-m会自动在/home目录下增加该用户的目录
# 不使用-m创建用户的时候，在/home目录下不会自动增加该用户的目录
useradd -m user_name 
# 设置密码
passwd user_name
# 查看用户信息
id user_name
# 查看当前用户
whoami
# 切换用户su: switch user
su user_name
# 删除用户
userdel user_name

# 添加用户到sudo用户组，以下命令适用于centos系统，在 CentOS 中，wheel 用户组的成员具有 sudo 权限：
usermod -aG wheel username

# 使用 id 命令来验证用户是否已经被添加到 wheel 用户组：
id username

# 为了执行sudo命令时不用输入密码，需要在 /etc/sudoers 文件中添加一行数据
%wheel ALL=(ALL) NOPASSWD: ALL # NoPASSWD可以让执行sudo的时候不用输入密码

# 如果没有通过usermod -aG wheel username把用户加到sudo用户组的话，可以直接往/etc/sudoers里增加以下内容：
username ALL=(ALL) NOPASSWD: ALL # 指定特定用户拥有sudo权限
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

  注意：**必须确保该用户是.ssh目录和authorized_keys文件的owner，否则还是不能登录。**
  
  **建议切换到对应的登录用户，然后使用该用户来创建.ssh目录和authorized_keys文件**)。
  
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

如果要下载某远程主机上的文件到本机上，则执行如下命令，表示把目标机器192.168.42.55的xxx.tar文件下载到本机的images目录下。

```bash
rsync  root@192.168.42.55:/root/images/xxx.tar images/
```

**注意**：如果执行rsync提示如下问题，则是由于本机的~/.ssh/id_rsa私钥文件权限过大，可以执行`chmod 600 id_rsa` 来解决。

```bash
Permissions 0644 for '/root/.ssh/id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/root/.ssh/id_rsa": bad permissions
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
rsync: connection unexpectedly closed (0 bytes received so far) [sender]
rsync error: unexplained error (code 255) at io.c(226) [sender=3.1.2]
```



## scp文件传输

```bash
## 传输本机文件到目标机器192.168.10.12的/target_machine/path目录
## 要把本机的ssh key加到目标机器的username用户下的.ssh/authorized_keys里
scp -r /path/to/local/directory username@192.168.10.12:/target_machine/path
```



## 网络盘挂载

场景：把IP为192.168.10.214的Linux服务器的/home/web/share挂载到本机的/home/test目录

操作步骤：

* 第一步，把本机生成的ssh public key放到192.168.10.214的web用户下的.ssh/authorized_keys文件里
* 第二步，在本机执行如下命令：

```bash
sshfs web@192.168.10.214:/home/web/share /home/test/
```

* 第三步，在本机上操作，把第二步的命令一模一样复制到~/.bashrc文件里

注意：需要先在本机和目标机器上安装sshfs，命令为

```bash
yum install sshfs
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

### 关闭防火墙

以下是如何使用firewall-cmd命令关闭防火墙的步骤:

1. 打开终端。

2. 输入以下命令来关闭防火墙：

   ```bash
   sudo firewall-cmd --state
   ```

   这个命令会显示出防火墙的当前状态，如果显示running，说明防火墙正在运行。

3. 如果防火墙正在运行，输入以下命令来停止防火墙：

   ```bash
   sudo systemctl stop firewalld
   ```

4. 在防火墙已停止的情况下，如果你想要禁止防火墙在系统启动时启动，你可以运行以下命令：

   ```bash
   sudo systemctl disable firewalld
   ```

5. 你可以再次使用以下命令来检查防火墙的状态，确认防火墙已关闭：

   ```bash
   sudo firewall-cmd --state
   ```

   这次应该返回not running，说明防火墙已经关闭。

此时，你的防火墙就已经关闭并且在下次启动时也不会自动运行。

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

* 方法1：`hostnamectl` 命令不仅可以显示你的主机信息，还可以显示操作系统相关的详细信息，包括操作系统名称和版本。

```bash
[xx@machine]$ hostnamectl
 Static hostname: machine-super
       Icon name: computer-server
         Chassis: server 🖳
      Machine ID: 9fcfa28b74c04b3387bbe7c2d2180486
         Boot ID: 7067fbf492414f6791f9a85ed1c0f4ef
Operating System: Rocky Linux 9.3 (Blue Onyx)
     CPE OS Name: cpe:/o:rocky:rocky:9::baseos
          Kernel: Linux 5.14.0-362.8.1.el9_3.x86_64
    Architecture: x86-64
 Hardware Vendor: AMAX
  Hardware Model: AS -4124GS-TNR
Firmware Version: 2.4
```



* 方法2：`cat /etc/*release` 这个命令会显示包含“release”的所有文件内容，其中通常包括了发行版和版本号等信息。

```bash
NAME="Rocky Linux"
VERSION="9.3 (Blue Onyx)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="9.3"
PLATFORM_ID="platform:el9"
PRETTY_NAME="Rocky Linux 9.3 (Blue Onyx)"
ANSI_COLOR="0;32"
LOGO="fedora-logo-icon"
CPE_NAME="cpe:/o:rocky:rocky:9::baseos"
HOME_URL="https://rockylinux.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
SUPPORT_END="2032-05-31"
ROCKY_SUPPORT_PRODUCT="Rocky-Linux-9"
ROCKY_SUPPORT_PRODUCT_VERSION="9.3"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="9.3"
Rocky Linux release 9.3 (Blue Onyx)
Rocky Linux release 9.3 (Blue Onyx)
Rocky Linux release 9.3 (Blue Onyx)
```

* 方法3：命令：`lsb_release -a`。

```bash
# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.9.2009 (Core)
Release:	7.9.2009
Codename:	Core
```

如果提示command not found，那先安装redhat-lsb包，命令为`yum install redhat-lsb`。



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

## Nginx加入到systemd管理

### 1. 创建 systemd 服务文件

如果 OpenResty 安装时没有自动生成 `systemd` 服务文件，您需要手动创建一个。通常，这个服务文件应该放在 `/etc/systemd/system/` 目录下，并以 `.service` 结尾。如果已经存在，您可以跳过这一步，直接到启用步骤。

1. 打开一个文本编辑器来创建服务文件：

   ```bash
   sudo vim /etc/systemd/system/nginx.service
   ```

2. 将以下内容添加到 `nginx.service` 文件中（确保根据您的 OpenResty 安装调整 `ExecStart` 和 `ExecReload` 中的路径）：

   ```ini
   [Unit]
   Description=The NGINX HTTP and reverse proxy server
   After=network.target
   
   [Service]
   Type=forking
   ExecStart=/usr/local/openresty/nginx/sbin/nginx -c /usr/local/openresty/nginx/conf/nginx.conf
   ExecReload=/usr/local/openresty/nginx/sbin/nginx -s reload
   ExecStop=/usr/local/openresty/nginx/sbin/nginx -s quit
   
   [Install]
   WantedBy=multi-user.target
   ```

   确保将 `/usr/local/openresty/nginx/sbin/nginx` 和 `/usr/local/openresty/nginx/conf/nginx.conf` 替换为您实际的 OpenResty Nginx 执行程序和配置文件的路径。

### 2. 启用服务

创建服务文件后，您需要启用服务以便在开机时自动启动。

1. 重新加载 `systemd` 以识别新的或更改过的服务文件：

   ```bash
   sudo systemctl daemon-reload
   ```

2. 启用 Nginx 服务：

   ```bash
   sudo systemctl enable nginx ## 设置ngin开机自启动
   sudo systemctl is-enabled nginx ## 查看nginx是否设置了开机自启动
   ```

   这会将 nginx 服务设置为开机自启动。

3. 日常运维

   ```bash
   sudo systemctl status nginx
   sudo systemctl restart nginx
   sudo systemctl stop nginx
   ```

   

## SELinux

SELinux: Security-Enhanced Linux

有时候使用Nginx作为Web服务器，访问网站的静态资源文件会出现权限问题，导致静态资源获取不到，网站样式错乱。

如果确认了静态资源目录和文件权限都正常的话，这个问题大概率是由于服务器开启了SELinux引起的。

- 首先，查看 SELinux 的状态：

  ```bash
  getenforce
  ```

- 如果返回 `Enforcing`，那么 SELinux 是启用的，并且可能是阻止访问的原因。

- 您可以临时将 SELinux 设置为 `Permissive` 模式，以排除它是问题的原因（但请注意，这种模式下，会记录违反策略的行为，但不会强制阻止）：

  ```bash
  setenforce 0
  ```

- 然后，尝试再次访问文件。如果这次成功，那么问题确实由 SELinux 的策略引起。

- 要为 `nginx` 访问静态文件设置正确的 SELinux 类型标签，可以对要访问的目录执行如下操作：

  ```bash
  sudo chcon -R -t httpd_sys_content_t /path/to/static/
  ```

- 请记得，完成测试后重新启用 SELinux：

  ```
  setenforce 1
  ```

**注意**： chcon命令修改的SELinux权限在机器重启会失效，要持久化保存，需要使用 `semanage fcontext` 命令来管理文件上下文的默认规则，并使用 `restorecon` 命令应用这些更改。

### 1. 使用 `semanage fcontext` 添加或修改规则

首先，确认已经安装了 `policycoreutils-python-utils` 包，这个包通常包含了 `semanage` 命令。如果没有，可以通过您的发行版的包管理器安装它。以 CentOS/RHEL 为例：

```bash
sudo yum install policycoreutils-python-utils
```

或者在 Fedora 上：

```bash
sudo dnf install policycoreutils-python-utils
```

接下来，添加一个新的规则，以指定目标路径的正确的 SELinux 类型 (`httpd_sys_content_t`)。如果您想要更改 `/path/to/your/web/content` 目录及其子目录中所有文件的类型，请运行：

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/path/to/your/web/content(/.*)?"
```

这个命令告诉 SELinux：“为 `/path/to/your/web/content` 以及其下所有子目录和文件添加一个规则，使它们的默认类型为 `httpd_sys_content_t`。”

正则表达式 `(/.*)?` 确保规则应用于目录本身以及目录下的所有内容。

### 2. 使用 `restorecon` 应用更改

在添加或修改了文件的 SELinux 类型规则后，您需要使用 `restorecon` 命令来实际应用这些更改。为了递归地（即包括所有子目录和文件）应用更改，请运行：

```bash
sudo restorecon -Rv /path/to/your/web/content
```

选项说明：

- `-R` 或 `--recursive`：递归地为所有对象应用指定的更改。
- `-v` 或 `--verbose`：在更改每个文件的上下文时输出详情。

执行这条命令后，`/path/to/your/web/content` 目录及其子目录和文件的 SELinux 上下文类型将被更新为 `httpd_sys_content_t`，应用了您通过 `semanage fcontext` 命令所设定的规则。

### 注意

- **确保路径正确**：在使用上述命令时，请确保把 `/path/to/your/web/content` 替换为您实际想要更改 SELinux 上下文的具体路径。
- **权限**：运行这些命令需要管理员或 root 权限。
- **重启服务**：在更改完文件的 SELinux 上下文后，您可能需要重启 Web 服务器，例如 Nginx 或 Apache，来确保更改生效。

通过上述步骤，您就可以永久地为文件或目录设置 SELinux 上下文，从而允许 Web 服务器安全地访问这些内容。

https://megamorf.gitlab.io/2020/06/10/troubleshoot-nginx-access-forbidden-error-caused-by-selinux/



## 添加路由

https://blog.csdn.net/wangzhen209/article/details/77748107



## crontab定时任务

CentOS系统，crontab的执行日志在/var/log/cron文件里

```bash
grep 'check_update_time.py' /var/log/cron
```



## Xshell and Xftp

* [Xshell如何记住用户秘钥，不用每次登录时选择private key文件](./network/xshell_public_key.md)



# Git命令

## 迁移repo A里某个指定分支到另外一个新repo B

* Step 1: 在Gitlab创建一个新的空仓库B，里面不要添加README.md以及任何License和.gitignore文件。
* Step 2: 按照如下命令操作即可。 

```bash
# 进入repo A目录，切换到指定分支
$ cd repo_A
$ git checkout target_branch

# 上传repo A里的指定分支到repo B仓库
$ git push git@repo_B:xx.git

# 上传repo A里的所有分支到repo B仓库
$ git push git@repo_B:xx.git --all

# 上传repo A里的所有tags到repo B仓库
$ git push git@repo_B:xx.git --tags
```

* https://blog.csdn.net/anniaxie/article/details/124122763

## 放弃本地修改

```bash
## git pull的时候提示本地有未提交的修改，可以执行以下命令放弃本地修改
git checkout -- /path/to/file
```



# Mac

Mac电脑如果需要远程访问windows电脑，可以安装如下这个软件或者使用todesk等远程控制软件：

https://install.appcenter.ms/orgs/rdmacios-k2vy/apps/microsoft-remote-desktop-for-mac/distribution_groups/all-users-of-microsoft-remote-desktop-for-mac

* 参考资料：https://zhuanlan.zhihu.com/p/460311929



# Windows

* 查找Windows指定路径下的进程

  ```bat
  wmic process get ProcessID,ExecutablePath | findstr /i "C:\Program\ Files\Tencent\QQNT\QQ.exe"
  ```

  路径参数里如果有**空格**，空格前面要加`\`来进行转移。
* 根据进程名查找进程ID
  ```bat
  tasklist | findstr process_xxx
  ```
* 根据进程ID查找exe进程所在的目录
  ```bat
  wmic process where processid="PID" get ExecutablePath
  ```
* kill相关进程
  ```bat
  # 根据进程名kill
  taskkill /F /IM "process_name"
  taskkill /F /T /IM "process_name"
  ```
  ```bat
  # 根据进程ID来kill
  taskkill /F /PID pid_number
  taskkill /F /T /PID pid_number
  ```
  /F表示Force，强制终止；/T表示Tree，kill该进程及其子进程

* 给windows用户登录用户名和显示用户名

  ```bat
  计算机管理->本地用户和组->修改Name和Full name(Name是登录用户名，Full Name是欢迎界面显示的名字)
  任务管理器->用户->注销用户
  控制面板->用户账户

## MySQL操作

* 机器上安装mysql client

  ```bash
  # centos为例，安装后就可以用mysql命令去连接mysql数据库了
  yum install mysql
  ```

  

* 创建用户

  ```sql
  ## 该用户只能从数据库所在机器访问数据库
  CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
  ## 该用户只能从192.168.10.214这台机器访问数据库
  CREATE USER 'username'@'192.168.10.214' IDENTIFIED BY 'password';
  ## 该用户可以从任意机器机器访问数据库
  CREATE USER 'username'@'%' IDENTIFIED BY 'password';
  ## 该用户可以从任意机器机器访问数据库
  CREATE USER 'username' IDENTIFIED BY 'password';
  ```

* 给数据库用户赋权限

  ```sql
  GRANT SELECT, INSERT, UPDATE, DELETE ON mydatabase.* TO 'testuser'@'localhost';
  FLUSH PRIVILEGES;
  ```

* 创建DB

  ```sql
  CREATE DATABASE mydb
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_general_ci;
  ```

* 创建table

  ```sql
  CREATE TABLE Customers (
      CustomerID int NOT NULL AUTO_INCREMENT,
      FirstName varchar(255),
      LastName varchar(255),
      Email varchar(255),
      PRIMARY KEY (CustomerID)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
  ```

  

## Docker

* Docker安装

  ```bash
  curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
  ```


* 查看现有容器

  ```bash
  docker ps
  ```

* 进入Docker容器

  ```bash
  docker exec -it container_id bash
  ```

  在这个命令中：

  - `-i` 选项表示以交互模式运行命令，确保你可以输入命令；
  - `-t` 选项分配一个伪终端，这对于保持输出格式正确很有帮助；
  - `container_id` 应替换为你的容器ID或名称；
  - `bash` 是你希望在容器内运行以获取交互式shell的命令。如果容器中没有bash，你可以尝试使用`sh`或其他可用的shell程序，比如：`docker exec -it <容器ID或名称> sh`。

* 宿主机和容器之间文件拷贝

  ```bash
  docker cp <容器名或ID>:<容器内文件的路径> <要复制到的宿主机的目标路径>
  docker cp my_container:/data/config.json /backup/config_backup.json
  
  docker cp <宿主机文件路径> <容器ID或名称>:<容器内文件路径>
  # 假设你有一个文件叫做example.txt，位于宿主机的/home/user/目录下，你想要将这个文件拷贝到名为mycontainer的容器的/usr/src/app目录下。那么你可以使用以下命令：
  docker cp /home/user/example.txt mycontainer:/usr/src/app/example.txt
  # 如果你想要将整个目录（假设目录名为mydir）从宿主机拷贝到容器中，你可以这样做：
  docker cp /home/user/mydir mycontainer:/usr/src/app/
  
  ```
  

## CTP硬件留痕信息

|                | Windows系统 | 参考命令                                                     |
| -------------- | ----------- | ------------------------------------------------------------ |
| 私网IP地址     |             | ipconfig /all                                                |
| 公网IP地址     |             | ipconfig /all                                                |
| 网卡MAC地址    |             | ipconfig /all                                                |
| 设备名         |             | ipconfig /all                                                |
| 操作系统版本   |             | ver                                                          |
| 硬盘序列号     |             | wmic path win32_physicalmedia get serialnumber               |
| CPU序列号      |             | wmic CPU get ProcessorID                                     |
| BIOS序列号     |             | wmic bios get serialnumber                                   |
| 系统盘分区信息 |             | vol                                                          |
|                |             |                                                              |
|                | Linux系统   | 参考命令                                                     |
| 私网IP地址     |             | ifconfig                                                     |
| 公网IP地址     |             | ifconfig                                                     |
| 网卡MAC地址    |             | ifconfig                                                     |
| 设备名         |             | hostname                                                     |
| 操作系统版本   |             | uname -a                                                     |
| 硬盘序列号     |             | 无阵列： cat /proc/scsi/scsi sudo hdparm -I /dev/sda 有阵列： sudo lshw -c disk 需要安装lshw |
| CPU序列号      |             | dmidecode -t 4 \|grep ID                                     |
| BIOS序列号     |             | dmidecode -t 1\|grep Serial                                  |
