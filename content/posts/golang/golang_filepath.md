---
title: "golang程序中的当前目录，当前文件目录，当前执行目录..."   
author: "Kingram"  
date: 2022-04-22
lastmod: 2022-04-22

tags: [  
"golang",
"goland",
]
---
## 问题描述
通常我们需要写一个os.Open()去打开配置文件的时候，这里面目录应该怎么写呢？写绝对路径吧，调试生产路径不一样，打镜像上生产的时候又容易忘掉改，写相对路径吧，总是遇到各种莫名其妙的找不到文件错误
```shell
panic: open ./internal/pkg/util/web3/contract/abi/Controller.abi: no such file or directory
```
## 实际场景
要搞清楚为什么找不到文件，首先要搞清楚几个目录，就是标题中的几个目录，这几个目录是相对于程序而言的，所以可能有点怪
- 当前目录，编译好的可执行文件所在的目录
- 当前文件目录，项目的main.go文件所在的目录
- 当前执行目录，实际执行这个可执行文件的时候所在的目录

下面模拟一个实际的场景就能理解这三个目录了

首先是项目结构
```shell
/Users/kingram/workerspace/TestPath
├── cfg
│   └── cfg.json
├── cmd
│   └── main.go
└── go.mod
```
在main里打开配置文件
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	_, err := os.Open("./cfg/cfg.json")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	fmt.Println("i am safe")
}

```
然后到cmd目录下构建项目，得到main可执行文件，再将可执行文件和配置目录拷贝到`/Users/kingram/workerspace/projectPath`
```shell
cd /Users/kingram/workerspace/TestPath/cmd
go build -o main 

cp main /Users/kingram/workerspace/projectPath/ 
cp -r ../cfg  /Users/kingram/workerspace/projectPath/
```

此时
```shell
/Users/kingram/workerspace/projectPath
├── cfg
│   └── cfg.json
└── main
```
- 当前目录 = `/Users/kingram/workerspace/projectPath`
- 当前文件目录 = `/Users/kingram/workerspace/TestPath/cmd/main.go`

那么当前执行目录呢，我们cd到~目录下执行编译好的执行文件
```shell
cd /Users/kingram/

/Users/kingram/workerspace/projectPath/main
```
此时执行目录就是`/Users/kingram/`

然后发现报了错
```shell
open ./cfg/cfg.json: no such file or directory
```

很奇怪，配置文件明明拷贝到`/Users/kingram/workerspace/projectPath/`下了，为什么找不到？

原因就是open()中`./cfg/cfg.json`是相对于当前执行目录，补全后就是`/Users/kingram/cfg/cfg.json`

## Golang中获取当前目录、当前文件目录、当前执行目录

```go
package main

import (
	"fmt"
	"log"
	"os"
	"path/filepath"
	"runtime"
	"strings"
)

func main() {
	showPath()
	_, err := os.Open(GetCurrentDirectory()+"/cfg/cfg.json")
	if err != nil {
		fmt.Println(err.Error())
		return
	}
	fmt.Println("i am safe")
}

func showPath() {
	fmt.Println("当前文件目录=>",CurrentFile())
	fmt.Println("当前目录=>",GetCurrentDirectory())
	fmt.Println("当前执行目录=>",CurrentPath())
}

func CurrentFile() string {
	_, file, _, ok := runtime.Caller(1)
	if !ok {
		panic("Can not get current file info")
	}
	return file
}

func GetCurrentDirectory() string {
	dir, err := filepath.Abs(filepath.Dir(os.Args[0]))
	if err != nil {
		log.Fatal(err)
	}

	return strings.Replace(dir, "\\", "/", -1)
}

func CurrentPath()string{
	getwd, err := os.Getwd()
	if err != nil {
		return ""
	}
	return getwd
}
```
再次模拟上面的场景，输出如下：
```shell
当前文件目录=> /Users/kingram/workerspace/TestPath/cmd/main.go
当前目录=> /Users/kingram/workerspace/projectPath
当前执行目录=> /Users/kingram
i am safe
```
验证无误

## Goland设置输出目录
如果按照上面的写法，会发现点击Goland的三角形直接运行或者Debug会找不到文件，这是因为我们没有设置Goland的go build输出路径

设置在右上角`go build TestPath/cmd`点开把输出目录设置成`/Users/kingram/workerspace/TestPath`

重新点击后，即可使用绿色三角运行或者Debug项目了
