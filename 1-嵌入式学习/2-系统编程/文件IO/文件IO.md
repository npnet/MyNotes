## 文件IO

### 1.第一天

![系统IO和标准IO-1.jpg](https://i.loli.net/2021/07/28/2SkjziJtVYHXBfx.jpg)

数据量大：标准IO
管道、套接字等：系统IO

![系统IO和标准IO-2.jpg](https://i.loli.net/2021/07/28/8Xf7ZzD4W3x6uO2.jpg)

#### 系统IO

【即可操作文本文件，又可操作二进制文件】

打开：open()
读：read()
写：write()
关闭：close()



#### 标准IO

打开：
fopen()

读：
fread() 按块读 【即可操作文本文件，又可操作二进制文件】
【以下接口只可操作文本文件】
fgets() 按行读
getchar() 从键盘按字符读
scanf() 从键盘按格式读
fgetc() 按字符读
fscanf() 按格式读

写：
fwrite() 按块写【即可操作文本文件，又可操作二进制文件】
【以下接口只可操作文本文件】
fputs() 按行写
putchar() 从键盘按字符写
printf() 从键盘按格式写
fputc() 按字符写
fprintf() 按格式写

关闭：
fclose()