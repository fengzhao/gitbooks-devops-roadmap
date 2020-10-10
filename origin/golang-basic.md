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

