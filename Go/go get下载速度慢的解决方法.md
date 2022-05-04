## go  get下载速度慢的解决方法

### 1.建议使用go version在1.13以上



### 2.打开终端并执行

```shell
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```



### 设置环境变量

Window

打开powershell，输入

```shell
$env:GOPROXY = "https://goproxy.cn"
```



Linux

```shell
echo "export GOPROXY=https://goproxy.cn" >> ~/.profile && source ~/.profile
```

