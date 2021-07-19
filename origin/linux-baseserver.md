# 一、NFS

## 1、安装

```
yum install -y nfs-utils rpcbind ;\
    systemctl enable nfs ;\
    systemctl enable rpcbind ;\
    systemctl start nfs ;\
    systemctl start rpcbind
```

## 2、配置

```
vi /etc/exports
要共享的目录 指定客户端IP地址1(权限) 指定客户端IP地址2(权限)

/NFS 172.16.2.0/24(ro,rw)
/NFS 172.16.2.0/24(ro,rw) 192.168.0.2(ro,rw)
```

客户端IP地址的指定方式： 

```
指定ip地址的主机：192.168.0.100
指定子网中的所有主机：192.168.0.0/24 或 192.168.0.0/255.255.255.0
指定域名的主机：nfs.test.com
指定域中的所有主机：*.test.com
所有主机：*
```

权限 

```
ro：共享目录只读；
rw：共享目录可读可写；
all_squash：所有访问用户都映射为匿名用户或用户组；
no_all_squash（默认）：访问用户先与本机用户匹配，匹配失败后再映射为匿名用户或用户组；
root_squash（默认）：将来访的root用户映射为匿名用户或用户组；
no_root_squash：来访的root用户保持root帐号权限；
anonuid=<UID>：指定匿名访问用户的本地用户UID，默认为nfsnobody（65534）；
anongid=<GID>：指定匿名访问用户的本地用户组GID，默认为nfsnobody（65534）；
secure（默认）：限制客户端只能从小于1024的tcp/ip端口连接服务器；
insecure：允许客户端从大于1024的tcp/ip端口连接服务器；
sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；
async：将数据先保存在内存缓冲区中，必要时才写入磁盘；
wdelay（默认）：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率；
no_wdelay：若有写操作则立即执行，应与sync配合使用；
subtree_check（默认） ：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限；
no_subtree_check ：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；
```

## 3、生效配置 

```
exportfs -a
```

## 4、客户端配置

### ①客户端安装

```
yum install nfs-utils rpcbind
```

### ②手动挂载

```
mount -t nfs Server_IP:Share_Dir  Mount_Dir 
```

### ③自动挂载

```
echo "Server_IP:Share_Dir Mount_dir nfs 权限 0 1" >> /etc/fstab
mount -a
```

### ④查看服务端共享的目录 

```
showmount -e Server_IP
```

## 5、卸载出现“设备正忙”的挂载

```bash
# 找到哪个进程正在使用挂载点的文件
lsof | grep '挂载点' 
# 查看使用挂载点文件的进程命令，如果无关紧要，可以直接杀掉
kill -9 对应的进程号
```



# 二、FTP的虚拟用户

 vsftpd服务器同时支持匿名用户、本地用户和虚拟用户三类用户账号，使用虚拟用户账号可以提供集中管理的FTP根目录，方便了管理员的管理，同时将用于FTP登录的用户名、密码与系统用户账号区别开，进一步增强了FTP服务器的安全性。

- 设置不使用ftp所在主机的系统用户及匿名用户登录
- 限制登录的虚拟用户只在指定的目录下操作
- 方便添加虚拟用户

## 1、安装

```bash
yum install -y vsftpd
vsftpd -version
```

## 2、配置

创建系统用户，所有的虚拟用户都对应到这个用户

```bash
useradd -d /data/ftp/ -s /sbin/nologin vftpuser 
```

创建虚拟用户的账号密码文本

```bash
sehll >vi /etc/vsftpd/vuser.txt
testuser
1234test
#奇数行是用户名，偶数是密码

#生成虚拟数据库文件（*如果db_load没有安装，yum install db4-utils db4-devel db4-4.3安装才能使用。）
db_load –T –t hash –f /etc/vsftpd/vuser.txt  /etc/vsftpd/vuser.db
#修改虚拟数据库文件vuser.db的权限为 700
chmod   700  /etc/vsftpd/vuser.db  

#配置PAM文件，目的是对客户端进行验证.编辑/etc/pam.d/vsftpd文件，注释所有内容，后添加：
vi /etc/pam.d/vsftpd
auth                 required     pam_userdb.so   db=/etc/vsftpd/vuser  
account              required     pam_userdb.so   db=/etc/vsftpd/vuser  
#不能写成db=/etc/vsftpd/vuser.db
```

配置vsftp的配置文件`/etc/vsftpd/vsftpd.conf`

```bash
anonymous_enable=NO       #是否允许匿名用户登录
local_enable=YES           #是否允许vsftp所在主机的本地用户登录
listen=YES
listen_ipv6=NO
guest_enable=YES           #激活虚拟账户  
guest_username=vftpuser    #把虚拟账户绑定为系统账户vftpuser   
pam_service_name=vsftpd    #使用PAM验证，指定PAM配置文件，文件已经在/etc/pam.d/存在（第二步配置的）
user_config_dir=/etc/vsftpd/vsftpd_user_conf  #设置虚拟用户的配置文件目录，配置文件名与虚拟用户名同名
write_enable=NO
```

配置虚拟用户的配置文件`/etc/vsftpd/vsftpd_user_conf/testuser`

```bash
anon_world_readable_only=NO        #浏览FTP目录
anon_upload_enable=YES            #允许上传
anon_mkdir_write_enable=YES        #建立和删除目录  
anon_other_write_enable=YES    #改名和删除文件
local_root=/data/ftp/test   #指定虚拟用户在系统用户下面的路径，限制虚拟用户家目录，虚拟用户登录后主目录
write_enable=YES                #启用/禁止用户的写权限
allow_writeable_chroot=YES
download_enable=NO							#设置只能上传不能下载
cmds_denied=DELE								#禁用掉删除DELE命令
#每行配置项结尾不能有空格
```

创建配置文件中设置的目录并设置相关权限

```bash
mkdir -p /data/ftp/test ;\
chown -R vftpuser:vftpuser /data/ftp/test ;\
chmod -R 770 /data/ftp/test
```

## 3、启动验证

```bash
systemctl start vsftpd
systemctl status vsftpd -l

ftp 127.0.0.1
> ls
> put local_file_path ftp_file_path
```

## 4、配置中出现的错误

### ①`refusing to run with writable root inside chroot ()`

限定了用户不能跳出其主目录之后，使用该用户登录FTP时往往会遇到这个错误

```
500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
```

从2.3.5之后，vsftpd增强了安全检查，如果用户被限定在了其主目录下，则该用户的主目录不能再具有写权限了！如果检查发现还有写权限，就会报该错误。

 要修复这个错误，可以用命令`chmod a-w /home/user`去除用户主目录的写权限（注意把目录替换成你自己的）。或者你可以在vsftpd_user_conf下的虚拟用户配置文件中增加下列一项：`allow_writeable_chroot=YES`

### ②`530 login incrrect`无法登陆。 

查看日志`tail -f /var/log/secure或者systemctl status vsftpd -l`

```
PAM unable to dlopen(/lib/security/pam_userdb.so): /lib/security/pam_userdb.so: cannot open shared object file: No such file or directory
```

原来pam_userdb.so在`/lib64/security/pam_userdb.so`,修改`/etc/pam.d/vsftpd`

```bash
#将下列内容
auth    required      /lib/security/pam_userdb.so     db=/etc/vsftpd/vuser
account required      /lib/security/pam_userdb.so     db=/etc/vsftpd/vuser
#替换成
auth    required      /lib64/security/pam_userdb.so     db=/etc/vsftpd/vuser
account required      /lib64/security/pam_userdb.so     db=/etc/vsftpd/vuser
#或者
auth    required     pam_userdb.so   db=/etc/vsftpd/vuser  
account required     pam_userdb.so   db=/etc/vsftpd/vuser 
```

 保存重启vsftpd服务。

### ③`425 Security: Bad IP connecting`

当使用非21端口进行端口转发连接的话，会出现上述情况。

解决方案：

```bash
1.#vim /etc/vsftpd/vsftpd.conf 
2.添加：pasv_promiscuous=YES 
3.保存后退出 
4.重启vsftpd #service vsftpd restart

#pasv_promiscuous选项参数说明：
此选项激活时，将关闭PASV模式的安全检查。该检查确保数据连接和控制连接是来自同一个IP地址。小心打开此选项。此选项唯一合理的用法是存在于由安全隧道方案构成的组织中。默认值为NO。 
合理的用法是：在一些安全隧道配置环境下，或者更好地支持FXP时(才启用它)。
```

 