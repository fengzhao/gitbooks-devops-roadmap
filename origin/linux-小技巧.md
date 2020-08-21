# 1、Linux SSH安全设置

## 只允许某用户从指定IP地址登陆

    sed -i '$a AllowUsers CR@192.168.1.12 root@192.168.1.12' /etc/ssh/sshd_config ;\
    systemctl restart sshd

## 修改会话保持时间

    #ClientAliveInterval 0
    #ClientAliveCountMax 3
    修改成
    ClientAliveInterval 30    #（每30秒往客户端发送会话请求，保持连接）
    ClientAliveCountMax 3     #（去掉注释即可，3表示重连3次失败后，重启SSH会话）

## 增加ssh登陆的验证次数

    MaxAuthTries 20

## 允许root用户登录

    sed -i 's/PermitRootLogin no/PermitRootLogin yes/g'  /etc/ssh/sshd_config ;\
    systemctl restart sshd

## 设置登录方式

    #AuthorizedKeysFile   .ssh/authorized_keys   //公钥公钥认证文件
    #PubkeyAuthentication yes   //可以使用公钥登录
    #PasswordAuthentication no  //不允许使用密码登录

# 2、bash不显示路径

命令行会变成-bash-3.2$主要原因可能是用户主目录下的配置文件丢失

    # 方式一
    cp -a /etc/skel/. ~
    
    # 方式二
    echo "export PS1='[\u@\h \W]\$'" >> ~/.bash_profile ;\
    source ~/.bash_profile

# 3、同时监控多个文件

    tail -f file1 file2

# 4、查看网卡

    # 方式一
    ifconfig -a
        
    # 方式二
    cat /proc/net/dev

# 5、cp目录下的带隐藏文件的子目录

    cp -R /home/test/* /tmp/test

/home/test下的隐藏文件都不会被拷贝，子目录下的隐藏文件倒是会的

    cp -R /home/test/. /tmp/test

cp的时候有重复的文件需要覆盖时会让不停的输入yes来确认，可以使用yes|

    yes|cp -r /home/test/. /tmp/test


# 6、获取出口IP地址

```bash
curl http://members.3322.org/dyndns/getip
curl https://ip.cn
curl cip.cc
curl myip.ipip.net
curl ifconfig.me
```

# 7、ISO自动挂载

    echo "/mnt/iso/CentOS-7-x86_64-Minimal-1804.iso /mnt/cdrom iso9660 defaults,loop  0 0" >> /etc/fstab && \
    mount -a && \
    df -mh

# 8、查看系统版本号和内核信息

    cat /proc/version
    uname -a
    lsb_release -a
    cat /etc/redhat-release
    cat /etc/issue
    rpm -q redhat-release

# 9、查看物理CPU个数、核数、逻辑CPU个数

CPU总核数 = 物理CPU个数 * 每颗物理CPU的核数 
总逻辑CPU数 = 物理CPU个数 * 每颗物理CPU的核数 * 超线程数

    # 查看CPU信息（型号）
    cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
    
    # 查看物理CPU个数
    cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
    
    # 查看每个物理CPU中core的个数(即核数)
    cat /proc/cpuinfo| grep "cpu cores"| uniq
    
    # 查看逻辑CPU的个数
    cat /proc/cpuinfo| grep "processor"| wc -l

# 10、Linux缓存

cached是cpu与内存间的，buffer是内存与磁盘间的，都是为了解决速度不对等的问题。buffer是即将要被写入磁盘的，而cache是被从磁盘中读出来的

- **buff**：作为buffer cache的内存，是块设备的读写缓冲区
- **cache**：作为page cache的内存，文件系统的cache。Buffer cache是针对磁盘块的缓存，也就是在没有文件系统的情况下，直接对磁盘进行操作的数据会缓存到buffer cache中。
- **pagecache**：页面缓存（pagecache）可以包含磁盘块的任何内存映射。这可以是缓冲I/O，内存映射文件，可执行文件的分页区域——操作系统可以从文件保存在内存中的任何内容。Page cache实际上是针对文件系统的，是文件的缓存，在文件层面上的数据会缓存到page cache。
- **dentries**：表示目录的数据结构
- **inodes**：表示文件的数据结构

```bash
#内核配置接口 /proc/sys/vm/drop_caches 可以允许用户手动清理cache来达到释放内存的作用，这个文件有三个值：1、2、3（默认值为0）

#释放pagecache
echo 1 > /proc/sys/vm/drop_caches

#释放dentries、inodes
echo 2 > /proc/sys/vm/drop_caches

#释放pagecache、dentries、inodes
echo 3 > /proc/sys/vm/drop_caches
```

# 11、设置代理

```bash
$> bash -c 'cat >> /etc/profile <<EOF
# HTTP协议使用代理服务器地址
export http_proxy=http://1.2.3.4:3128
# HTTPS协议使用代理服务器地址
export https_proxy=https://1.2.3.4:3128
# FTP协议使用代理服务器地址
export https_proxy=https://1.2.3.4:3128
# 不使用代理的IP或主机
export no_proxy=.abc.com,127.0.0.0/8,192.168.0.0/16,.local,localhost,127.0.0.1

export HTTP_PROXY="http://1.2.3.4:3128"
export HTTPS_PROXY="http://1.2.3.4:3128"
export NO_PROXY="192.168.0.0/16,.taobao.com,.okd311.curiouser.com"

export 
EOF' ;\
   sed -i '/^##/d' /etc/profile ;\
   source /etc/profile
```

**注意**：

- 当使用“**export http_proxy**”和“**export https_proxy**”设置代理时，curl默认所有的请求都是走的代理，请求域名不通过/etc/hosts解析。

- 所以当有需求curl命令不走代理，通过/etc/hosts解析时，代理设置要通过“**export HTTP_PROXY**”和“**export HTTPS_PROXY**”设置。（原因是url.c（版本7.39中的第4337行）处看先检查小写版本，如果找不到，则检查大写。链接：https://stackoverflow.com/questions/9445489/performing-http-requests-with-curl-using-proxy）
- **no_proxy不支持模糊匹配**。不支持`*.a.com`，支持`.a.com`

# 12、查看网卡UUID

    nmcli con | sed -n '1,2p'

# 13、时间戳与日期

## 日期与时间戳的相互转换

    #将日期转换为Unix时间戳
    date +%s
    
    #将Unix时间戳转换为指定格式化的日期时间
    date -d @1361542596 +"%Y-%m-%d %H:%M:%S"

## date日期操作

    date +%Y%m%d               #显示前天年月日
    date -d "+1 day" +%Y%m%d   #显示前一天的日期
    date -d "-1 day" +%Y%m%d   #显示后一天的日期
    date -d "-1 month" +%Y%m%d #显示上一月的日期
    date -d "+1 month" +%Y%m%d #显示下一月的日期
    date -d "-1 year" +%Y%m%d  #显示前一年的日期
    date -d "+1 year" +%Y%m%d  #显示下一年的日期

## 获得毫秒级的时间戳

在linux Shell中并没有毫秒级的时间单位，只有秒和纳秒。其实这样就足够了，因为纳秒的单位范围是（000000000..999999999），所以从纳秒也是可以的到毫秒的

    current=`date "+%Y-%m-%d %H:%M:%S"`     #获取当前时间，例：2015-03-11 12:33:41
    timeStamp=`date -d "$current" +%s`      #将current转换为时间戳，精确到秒
    currentTimeStamp=$((timeStamp*1000+`date "+%N"`/1000000)) #将current转换为时间戳，精确到毫秒
    echo $currentTimeStamp

# 14、nohup手动后台运行进程并记录进程号

```bash
nohup jar -jar jar包 </dev/null > /data/app/logs/app.log 2>&1 & echo $! > /data/app/run.pid

# 2>&1是把标准错误2重定向到标准输出1中，而标准输出又导入文件里面，所以标准错误和标准输出都会输出到文件。
# 同时把启动的进程号pid输出到文件

注意：
	如果运行时的shell为zsh，将任务放置后台的命令由”&“变为”&!“。例如：nohup jar -jar jar包 </dev/null > /data/app/logs/app.log 2>&1 &! echo $! > /data/app/run.pid
	参考：https://stackoverflow.com/questions/19302913/exit-zsh-but-leave-running-jobs-open
```

# 15、生成文件的MD值

在网络传输、设备之间转存、复制大文件等时，可能会出现传输前后数据不一致的情况。这种情况在网络这种相对更不稳定的环境中，容易出现。那么校验文件的完整性，也是势在必行的。

在网络传输时，我们校验源文件获得其md5sum，传输完毕后，校验其目标文件，并对比如果源文件和目标文件md5 一致的话，则表示文件传输无异常。否则说明文件在传输过程中未正确传输。

md5值是一个128位的二进制数据，转换成16进制则是32（128/4）位的进制值。
md5校验，有很小的概率不同的文件生成的md5可能相同。比md5更安全的校验算法还有SHA*系列的。

## **Linux的md5sum命令**

md5sum命令用于生成和校验文件的md5值。它会逐位对文件的内容进行校验。是文件的内容，与文件名无关，也就是文件内容相同，其md5值相同。

```bash
#md5sum命令的详解
$> md5sum --h
Usage: md5sum [OPTION]... [FILE]
With no FILE, or when FILE is -, read standard input.
-b, --binary         二进制模式读取文件
-c, --check          从文件中读取、校验MD5值
      --tag          创建一个BSD-style风格的校验值
-t, --text           文本模式读取文件（默认）
#校验文件MD5值使用的参数
The following four options are useful only when verifying checksums:
      --quiet          don't print OK for each successfully verified file
      --status         don't output anything, status code shows success
      --strict         exit non-zero for improperly formatted checksum lines
  -w, --warn           warn about improperly formatted checksum lines

      --help     display this help and exit
      --version  output version information and exit


#生成的MD5值重定向到文件中
$>md5sum filename > filename.md5
#生成的MD5值重定向追加到文件中
$> md5sum filename >>filename.md5
#多个文件输出到一个md5文件中，这要使用通配符*
$> md5sum *.iso > iso.md5
#同时计算多个文件的MD5值
$> md5sum filetohashA.txt filetohashB.txt filetohashC.txt > hash.md5

#校验MD5:把下载的文件file和该文件的file.md5报文摘要文件放在同一个目录下
$> md5sum -c file.md5
#创建一个BSD风格的校验值
$> md5sum --tag file.md5
MD5 (file.md5) = 9192e127b087ed0ae24bb12070f3051a
```

## **Python生成MD5值**

```bash
# 方式一：使用md5包

import md5

src = 'this is a md5 test.'
m1 = md5.new()
m1.update(src)
print m1.hexdigest()

# 方式二：使用hashlib（推荐）

import hashlib   

m2 = hashlib.md5()   
m2.update(src)   
print m2.hexdigest()

# 加密常见的问题：

1：Unicode-objects must be encoded before hashing

　　解决方案：import hashlib
　　　　　　　m2 = hashlib.md5()
　　　　　　　m2.update(src．encode('utf-8'))
　　　　　　　print m2.hexdigest()
```

## **Java生成MD5值**

```java
import java.security.MessageDigest;
public static void main(String[] args) {  
        String password = "123456";  
        try {  
            MessageDigest instance = MessageDigest.getInstance("MD5");// 获取MD5算法对象  
            byte[] digest = instance.digest(password.getBytes());// 对字符串加密,返回字节数组  
  
            StringBuffer sb = new StringBuffer();  
            for (byte b : digest) {  
                int i = b & 0xff;// 获取字节的低八位有效值  
                String hexString = Integer.toHexString(i);// 将整数转为16进制  
                // System.out.println(hexString);  
  
                if (hexString.length() < 2) {  
                    hexString = "0" + hexString;// 如果是1位的话,补0  
                }  
  
                sb.append(hexString);  
            }  
  
            System.out.println("md5:" + sb.toString());  
            System.out.println("md5 length:" + sb.toString().length());//Md5都是32位  
  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
            // 没有该算法时,抛出异常, 不会走到这里  
        }  
    }  
```

# 16、添加用户

    useradd (选项) （参数）
    
    #选项
    －c：加上备注文字，备注文字保存在passwd的备注栏中
    －d：指定用户登入时的启始目录
    －D：变更预设值
    －e：指定账号的有效期限，缺省表示永久有效
    －f：指定在密码过期后多少天即关闭该账号
    －g：指定用户所属的起始群组
    －G：指定用户所属的附加群组
    －m：自动建立用户的登入目录
    －M：不要自动建立用户的登入目录
    －n：取消建立以用户名称为名的群组
    －r：建立系统账号
    －s：指定用户登入后所使用的shell
    －u：指定用户ID号

# 17、su 与 sudo

**`su`** : switch to another user 切换用户

**`sudo`** : superuser do 允许用户使用superuser的身份执行命令

```bash
su username ：切换为username，需要输入username密码
su : 切换为root用户，需要输入root密码
su - : 切换为root用户，需要输入root密码，且环境变量也改变
su - -c "command" ：使用root身份执行命令，完成后即退出root身份
sudo command : 与su -c相似，需要输入当前用户（superuser，/etc/sudoers中指定）密码
sudo su -：使用当前用户密码实现root身份的切换
su - hdfs -c command    切换用户并以某用户的身份去执行一条命令
su - hdfs  test.sh  切换用户并以某用户的身份去执行一个shell文件
```

# 18、重新开启SELinux

如果在使用setenforce命令设置selinux状态的时候出现这个提示：`setenforce: SELinux is disabled`。那么说明selinux已经被彻底的关闭了,如果需要重新开启selinux

```bash
vi /etc/selinux/config

更改为：SELINUX=1

必须重启linux，不重启是没办法立刻开启selinux的
```

重启完以后，使用getenforce,setenforce等命令就不会报“setenforce: SELinux is disabled”了。这时，我们就可以用setenforce命令来动态的调整当前是否开启selinux。

# 19、检查软件是否已安装，没有就自动安装

```bash
rpm -qa |grep "jq"
if [ $? -eq 0 ] ;then
    echo "jq hava been installed "
else
    yum -y install epel-release && yum -y install jq
fi
```

# 20、使用privoxy代理http，https流量使用socket连接ShadowSocks服务器

```bash
echo "安装ShadowSocks" && \
yum -y install epel-release && yum -y install python-pip && pip install shadowsocks && \
bash -c 'cat > /etc/shadowsocks.json <<EOF
{
"server": "***.***.***.***",
"server_port": "443",
"local_address": "127.0.0.1",
"local_port":"1080",
"password": "******",
"timeout":300,
"method": "aes-256-cfb",
"fast_open": false
}
EOF' && \
bash -c 'cat > /etc/systemd/system/shadowsocks.service << EOF
[Unit]
Description=Shadowsocks
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
EOF' && \
  systemctl daemon-reload  && \
  systemctl enable shadowsocks && \
  systemctl start shadowsocks

yum install -y privoxy && \
sed -i 's/#        forward-socks5t   \/               127.0.0.1:9050 ./        forward-socks5t   \/               127.0.0.1:1080 ./' /etc/privoxy/config && \
privoxy --user privoxy /etc/privoxy/config && \
echo "export http_proxy=http://127.0.0.1:8118" >> /etc/profile && \
echo "export https_proxy=http://127.0.0.1:8118" >> /etc/profile && \
source /etc/profile && \
curl www.google.com
```

# 21、批量打通指定主机SSH免密钥登录脚本

**CentOS**

```bash
$> bash -c 'cat > ./HitthroughSSH.sh <<EOF
#!/bin/bash

##
#===========================================================
echo "script    usage : ./HitthroughSSH.sh hosts.txt"
echo "hosts.txt format: host_ip:root_password"

#=========================================================
echo "==Setup1:Check if cmd expect exist,if no,install automatically"
rpm -qa | grep expect 
if [ \$? -ne 0 ];then
yum install -y expect
fi
#=====================================
echo "==Setup2:Check if have been generated ssh private and public key.if no ,generate automatically "

if [ ! -f ~/.ssh/id_rsa ];then
  ssh-keygen -t rsa  -P "" -f ~/.ssh/id_rsa
fi
#===========================================================
echo "Setup3:Read IP and root password from text"
echo "Setup4:Begin to hit root ssh login without password thorough hosts what defined in the hosts.txt"
for p in \$(cat \$1)    
do   
    ip=\$(echo "\$p"|cut -f1 -d":")         
    password=\$(echo "\$p"|cut -f2 -d":")  
    expect -c "   
            spawn ssh-copy-id -i /root/.ssh/id_rsa.pub root@\$ip  
            expect {   
                \"*yes/no*\" {send \"yes\r\"; exp_continue}   
                \"*password*\" {send \"\$password\r\"; exp_continue}   
                \"*Password*\" {send \"\$password\r\";}   
            }   
        "
    ssh root@\$ip "date"
done
EOF' ;\
  sed -i -c -e '/^$/d;/^##/d' ./HitthroughSSH.sh ;\
  chmod +x ./HitthroughSSH.sh ;\
  bash -c 'cat > ./hosts.txt <<EOF
172.16.0.3:Abc@1234
172.16.0.4:Abc@1234
172.16.0.5:Abc@1234
172.16.0.6:Abc@1234
172.16.0.7:Abc@1234
EOF' ;\
  ./HitthroughSSH.sh ./hosts.txt ;\
  rm -rf ./HitthroughSSH.sh ./hosts.txt
```

# 22、硬盘自动分区，格式化，开机自动挂载到/data

```bash
disk=/dev/sdc;\
bash -c "fdisk ${disk}<<End
n
p
1


wq
End" ;\
mkfs.ext4 ${disk}1 ;\
blkid | grep ${disk}1 | cut -d ' ' -f 2 >>/etc/fstab ;\
sed -i '$ s/$/ \/data ext4 defaults 0 0/' /etc/fstab ;\
mkdir /data ;\
mount -a ;\
df -h
```

# 23、在hosts文件中添加IP地址与主机名的域名映射

```bash
ipaddr=$(ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'| sed -n '1p') && \
echo $ipaddr $HOSTNAME >> /etc/hosts
```

# 24、禁用透明大页

**Redhat**

```bash
sed -i '$a echo nerver > /sys/kernel/mm/redhat_transparent_hugepage/defrag\necho nerver > /sys/kernel/mm/redhat_transparent_hugepage/enabled'
```

**CentOS**

```bash
echo never > /sys/kernel/mm/transparent_hugepage/defrag ;\
echo never > /sys/kernel/mm/transparent_hugepage/enabled ;\
sed -i '/GRUB_CMDLINE_LINUX/ s/"$/ transparent_hugepage=never"/' /etc/default/grub ;\
grub2-mkconfig -o /boot/grub2/grub.cfg
```

# 25、安装JDK环境

**Prerequisite**：
1. JDK安装包已下载在内网HTTP服务器中

```bash
wget http://192.168.1.2/jdk/jdk-8u111-linux-x64.tar.gz;\
tar -zxvf jdk-8u111-linux-x64.tar.gz -C /opt;\
rm -rf jdk-8u111-linux-x64.tar.gz;\
ln -s /opt/jdk1.8.0_111 /opt/jdk;\
sed -i '$a export JAVA_HOME=/opt/jdk\nexport CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar\nexport PATH=$PATH:$JAVA_HOME/bin' /etc/profile;\
source /etc/profile;\
ln -s /opt/jdk/bin/java /usr/bin/java;\
java -version;\
javac -version
```

# 26、安装Tomcat，并由systemctl托管

**Prerequisite**：
1. 已安装JDK
2. Tomcat安装包已下载在内网HTTP服务器中

```bash
wget http://192.168.1.2/tomcat/apache-tomcat-8.5.20.tar.gz;\
tar -zxvf apache-tomcat-8.5.20.tar.gz -C /opt;\
rm -rf apache-tomcat-8.5.20.tar.gz;\
ln -s /opt/apache-tomcat-8.5.20 /opt/tomcat;\
bash -c 'cat > /lib/systemd/system/tomcat.service <<EOF
[unit]
Description=Tomcat
After=network.target
[Service]
Type=forking
PIDFile=/opt/tomcat/tomcat.pid
ExecStart=/opt/tomcat/bin/catalina.sh start
ExecReload=/opt/tomcat/bin/catalina.sh restart
ExecStop=/opt/tomcat/bin/catalina.sh stop
[Install]
WantedBy=multi-user.target
EOF';\
ln -s /lib/systemd/system/tomcat.service /etc/systemd/system/multi-user.target.wants/tomcat.service;\
sed -i '1a CATALINA_PID=/opt/tomcat/tomcat.pid' /opt/tomcat/bin/catalina.sh;\
systemctl daemon-reload;\
systemctl start tomcat;\
systemctl status tomcat;\
systemctl stop tomcat;\
systemctl status tomcat;\
systemctl enable tomcat;\
systemctl status tomcat
```

# 27、安装Nginx

```bash
bash -c 'cat > /etc/yum.repos.d/nginx.repo <<EOF
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/\$basearch/
gpgcheck=0
enabled=1
EOF' ;\
  yum install nginx -y
```

# 28、安装单机版的Zookeeper

**Prerequisite**：
1. 已安装JDK

```bash
version=3.4.14
curl https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/stable/zookeeper-$version.tar.gz -o /opt/zookeeper-$version.tar.gz 
tar -zxvf /opt/zookeeper-*.tar.gz -C /opt/ ;\
rm -rf /opt/zookeeper-*.tar.gz ;\
ln -s /opt/zookeeper-$version/ /opt/zookeeper ;\
sed -i '$a export ZOOKEEPER_HOME=/opt/zookeeper\nexport PATH=$PATH:$ZOOKEEPER_HOME/bin' /etc/profile ;\
source /etc/profile ;\
mv /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg  ;\
sed -i -e '/dataDir/d' -e '/dataLogDir/d' /opt/zookeeper/conf/zoo.cfg ;\
sed -i -e '$a dataDir=/data/zookeeper/data\ndataLogDir=/data/zookeeper/logs\nserver.1=127.0.0.1:2888:3888\nautopurge.purgeInterval=24\nautopurge.purgeInterval=5' /opt/zookeeper/conf/zoo.cfg ;\
mkdir -p /data/zookeeper/{data,logs} ;\
echo "1" > /data/zookeeper/data/myid ;\
zkServer.sh start ;\
zkServer.sh status ;\
jps -l
```

# 29、安装单机版的Kafka

**Prerequisite**：
1. 已安装Zookeeper

```bash
version=2.12-2.2.0
curl https://mirrors.tuna.tsinghua.edu.cn/apache/kafka/2.2.0/kafka_$version.tgz -o /opt/kafka_$version.tgz ;\
tar -zxvf /opt/kafka_$version.tgz -C /opt;\
rm -rf /opt/kafka_$version.tgz ;\
ln -s /opt/kafka_$version /opt/kafka ;\
sed -i '$a export KAFKA_HOME=/opt/kafka\nexport PATH=$PATH:$KAFKA_HOME/bin' /etc/profile ;\
source /etc/profile ;\
sed -i -e 's/log.dirs=\/tmp\/kafka\/logs/log.dirs=\/data\/kafka\/logs/g' -e 's/log.retention.hours=168/log.retention.hours=1/g' -e '$a auto.create.topics.enable=true\ndelete.topic.enable=true'  /opt/kafka/config/server.properties ;\
mkdir -p /data/kafka/{logs,data} ;\
kafka-server-start.sh -daemon /opt/kafka/config/server.properties ;\
jps -l
```

# 30、安装Hadoop客户端

以hadoop 2.8.3版本为例

```bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.8.3/hadoop-2.8.3.tar.gz ;\
tar -xvf hadoop-2.8.3.tar.gz -C /opt ;\
rm -rf hadoop-2.8.3.tar.gz ;\
ln -s /opt/hadoop-2.8.3 /opt/hadoop ;\
sed -i '$a export HADOOP_HOME=/opt/hadoop\nexport PATH=$PATH:$HADOOP_HOME/bin' /etc/profile ;\
source /etc/profile
#然后在/opt/hadoop-2.8.3/etc/hadoop/core-site.xml配置文件<configuration>标签中填写HDFS NameNode节点的IP地址及端口号
<property>
   <name>fs.default.name</name>
   <value>hdfs://172.16.3.10:9000</value>
   <description> </description>
</property>


hdfs dfs -ls /
```

# 31、安装Maven环境

```bash
curl https://mirrors.tuna.tsinghua.edu.cn/apache/maven/binaries/apache-maven-3.2.2-bin.tar.gz -o /opt/apache-maven-3.2.2-bin.tar.gz && \
tar -zxvf /opt/apache-maven-*.tar.gz -C /opt/ && \
rm -rf /opt/apache-maven-*.tar.gz && \
ln -s /opt/apache-maven-3.2.2 /opt/maven && \
sed -i '$a export M2_HOME=/opt/maven\nexport PATH=$PATH:$M2_HOME/bin' /etc/profile && \
source /etc/profile && \
mvn version
```

# 32、安装NodeJS环境

```bash
wget https://nodejs.org/dist/v8.9.4/node-v8.9.4-linux-x64.tar.xz ;\
tar -xvf node-v8.9.4-linux-x64.tar.xz -C /opt/ ;\
rm -rf node-v8.9.4-linux-x64.tar.xz ;\
ln -s /opt/node-v8.9.4-linux-x64 /opt/nodejs ;\
sed -i '$a export NODEJS_HOME=/opt/nodejs\nexport PATH=$PATH:$NODEJS_HOME/bin' /etc/profile;\
source /etc/profile;\
yum install gcc-c++ make -y;\
npm config set registry https://registry.npm.taobao.org ;\
npm config set sass_binary_site https://npm.taobao.org/mirrors/node-sass/ ;\
npm version
```

# 33、安装Docker，并设置新硬盘LVM成docker的数据目录

```bash
wget https://download.docker.com/linux/centos/docker-ce.repo -O  /etc/yum.repos.d/docker-ce.repo ;\
yum makecache ;\
yum install docker-ce-17.12.1.ce -y ;\
systemctl enable docker ;\
mkdir /etc/docker ;\
touch /etc/docker/daemon.json ;\
bash -c ' tee  /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://0gxg9a07.mirror.aliyuncs.com"],
  "insecure-registries": ["0.0.0.0/0"]
}
EOF' ;\
systemctl daemon-reload ;\
disk=/dev/sdc ;\
yum install -y lvm2 ;\
pvcreate ${disk} ;\
vgcreate -s 4M docker ${disk} ;\
PE_Number=`vgdisplay ${disk}|grep "Free  PE"|awk '{print $5}'` ;\
lvcreate -l ${PE_Number} -n docker-lib docker ;\
mkfs.xfs /dev/docker/docker-lib ;\
mkdir /var/lib/docker ;\
echo "/dev/docker/docker-lib /var/lib/docker/ xfs defaults 0 0" >> /etc/fstab ;\
df -mh ;\
systemctl start docker ;\
ls /var/lib/docker/ ;\
docker info |grep "Insecure Registries:" -A 4
```

# 34、字符转换命令expand/unexpand

用于将文件的制表符（Tab）转换为空格符（Space），默认一个Tab对应8个空格符，并将结果输出到标准输出。若不指定任何文件名或所给文件名为”-“，则expand会从标准输入读取数据。

功能与之相反的命令是unexpand，是将空格符转成Tab符。

vi/vim在命令模式下通过设置":set list"可显示文件中的制表符“^I”

**expand命令参数**

```bash
-i, --initial       do not convert tabs after non blanks
-t, --tabs=NUMBER   have tabs NUMBER characters apart, not 8
-t, --tabs=LIST     use comma separated list of explicit tab positions
    --help     display this help and exit
    --version  output version information and exit
```

**unexpand命令参数**

```bash
-a, --all        convert all blanks, instead of just initial blanks
    --first-only  convert only leading sequences of blanks (overrides -a)
-t, --tabs=N     have tabs N characters apart instead of 8 (enables -a)
-t, --tabs=LIST  use comma separated LIST of tab positions (enables -a)
    --help     display this help and exit
    --version  output version information and exit
```

**实例**

将文件中每行第一个Tab符替换为4个空格符，非空白符后的制表符不作转换

```bash
#使用"----"或"--"代表一个制表符，使用":"代表一个空格
----abcd--e

$ expand -i -t 4 old-file > new-file

::::abcd--e
```

**注意**

不是所有的Tab都会转换为默认或指定数量的空格符，expand会以对齐为原则将Tab符替换为适当数量的空格符，替换的原则是使后面非Tab符处在一个物理Tab边界（即Tab size的整数倍。例如：

```bash
#使用"----"或"--"代表一个制表符，使用":"代表一个空格
abcd----efg--hi

$ expand -t 4 file

abcd::::efg::hi
```

# 35、修改时区

1. Docker容器中

   - 添加环境变量：TZ = Asia/Shanghai

2. Linux主机

   ```bash
   timedatectl set-timezone "Asia/Shanghai"
   # 设置时区
   timedatectl status 
   # 查看当前的时区状态
   date -R
   # 查看时区
   ```

   或者
   
   ```bash
   cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   ```
# 36、shell脚本的调试

  - 在脚本运行时添加`-x`参数    
  - 在脚本中开头添加`set -x`

# 37、删除“-”开头的文件或文件夹

当直接使用`rm -f`删除以`-`开头的文件与文件夹时，rm或其他命令报参数错误，会误认为`-`后面的内容是命令的参数

```bash
rm  -rf -- -XGET

cd -- -XGET
```

# 38、硬盘快速分区

## 方式一：使用parted命令

parted命令详解：https://www.cnblogs.com/Cherry-Linux/p/10103172.html

 ```bash
disk=/dev/vdb && \
parted -s -a optimal $disk mklabel gpt -- mkpart primary ext4 1 -1
 ```

## 方式二：使用fdisk

```bash
disk=/dev/vdb && \
bash -c "fdisk ${disk}<<End
n
p
1


wq
End"
```

# 39、别名传参

别名并不能直接传参，但是可以使用以下方式代替：

## 方式一：使用functions替代

```bash
$ test () { num=${1:-5} dmesg |grep -iw usb|tail -$num }
$ test 5

```

## 方式二：使用read读取输入，然后使用变量替换命令中的参数

```bash
$ alias taila='{ IFS= read -r line_num && tail -n $line_num /var/logs/message ;} <<<'
$ taila 50
```

参考：

1. https://askubuntu.com/questions/626458/can-i-pass-arguments-to-an-alias-command
2. https://www.kutu66.com//ubuntu/article_158110

# 40、Ubuntu/Debian的镜像源URL字段

Nexus设置apt proxy仓库，代理http://archive.ubuntu.com/ubuntu/

```bash
deb http://192.168.1.6:8080/repository/apt-ubuntu/ bionic main restricted
deb http://192.168.1.6:8080/repository/apt-ubuntu/ bionic-security main restricted
deb http://192.168.1.6:8080/repository/apt-ubuntu/ bionic-updates main restricted
deb http://192.168.1.6:8080/repository/apt-ubuntu/ bionic-proposed main restricted
deb http://192.168.1.6:8080/repository/apt-ubuntu/ bionic-backports main restricted
```

**第一字段，指示包类型。**

- **deb**：二进制包
- **deb-src**：源码包

**第二字段，指示镜像站点，即「源」！**
URL 定位到某个目录，该目录下必有「dists」「pool」两个子目录。如：

- http://ftp.cn.debian.org/debian/
- http://ftp.sjtu.edu.cn/ubuntu/

**第三字段，指示包的「版本类型」，姑且称为「仓库」。**
打开某源，进入「dists」子目录可见该源中有哪些仓库，即其下诸子目录。命名形式为「系统发行版名-仓库名」，如 Debian 的「jessie-backports」「stretch-updates」，Ubuntu 的「vivid-updates」「wily-proposed」。无仓库名的即为主仓库。

Debian 的 stable、testing 为链接，指向具体系统发行版，会随时间而变。比如，当前 stable 为 jessie，所以 stable-backports 与 jessie-backports 等效。但本人不建义使用 stable、testing，因为下一个 stable 发布后，你的源便自动指向了一个新版本，然而你并未阅读新版本的发行说明，并未做好升级的准备。

Debian 的仓库自 squeeze 起与 Ubuntu 基本相同。除主仓库外，有：

- **security**：Ubuntu 用于指安全性更新。即影响系统安全的 bug 修补。Debian 特殊一些，见下文。
- **updates**：非安全性更新。即不影响到系统安全的 bug 修补。
- **proposed-updates**：预更新。小 beta 版。过后会进入「updates」或「security」。Ubuntu 仅用proposed」，无后缀「updates」。
- **backports**：后备。Debian stable 发布后，Ubuntu 某版本正式发布后，其所有软件版本号便已被冻结，所有软件只修 bug，不增加任何特性。但有人可能需要新特性，甚至某些较新的软件原来根本就没有。该仓库正因此而设，但欠官方维护，且可能在系统正式发布之后过一段时间才有内容。此仓库处于第二优先顺序，而上述几个仓库处于第一优先顺序。安装第二优先顺序的包必须特别指明，见 apt-get(8) aptitude(8) 的 --target-release 选项。
  **提示：**并非所有版本都设有上述全部仓库，请打开源中 dists 目录查看。

**后续字段，指示包许可类型。**
后续字段排名不分先后，最终结果取其并集。按包本身的许可及所直接依赖的包的许可划分。打开某仓库，可见几个子目录。
Debian 最多有三种

- **main**：本身是自由软件，且所有依赖的包也都是自由软件，此类可称纯自由软件，见 https://www.debian.org/distrib/packages《Debian自由软件指导方针》。

- **contrib**：本身是自由软件，但依赖不纯，即依赖中至少有一例 contrib 或 non-free 者。

- **non-free**：本身并非自由软件，无论依赖如何。当然，该软件是可免费使用或试用的。免费一例 https://packages.debian.org/jessie/unrar，试用xx天一例 https://packages.debian.org/jessie/rar。

Ubuntu 最多有四种

- **main**：官方维护的自由软件。

- **universe**：社区维护的自由软件。

- **restricted**：设备专有驱动。

- **multiverse**：同 Debian 的「non-free」。

  某些另类的第三方源，未必遵循上述惯例。总之，打开仓库目录自己看。


特别之处：

**Debian 安全性更新**
不像 Ubuntu 放在「security」仓库，而是放在单独一个源中。各大镜像站通常都把一般的包放在根下来一级的「debian」目录中，而安全性更新则会放在「debian-security」目录中，如果有的话，如 http://ftp.cn.debian.org/debian-security/。
Debian 官方建议，所有安全性更新，只从官方主站更新，勿使用其它镜像站，除非你对镜像站非常有信心，见 https://www.debian.org/security/index.en.html。所以，很多镜像站并不提供安全更新源。
安全性更新的第三字段形式固定为「版本名/updates」，如「wheezy/updates」「jessie/updates」。

**Debian 多媒体源**
一些多媒体软件因牵涉到版权问题，包括硬件解码器，Debian 官方并未收录，有一网站专门填补该空缺，见 [http://www.deb-multimedia.org](https://www.deb-multimedia.org/)。


最后忠告：
不要同时启用多个源，同一仓库的源启用一个即可，否则容易引起混乱。以下实例便是列有多套而仅启用一套。

## 参考

https://forum.ubuntu.org.cn/viewtopic.php?t=366506

# 41、裸磁盘分区扩容

①停掉向挂载路径写文件的服务或进程

② 卸载挂载

```bash
umount /data
```

如果提示`umount:/data:target is bus`,使用`fuser`找出正在往挂载路径写文件的进程并kill掉，再次卸载挂载

```bash
yum install psmisc -y
fuser -mv /data
                     USER        PID ACCESS COMMAND
/data:                root     kernel mount /data
                     root      13830 ..c.. bash
```

③修复分区表

磁盘扩大容量后，分区表中记录的柱头等信息需要更新，否则创建新分区时会报`GPT PMBR size mismatch`

```bash
parted -l
在弹出Fix/Ignore?的提示时输入Fix后回车即可。
```

④删掉旧分区再重建新分区

```bash
fdisk /dev/sdb
	d			# 删除原来的分区/dev/sdb1
	n			#	创建新的分区		
	1			# 分区号与旧的保持一致
	w			# 写入分区表并生效
```

⑤调整分区

```bash
e2fsck -f /dev/sdb1 检查分区信息
resize2fs /dev/sdb1 调整分区大小
```

⑥重新挂载并验证数据是否丢失？容量是否扩容？

# 42、MacOS下`tar`归档文件时,排错`._*`文件

MacOS下的`tar`命令，在归档压缩文件或文件夹时，会产生`._*`的隐藏文件()也一并归档到压缩包中，增加压缩包体积。可以在归档时不包含这些文件

```bash
COPYFILE_DISABLE=1 tar czf test.tar /your/files
# 去除旧压缩包中的“._*”文件
tar -cf newTar --include='some/path/*' oldTar
```

参考：

1. https://stackoverflow.com/questions/30962501/how-do-i-delete-a-single-file-from-a-tar-gz-archive
2. https://superuser.com/questions/259703/get-mac-tar-to-stop-putting-filenames-in-tar-archives

# 43、echo 换行

```bash
echo -e "test\ndasdasd" > test
```

# 44、安装Docker

```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum list docker-ce --showduplicates | sort -r
yum install -y docker-ce-18.06.3.ce docker-compose
bash -c 'cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com"
  ],
  "max-concurrent-downloads": 10,
  "log-driver": "json-file",
  "log-level": "warn",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
    },
  "data-root": "/var/lib/docker"
}
EOF'
systemctl enable docker
systemctl start docker
docker info
```

# 45 、生成随机字符串

```bash
# 根据时间戳加随机数计算md5值并取前10位
echo $(date +%s)$RANDOM | md5sum | head -c 10

head -c 16 /dev/random | base64

openssl rand -hex 10

cat /proc/sys/kernel/random/uuid| cksum |cut -f1 -d" " | base64

head -n 5 /dev/urandom |sed 's/[^a-Z0-9]//g'|strings -n 4

tr -dc '_A-Z#\-+=a-z(0-9%^>)]{<|' </dev/urandom | head -c 15; echo
```

# 46、使用curl命令发送邮件

```bash
curl -s --ssl-reqd --write-out %{http_code} --output /dev/null \
  --url "smtp://发件人SMTP服务器地址:发件人SMTP服务器端口" \
  --user "发件人SMTP服务器用户名:发件人SMTP服务器密码" \
  --mail-from 发件人邮箱地址 \
  --mail-rcpt 收件人邮箱地址 \
  --upload-file /tmp/emai-data.txt
  
# /tmp/emai-data.txt的内容

FROM: 发件人邮箱地址
To: 收件人邮箱地址
CC: 抄送人邮箱地址
Subject: 主题

Content-Type: multipart/mixed; boundary=MixedBoundary

--MixedBoundary
Content-Type: multipart/related; boundary=AlternativeBoundary

--AlternativeBoundary
Content-Type: multipart/related; boundary=RelatedBoundary

--RelatedBoundary
Content-Type: text/html; charset=utf-8

<html>
<body>
<h1>测试<h1>
</body>
</html>

--RelatedBoundary--

--AlternativeBoundary--

--MixedBoundary
Content-Type: text/plain; name=test.txt
Content-Transfer-Encoding: base64
Content-Disposition: attachment; filename=test.txt

[base64编码的附件内容]

--MixedBoundary--
```

参考：

1. https://skeletonkey.com/filemaker-18-smtp-curl/

2. https://www.soliantconsulting.com/blog/html-email-filemaker/

3. https://stackoverflow.com/questions/44728855/curl-send-html-email-with-embedded-image-and-attachment

   