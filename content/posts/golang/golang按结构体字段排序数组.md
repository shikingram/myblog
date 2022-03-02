---
title: "Golang按结构体字段排序数组"   
author: "Kingram"  
date: 2022-03-02   
lastmod: 2022-03-02

tags: [  
"golang",
]
---

golang的sort包提供了自定义的排序接口Interface，此接口包括三个方法。实现这三个函数即可调用sort包的函数进行排序，示例如下：
```sql
package main

import (
	"fmt"
	"sort"
	"time"
)

type Event struct {
	id int
	CreationTimestamp time.Time
}

type EventList []Event

func (e EventList) Len() int { return len(e) }

func (e EventList) Swap(i, j int) { e[i], e[j] = e[j], e[i] }

func (e EventList) Less(i, j int) bool {
	return e[i].CreationTimestamp.Unix() > e[j].CreationTimestamp.Unix()
}

func main() {
	e1 := Event{
		id : 1,
		CreationTimestamp: time.Now(),
	}
	e2 := Event{
		id : 2,
		CreationTimestamp: time.Now().Add(time.Second * 60),
	}
	e3 := Event{
		id : 3,
		CreationTimestamp: time.Now().Add(time.Second * 120),
	}
	var arr []Event
	arr = append(arr,e1,e2,e3)

	sort.Sort(EventList(arr))

	for _,v := range arr {
		fmt.Println(v)
	}
}
```
