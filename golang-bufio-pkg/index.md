# 【Golang】bufio包的使用


## bufio包介绍

`bufio`包可以从文件或标准输入读取数据，还可以向文件写入数据。

读取数据时，先将数据读入缓冲区，然后再从缓冲区中读取。写入数据时，先把数据写入缓冲区，然后在某个时间节点，一次性地将数据写入文件。

通过缓冲区，能有效减少读写磁盘的次数。

注意，此处的缓冲区是指`Golang`为应用程序分配的一块内存空间。


## 使用

下面仅介绍一些常见用法。

### 1. 读取数据

```go
func main() {
	file1, err := os.OpenFile("./data.txt", os.O_CREATE|os.O_RDWR, os.ModePerm)
	if err != nil {
		log.Fatal(err)
	}
	defer file1.Close()

	// 创建缓冲区，默认大小是4096
	buf := bufio.NewReader(file1)

	b := make([]byte, 64)
	n, err := buf.Read(b)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(buf.Buffered())
	fmt.Println(string(b[:n]))
}
```

### 2. 按行读取

```go
func main() {
	file1, err := os.OpenFile("./data.txt", os.O_CREATE|os.O_RDWR, os.ModePerm)
	if err != nil {
		log.Fatal(err)
	}
	defer file1.Close()

	// 创建缓冲区，默认大小是4096
	buf := bufio.NewReader(file1)

	// 按行读取
	for {
		bs1, _, err := buf.ReadLine()
		if err == io.EOF {
			break
		}
		fmt.Println(string(bs1))
	}
}
```

### 3. 读取多个字节

```go
func main() {
  	file1, err := os.OpenFile("./data.txt", os.O_CREATE|os.O_RDWR, os.ModePerm)
	if err != nil {
		log.Fatal(err)
	}
	defer file1.Close()

	// 创建缓冲区，默认大小是4096
	buf := bufio.NewReader(file1)

	// 指定每次读取的分割符
	for {
		bs2, err := buf.ReadBytes('\n')
		if err == io.EOF {
			break
		}
		fmt.Print(string(bs2))
	}
}
```

### 4. 读取字符串

```go
func main() {
  	file1, err := os.OpenFile("./data.txt", os.O_CREATE|os.O_RDWR, os.ModePerm)
	if err != nil {
		log.Fatal(err)
	}
	defer file1.Close()

	// 创建缓冲区，默认大小是4096
	buf := bufio.NewReader(file1)

	for {
		str, err := buf.ReadString('\n')
		if err != nil {
			break
		}
		fmt.Print(str)
	}
}
```

### 5. 从标准输入读取

```go
func main() {
	// 读取输入
	buf := bufio.NewReader(os.Stdin)
	for {
		s2, _, err := buf.ReadLine()
		fmt.Println(string(s2))
		fmt.Println(err)
		if string(s2) == "q" {
			break
		}
	}
}
```

### 6. 写入数据到文件

```go
func main() {
	// 写入数据
	file, err := os.OpenFile("./data1.txt", os.O_CREATE|os.O_RDWR, os.ModePerm)
	if err != nil {
		log.Fatal(err)
	}
	buf2 := bufio.NewWriter(file)
	n, err := buf2.WriteString("今夜特别漫长\n")
	if err != nil {
		log.Fatal(err)
	}
	// 将缓冲区数据刷入磁盘文件
	buf2.Flush()
	fmt.Println(n)
}
```

## 源码解析

### 1. bufio.NewReader

用于生成一个读缓冲区。

方法的具体定义为：

```go
func NewReader(rd io.Reader) *Reader {
	return NewReaderSize(rd, defaultBufSize)
}
```

传入的参数`rd`需要实现`io.Reader`接口，通过`os.Open`或者`os.OpenFile`返回的`*File`对象就实现了这个接口。

`*Reader`结构体中包含`buf`字段，这是一个字节切片，用做缓冲区来存放数据。

`defaultBufSize`用于定义`buf`的长度和容量，默认4096个字节。

### 2. bufio.Read

方法的声明为：

```go
func (b *Reader) Read(p []byte) (n int, err error) 
```

- 当p的长度为0时，不读取数据。

- 当缓冲区为空，且p的长度大于等于缓冲区长度时，不经过缓冲区，直接从文件中读取数据。

- 当缓冲区为空，且p的长度小于缓冲区长度时，先将数据读入缓冲区，再从缓冲区读取需要的字节。

- 如果缓冲不为空，直接从缓冲区读取，当p读满了，或者缓冲区读空了，就返回结果。

### 3. bufio.NewWriter

```go
func NewWriter(w io.Writer) *Writer {
	return NewWriterSize(w, defaultBufSize)
}
```

用于生成一个写缓冲区。逻辑类似于`bufio.NewReader`，返回一个`*Writer`结构体地址。

`Writer`中含有`buf`字段，用做缓冲区来存放数据，默认长度为4096个字节。

### 4. bufio.Write

```go
func (b *Writer) Write(p []byte) (nn int, err error)
```

- 如果写入的数据大于缓冲区可用长度，当缓冲区内无数据时，直接写入文件。

- 如果写入数据大于缓冲区可用长度，且缓冲区有数据，则先将数据写入缓冲区，再将缓冲区写入文件并清空缓冲区。不断重复该过程，直到剩余数据不大于缓冲区可用长度，然后仅将数据写入缓冲区，结束。

- 如果写入数据小于等于缓冲区可用长度，则仅仅写入缓冲区。
