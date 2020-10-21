

# Python 多用户工作台JupyterHub

# 一、简介

# 二、安装

## 1、二进制安装

以Ubuntu为例

```bash
apt-get install npm
npm config set registry https://registry.npm.taobao.org --global
mkdir ~/.pip
echo -e "[global]\nindex-url = https://mirrors.aliyun.com/pypi/simple/\n[install]\ntrusted-host=mirrors.aliyun.com\n" > ~/.pip/pip.conf
python3 -m pip install jupyterhub
npm install -g configurable-http-proxy
python3 -m pip install notebook
```

生成默认配置文件

```bash
jupyterhub --generate-config
```

修改配置文件`~/jupyterhub_config.py`并启动jupyterhub

```	bash
nohup jupyterhub --generate-config -f ~/jupyterhub_config.py 2>&1 >> /var/log/jupyterhub.log &
echo $! > /var/log/jupyterhub.pid

# 或者
nohup jupyterhub --generate-config -f ~/jupyterhub_config.py 2>&1 >> /var/log/jupyterhub.log &!
echo $! > /var/log/jupyterhub.pid
```

## 2、docker安装

## 3、kubernetes安装

```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/ && \
helm repo update

RELEASE=jhub
NAMESPACE=jhub
helm upgrade --cleanup-on-fail \
  --install $RELEASE jupyterhub/jupyterhub \
  --namespace $NAMESPACE \
  --create-namespace \
  --version=0.9.0 \
  --values config.yaml
```



# 三、jupyterhub 命令详解

## 命令格式

```bash
jupyterhub cmd [args]
```

## 全局命令参数

```bash
--debug
    set log level to logging.DEBUG (maximize logging output)
--generate-config
    generate default config file
--generate-certs
    generate certificates used for internal ssl
--no-db
    disable persisting state database to disk
--upgrade-db
    Automatically upgrade the database if needed on startup.

    Only safe if the database has been backed up.
    Only SQLite database files will be backed up automatically.
--no-ssl
    [DEPRECATED in 0.7: does nothing]
--base-url=<URLPrefix> (JupyterHub.base_url)
    Default: '/'
    The base URL of the entire application.
    Add this to the beginning of all JupyterHub URLs. Use base_url to run
    JupyterHub within an existing website.
    .. deprecated: 0.9
        Use JupyterHub.bind_url
-y <Bool> (JupyterHub.answer_yes)
    Default: False
    Answer yes to any questions (e.g. confirm overwrite)
--ssl-key=<Unicode> (JupyterHub.ssl_key)
    Default: ''
    Path to SSL key file for the public facing interface of the proxy
    When setting this, you should also set ssl_cert
--ssl-cert=<Unicode> (JupyterHub.ssl_cert)
    Default: ''
    Path to SSL certificate file for the public facing interface of the proxy
    When setting this, you should also set ssl_key
--url=<Unicode> (JupyterHub.bind_url)
    Default: 'http://:8000'
    The public facing URL of the whole JupyterHub application.
    This is the address on which the proxy will bind. Sets protocol, ip,
    base_url
--ip=<Unicode> (JupyterHub.ip)
    Default: ''
    The public facing ip of the whole JupyterHub application (specifically
    referred to as the proxy).
    This is the address on which the proxy will listen. The default is to listen
    on all interfaces. This is the only address through which JupyterHub should
    be accessed by users.
    .. deprecated: 0.9
        Use JupyterHub.bind_url
--port=<Int> (JupyterHub.port)
    Default: 8000
    The public facing port of the proxy.
    This is the port on which the proxy will listen. This is the only port
    through which JupyterHub should be accessed by users.
    .. deprecated: 0.9
        Use JupyterHub.bind_url
--pid-file=<Unicode> (JupyterHub.pid_file)
    Default: ''
    File to write PID Useful for daemonizing JupyterHub.
--log-file=<Unicode> (JupyterHub.extra_log_file)
    Default: ''
    DEPRECATED: use output redirection instead, e.g.
    jupyterhub &>> /var/log/jupyterhub.log
--log-level=<Enum> (Application.log_level)
    Default: 30
    Choices: (0, 10, 20, 30, 40, 50, 'DEBUG', 'INFO', 'WARN', 'ERROR', 'CRITICAL')
    Set the log level by value or name.
-f <Unicode> (JupyterHub.config_file)
    Default: 'jupyterhub_config.py'
    The config file to load
--config=<Unicode> (JupyterHub.config_file)
    Default: 'jupyterhub_config.py'
    The config file to load
--db=<Unicode> (JupyterHub.db_url)
    Default: 'sqlite:///jupyterhub.sqlite'
    url for the database. e.g. `sqlite:///jupyterhub.sqlite`
```

## 子命令

### token：生成用户API token

命令格式

```bash
jupyterhub token [username]
```

命令参数

```bash
--log-level=<Enum> (Application.log_level)
    Default: 30
    Choices: (0, 10, 20, 30, 40, 50, 'DEBUG', 'INFO', 'WARN', 'ERROR', 'CRITICAL')
    Set the log level by value or name.
-f <Unicode> (JupyterHub.config_file)
    Default: 'jupyterhub_config.py'
    The config file to load
--config=<Unicode> (JupyterHub.config_file)
    Default: 'jupyterhub_config.py'
    The config file to load
--db=<Unicode> (JupyterHub.db_url)
    Default: 'sqlite:///jupyterhub.sqlite'
    url for the database. e.g. `sqlite:///jupyterhub.sqlite`
 
# 示例
$> jupyterhub token kaylee
ab01cd23ef45
```


# 四、其他命令

## Jupyter kernel的管理

```bash
jupyter-kernelspec  list
jupyter-kernelspec install
jupyter-kernelspec uninstall
jupyter-kernelspec remove
```



# 五、使用LDAP进行用户认证

Github：https://github.com/jupyterhub/ldapauthenticator

```bash
pip3 install jupyterhub-ldapauthenticator
```



```bash
c.JupyterHub.authenticator_class = 'ldapauthenticator.LDAPAuthenticator'
#c.LDAPAuthenticator.server_address = '192.168.1.7'
c.LDAPAuthenticator.server_hosts = ['ldap://192.168.1.7:389']
c.LDAPAuthenticator.bind_user_dn = 'uid=root,cn=users,dc=ldap,dc=synology,dc=curiouser,dc=com'
c.LDAPAuthenticator.bind_user_password = 'jL6u49t5A9P5'
c.LDAPAuthenticator.user_search_base = 'cn=users,dc=ldap,dc=synology,dc=curiouser,dc=com'
c.LDAPAuthenticator.user_search_filte = '(&(memberOf=cn=jupyterhub,cn=groups,dc=ldap,dc=synology,dc=curiouser,dc=com)(cn={0}))'
c.LDAPAuthenticator.user_attribute = 'cn'
c.LDAPAuthenticator.create_user_home_dir = True
c.LDAPAuthenticator.create_user_home_dir_cmd = ['mkhomedir_helper']



c.LDAPAuthenticator.lookup_dn = True
c.LDAPAuthenticator.lookup_dn_search_filter = '({login_attr}={login})'
c.LDAPAuthenticator.lookup_dn_search_user = 'ldap_search_user_technical_account'
c.LDAPAuthenticator.lookup_dn_search_password = 'secret'
c.LDAPAuthenticator.user_search_base = 'ou=people,dc=wikimedia,dc=org'
c.LDAPAuthenticator.user_attribute = 'sAMAccountName'
c.LDAPAuthenticator.lookup_dn_user_dn_attribute = 'cn'
c.LDAPAuthenticator.escape_userdn = False
c.LDAPAuthenticator.bind_dn_template = '{username}'
```



