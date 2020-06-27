---
title: "Go Slice"
date: 2020-04-18T11:32:30+08:00
draft: true
---

# 1.切片是啥

​		Go 的 **Slice（切片）**类型提供了一种方便有效的方法来处理类型化数据序列。 slice 与其他语言的数组类似却又不同，简单来说，切片更加灵活，用起来更方便，其原因就是可以扩容。



# 2.举例分析

## 2.1. 认识切片第一步

```go
package main

import "fmt"

func main() {
	sliceExample()
}

func sliceExample() {
	slc := make([]int, 0)
	slc = append(slc, 1)
	fmt.Println(slc)

	var slc1 []int
	slc1 = append(slc1, 1)
	fmt.Println(slc1)

	var slc2 []int
	slc2 = append(slc2, 1)
	fmt.Println(slc2)
}
```

上面的代码是最基本的使用方式，首先看一下上面的代码在底层都做了哪些东西（不要害怕汇编，其实很简单，我们主要明白部分汇编代码即可）

命令：`go tool compile -S slice.go` 我的 go 文件为 slice.go

```assembly
"".main STEXT size=48 args=0x0 locals=0x8
	0x0000 00000 (slice.go:5)	TEXT	"".main(SB), ABIInternal, $8-0
	......
	0x0021 00033 (slice.go:10)	PCDATA	$2, $1
	0x0021 00033 (slice.go:10)	PCDATA	$0, $0
	0x0021 00033 (slice.go:10)	LEAQ	type.int(SB), AX
	0x0028 00040 (slice.go:10)	PCDATA	$2, $0
	0x0028 00040 (slice.go:10)	MOVQ	AX, (SP)
	0x002c 00044 (slice.go:10)	XORPS	X0, X0
	0x002f 00047 (slice.go:10)	MOVUPS	X0, 8(SP)
	0x0034 00052 (slice.go:10)	CALL	runtime.makeslice(SB)
	0x0039 00057 (slice.go:10)	PCDATA	$2, $1
	0x0039 00057 (slice.go:10)	MOVQ	24(SP), AX
	0x003e 00062 (slice.go:11)	PCDATA	$2, $2
	0x003e 00062 (slice.go:11)	LEAQ	type.int(SB), CX
	0x0045 00069 (slice.go:11)	PCDATA	$2, $1
	0x0045 00069 (slice.go:11)	MOVQ	CX, (SP)
	0x0049 00073 (slice.go:11)	PCDATA	$2, $0
	0x0049 00073 (slice.go:11)	MOVQ	AX, 8(SP)
	0x004e 00078 (slice.go:11)	XORPS	X0, X0
	0x0051 00081 (slice.go:11)	MOVUPS	X0, 16(SP)
	0x0056 00086 (slice.go:11)	MOVQ	$1, 32(SP)
	0x005f 00095 (slice.go:11)	CALL	runtime.growslice(SB)
	0x0064 00100 (slice.go:11)	PCDATA	$2, $1
	0x0064 00100 (slice.go:11)	MOVQ	40(SP), AX
	0x0069 00105 (slice.go:11)	MOVQ	48(SP), CX
	0x006e 00110 (slice.go:11)	MOVQ	56(SP), DX
	0x0073 00115 (slice.go:11)	MOVQ	$1, (AX)
	0x007a 00122 (slice.go:12)	PCDATA	$2, $0
	0x007a 00122 (slice.go:12)	MOVQ	AX, (SP)
	0x007e 00126 (slice.go:11)	LEAQ	1(CX), AX
	0x0082 00130 (slice.go:12)	MOVQ	AX, 8(SP)
	0x0087 00135 (slice.go:12)	MOVQ	DX, 16(SP)
	0x008c 00140 (slice.go:12)	CALL	runtime.convTslice(SB)
	......
	0x00e8 00232 (slice.go:15)	PCDATA	$2, $1
	0x00e8 00232 (slice.go:15)	LEAQ	type.int(SB), AX
	0x00ef 00239 (slice.go:15)	PCDATA	$2, $0
	0x00ef 00239 (slice.go:15)	MOVQ	AX, (SP)
	0x00f3 00243 (slice.go:15)	XORPS	X0, X0
	0x00f6 00246 (slice.go:15)	MOVUPS	X0, 8(SP)
	0x00fb 00251 (slice.go:15)	MOVQ	$0, 24(SP)
	0x0104 00260 (slice.go:15)	MOVQ	$1, 32(SP)
	0x010d 00269 (slice.go:15)	CALL	runtime.growslice(SB)
	0x0112 00274 (slice.go:15)	PCDATA	$2, $1
	0x0112 00274 (slice.go:15)	MOVQ	40(SP), AX
	0x0117 00279 (slice.go:15)	MOVQ	48(SP), CX
	0x011c 00284 (slice.go:15)	MOVQ	56(SP), DX
	0x0121 00289 (slice.go:15)	MOVQ	$1, (AX)
	0x0128 00296 (slice.go:16)	PCDATA	$2, $0
	0x0128 00296 (slice.go:16)	MOVQ	AX, (SP)
	0x012c 00300 (slice.go:15)	LEAQ	1(CX), AX
	0x0130 00304 (slice.go:16)	MOVQ	AX, 8(SP)
	0x0135 00309 (slice.go:16)	MOVQ	DX, 16(SP)
	0x013a 00314 (slice.go:16)	CALL	runtime.convTslice(SB)
	......
	0x0196 00406 (slice.go:18)	PCDATA	$2, $1
	0x0196 00406 (slice.go:18)	LEAQ	type.[0]int(SB), AX
	0x019d 00413 (slice.go:18)	PCDATA	$2, $0
	0x019d 00413 (slice.go:18)	MOVQ	AX, (SP)
	0x01a1 00417 (slice.go:18)	CALL	runtime.newobject(SB)
	0x01a6 00422 (slice.go:19)	PCDATA	$2, $1
	0x01a6 00422 (slice.go:19)	LEAQ	type.int(SB), AX
	0x01ad 00429 (slice.go:19)	PCDATA	$2, $0
	0x01ad 00429 (slice.go:19)	MOVQ	AX, (SP)
	0x01b1 00433 (slice.go:19)	XORPS	X0, X0
	0x01b4 00436 (slice.go:19)	MOVUPS	X0, 16(SP)
	0x01b9 00441 (slice.go:19)	MOVQ	$1, 32(SP)
	0x01c2 00450 (slice.go:19)	CALL	runtime.growslice(SB)
	0x01c7 00455 (slice.go:19)	PCDATA	$2, $1
	0x01c7 00455 (slice.go:19)	MOVQ	40(SP), AX
	0x01cc 00460 (slice.go:19)	MOVQ	48(SP), CX
	0x01d1 00465 (slice.go:19)	MOVQ	56(SP), DX
	0x01d6 00470 (slice.go:19)	MOVQ	$1, (AX)
	0x01dd 00477 (slice.go:20)	PCDATA	$2, $0
	0x01dd 00477 (slice.go:20)	MOVQ	AX, (SP)
	0x01e1 00481 (slice.go:19)	LEAQ	1(CX), AX
	0x01e5 00485 (slice.go:20)	MOVQ	AX, 8(SP)
	0x01ea 00490 (slice.go:20)	MOVQ	DX, 16(SP)
	0x01ef 00495 (slice.go:20)	CALL	runtime.convTslice(SB)
```



上面汇编我们不需要全部看明白只需要看懂6、11、15、23、34、37、44、55、58、61、69、80这几行就可以，也就是下面的代码：

```assembly
slc := make([]int, 0)
slc = append(slc, 1)
fmt.Println(slc)
	
0x0021 00033 (slice.go:10)	LEAQ	type.int(SB), AX
0x0034 00052 (slice.go:10)	CALL	runtime.makeslice(SB)
0x003e 00062 (slice.go:11)	LEAQ	type.int(SB), CX
0x005f 00095 (slice.go:11)	CALL	runtime.growslice(SB)
0x008c 00140 (slice.go:12)	CALL	runtime.convTslice(SB)
============================
var slc1 []int
slc1 = append(slc1, 1)
fmt.Println(slc1)

0x00e8 00232 (slice.go:15)	LEAQ	type.int(SB), AX
0x010d 00269 (slice.go:15)	CALL	runtime.growslice(SB)
0x013a 00314 (slice.go:16)	CALL	runtime.convTslice(SB)
0x0196 00406 (slice.go:18)	LEAQ	type.[0]int(SB), AX
============================
slc2 := []int{}
slc2 = append(slc2, 1)
fmt.Println(slc2)

0x01a1 00417 (slice.go:18)	CALL	runtime.newobject(SB)
0x01c2 00450 (slice.go:19)	CALL	runtime.growslice(SB)
0x01ef 00495 (slice.go:20)	CALL	runtime.convTslice(SB)
```

​		为什么要看这些汇编代码呢？go 里面很多内置的方法我们是不能直接找到对应的实现函数的，所以我们通过这个 go tool compile 工具来看一下，就知道了。上面代码的第5、6行，就是 make slice 的实现，也就是说调用的函数就是 runtime 包中的 makeslice 函数。接下来的7、8行就是 appen 的具体实现，同样是 runtime 包，growslice 函数。第9行就是 fmt.Println 的具体实现，也就是 runtime.convTslice 函数。这样子我们就知道该去哪里看对应代码了。其实通过汇编能看到很多东西，这里我们只说 slice，以后有机会会继续和大家分享。

​		上面的汇编代码我分为了三部分，也就是对应三中 slice 的声明，估计大家都看出来了，每种声明方式对应的实现函数都是不一样的，第一个是 runtime.makeslice，第二个没有对应的实现，第三个是 runtime.newobject，这三种方式区别还是有的，但是最终都可以实现我们的目标，因为最终都是调用了 runtime.mallocgc 函数，也就是分配内存，看到这有同学就会问了，第二种方式我们并没有对应的实现啊，也就是说并没有分配内存啊。是的，第二种方式我们确实没有直接分配内存，并且我们直接 append 了，其实是因为 append 操作会对没有分配内存的切片再分配一下内存（感兴趣的同学可以仔细看一下 growslice 源码）。所以我们在写代码时，使用 `var` 关键字声明切片是完全可以的，并且对于长度为0的切片我们非常建议这样声明。

​		我们简单看一下 `makeslice` 代码：

```go
func makeslice(et *_type, len, cap int) unsafe.Pointer {
    // 判断 cap 是否溢出
	mem, overflow := math.MulUintptr(et.size, uintptr(cap))
    
    // 溢出或者 len、cap 不符合要求
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

    // 真正的分配内存
	return mallocgc(mem, et, true)
}
```



​		再看一下 `newobject` 代码：

```go
func newobject(typ *_type) unsafe.Pointer {
	return mallocgc(typ.size, typ, true)
}
```

​		没错，上面两个的最终都是调用 `mallocgc` 函数。只不过 `makeslice` 先做了一些判断。



## 2.2. 切片源码

我们看一下切片的源码（go/src/runtime/slice.go）：

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

就是这么简单，一个切片就是三个字段，第一个是内存开始的指针，第二个是切片的长度，最后一个就是切片的容量。`slice.go` 文件中其他函数都是针对切片的函数：

* `makeslice`：初始化切片。
* `makeslice64`：初始化长度和容量为 int64 类型的切片，最终还是调用 `makeslice`。
* `growslice`：切片扩容（内置 `append` 函数）。
* `slicecopy`：切片的拷贝（内置 `copy` 函数）。



## 2.3. 切片和数组

切片是数组的片段，也就是说从数组上截取一段，然后我们切片可以在这个数组上“活动”，同样我们都可以使用下标来访问某个元素，但是如果我们给切片添加元素时，如果长度超过了当前数组的长度，那我们就再寻找一个新的内存，同样是新的数组、新的切片，但是对于我们代码使用者来说，切片还是不变的，但是它在内存的位置改变了。



## 2.4. append

`append` 应该是我们最常用的操作了，在文章开始部分我们已经看过简单的汇编语言了，并且已经知道其源码就是 `runtime/slice.go` 中的 `growslice` 函数。估计大家都知道切片是如何扩容的，网上大多数资料都是当原 slice 容量小于 `1024` 的时候，新 slice 容量变成原来的 `2` 倍；原 slice 容量超过 `1024`，新 slice 容量变成原来的`1.25`倍。这样说我们不能说完全正确，这样的说法还不算严谨。为啥不严谨呢，我们接着往下看。

`append` 参数可以是多个，那么我们就有一下两种写法：

```go
s := []string{"a", "b"}

// append 多个元素
s := append(s,"a", "b", "c")

// append 单个元素
s = append(s, "c")
s = append(s, "d")
s = append(s, "e")
```

那么这两种写法最后的切片 s 的长度和容量都是多少呢？

```go
s := []string{"a", "b"} // 此时切片长度为2，容量也为2。
s = append(s, "c")
s = append(s, "d")
s = append(s, "e")
fmt.Printf("len=%d, cap=%d", len(s), cap(s)) // 结果：len=5, cap=8
```

```go
s := []string{"a", "b"} // 此时切片长度为2，容量也为2。
s = append(s, "c", "d", "e")
fmt.Printf("len=%d, cap=%d", len(s), cap(s)) // 结果：len=5, cap=5
```

事实证明，不同的写法对切片最终的容量是有影响的。也就是说，对于 `append` 多个参数时，并不会对每一个元素添加是都会进行扩容，而是对整体的所有元素来进行扩容，并且在元素类型不同时，最终的容量也是不同的。

```go
s := []int{1, 2} // 此时切片长度为2，容量也为2。
s = append(s, 3, 4, 5)
fmt.Printf("len=%d, cap=%d", len(s), cap(s)) // 结果：len=5, cap=6
```

其原因是元素类型所占的内存大小是不一样的，从而导致 `append` 操作时进行 **内存对齐** 的结果也不一样（内存对齐这里不再赘述，以后有时间也会写类似文章）。所以我们上面三种代码最终的结果都是不同的。

结论：在 appen 单个元素时，扩容规律确实是2倍或者1.25倍，但是appen 多个元素时，结果和元素类型是相关的，容量最小和长度相同。



## 2.5. 切片截取

我们还要注意的是，扩容时如果容量大于原有数据的长度，我们重新分配内存，其操作不会影响原有的数据。但是没有分配新的内存，也就是说还是原来数组的基础上添加元素，那么新的切片操作就会影响原有的数组。这部分依然不再赘述，看一下下面代码，大家都明白了：

```go
s := []int{1, 2, 3, 4}
s1 := s[1:2]
fmt.Printf("len=%d, cap=%d\n", len(s), cap(s)) // len=4, cap=4
fmt.Println(s) // [1 2 3 4]
fmt.Println(s1) // [2]
fmt.Printf("len=%d, cap=%d\n", len(s1), cap(s1)) // len=1, cap=3

s1[0] = 5
fmt.Println(s) // [1 5 3 4]
fmt.Println(s1) // [5]

s1 = append(s1, 1)
fmt.Printf("len=%d, cap=%d\n", len(s1), cap(s1)) // len=2, cap=3
s1[0] = 6
fmt.Println(s) // [1 6 1 4]
fmt.Println(s1) // [6 1]

s1 = append(s1, 1, 2, 3)
fmt.Printf("len=%d, cap=%d\n", len(s1), cap(s1)) // len=5, cap=6
s1[0] = 7
fmt.Println(s) // [1 6 1 4]
fmt.Println(s1) // [7 1 1 2 3]
```

另外还有一点，切片截取也是可以截取 cap 的：

```go
s := []int{1, 2, 3, 4}
fmt.Printf("len=%d, cap=%d\n", len(s), cap(s)) // len=4, cap=4
	
s1 := s[1:2:3]
fmt.Printf("len=%d, cap=%d\n", len(s1), cap(s1)) // len=1, cap=2
	
s2 := s[1:2]
fmt.Printf("len=%d, cap=%d\n", len(s2), cap(s2)) // len=1, cap=3
```



## 2.6. 拷贝

切片的拷贝可以使用内置 copy 函数：

```go
s := []int{1, 2, 3, 4}
var s1 []int
copy(s1, s)
fmt.Println(s1) // []
fmt.Println(s) // [1 2 3 4]

s1 = make([]int, 2)
count := copy(s1, s)
fmt.Println(s1) // [1 2]
fmt.Println(s) // [1 2 3 4]
fmt.Println(count) // 2

s1[0] = 5
fmt.Println(s1) // [5 2]
fmt.Println(s) // [1 2 3 4]
```

上面的例子可以看出来，copy 只会拷贝目标切片长度个元素，并且 copy 后两个切片是没有影响的。

还有一种更加高效的实现方式：

```go
s := []int{1, 2, 3, 4}
s2 := append(s[:0:0], s...)
fmt.Println(s2) // [1 2 3 4]
s2[0] = 5
fmt.Println(s2) // [5 2 3 4]
fmt.Println(s) // [1 2 3 4]
```

上面这种方式算是一个比较取巧的方法，并且可以达到 copy 的效果，并且速度要比 copy 快很多。需要注意的是第2行中 cap 的下标是一定要写的，并且建议写0，这样效率是最高的。



## 2.7. 传递

Go 语言函数参数都是值传递，在函数内部会拷贝一份，所以 slice 传递后拷贝一份都是新的 slice，但是 map 这种初始化之后就是一个 *hmap，所以参数传递后还是指向同一个 map（map 以后再详解）。

```go
func slice1() {
	s := []string{"a", "b"}
	fmt.Println(s)          // [a b]
	fmt.Printf("%p \n", &s) // 0xc0000a6020
	slice2(s)
	fmt.Println(s)          // [a c]
}

func slice2(s []string) {
	s[1] = "c"
	fmt.Println(s)          // [a c]
	fmt.Printf("%p \n", &s) // 0xc0000a6080
}
```

上面的代码将切片作为参数确实在 slice2 函数中修改了传进来的参数s，并且在 slice1 函数中的 s 也确实改变了。但是这并不叫做引用传递，我们可以看到两个切片 s 的地址是不一样的，我们传给 slice2 函数的切片是一个新的切片，并不是 slice1 中的 s，但是他们底层都是同一个数组，所以函数 slice2 中修改了切片s底层的数组，所以 slice1 函数中的切片 s 的底层也修改了，但是记住，这依然不叫做引用传递，但是看起来却和引用传递是一样的。



## 2.8. 总结

切片暂时告一段落，最后总结一下：

* 切片是对数组的封装，切片可以自动扩容。
* 切片添加后，新的容量小于之前的容量，那么还是使用原有的数组，并且两者之间的元素修改有影响，因为底层是同一块内存。
* 切片的扩容并不一定总是原有容量的2倍或者1.25倍，当 `append` 多个元素时，会有内存对齐 ，最终的容量大于等于长度（这是一句废话，容量小于长度就 panic 了）。 
* 使用 `var` 声明的切片可以直接 `append`，并且我们建议这样，因为在声明时不会分配内存。
* 已知切片容量或者长度时，声明时最好也指定容量或者长度，因为扩容导致重新分配内存消耗太大了。





参考链接：

* https://qcrao.github.io/2019/04/02/dive-into-go-slice/（饶大）
* https://xargin.com/go-slice/（曹大）

* https://www.tutorialspoint.com/go/go_slice.htm（需要梯子）
* https://medium.com/rungo/the-anatomy-of-slices-in-go-6450e3bb2b94（需要梯子）
* https://www.thegeekstuff.com/2019/03/golang-slice-examples/（需要梯子）

