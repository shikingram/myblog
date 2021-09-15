{
  "title":"golang通知协程退出",
  "tags":[
    "golang"
  ],
  "date":"2021-09-15",
  "lastmod":"2021-09-15",
  "draft":"false",
  "author":"kingram"
}
 
```
/*
	一个安全关闭的的协程模型
*/
type MyChannel struct {
	C    chan struct{}
	once sync.Once
}

func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan struct{})}
}

func (mc *MyChannel) SafeClose() {
	mc.once.Do(func() {
		close(mc.C)
	})
}
```

使用waitgroup和chanel是否关闭的判断来通知和等待协程退出

```
func main() {
	myChan := NewMyChannel()
	wg := &sync.WaitGroup{}
	wg.Add(1)
	go func() {
		for {
			select {
			case _,ok  := <-myChan.C:
				if !ok {
					fmt.Println("-> task out")
					wg.Done()
					return
				}
			default:
				fmt.Println("task ... ")
			}
			time.Sleep(time.Millisecond * 200)
		}
	}()

	time.Sleep(time.Second * 5)

	// 通知退出
	myChan.SafeClose()
	// 等待退出
	wg.Wait()
}
```