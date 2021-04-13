# Nginx日志写入Kafka

# 一、简介

## 方案

- **Nginx module**：编译集成第三方kafka相关模块，可直接将日志发送到kafka
- **Nginx + 第三方应用**：结合第三方应用将Nginx日志发送到kafka
  - `tail | kafkacat ———> kafka`
  - `rsyslog ———> kafka`
  - `Nginx stdout |  k8s 第三方operator ———> kafka`

# 二、Nginx module -> kafka



```bash
openssl_version=1.1.1 && \
nginx_version=1.18.0 && \
mkdir compile-dir && cd compile-dir && \
curl -s -# https://www.openssl.org/source/openssl-$openssl_version.tar.gz | tar zxf - -C ./ && \
curl -s -# https://nginx.org/download/nginx-$nginx_version.tar.gz | tar zxf - -C ./ && \
cd nginx-$nginx_version && \
./configure \
--prefix=/opt/nginx-1.18.0 \
--user=nginx \
--group=nginx \
--modules-path=/opt/nginx-1.18.0/modules \
--sbin-path=/opt/nginx-1.18.0/sbin/nginx \
--error-log-path=/opt/nginx-1.18.0/logs/error.log \
--http-log-path=/opt/nginx-1.18.0/logs/access.log \
--conf-path=/opt/nginx-1.18.0/conf/nginx.conf \
--pid-path=/opt/nginx-1.18.0/nginx.pid \
--lock-path=/opt/nginx-1.18.0/nginx.lock \
--with-pcre \
--with-openssl=../openssl-1.1.1 \
--with-http_stub_status_module \
--with-http_realip_module \
--with-http_addition_module \
--with-http_sub_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_mp4_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_xslt_module=dynamic \
--with-http_image_filter_module=dynamic \
--with-http_geoip_module=dynamic \
--with-http_image_filter_module \
--with-http_v2_module \
--with-http_slice_module \
--with-threads \
--with-stream \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-stream_realip_module \
--with-stream_geoip_module=dynamic \
--with-mail \
--with-mail_ssl_module \
--with-compat
```



参考：

- https://github.com/brg-liuwei/ngx_kafka_module
- https://github.com/kaltura/nginx-kafka-log-module

# 三、tail | kafkacat -> kafka

参考：

- https://github.com/edenhill/kafkacat

# 四、nginx -> rsyslog -> kafka

参考：

- http://zhang-jc.github.io/2019/03/15/%E4%BD%BF%E7%94%A8-Rsyslog-%E5%B0%86-Nginx-Access-Log-%E5%86%99%E5%85%A5-Kafka/

# 五、Nginx stdout -> k8s 第三方operator-> kafka

参考：

- https://banzaicloud.com/docs/one-eye/logging-operator/quickstarts/kafka-nginx/

