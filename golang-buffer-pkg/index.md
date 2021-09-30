# 【Golang】buffer包的使用


## 定义

`Buffer`是一个可变长的字节缓冲区，具备读写的方法。`Buffer`的零值是一个空的缓冲区。

```go
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

## 声明

有四种声明方法。

1. 直接定义一个`Buffer`变量。

```go
var b bytes.Buffer
```

2. 使用`New`返回`Buffer`变量指针。

```go
b := new(bytes.Buffer)
```

3. 使用`[]byte`切片初始化。

```go
b := bytes.NewBuffer([]byte("hello"))
```

4. 使用`string`初始化。

```go
b := bytes.NewBufferString("hello")
```

## 写数据

`Buffer`提供了5种写数据的方法。

1. 将`[]byte`切片写入尾部。

```go
func (b *Buffer) Write(p []byte) (n int, err error)
```

2. 将`string`写入尾部。

```go
func (b *Buffer) WriteString(s string) (n int, err error)
```

3. 将字符写入尾部。

```go
func (b *Buffer) WriteByte(c byte) error
```

4. 将`rune`写入尾部。

```go
func (b *Buffer) WriteRune(r rune) (n int, err error)
```

5. 将接口对象写入尾部。

```go
func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error)
```

## 读数据

`Buffer`提供了7种读数据的方法。

1. 读取`n`个字符。

```go
func (b *Buffer) Next(n int) []byte
```

2. 将数据读入`[]byte`。

```go
func (b *Buffer) Read(p []byte) (n int, err error)
```

3. 读取第一个`byte`。

```go
func (b *Buffer) ReadByte() (byte, error)
```

4. 读取第一个`UTF8`编码字符。

```go
func (b *Buffer) ReadRune() (r rune, size int, err error)
```

5. 读取分隔符前的内容，并且返回切片。

```go
func (b *Buffer) ReadBytes(delim byte) (line []byte, err error)
```

6. 读取分隔符前的内容，并且返回字符串。

```go
func (b *Buffer) ReadString(delim byte) (line string, err error)
```

7. 将内容输出到接口对象。

```go
func (b *Buffer) WriteTo(w io.Writer) (n int64, err error)
```

## 其他方法

1. 返回字节切片。

```go
func (b *Buffer) Bytes() []byte
```

2. 返回字节切片容量。

```go
func (b *Buffer) Cap() int
```

3. 为容量增加`n`个字节。

```go
func (b *Buffer) Grow(n int)
```

4. 返回缓冲区长度。

```go
func (b *Buffer) Len() int
```

5. 清空数据。

```go
func (b *Buffer) Reset()
```

6. 字符串化。

```go
func (b *Buffer) String() string
```

7. 丢弃前`n`个未读字节以外的数据。

```go
func (b *Buffer) Truncate(n int)
```

8. 将最后一次成功读取的字节，设为未读取状态。

```go
func (b *Buffer) UnreadByte() error
```

9. 将最后一次读取的`rune`字符，设为未读取状态。

```go
func (b *Buffer) UnreadRune() error
```

