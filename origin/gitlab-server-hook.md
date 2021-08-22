# Gitlab的服务端钩子Hook

# 一、简介

官方文档：https://docs.gitlab.com/ee/administration/server_hooks.html

Git除了客户端有各种钩子进行配置使用，远程存储库，例如gitlab也支持服务端的钩子。比如本地git客户端推送代码到远程仓库时，可以指定执行钩子脚本。

**Gitlab支持的服务端钩子**

- **pre-receive**：`git push`向仓库推送代码时被执行
- **post-receive**：在成功推送后被调用，适合用于发送通知。这个脚本没有参数，但和pre-receive一样通过标准输入读取
- **update**：在pre-receive之后被调用。它也是在实际更新前被调用的，但它可以分别被每个推送上来的引用分别调用。也就是说如果用户尝试推送到4个分支，update会被执行4次。不需要读取标准输入。事实上，它接受三个参数：
  - 更新的引用名称
  - 引用中存放的旧的对象名称
  - 引用中存放的新的对象名称

**Gitlab服务端钩子支持配置的范围**

- **单个仓库**
- **全局所有的仓库**





# 二、单个仓库钩子



## 1、

## 2、创建钩子脚本文件夹

在项目经过Hashed过的项目路径下创建钩子脚本文件（项目hashed的路径参考[附录一](#附录)）

```bash
mkdir /var/opt/gitlab/git-data/repositories/@hashed/项目hashed路径/custom_hooks
```

## 3、创建`pre-receive`脚本

```bash
#!/bin/bash
z40=0000000000000000000000000000000000000000
echo $GL_USERNAME
while read oldrev newrev refname; do
   if [ $oldrev == $z40 ]; then
     # Commit being pushed is for a new branch
     oldrev=4b825dc642cb6eb9a060e54bf8d69288fbee4904
   fi
   yaml_file_name=".test.yml"
   approval_user="root"
   file_name=$(git diff --name-only $oldrev $newrev)
   if [ "$yaml_file_name" == "$file_name" ] && [ $GL_USERNAME != "$approval_user" ]; then
           echo -e "\033[31m####################################\033[0m"
           echo -e "\033[31m###                              ###\033[0m"
           echo -e "\033[31m###   请不要修改${yaml_file_name}文件!   ###\033[0m"
           echo -e "\033[31m###                              ###\033[0m"
           echo -e "\033[31m####################################\033[0m"
           exit 1
   fi
done
echo "I'm a hooked file on server"
```

```bash
chmod +x /var/opt/gitlab/git-data/repositories/@hashed/项目hashed路径/custom_hooks/pre-receive
```



## 4、验证测试

```bash
echo "test" >> .test.yaml
git commit -am "测试"
git push origin
```

![](../assets/image-20210813141326213.png)

# 三、全局仓库钩子

```bash
cd /opt/gitlab/embedded/service/gitlab-shell/
mkdir -p hooks/{pre-receive.d,post-receive.d,update.d}
```

`detect-modify-cicd-scripts-files.sh`

```bash
#!/bin/bash
z40=0000000000000000000000000000000000000000
echo $GL_USERNAME
while read oldrev newrev refname; do
   if [ $oldrev == $z40 ]; then
     # Commit being pushed is for a new branch
     oldrev=4b825dc642cb6eb9a060e54bf8d69288fbee4904
   fi
   yaml_file_name=".test.yml"
   approval_user="root"
   d=$(git diff --name-only $oldrev $newrev)
   if [ "$yaml_file_name" == "$file_name" ] && [ $GL_USERNAME != "$approval_user" ]; then
           echo -e "\033[31m####################################\033[0m"
           echo -e "\033[31m###                              ###\033[0m"
           echo -e "\033[31m###   请不要修改${yaml_file_name}文件!   ###\033[0m"
           echo -e "\033[31m###                              ###\033[0m"
           echo -e "\033[31m####################################\033[0m"
           exit 1
   fi
done
echo "I'm a hooked file on server"
```

# 四、应用场景

## 1、检查推送代码是否修改某些文件



## 2、





# 附录

## 1、显示仓库文件Hash之后的存储路径

- 在gitlab-rails控制台中使用命令显示

  `/var/opt/gitlab/git-data/repositories/`

  ```bash
  # 进入gitlab-rails控制台
  gitlab-rails console
  
  => Project.find(项目ID).disk_path
  "@hashed/6f/4b/6f4b6612125fb3a0daecd2799dfd6c9c299424fd920f9b308110a2c1fbd8f443"
  => Project.find_by_full_path('组/仓库').disk_path
  "@hashed/6f/4b/6f4b6612125fb3a0daecd2799dfd6c9c299424fd920f9b308110a2c1fbd8f443"
  ```

- 拼凑

  项目仓库文件Hash之后的存储路径是Project_ID进行SHA256加密过的字符串构成。例如

  ```bash
  /var/opt/gitlab/git-data/repositories/@hashed/
  		|--Project_ID的SHA256加密字符串前两位/
  					|--Project_ID的SHA256加密字符串第三位和第四位
  					      |--Project_ID的SHA256加密字符串.git
  					      |--Project_ID的SHA256加密字符串.wiki.git
  ```

  





# 参考

1. https://docs.gitlab.com/ee/administration/server_hooks.html
2. https://blog.csdn.net/ygdlx521/article/details/115751703