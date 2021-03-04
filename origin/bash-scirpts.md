# 1、判断curl返回状态码

```bash
#!/bin/bash
response=$(curl -sL -o /dev/null -w %{http_code} https://baidu.com)
if [[ $response -ge 200 && $response -le 299 ]] ;then 
	echo 'check point success'
else 
	echo 'check point fail'
fi
```

# 2、读取文件中的配置到变量中

```bash
#!/bin/bash
# 配置文件中的配置项格式为key1=value1，一行一个配置项
while read line;do
	eval "$line"
done < /etc/openvpn/server/smtp.conf
echo $key1
```

# 3、根据console输入条件执行

```bash
#!/bin/bash
echo "选择以下功能:"
echo "   0) 功能0"
echo "   1) 功能1"
echo "   2) 功能2"
echo "   3) 功能3"
echo "   4) 功能4"
read -p "功能选项[4]: " option
until [[ -z "$option" || "$option" =~ ^[0-4]$ ]]; do
	read -p "$option为无效的选项，请重新输入功能选项: " option
done
case "$option" in
    0) 
    	echo "功能0已执行!" 
    ;;
		1)
			echo "功能1已执行!" 
      exit
    ;;
		2)
			echo "功能2已执行!" 
			exit
		;;
		3)
			echo "功能3已执行!"  
			exit
		;;
		# 默认选项
		4|"")
			echo "功能4已执行!" 
			exit
		;;
esac
```

# 4、将指定输出内容写入文件

```bash
{
echo "hahh"
echo "lalal"
} > /tmp/test
```

# 5、判断变量是否存在或为空

```bash
if [ -z ${var+x} ]; then 
	echo "var is unset"; 
else 
	echo "var is set to '$var'"; 
fi
```

参考：https://stackoverflow.com/questions/3601515/how-to-check-if-a-variable-is-set-in-bash

# 6、换算秒为分钟、小时

```bash
#!/bin/bash

a=60100
swap_seconds ()
{
    SEC=$1
    (( SEC < 60 )) && echo -e "持续时间: $SEC秒\c"
    (( SEC >= 60 && SEC < 3600 )) && echo -e "持续时间: $(( SEC / 60 ))分钟$(( SEC % 60 ))秒\c"
    (( SEC > 3600 )) && echo -e "持续时间: $(( SEC / 3600 ))小时$(( (SEC % 3600) / 60 ))分钟$(( (SEC % 3600) % 60 ))秒\c"
}

b=`swap_seconds $a`
echo $b
```

输出

```bash
持续时间: 16小时41分钟40秒
```

## 7、脚本命令行参数的传递与判断

```bash
#!/bin/bash

main() {
	if [[ $# == 1 ]]; then
        case $1 in
        "-h")
            echo "脚本使用方法: "
            echo "  ./gitlab-pipeline.sh git仓库名1 git仓库名2 ... tag名(tag命名规则为: *-v加数字)"
            exit
            ;;
        "--help")
            echo "脚本使用方法: "
            echo "  ./gitlab-pipeline.sh git仓库名1 git仓库名2 ... tag名(tag命名规则为: *-v加数字)"
            exit
            ;;
        *)
            echo "参数错误！"
            exit
            ;;
        esac
    fi
}

main $*
```



# 8、检测docker 容器的启动状态



```bash
# 第一步：判断镜像是否存在
if [ `docker images --format {{.Repository}}:{{.Tag}} |grep -Fx 192.168.1.7:32772/applications/$CI_PROJECT_NAME:${CI_COMMIT_SHORT_SHA};echo $?` -eq 0 ];then
   docker pull 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
fi;
# 第二步：判断是否已经有重名的容器在运行或者处在其他状态。重名的，先删掉，在启动；不重名的直接启动
if [ `docker ps -a --format {{.Names}} |grep -Fx $CI_PROJECT_NAME > /dev/null ;echo $?` -eq 0 ] ;then
   docker rm -f $CI_PROJECT_NAME ;
   docker run -d --name $CI_PROJECT_NAME -p 30088:8080 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
else
   docker run -d --name $CI_PROJECT_NAME -p 30088:8080 192.168.1.7:32772/applications/$CI_PROJECT_NAME:$CI_COMMIT_SHORT_SHA ;
fi;
# 第三步：循环五次判断容器的监控检测是否处于什么状态。健康状态就直接退出循环，不健康显示健康检查日志，正在启动的直接显示。处于其他状态的直接显示状态
n=0;
while true ;do
  container_state=`docker inspect --format='{{json .State.Health.Status}}' $CI_PROJECT_NAME`;
  case $container_state in
    '"starting"' )
      echo "应用容器正在启动！";
    ;;
    '"healthy"' )
      echo "应用容器已启动，状态健康！";
    break;
    ;;
    '"unhealthy"' )
      echo "应用容器健康检测失败！";
      docker inspect --format='{{json .State.Health.Log}}' $CI_PROJECT_NAME;
    ;;
    * )
      echo "未知的状态:$container_state";
    ;;
  esac;
  sleep 1s;
  n=$(($n+1));
  if [ $n -eq 5 ];
     then break ;
  fi ;
done

```

