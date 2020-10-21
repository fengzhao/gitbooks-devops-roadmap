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

