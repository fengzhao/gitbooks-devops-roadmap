# MacOS小技巧

## 1、DMG 格式文件制作以及 ISO 转换互转

DMG 格式是 Mac OS X 中常用的打包格式

- 创建 DMG 格式的文件

```bash
$ hdiutil create -size 100M -stdinpass -format UDZO -srcfolder folder_to_compress archive_name.dmg

UDZO（压缩格式，默认）
UDRO（只读格式）
UDBZ（Better compressed image）
UDRW（可读写格式）
UDTO（DVD 格式）
```

- 修改 DMG 文件的大小

```bash
$ hdiutil resize 150M /path/to/the/diskimage
```

- 修改 DMG 格式中的加密口令

```bash
$ hdiutil chpass /path/to/the/diskimage
```

- 挂载 DMG 格式的文件

```bash
$ hdiutil attach archive_name.dmg
```

它的挂载点在 /Volumes 目录的同名目录下

```bash
$ ls -lah /Volumes/archive_name/
```

- 卸载 DMG 文件

```bash
$ hdiutil eject /Volumes/archive_name/
```

- 将 ISO 格式的文件转为 DMG 格式的文件

```bash
$ hdiutil convert /path/imagefile.iso -format UDRW -o /path/convertedimage.dmg
```

- 将 DMG 格式的文件转为 ISO 格式的文件

```bash
$ hdiutil convert /path/imagefile.dmg -format UDTO -o /path/convertedimage.cdr
$ hdiutil makehybrid /path/convertedimage.cdr -iso -joliet -o /path/convertedimage.iso
```

## 2、删除虚拟网络设备

```bash
sudo ifconfig utun3 delete
```

## 3、路由修改

```bash
# 删除路由
ip route delete 172.16.1.2/32 
# 添加路由
sudo route add 172.16.1.2/32 -interface utun2
```

