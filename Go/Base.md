# string

```go
func main() {
    s := "golang"
    for _, v := range s {
        fmt.Printf("v:%T", v)
    }

    fmt.Printf("v:%T", s[0])
}
```

- range遍历字符串：rune类型(int32)

- 切片索引字符串：byte类型(int8)



# array

1、求两个数组是否存在连续交集：

```go
start := max(section1[0], section2[0])
end := min(section1[1], section2[1])
if start <= end {
    // 存在交集
}
```


