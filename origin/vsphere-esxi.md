



# ESXI 管理常用命令

## esxcli

### 维护模式管理

```bash
esxcli system maintenanceMode {cmd} [cmd options]

Available Commands:
  get                   获取系统维护状态
  set                   Enable or disable the maintenance mode of the system.
  	-e|--enable           开启维护模式 (必须)
    -t|--timeout=<long>   多少秒后进入维护模式 (默认0秒)
    -m|--vsanmode=<str>   在主机进入维护模式(默认ensureObjectAccessibility)之前，VSAN服务必须执行														的操作。允许的值是:
														ensureObjectAccessibility:
																在进入维护模式之前，从磁盘中提取数据以确保虚拟SAN集群中的对象可访问性。
														evacuateAllData:在进入维护模式之前，从磁盘中撤离所有数据。
														noAction:在进入维护模式之前，不要将虚拟SAN数据移出磁盘。

```

### 关机重启管理（必须进入维护模式）

```bash
esxcli system shutdown {cmd} [cmd options]

Available Commands:
  poweroff              断开电源
    -d|--delay=<long>     多少秒后关机，范围在10-4294967295
    -r|--reason=<str>     执行该操作的原因
  reboot                重启系统
    -d|--delay=<long>     多少秒后关机，范围在10-4294967295
    -r|--reason=<str>     执行该操作的原因
```

### 系统时间管理

```bash
esxcli system time set [cmd options]

Cmd options:
  -d|--day=<long>       Day
  -H|--hour=<long>      Hour
  -m|--min=<long>       Minute
  -M|--month=<long>     Month
  -s|--sec=<long>       Second
  -y|--year=<long>      Year
```

### 创建datastore

创建NFS类型的Datastore

```bash
# ESXI安装Synology NFS VAAI
参考附录2。ESXI安装完Synology NFS VAAI后再创建NFS类型Datastore后，会显示Datastore已支持硬件加速

esxcfg-nas -a synology-nfs-datastore -o 192.168.1.7 -s /volume2/ESXI

# 删除Datastore
esxcfg-nas -d synology-nfs-datastore


```



# 附录

## 1、ESXI VAAI

在虚拟化环境中，从资源角度来看，传统上的存储操作非常昂贵。与主机相比，存储设备可以更高效地执行克隆和快照等功能。VMware vSphere存储API阵列集成（VAAI），也称为硬件加速或硬件卸载API，是一组API，用于启用VMware vSphere ESXi主机与存储设备之间的通信。这些API定义了一组“存储原语”，它们使ESXi主机能够将某些存储操作卸载到阵列上，从而减少了ESXi主机上的资源开销，并可以显着提高存储密集型操作（如存储克隆，清零等）的性能。VAAI的目标是帮助存储供应商提供硬件帮助，以加快在存储硬件中更有效地完成的VMware I / O操作。

参考：https://www.vmware.com/techpapers/2012/vmware-vsphere-storage-apis-array-integration-10337.html

## 2、ESXI安装Synology NFS VAAI

下载地址：https://www.synology.cn/en-global/support/download/DS110+#utilities

安装参看：https://global.download.synology.com/download/www-res/dsm/Tools/NFSVAAIPlugin/README

```bash
scp synonfs-vaai-plugin.vib root@192.168.1.103:/tmp

esxcli software vib install -v /tmp/synonfs-vaai-plugin.vib --no-sig-check
# 或
esxcli software vib install -d /tmp/synonfs-vaai-plugin.zip --no-sig-check

重启ESXI

# 查看ESXI插件中是否已安装Synology NFS VAAI
esxcli software vib list | more

# 删除ESXI插件
esxcli software vib remove -n {PLUGIN_NAME}
```

参考：https://www.jonathanmedd.net/category/nfs

