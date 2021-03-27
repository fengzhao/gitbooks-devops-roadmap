# 1. CMD下的换行符

    在CMD下,可以用^作为换行符,类似于Linux下的\

# 2. CMD下查看端口使用情况

    netstat -ano |findstr 8080

# 3. CMD下杀掉进程

    taskkill /pid 8080 -t -f

# 4. CMD下校验文件的MD5、SHA1、SHA256值

    certutil -hashfile yourfilename.ext MD5
    certutil -hashfile yourfilename.ext SHA1
    certutil -hashfile yourfilename.ext SHA256

# 5. CMD下激活windows系统

以管理员身份运行CMD

卸载之前的激活密钥

    slmgr -upk

设置KMS服务器

    slmgr -skms KMS服务器

常用的KMS服务器

    kms.03k.org
    kms.chinancce.com
    kms.lotro.cc
    cy2617.jios.org
    kms.shuax.com
    kms.luody.info
    kms.cangshui.net
    zh.us.to
    122.226.152.230
    kms.digiboy.ir
    kms.library.hk
    kms.bluskai.com

输入新的密钥

    slmgr -ipk 激活密钥

密钥

    win10专业版密钥
    W269N-WFGWX-YVC9B-4J6C9-T83GX

激活

    slmgr -ato

# 6. PowerShell下载文件

    $client = new-object System.Net.WebClient
    $client.DownloadFile('#1', '#2')
    # #1为下载链接 #2为文件保存的路径

**`Note`**：
- 一定要在路径中写上保存的新文件的全名（包括后缀）
- 建议保存的文件格式与下载的文件格式一致

# 7. 离线安装.NET Framework 3.5

**`Preflight`**
- windows 10 的系统ISO镜像
- 以管理员身份运行的CMD


将ISO镜像中source/sxs目录拷贝到某个路径下（以桌面为例）

![](../assets/windows-小技巧-1.png)

在以管理员身份运行的CMD执行以下命令
    
    dism.exe /online /enable-feature /featurename:netfx3 /Source:C:\Users\user\Desktop\sxs

![](../assets/windows-小技巧-2.png)

# 8. 添加开机自启动bat脚本

**`方法一`**：（推荐）

    将脚本放置“C:\Users\Curiouser\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup”路径下

**`方法二`**：

![](../assets/windows-小技巧-3.png)
![](../assets/windows-小技巧-4.png)
![](../assets/windows-小技巧-5.png)

# 9. 修改远程桌面的默认端口3389

Windows+R,输入regedit，打开注册表，修改一下注册表的值(十进制)，然后重启远程桌面

    HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\PortNumber

防火墙放行新指定的远程桌面端口

# 10. 防火墙放行指定端口

![](../assets/windows-小技巧-6.png)
![](../assets/windows-小技巧-7.png)
![](../assets/windows-小技巧-8.png)
![](../assets/windows-小技巧-9.png)

# 11. CMD下的用户管理

- `net user`：查看目前系统存在的用户
   `net user username`：查看用户的详细信息
- `whoami`：查看计算机当前登陆的用户
- `query user`：查看已登陆用户的详细信息
- `logoff+空格+ID号`：注销用户
- `net user 用户名 密码 /add`：新增本地用户
- `net localgroup administrators 用户名 /add`：将本地用户加入管理员用户组
- `net user 用户名 /del`：删除用户
- `runas /user:用户 cmd`：以某个用户运行命令

# 12. Windows软件授权管理工具slmgr命令

![](../assets/windows-小技巧-10.png)
![](../assets/windows-小技巧-11.png)
![](../assets/windows-小技巧-12.png)
![](../assets/windows-小技巧-13.png)
![](../assets/windows-小技巧-14.png)

# 13、Window下类Unix终端Cygwin

官网下载地址：https://cygwin.com/install.html

注意：在安装时，会让选择预下载的软件，记得预下载`lynx、wget、curl、zsh`

![](../assets/windows-小技巧-15.png)

## ①安装apt-cyg包管理器

apt-cyg是Cygwin下类似于apt的包管理器，可安装Github 地址：https://github.com/transcode-open/apt-cyg

```bash
git clone https://github.com/transcode-open/apt-cyg.git
cd apt-cyg
install apt-cyg /bin
# 或者
lynx -source rawgit.com/transcode-open/apt-cyg/master/apt-cyg > apt-cyg # 先为lynx命令设置代理，不然下载很慢
install apt-cyg /bin
```

```bash
# 配置apt-cyg的镜像源
apt-cyg mirror http://mirrors.163.com/cygwin
# 更新源
apt-cyg update
# 安装软件
apt-cyg install jq vim 
```

参考：

1. https://zhuanlan.zhihu.com/p/66930502

## ②安装配置zsh及oh-my-zsh

参考[ZSH](linux-zsh.md)

## ③设置默认终端shell

```bash
$ mkpasswd > /etc/passwd
# 然后在/etc/passwd文件中设置当前用户为/bin/zsh
```

## ④为lynx命令设置代理

```bash
echo -e "http_proxy:http://localhost:80\nhttps_proxy:http://localhost:80" >> /etc/lynx.cfg
```







