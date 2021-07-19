# 一、struct与json之间的互转

```go
type Test struct {
  A string `json:"a`
  B int `json:"b"`
}
```

## struct转json

```go
var t Test
t.A="测试"
t.B=2

jsonBytes, err := json.Marshal(t)
if err != nil {
	fmt.Println(err)
}
fmt.Println(string(jsonBytes))
```

## json转struct

```bash
jsonStr := `{
					"a": "测试",
          "b": 2
        }`
var t Test
json.Unmarshal([]byte(jsonStr), &t)
fmt.Println(t)
```

# 二、主函数的初始化

```go
var (
	router *gin.Engine
)

func init (){
  router = gin.Default()
}

func main(){
  router.GET("/",func(context *gin.Context) {})
}
```

# 三、字符串的处理

## 1、分割字符串

```bash
str="aaaaaaaa\r\nBBBBBBBBBBB\r\n1111"
split_str := strings.Split(string(res[0:len(res)]), "\r\n")
# split_str类型为字符串数组
```

## 2、判断字符串前缀是否包含指定字符

```go
str="aaaaaaaa\r\nBBBBBBBBBBB\r\n1111"
res := strings.HasPrefix(s, "aaa")
// res为布尔值
```

# 四、命令行参数的设置

```go
import ("flag" )
var (
	omhost   string
	omport   string
	ompasswd string
)

func init (){
  flag.StringVar(&omhost, "host", "", "OpenVPN服务端地址")
	flag.StringVar(&omport, "port", "", "OpenVPN服务端管理端口，默认为空")
	flag.StringVar(&ompasswd, "passwd", "", "OpenVPN服务端管理端口密码")
	flag.Parse()
}

func main(){
	if omhost == "" && omport == "" {
		fmt.Println("请在启动命令后添加'-host','-port'参数设置")
		os.Exit(0)
	} else if omhost == "" {
		fmt.Println("请在启动命令后添加'-host'参数设置IP地址")
		os.Exit(0)
	} else if omport == "" {
		fmt.Println("请在启动命令后添加'-port'参数设置管理端口号")
		os.Exit(0)
	}
}
```

# 五、数组的遍历

```go
var test_array = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
for _,a := range test_array {
	fmt.Println(a)
}

var test_array = [3]string{"test1","test2"}
for _,a := range test_array {
	fmt.Println(a)
}
```

# 六、**golang程序初始化**

golang程序初始化先于main函数执行，由runtime进行初始化，初始化顺序如下：

1. 初始化导入的包（包的初始化顺序并不是按导入顺序（“从上到下”）执行的，runtime需要解析包依赖关系，没有依赖的包最先初始化，与变量初始化依赖关系类似，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
2. 初始化包作用域的变量（该作用域的变量的初始化也并非按照“从上到下、从左到右”的顺序，runtime解析变量依赖关系，没有依赖的变量最先初始化，参见[golang变量的初始化](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/PGDzMaYznZVuDiO6V-zYDw)）；
3. 执行包的init函数；



变量初始化->init()->main()
