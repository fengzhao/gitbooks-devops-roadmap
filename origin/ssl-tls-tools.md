# SSL证书生成工具

# 一、简介



# 二、Let's Encrypt

Let's Encrypt是一个于2015年三季度推出的数字证书认证机构，旨在以自动化流程消除手动创建和安装证书的复杂流程，并推广使万维网服务器的加密连接无所不在，为安全网站提供免费的SSL/TLS证书。

Let's Encrypt由互联网安全研究小组（缩写ISRG）提供服务。主要赞助商包括电子前哨基金会、Mozilla基金会、Akamai以及思科。2015年4月9日，ISRG与Linux基金会宣布合作。

用以实现新的数字证书认证机构的协议被称为自动证书管理环境（ACME）。GitHub上有这一规范的草案，且提案的一个版本已作为一个Internet草案发布。

Let's Encrypt宣称这一过程将十分简单、自动化并且免费

# 三、OpenSSL



```ini
[req]
default_bits = 4096
default_md = sha256
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
C = CN
ST = Shanghai
L = Shanghai
O = SHANGHAI CURIOUSER MEDICAL TECHNOLOGY CO.,LTD
OU = Software Development Department
CN = test.curiouser.com
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = test.curiouser.com
DNS.2 = test.curiouser.com
IP.1 = 192.168.1.18
```



```bash
openssl req -new -nodes -x509 -days 36000 -keyout test.curiouser.com.key -out test.curiouser.com.crt -config req
```





# 四、ACME.sh

简单来说acme.sh 实现了 acme 协议, 可以从 let‘s encrypt 生成免费的证书。
acme.sh 有以下特点：

- 一个纯粹用Shell（Unix shell）语言编写的ACME协议客户端。
- 完整的ACME协议实施。 支持ACME v1和ACME v2 支持ACME v2通配符证书
- 简单，功能强大且易于使用。你只需要3分钟就可以学习它。
- Let's Encrypt免费证书客户端最简单的shell脚本。
- 纯粹用Shell编写，不依赖于python或官方的Let's Encrypt客户端。
- 只需一个脚本即可自动颁发，续订和安装证书。 不需要root/sudoer访问权限。
- 支持在Docker内使用，支持IPv6

Github：https://github.com/acmesh-official/acme.sh



```bash
curl  https://get.acme.sh | sh
```





# 五、Certbot

https://certbot.eff.org/



```bash
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto --help
```



# 六、CFSSL

CFSSL是CloudFlare开源的一款PKI/TLS工具。 CFSSL 包含一个命令行工具 和一个用于 签名，验证并且捆绑TLS证书的 HTTP API 服务。 使用Go语言编写。

CFSSL包括：

- 一组用于生成自定义 TLS PKI 的工具
- `cfssl`程序，是CFSSL的命令行工具
- `multirootca`程序是可以使用多个签名密钥的证书颁发机构服务器
- `mkbundle`程序用于构建证书池
- `cfssljson`程序，从`cfssl`和`multirootca`程序获取JSON输出，并将证书，密钥，CSR和bundle写入磁盘

PKI借助数字证书和公钥加密技术提供可信任的网络身份。通常，证书就是一个包含如下身份信息的文件：

- 证书所有组织的信息
- 公钥
- 证书颁发组织的信息
- 证书颁发组织授予的权限，如证书有效期、适用的主机名、用途等
- 使用证书颁发组织私钥创建的数字签名

需要安裝`CFSSL`工具，这将会用來建立 TLS Certificates

Github： https://github.com/cloudflare/cfssl

下载地址： https://pkg.cfssl.org/



```bash
export CFSSL_URL="https://pkg.cfssl.org/R1.2"
wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl
wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson
chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson
```

# 七、Web服务器绑定

## 1、Nginx

```bash
server {
    listen 443 ssl;
		ssl_certificate /websvr/ssl/fullchain.cer; # 证书文件
    ssl_certificate_key /websvr/ssl/mydomain.key; # 私钥文件
    ssl_session_timeout 5m; # 会话缓存过期时间
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # 开启 SSL 支持
    ssl_prefer_server_ciphers on; # 设置协商加密算法时，优先使用服务端的加密套件
  	server_name test.curiouser.com;
    set $app test;
    if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
        set $tttt $1;
    }
    error_log  /var/log/nginx/nginx-test-443-error.log;
    access_log  /var/log/nginx/nginx-tes1t-443-access-$tttt.log  main;
    location / {
      root ~/test;
      index index.html index.htm;
      autoindex on;
    }
}
```



## 2、Apache

# 八、其他操作

## 1、openssl命令行获取服务器SSL证书

```bash
openssl s_client -showcerts -connect {HOSTNAME}:{PORT} </dev/null 2>/dev/null|openssl x509 -outform PEM > www.test.com.ssl.pem
```

```bash
openssl s_client -connect {HOSTNAME}:{PORT} -showcerts
```

参考：

1. https://superuser.com/questions/97201/how-to-save-a-remote-server-ssl-certificate-locally-as-a-file

## 2、Linux发行版导入自签证书

### CentOS

```bash
yum install -y ca-certificates

cp ca.crt /usr/local/share/ca-certificates/kubernetes.crt
update-ca-certificates
# 或者
cp *.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust extract
```

### Ubuntu/Debian

```bash
apt-get install ca-certificates

cp my.crt /usr/local/share/ca-certificates/
update-ca-certificates
# 或者
cp cacert.pem /usr/share/ca-certificates
dpkg-reconfigure ca-certificates
```

### Alpine

```bash
apk add --no-cache ca-certificates
mv my.crt /usr/local/share/ca-certificates/
update-ca-certificate
```

