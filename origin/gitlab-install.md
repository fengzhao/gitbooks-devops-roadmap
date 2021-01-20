# Gitlab的安装

# 一、Docker

## ARM

官方gitlab-ce暂没有ARM 架构的镜像,所以由第三方的镜像进行部署

参考：https://about.gitlab.com/handbook/engineering/development/enablement/distribution/maintenance/arm.html#why-dont-you-compile-arm32-bit-packages-on-arm64-for-speed

GitHub：https://github.com/ulm0/gitlab   

DockerHub：https://hub.docker.com/r/ulm0/gitlab

```bash
mkdir -p /data/gitlab/data /data/gitlab/logs /data/gitlab/config && \
docker run -d \
--hostname 192.168.1.8 \
-e GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.1.1:38080';gitlab_rails['lfs_enabled'] = true; gitlab_rails['gitlab_shell_ssh_port'] = 30022" \
-e RAILS_ENV="production" \
-e GITLAB_EMAIL_DISPLAY_NAME="Gitlab 13" \
-e GITLAB_EMAIL_FROM="*****@163.com" \
-e GITLAB_EMAIL_REPLY_TO="*****@163.com" \
-e GITLAB_EMAIL_SUBJECT_SUFFIX="Gitlab 13" \
-e GITLAB_ROOT_PASSWORD="*****" \
-p 38080:38080 \
-p 30022:22 \
--name gitlab \
--restart always \
-v /data/gitlab/config:/etc/gitlab \
-v /data/gitlab/logs:/var/log/gitlab \
-v /data/gitlab/data:/var/opt/gitlab \
ulm0/gitlab:13.2.6
```

