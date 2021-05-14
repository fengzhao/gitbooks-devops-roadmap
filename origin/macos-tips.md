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

## 4、HomeBrew安装使用

### ①简介

- **brew** 是从下载源码解压然后 `./configure && make install` 

- **formula**：定义了一个软件包。包括了这个软件的，依赖、源码位置及编译方法等

- **tap**：一个包含 formula 的 git 仓库

- **cask**：homebrew 的一个扩展仓库，用来安装 一些带界面的应用软件，下载好后会自动安装

- **bottle**：homebrew 提供的已经编译好的 formula。这些 bottle 可以在[这里](https://link.jianshu.com?t=https://bintray.com/homebrew/bottles)看到。在大部分的情况下，执行

- **Homebrew路径**

  ```bash
  cd "$(brew --repo)"
  ```

- **Cask仓库路径**

  ```bash
  cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
  ```

- **Core仓库路径**

  ```bash
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
  ```

### ②安装

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### ③使用

```bash
# 安装一个包
brew install <package_name>
# 搜索一个包
brew search <package_name>
brew search /正则表达式/ # 标准格式
brew search /^vi/   #规定了只能是vi开头
brew search /^vi\\w$/ 
# 查看这个包的信息
brew info <package_name>
# 安装图形化的软件
brew cask install <formula>
# 卸载对应包名字
brew uninstall <package_name>
# 列出过时的包
brew outdated
# 更新过时的包，不带包名就跟新所有包
brew upgrade [ package_name ]
# 更新HomeBrew自身
brew update
# 清除缓存
brew cleanup [包名]
# 列出已经安装的包
brew list
# 查看homebrew 的配置
brew config
# 添加或者删除仓库
brew [un]tap <github_userid/repo_name> 
```

### ④使用国内的镜像源

- 中科大

  ```bash
  # Homebrew 源代码仓库
  cd "$(brew --repo)"
  git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
  
  # Homebrew 核心软件仓库
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
  git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
  
  # Homebrew cask 软件仓库，提供 macOS 应用和大型二进制文件
  cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
  git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
  
  # Homebrew cask 其他版本 (alternative versions) 软件仓库，提供使用人数多的、需要的版本不在 cask 仓库中的应用。
  cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask-versions
  git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask-versions.git
  
  # Homebrew 预编译二进制软件包
  echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.zshrc
  source ~/.zshrc
  ```

## 5、HomeBrew的备份恢复

### ①简介

homebrew-bundle - https://github.com/Homebrew/homebrew-bundle

1. Mac 上非常常用的包管理器 Homebrew, 我们经常用它来安装其他的软件包
2. 还有 Homebrew-cask, 可以用来安装图形界面的 App
3. homebrew-bundle 类似 node 中的 package.json 或者 Cocoapods 中的 Podfile
4. 我们将需要的包和 App, 声明在一个 Brewfile 中, 然后执行 brew bundle 即可安装所有包

### ②备份内容

1. brew tap 中的软件库
2. brew 安装的命令行工具
3. brew cask 安装的 App
4. Mac App Store 安装的 App

### ③备份命令 

```bash
# 执行 brew bundle dump 
brew bundle dump --describe --force --file="~/Desktop/Brewfile"

# 参数说明 
--describe：为列表中的命令行工具加上说明性文字。
--force：直接覆盖之前生成的 Brewfile 文件。如果没有该参数，则询问你是否覆盖。
--file="~/Desktop/Brewfile"：在指定位置生成文件。如果没有该参数，则在当前目录生成 Brewfile 文件。
```

生成的Brewfile 文件内容

```bash
## 该部分是 brew 中的 tap，相当于一个个软件库 
tap "homebrew/bundle"
tap "homebrew/cask"
## 该部分是 brew 安装的命令行工具 
# Mac App Store command-line interface
brew "mas"
# UNIX shell (command interpreter)
brew "zsh"
# Fish shell like syntax highlighting for zsh
brew "zsh-syntax-highlighting"

## 该部分是 brew cask 安装的 app
cask "mounty"
cask "dteoh/sqa/slowquitapps"

## 该部分是 Mac App Store 安装的 app
mas "ting_en", id: 734383760
mas "Xcode", id: 497799835
```

### ④恢复命令

```bash
brew install mas
# 批量安装软件 
brew bundle --file="~/Desktop/Brewfile"
```

### 参考：

1. https://wsgzao.github.io/post/homebrew-bundle/

## 6、使用Brew安装的软件信息

### ①MySQL

- 日志和底层DB数据文件: `/usr/local/var/mysql`
- bin文件路径：`/usr/local/Cellar/mysql@mysql版本/mysql版本`
- brew 启动命令：`brew services restart mysql@mysql版本 `

### ②Nginx

- 主配置文件路径：`/usr/local/etc/nginx/nginx.conf`
- bin文件路径: `/usr/local/Cellar/nginx/nginx版本号/`