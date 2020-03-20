# **Ubuntu基于OpenSSh的SFTP权限快速搭建**

### 说明

默认SFTP基于系统用户. 一般通过xshell连接的用户都可以连接到sftp.但是这种用户基本可以操作任何用户,权限方面没有限制.

改文章即提供SFTP相关权限的控制的设置.

### 系统环境

#### ssh-version

```
ssh -V
```

OpenSSH_7.2p2 Ubuntu-4ubuntu2.2, OpenSSL 1.0.2g 1 Mar 2016

**保证OpenSSH版本至少4.8p1,因为配置权限需要版本添加的新配置项 ChrootDirectory 来完成。**

#### Ubuntu版本

cat /proc/version

Linux version 4.4.0-97-generic (buildd@lcy01-33) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) ) #120-Ubuntu SMP Tue Sep 19 17:28:18 UTC 2017

cat /etc/issue

Ubuntu 16.04.3 LTS \n \l

### 操作步骤

#### 切换到ROOT用户

1. 创建SFTP用户组

```bash
groupadd sftpusers
```

1. 创建用户
2. useradd -s /bin/false -G sftpusers user

**/bin/fasle指定用户没有登录shell的权限**

1. 设置用户密码
2. passwd passwd
3. 配置ssh权限
4. vim /etc/ssh/sshd_config
5. 4.1. 修改以下内容

```bash
#Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

1. 4.2. 在尾部添加权限

```bash
# 匹配用户组，如果要匹配多个组，多个组之间用逗号分割
Match Group sftpusers
# 指定登陆用户到自己的用户目录
# 开放Story
ChrootDirectory /data/test
# 指定 sftp 命令
ForceCommand internal-sftp
# 这两行，如果不希望该用户能使用端口转发的话就加上，否则删掉
X11Forwarding no
AllowTcpForwarding no
```

为什么实用 internal-sftp 而不用默认的 sftp-server，这是因为：

这是一个进程内的 sftp 服务，当用户 ChrootDirectory 的时候，将不请求任何文件；

更好的性能，不用为 sftp 再开一个进程。

- 修改数据目录权限
- *假设目录为**/data/test*
- /data/test`必须为root用户, 且目录权限为755
- /data/test的上层目录/data也必须为root用户, 且目录权限为755
- **操作如下**

```bash
chown -R root:root /data
chmod -R 755 /data
```

- 添加上传目录

```bash
mkdir /data/test/upload
chown -R test:sftpusers /data/test/upload
chmod -R 755 /data/test/upload
```

- 重启SSH服务

```bash
service sshd restar
```

- 查看 sshd 服务状态 

```bash
service sshd status
```