# 【Linux】du命令详解


在`Linux`环境中，查看目录或文件的磁盘空间大小是很常见的需求，使用`du`命令即可帮助我们达到该目的。

## 命令格式

```shell
du [选项] [文件或者目录]
```

## 选项

常用的可选参数为：

- `-a`: `--all`，列出所有文件和目录的容量大小（默认只列出目录容量的大小）。

- `-h`: 以已读的形式显示。

- `-B`：`--block-size=SIZE`，指定容量的单位值，size可以是1，k，或者m，如`-B1`。

- `-k`：`--block-size=1k`，以`KB`为单位。

- `-m`：`--block-size=1m`，以`MB`为单位。

- `-c`：`--total`，额外显示总的容量大小。

- `-s`：仅显示总量大小。

- `--max-depth`：显示指定层级的目录。

- `--exclude=<文件或目录>`：忽略指定目录或文件。

## 示例

- 显示当前目录及子目录大小

  ```shell
  du
  ```

- 显示指定文件所占空间

  ```shell
  du filename
  ```

- 仅显示当前目录所占空间大小

  ```shell
  du --max-depth=1
  ```

- 仅显示当前目录总的容量大小

  ```shell
  # 使用 -s
  du -sh
  # 使用--max-depth
  du -h --max-depth=0
  ```

## `du`和`ls -l`

除了可以使用`du`查看目录或文件大小外，还可以使用`ls -l`查看文件或目录的大小，但是二者获得的结果有所差异。

- `ls -l`的结果小于`du`的结果
  
  `du`是文件占用`block`的大小，在`linux`中，一个`block`的大小为`4k`。而`ls -l`是文件的实际大小，所以即使文件只有`1bytes`，`du`获得的结果也会是`4k`。

- `ls -l`的结果大于`du`的结果
  
  当文件出现空洞的时候会出现这种结果，即磁盘空间被占用，但是里面没有数据。