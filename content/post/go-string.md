---
title: "Go String"
date: 2020-05-17T17:26:57+08:00
draft: true
---

# 1. string 简介

​	string 肯定不陌生了，这个可以说是我们平时最常用的，其实字符串就是内存中一片连续的空间，我们也可以理解成字符的数组。在 Go 中我们是可以将 []byte 与 string 直接转换的，今天我们就仔细的聊一下 Go 语言中的 string 类型的一些东西。



# 2. 源码

## 2.1. string 类型源码

​	在 go/src/reflect/value.go 文件中，对于 string 类型有两个结构体，如下：

```go
// StringHeader is the runtime representation of a string.
// It cannot be used safely or portably and its representation may
// change in a later release.
// Moreover, the Data field is not sufficient to guarantee the data
// it references will not be garbage collected, so programs must keep
// a separate, correctly typed pointer to the underlying data.
type StringHeader struct {
	Data uintptr
	Len  int
}

// stringHeader is a safe version of StringHeader used within this package.
type stringHeader struct {
	Data unsafe.Pointer
	Len  int
}
```

​		在 go/src/runtime/string.go 文件中，和string ，类型相关结构如下：

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

​	暂时不说上面每份代码是做什么的，但是通过上面两份代码我们大概就可以知道 string 类型是什么样了，没错和 slice 结构非常相似，就是相差一个 cap 字段。string 类型就是一个指针加上一个长度，也就是这段连续的内存就是一个 string 类型的数据。

​	看上面代码的注释可以知道，reflect/value.go 中的 StringHeader 结构是代表运行时的 string 类型。而 runtime/string.go 中的 stringStruct 结构是运行时 string 类型部分操作使用的一个结构，经常将 string 类型转成 stringStruct 类型然后在给两个字段赋值。



## 2.2. + 拼接

​	首先我们对于字符串最常用的应该就是 `+` 的操作了，我们就先看这个操作的源码：

```go
// The constant is known to the compiler.
// There is no fundamental theory behind this number.
const tmpStringBufSize = 32

type tmpBuf [tmpStringBufSize]byte

// concatstrings implements a Go string concatenation x+y+z+...
// The operands are passed in the slice a.
// If buf != nil, the compiler has determined that the result does not
// escape the calling function, so the string data can be stored in buf
// if small enough.
func concatstrings(buf *tmpBuf, a []string) string {
	idx := 0
	l := 0
	count := 0
	for i, x := range a {
		n := len(x)
		if n == 0 {
			continue
		}
		if l+n < l {
			throw("string concatenation too long")
		}
		l += n
		count++
		idx = i
	}
	if count == 0 {
		return ""
	}

	// If there is just one string and either it is not on the stack
	// or our result does not escape the calling frame (buf != nil),
	// then we can return that string directly.
	if count == 1 && (buf != nil || !stringDataOnStack(a[idx])) {
		return a[idx]
	}
	s, b := rawstringtmp(buf, l)
	for _, x := range a {
		copy(b, x)
		b = b[len(x):]
	}
	return s
}

func concatstring2(buf *tmpBuf, a [2]string) string {
	return concatstrings(buf, a[:])
}

func concatstring3(buf *tmpBuf, a [3]string) string {
	return concatstrings(buf, a[:])
}

func concatstring4(buf *tmpBuf, a [4]string) string {
	return concatstrings(buf, a[:])
}

func concatstring5(buf *tmpBuf, a [5]string) string {
	return concatstrings(buf, a[:])
}
```

​	可以看到最主要的就是 `concatstrings` 这个函数，其他函数都是对其的封装，在我们使用 `+` 拼接的字符串个数超过5个，就会直接调用 `concatstrings` ，当拼接的个数为2两个那么就会调用 `concatstring2` 。所以我们简单分析一下最主要的函数就可以了。

​	`tmpBuf` 是一个长度为32的字节数组（为什么长度是32，注释中说没有任何理论依据，大概就是任性吧。但是我个人觉得在大多数情况下32的长度已经足够了，这样就不用再重新申请内存空间了，也是为了提升效率）。第二个参数是 string 的切片，也就是我们要拼接的字符串列表。

* 首先我们遍历一遍参数 `a []string`，如果切片中的一个 string 元素长度溢出，程序会 panic。如果  `a` 长度为0，直接返回空字符串。
* 如果只有一个字符串元素，并且  `buf` 不为 `nil`，同时不在堆栈上，那么就直接返回这个字符串。
* 根据 `buf（长度32的字节数组）` 和 `l（拼接字符串总长度）` 生成一个字符串 `s` 和一个切片 `b`，同时 `b` 就是字符串 `s` 中的内存字节数组。再将参数 `a` 中的每个元素拷贝到切片 `b` 中。

此时就完成的字符串的 `+` 操作。



## 2.3. string 与 slice

### 2.3.1 源码

```go
s := "hello"
b := []byte(s)
s = string(b)
```

​	上面的代码应该是写的比较多的了，也就是说 Go 中 `string` 和 `[]byte` 是可以轻松转换的，大家可以回想一下 `slice` 和 `string` 的底层结构，相同的地方是都有一个指针和长度，不同的是 `slice` 还有一个 `cap`。所以大家可以试想一下，对于 `string` 和 `slice` 两种结构的转换，如果是你，你会如何实现？

​	我们还是使用 `go tool compile -S` 这个命令来看一下上面的代码，这里就直接看 Go 语言是如何实现的。在 `go/src/runtime/string.go` 文件中，有两个函数 `stringtoslicebyte` 和 `slicebytetostring`，这两个就是其实现的函数。

```go
// Buf is a fixed-size buffer for the result,
// it is not nil if the result does not escape.
func slicebytetostring(buf *tmpBuf, b []byte) (str string) {
	l := len(b)
	if l == 0 {
		// Turns out to be a relatively common case.
		// Consider that you want to parse out data between parens in "foo()bar",
		// you find the indices and convert the subslice to string.
		return ""
	}
	if raceenabled {
		racereadrangepc(unsafe.Pointer(&b[0]),
			uintptr(l),
			getcallerpc(),
			funcPC(slicebytetostring))
	}
	if msanenabled {
		msanread(unsafe.Pointer(&b[0]), uintptr(l))
	}
	if l == 1 {
		stringStructOf(&str).str = unsafe.Pointer(&staticbytes[b[0]])
		stringStructOf(&str).len = 1
		return
	}

	var p unsafe.Pointer
	if buf != nil && len(b) <= len(buf) {
		p = unsafe.Pointer(buf)
	} else {
		p = mallocgc(uintptr(len(b)), nil, false)
	}
	stringStructOf(&str).str = p
	stringStructOf(&str).len = len(b)
	memmove(p, (*(*slice)(unsafe.Pointer(&b))).array, uintptr(len(b)))
	return
}

type stringStruct struct {
	str unsafe.Pointer
	len int
}

func stringStructOf(sp *string) *stringStruct {
	return (*stringStruct)(unsafe.Pointer(sp))
}
```

​	上面的代码是 `[]byet` 转 `string` 的实现（核心代码就是26行至35行）：

* 首先定义指针 p，如果 buf 不是 nil并且切片 b 的长度小于等于 buf 的长度，那么 p 就指向 buf 数组。否则分配一块内存，大小为切片 b 的长度，并且 p 指向这块内存。

* 将返回值 str 字符串指针转成 stringStruct 的指针，其 str 字段就是上一步的指针 p，len 字段就是切片 b 的长度。

* 再将切片 b 内存数据移动到指针 p 下。此时 返回值 str 字符串变量就是切片 b 的数据。

    另外一个操作就是 `string` 转 `[]byte` ，代码也非常简单，这里就不把代码拿过来了，感兴趣大家可以去源码看下，其实现方式最主要的就是将字符串下的指针指向的内存拷贝到目标切片中。



### 2.3.2 语法糖

​	大家都知道内置的 `copy` 和 `appen` 函数可以操作切片，同样也是可以操作 `string` 类型的：

```go
slice1 := make([]byte, 10)
string1 := "hello"
copy(slice1, string1)
fmt.Println(string(slice1)) // hello

slice2 := make([]byte, 0)
string2 := "world"
slice2 = append(slice2, string2...)
fmt.Println(string(slice2)) // world
```

​	 这种写法可以算是 Go 提供给我们的一种语法糖，也就是我们可以省略将 `string` 类型转换成 `[]byte` 类型这一步。



### 2.3.3. string 只读内存

```go
str := "hello"
p := unsafe.Pointer(&str)
var sl1 = new([]byte)
sl1 = (*[]byte)(p)
(*sl1)[0] = 1
fmt.Println(string(*sl1))
```

​	上面的代码很简单，主要目的就是对于变量 `str` 的空间更改一下数据，运行这段代码是会报错的，原因就是 `string` 都是分配在只读内存上，强行修改就会出错。



## 2.3. for 循环与 rune

```go
s := "hello"
for k, v := range s {
	fmt.Println(k)          // 0,1,2,3,4
	fmt.Println(string(v))  // h,e,l,l,o
}

s = "hello世界哈"
for k, v := range s {
	fmt.Println(k)         // 0,1,2,3,4,5,8,11
	fmt.Println(string(v)) // h,e,l,l,o,世,界,哈
}
```

​	上面代码大家执行结果和大家预想的是否一致？当我们使用 `go tool compile -S` 命令时，我们就会发现，对于 `string`  类型的底层使用了 `runtime.decoderune` 这个函数，位置在：`go/src/runtime/utf8.go` 中。这里我们不再细讲这个函数的实现，我们首先要明白对于汉字和英文字母其大小是不一样的，Go 中使用的是 utf-8，一个因为字母是1字节，一个汉字是3字节（关于编码部分会在以后的文章中和大家讨论）。所以说对于字符串的 for 遍历，其实遍历的是 `码点（codepoint）` ，也就是对于汉字这种大小不是1字节的，我们使用 for 循环时是不用担心的。

​	这是大家可能就会想对于 `hello世界哈` 这个字符串 for 循环时下标没有`5,6,7,9,10`，那么我们修改了会怎样呢？看下面的代码：

```go
s := "hello世界哈"
fmt.Println(string(s[5])) // ä
```

​	对于上面代码，如果我们不知道 Go 中 `string` 的底层或者不明白 `utf-8` 或者码点等，那么就可能认为字符串 `hello世界哈` 下标为5的应该是汉字 `世`，但实际情况并不是，我们已经通过 for 循环的形式知道了 `string` 底层下标的规则了，想必这个问题大家以后敲代码应该不会再犯这个错误了。

​	此时应该有同学问了，那如果想根据下标修改或者查看某个位置怎么办？那应该就是下面的情况了：

```go
s := "hello世界哈"
sb := []byte(s)
fmt.Println(string(sb)) // hello世界哈
sb[1] = 97
fmt.Println(string(sb)) // hallo世界哈
sb[5] = 97
fmt.Println(string(sb)) // halloa��界哈
```

​	那么怎么解决上面的问题呢？那就继续看下面的代码：

```go
s := "hello世界哈"
sr := []rune(s)
fmt.Println(string(sr))    // hello世界哈
fmt.Println(len(sr))       // 8
fmt.Println(string(sr[7])) // 哈

s1 := "你好"
s1r := []rune(s1)
sr[7] = s1r[1]
fmt.Println(string(sr))   // hello世界好
```

​	上面代码就将 `hello世界哈` 中的 `哈` 字改成了 `好` 字。



## 2.4. 总结一下

​	关于 `string ` 类型就告一段落，基本上常用的都说了，希望大家下次用到时可以再回想一下这篇文章，有一些错误就不会再遇到了，而且对于基本的使用也更顺手了。



> 补充：对于 rune 类型本文并没有过多介绍，关于编码等问题可以在以后的文章中再和大家聊一聊。那时我们再仔细的聊一下 rune （其实就是 int32 哈哈）。

