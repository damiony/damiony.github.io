# golang文件相关操作


## 介绍

golang提供了文件相关的各种操作，包括创建、删除、和读写等。

## 文件基本操作方法

### 1. 查看文件信息

```go
func main() {
	// 查看文件信息
	fileInfo, err := os.Stat("./data.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Stat:", fileInfo.Name())
	fmt.Println("Stat:", fileInfo.IsDir())
	fmt.Println("Stat:", fileInfo.Size())
	fmt.Println("Stat:", fileInfo.Mode())
	fmt.Println("Stat:", fileInfo.ModTime())
}
```

### 2. 路径操作

```go
func main() {
	// 路径操作
	filename1 := "./file.txt"
	fmt.Println("IsAbs:", filepath.IsAbs(filename1))

	// 获取绝对路径
	fileabs, err := filepath.Abs(filename1)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(fileabs)

	// 绝对路径的父级目录
	fmt.Println("Join:", path.Join(fileabs, ".."))
}
```

### 3. 创建一级文件夹

```go
func main() {
	// 创建一级文件夹
  err := os.Mkdir("./aa", os.ModePerm)
	if err != nil {
		fmt.Println(err)
	}
}
```

### 4. 创建多级文件夹

```go
func main() {
	// 创建多级文件夹
  err := os.MkdirAll("./bb/cc", os.ModePerm)
	if err != nil {
		fmt.Println(err)
	}
}
```

### 5. 创建文件

```go
func main() {
	// 创建文件，权限默认为0666，如果文件存在，会清空数据
	file, err := os.Create("./file.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	file.Close()
}
```

### 5. 只读模式打开文件

```go
func main() {
	// 只读模式打开文件
  file, err := os.Open("./file.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	file.Close()
}
```

### 6. 以指定模式打开文件

```go
func main() {
	// 以指定模式打开文件，如果文件不存在，则以指定权限0666创建文件
  file, err := os.OpenFile("./file.txt", os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0666)
	if err != nil {
		fmt.Println(err)
		return
	}
	file.Close()
}
```

### 7. 删除空目录或文件

```go
func main() {
	// 删除空目录
  err := os.Remove("./aa")
	if err != nil {
		fmt.Println("remove:", err)
		return
	}

	// 删除文件
	err = os.Remove("./file.txt")
	if err != nil {
		fmt.Println("remove:", err)
		return
	}
}
```

### 8. 删除指定目录或文件，即使目录非空

```go
func main() {
	// 删除所有目录，包括非空目录
	err := os.RemoveAll("./aa.txt")
	if err != nil {
		fmt.Println("RemoveAll:", err)
		return
	}
	err = os.RemoveAll("./data.txt")
	if err != nil {
		fmt.Println("RemoveAll:", err)
		return
	}
}
```

## 读取文件内容

除了上述的文件基本操作外，golang还为读文件提供了多种方式。

### `file.Read`

```go
func main() {
    file, err := os.Open("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()
 
    b := make([]byte, 128)
    for {
        n, err := file.Read(b)
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("%s", string(b[:n]))
    }
}
```

### `io.ReadFull`

```go
  1 package main
  2
  3 import (
  4     "fmt"
  5     "io"
  6     "log"
  7     "os"
  8 )
  9
 10 func main() {
 11     file, err := os.Open("./data.txt")
 12     if err != nil {
 13         log.Fatal(err)
 14     }
 15		defer file.Close()
 16
 17     // 读取正好1024个字节，内容不够会报错
 18     b := make([]byte, 1024)
 19     n, err := io.ReadFull(file, b)
 20     if err != nil {
 21         log.Fatal(err)
 22     }
 23
 24     fmt.Printf("%s", string(b[:n]))
 25 }
```

### `io.ReadAtLeast`

```go
func main() {
    file, err := os.Open("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    // 至少读取8个字节，不够的话会报错
    b := make([]byte, 1024)
    n, err := io.ReadAtLeast(file, b, 8)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s", string(b[:n]))
}
```

### `bufio.NewReader`

bufio提供了多种读取文件的方法，但方法名都是以`Read`开头。

```go
func main() {
    file, err := os.Open("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    reader := bufio.NewReader(file)
    for {
        str, err := reader.ReadString('\n')
        if err == io.EOF {
            break
        }
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("%s", str)
    }
}
```

### `ioutil.ReadAll`

```go
func main() {
    file, err := os.Open("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    data, err := ioutil.ReadAll(file)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("%s", string(data))
}
```

### `ioutil.ReadFile`

```go
func main() {
    b, err := ioutil.ReadFile("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b))
}
```

### `bufio.NewScanner`

```go
func main() {
    file, err := os.Open("./data.txt")
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
    err = scanner.Err()
    if err != nil {
        log.Fatal(err)
    }
}
```

## 写入文件

golang为写文件提供了以下方式。

### `file.Write`

```go
func main() {
    file, err := os.OpenFile("./data.txt", os.O_WRONLY|os.O_CREATE, 0644)
    if err != nil {
        log.Fatal(err)
    }
    defer file.Close()

    b := []byte("Hello")
    n, err := file.Write(b)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Wrote %d bytes.\n", n)
}
```

### `ioutil.WriteFile`

```go
func main() {
    err := ioutil.WriteFile("./data.txt", []byte("Hello"), 0644)
    if err != nil {
        log.Fatal(err)
    }
}
```

### `bufio.NewWriter`

bufio包中，写入文件的方法较多，但都是以`Write`开头。

```go
 func main() {
     file, err := os.OpenFile("./data.txt", os.O_CREATE|os.O_WRONLY|os.O_TRUNC, 0644)
     if err != nil {
         log.Fatal(err)
     }
     defer file.Close()
 
     writer := bufio.NewWriter(file)
     writer.WriteString("Hello world")
     writer.Flush()
 }
```
