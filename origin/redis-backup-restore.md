# Redis数据的备份恢复迁移

# 一、Redis-dump

Redis-dump Github：https://github.com/delano/redis-dump

## 1、安装

### MacOS

```bash
brew install ruby
gem sources --add https://mirrors.aliyun.com/rubygems/
gem sources --remove https://rubygems.org/
gem sources --list
gem install redis-dump -V
```

### CentOS

```bash
yum inatll -y ruby
gem sources --add https://mirrors.aliyun.com/rubygems/
gem sources --remove https://rubygems.org/
gem sources --list
# gem安装redis需要ruby版本高于2.3.0，CentOS7默认安装的ruby版本为2.0.0，所以先升级Ruby
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
curl -sSL https://get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm -v
rvm list known
rvm install 2.5
ruby -V
gem install redis-dump -V
source /etc/profile
redis-dump -V
```

## 2、redis-dump导出数据到JSON

```bash
Usage: redis-dump [global options] COMMAND [command options]
 -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])
 -d, --database=S                 Redis database (e.g. -d 15)
 -a, --password=S                 Redis password (e.g. -a 'my@pass/word')
 -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)
 -c, --count=S                    Chunk size (default: 10000)
 -f, --filter=S                   Filter selected keys (passed directly to redis' KEYS command)
 -b, --base64                     Encode key values as base64 (useful for binary values)
 -O, --without_optimizations      Disable run time optimizations
 -V, --version                    Display version
 -D, --debug
     --nosafe
```

### 示例

```bash
redis-dump -u redis://127.0.0.1:6379 -d 0 -c 50000 > redis-backup-$(date "+%Y%m%d-%H%M%S").json
```

## 3、redis-load导入JSON数据文件到Redis

```bash
redis-load [global options] COMMAND [command options]
  -u, --uri=S                      Redis URI (e.g. redis://hostname[:port])
  -d, --database=S                 Redis database (e.g. -d 15)
  -a, --password=S                 Redis password (e.g. -a 'my@pass/word')
  -s, --sleep=S                    Sleep for S seconds after dumping (for debugging)
  -b, --base64                     Decode key values from base64 (used with redis-dump -b)
  -n, --no_check_utf8
  -V, --version                    Display version
  -D, --debug
      --nosafe
```

### 示例

```bash
 cat redis-backup.json| redis-load -u redis://127.0.0.1:6379 -d 0
```

### 注意

1. 相同的Key，值会被覆盖



