## Linux 分卷压缩解压命令

### 1.使用tar分卷压缩

**格式**：

```shell
tar zcvf - filedir | split -d -b 50m - filename
```

**示例**：

```shell
tar zcvf - ./picture | split -d -b 10m - picture
```

将 ./picture 打包，并切割为10m的包

**注意**：

* 输出文件名为 filename00，filename01，filename02 ......
* 假设不加filename，则输出文件为 x00、x01、x02 ...
* 假设不加參数 -d。则输出aa、ab、ac ...



### 2.解压分卷

首先将分卷包合并

```shell
cat x* > myzip.tar.gz
```

然后再解压

```shell
tar zxvf myzip.tar.gz
```

**示例**：

```shell
cat picture* > picture.tar.gz
tar zxvf picture.tar.gz
```

