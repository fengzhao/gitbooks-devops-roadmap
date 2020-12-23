# Logstash常用filter实现的功能

# 1、截取带有文件路径字段中的文件名

```bash
filter{
 grok {
    match => {
      "[log][file][path]" => "%{GREEDYDATA}/%{GREEDYDATA:app}-access.log"
    }
  }
}
```

