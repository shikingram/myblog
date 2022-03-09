---
title: "go语言map拷贝陷阱"   
author: "Kingram"  
date: 2022-03-09   
lastmod: 2022-03-09

tags: [  
"golang",
]
---
## map 可以拷贝吗？

map 其实是不能拷贝的，如果想要拷贝一个 map ，只有一种办法就是循环赋值，就像这样
```go
originalMap := make(map[string]int)
originalMap["one"] = 1
originalMap["two"] = 2

// Create the target map
targetMap := make(map[string]int)

// Copy from the original map to the target map
for key, value := range originalMap {
    targetMap[key] = value
}
```

如果 map 中有指针，还要考虑深拷贝的过程
```go
originalMap := make(map[string]*int)
var num int = 1
originalMap["one"] = &num

// Create the target map
targetMap := make(map[string]*int)

// Copy from the original map to the target map
for key, value := range originalMap {
var tmpNum int = *value
    targetMap[key] = &tmpNum
}
```

如果想要更新 map 中的value，可以通过赋值来进行操作
```go
map["one"] = 1
```
但如果 value 是一个结构体，可以直接替换结构体，但无法更新结构体内部的值
```go
originalMap := make(map[string]Person)
originalMap["minibear2333"] = Person{age: 26}
originalMap["minibear2333"].age = 5
```