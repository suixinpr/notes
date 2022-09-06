# Samba实现共享文件并管理仓库
### 安装Samba
1. 检查Samba是否已经安装完成
```bash
rpm -qa | grep samba
```
如果已经安装则跳过下一步
2. 通过yum直接安装Samba服务端和客户端
```bash
yum -y install samba samba-client
```
### 配置环境
1. 创建samba用户，本例子的用户名字为share，可以更改
```bash
# 首先使用useradd在系统中创建用户
useradd share
# 将系统用户加入smb中，并输入密码
smbpasswd -a share
```
> smbpasswd -a username #添加用户
> smbpasswd -x username #删除用户
> smbpasswd -d username #冻结用户
> smbpasswd -n username #密码置空

2. 创建共享目录

创建目录的位置为~/shared，地址可以更改，但是一定要在之前所创的用户所在的目录下，其他用户的目录下是访问不到的！
```bash
#切换至share目录下
su share
#创建共享目录并赋予权限
mkdir shared
chmod 777 shared
```

3. 修改配置文件
```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.back
vi /etc/samba/smb.conf
```
在末尾添加上如下代码后保存并退出
```
[share]
      path = /home/share/shared  // 这里需要修改成你的用户名和共享文件夹名
      available = yes
      browseable = yes
      public = yes
      writable = yes
```
配置文件详解
>共享参数 [共享名]:
>* comment = 任意字符串
>说明：comment是对该共享的描述，可以是任意字符串。
>* path = 共享目录路径
>说明：path用来指定共享目录的路径。可以用%u、%m这样的宏来代替路径里的unix用户和客户机的Netbios名，用宏表示主要用于[homes]共享域。例如：如果我们不打算用home段做为客户的共享，而是在/home/share/下为每个Linux用户以他的用户名建个目录，作为他的共享目录，这样path就可以写成：path = /home/share/%u; 。用户在连接到这共享时具体的路径会被他的用户名代替，要注意这个用户名路径一定要存在，否则，客户机在访问时会找不到网络路径。同样，如果我们不是以用户来划分目录，而是以客户机来划分目录，为网络上每台可以访问samba的机器都各自建个以它的netbios名的路径，作为不同机器的共享资源，就可以这样写：path = /home/share/%m 。
>* browseable = yes/no
>说明：browseable用来指定该共享是否可以浏览。
>* writable = yes/no
说明：writable用来指定该共享路径是否可写。
>* available = yes/no
说明：available用来指定该共享资源是否可用。
>* admin users = 该共享的管理者
>说明：admin users用来指定该共享的管理员（对该共享具有完全控制权限）。在samba 3.0中，如果用户验证方式设置成“security=share”时，此项无效。
>例如：admin users =bobyuan，jane（多个用户中间用逗号隔开）。
>* valid users = 允许访问该共享的用户
>说明：valid users用来指定允许访问该共享资源的用户。
>例如：valid users = bobyuan，@bob，@tech（多个用户或者组中间用逗号隔开，如果要加入一个组就用“@+组名”表示。）
>* invalid users = 禁止访问该共享的用户
>说明：invalid users用来指定不允许访问该共享资源的用户。
>例如：invalid users = root，@bob（多个用户或者组中间用逗号隔开。）
>* write list = 允许写入该共享的用户
>说明：write list用来指定可以在该共享下写入文件的用户。
>例如：write list = bobyuan，@bob
>* public = yes/no
>说明：public用来指定该共享是否允许guest账户访问。
>* guest ok = yes/no
>说明：意义同“public”。
4. 启动服务(以及开机自启)
```bash
/bin/systemctl start nmb
/bin/systemctl start smb
systemctl enable smb.service
```
5. 查看samba服务状态
```bash
/bin/systemctl status smb.service
```
### Windows连接
WIN + R 在运行中输入 \\\\172.16.xx.xx，然后输入share以及创建share时设置的密码，进入界面并选中映射网络驱动器，然后建立映射。到这里就完成了通过Samba来实现共享文件!