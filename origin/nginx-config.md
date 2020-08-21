# Nginx优化配置

# 一、访问日志JSON格式

```bash
# ....全局配置省略..... ;

http {
		# ....HTTP模块其他配置省略..... ;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    # 设置日志格式
    log_format json_log '{ "@timestamp": "$time_iso8601", '
                           '"app": "$app", '
                           '"remote_addr": "$remote_addr", '
                           '"referer": "$http_referer", '
                           '"request": "$request", '
                           '"status": $status, '
                           '"bytes": $body_bytes_sent, '
                           '"agent": "$http_user_agent", '
                           '"x_forwarded": "$http_x_forwarded_for", '
                           '"up_addr": "$upstream_addr",'
                           '"up_host": "$upstream_http_host",'
                           '"up_resp_time": "$upstream_response_time",'
                           '"request_time": "$request_time"'
                        ' }';
    server {
        # .....虚拟主机其他配置省略.....
        set $app test ; # 设置变量app的值为“test”
        access_log  logs/host.access.log  json_log; # 设置访问日志按照“json_log”的格式进行输出
    }
}
```

**输出的JSON格式日志**

```json
{
    "@timestamp": "2020-03-09T17:54:49+08:00",
    "app": "test",
    "remote_addr": "127.0.0.1",
    "referer": "-",
    "request": "GET /Empty-2C4G80G.ovf HTTP/1.1",
    "status": 200,
    "bytes": 7532,
    "agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36",
    "x_forwarded": "-",
    "up_addr": "-",
    "up_host": "-",
    "up_resp_time": "-",
    "request_time": "0.000"
}
```



# 二、按天保留日志文件

```yaml
server{
...
		if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
       set $tttt $1;
    }
    access_log  logs/nginx-access-$tttt.log  main;
...
}
```

# 三、配置HTTP基础认证

Nginx 使用  `ngx_http_auth_basic_module`  模块支持 **HTTP基本身份验证** 功能 。

## 1、安装httpd-tools

yum

```bash
yum install -y httpd-tools
```

apt

```bash
apt install -y apache2-utils  
```

## 2、创建授权用户和密码

```bash
htpasswd -c -d /etc/nginx/basic-auth-pass-file test_user
```

htpasswd 其他操作参考：[htpasswd操作](linux-htpasswd.md)

## 3、配置nginx

```bash
server {
     # ...省略
    auth_basic   "登录认证";  
    auth_basic_user_file basic-auth-pass-file;

    # ...省略
    
    # 或者只设置某些URL进行登录认证
    # location /api {
    #    auth_basic "登录认证";
    #    auth_basic_user_file basic-auth-pass-file;
    #}
}
```

## 4、使用

```bash
# 浏览器中使用
直接在浏览器中输入地址, 会弹出用户密码输入框, 输入即可访问

# wget
wget --http-user=test_user --http-passwd=*** http://****

# curl
curl -u test_user:**** -O http://****
```

