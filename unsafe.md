# unsafe.pointer的用法

之前一直对unsafe.Pointer的使用不是特别理解，找了几篇文章看了下，其主要作用是转换任意类型的指针，**用在结构体中取数值比较常用。**



正常的go语言中，不同类型的指针，无法进行转换。

例如

```go
package main

import (
	"fmt"
)

func main() {
	i := 10
	ip := &i

	var fp *float64 = (*float64)(ip)

	fmt.Println(fp)
}

C:\gostudy\src\test\ttt>go run main.go
# command-line-arguments
.\main.go:11:30: cannot convert ip (type *int) to type *float64
```

使用unsafe.pointer可以将int类型的指针转换为float64类型的指针

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	i := 10
	ip := &i

	var fp *float64 = (*float64)(unsafe.Pointer(ip))

	*fp = *fp * 3

	fmt.Println(i)
}

```

在结构体中，利用偏移量进行计算的技巧

```
可以看到unsafe.Pointer其实就是一个*int,一个通用型的指针。

我们看下关于unsafe.Pointer的4个规则。

任何指针都可以转换为unsafe.Pointer
unsafe.Pointer可以转换为任何指针
uintptr可以转换为unsafe.Pointer
unsafe.Pointer可以转换为uintptr
```

举例

```go
func main() {
	u:=new(user)
	fmt.Println(*u)

	pName:=(*string)(unsafe.Pointer(u))
	*pName="张三"

	pAge:=(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(u))+unsafe.Offsetof(u.age)))
	*pAge = 20

	fmt.Println(*u)
}

type user struct {
	name string
	age int
}
```

```
第一个修改user的name值的时候，因为name是第一个字段，所以不用偏移，我们获取user的指针，然后通过unsafe.Pointer转为*string进行赋值操作即可。

第二个修改user的age值的时候，因为age不是第一个字段，所以我们需要内存偏移，内存偏移牵涉到的计算只能通过uintptr，所我们要先把user的指针地址转为uintptr，然后我们再通过unsafe.Offsetof(u.age)获取需要偏移的值，进行地址运算(+)偏移即可。
```

注意：这里的uintptr千万不要分开写，否则可能会牵扯出GC的问题，莫名奇妙就会出错。