# 并发变量在for 循环传递
今天刚好并发去call 做一个报告，如下例子

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	ch := make(chan struct{},3)
	for key, _ := range []string{"aa","bb","cc","dd","ee","ff","gg"} {
		ch <- struct{}{}
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println(key)
			// call api
			time.Sleep(time.Millisecond)
			<-ch
        }()
    }

	wg.Wait()
}
```
这个例子并发的普通例子，然后发现数据一直打印的是
aa
aa
aa
...

# 解决
大概顺序如下
1. 创建缓冲区
2. 开启携程,变量要放在go func(里面)
3. 结束任务释放缓冲区

```go
package main

import (
	"fmt"
	"sync"
	"time"
)


func main() {
	var wg sync.WaitGroup
	ch := make(chan struct{},3)
	for _, value := range []string{"aa","bb","cc","dd","ee","ff","gg"} {
		acct := value
		ch <- struct{}{}
		wg.Add(1)
		go func(acct string) {
			defer wg.Done()
			fmt.Println(value)
			// call api
			time.Sleep(time.Millisecond)
			<-ch
		}(acct)
	}

	wg.Wait()
}
```

