## Typora设置图床PicGo，以实现图片自动上传

### 1. 配置Github

1. 在GitHub建立公共图床，仓库属性设置为**public**，如果是私人仓库，github会给图片链接加上token，PicGo不知道怎么破解，所以只能设置成公共的仓库。

   <img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614101548975.png" alt="image-20220614101548975" style="zoom:67%;" />



2. 依次执行以下步骤，生成token

   `Settings` --> `Developer Settings`  -->  `Personal access tokens` -->  `Generate new token`

   复制生成的token

   <img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614102521751.png" alt="image-20220614102521751" style="zoom:67%;" />

   <img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614102613768.png" alt="image-20220614102613768" style="zoom:67%;" />



### 2. 配置PicGo

1. 下载安装PicGo  [PicGo](https://github.com/Molunerfinn/PicGo/releases)

2. 按照如下配置PicGo

   自定义域名是：https://raw.githubusercontent.com/**用户名**/**仓库名**/master

   <img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614102815267.png" alt="image-20220614102815267" style="zoom:67%;" />

3. 激活PicGo-Server

   打开PicGo详细页面，进入PicGo设置 —> 设置Server ---> 打开设置



### 3. 配置Typora

文件 —> 偏好设置 —> 图像，再按照下面截图配置：
**其中“PicGo路径”就是上面安装PicGo的路径，选到PicGo.exe为止。**

<img src="https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614103117473.png" alt="image-20220614103117473" style="zoom:67%;" />



### 4. 问题

若出现能上传到github仓库，但在本地图片无法显示的问题

![image-20220614103316518](https://raw.githubusercontent.com/Kownzird/Notes-Img/master/img/image-20220614103316518.png)

则在Window下打开改路径文件：`C:\Windows\System32\drivers\etc\hosts`

在文件末尾添加ip和域名的映射，通过此方法找到图片对应的ip地址

```shell
# Github Start 
140.82.113.3      github.com
140.82.114.20     gist.github.com

151.101.184.133    assets-cdn.github.com
151.101.184.133    raw.githubusercontent.com
199.232.28.133     raw.githubusercontent.com 
151.101.184.133    gist.githubusercontent.com
151.101.184.133    cloud.githubusercontent.com
151.101.184.133    camo.githubusercontent.com
199.232.96.133     avatars.githubusercontent.com
151.101.184.133    avatars0.githubusercontent.com
199.232.68.133     avatars0.githubusercontent.com
199.232.28.133     avatars0.githubusercontent.com 
199.232.28.133     avatars1.githubusercontent.com
151.101.184.133    avatars1.githubusercontent.com
151.101.108.133    avatars1.githubusercontent.com
151.101.184.133    avatars2.githubusercontent.com
199.232.28.133     avatars2.githubusercontent.com
151.101.184.133    avatars3.githubusercontent.com
199.232.68.133     avatars3.githubusercontent.com
151.101.184.133    avatars4.githubusercontent.com
199.232.68.133     avatars4.githubusercontent.com
151.101.184.133    avatars5.githubusercontent.com
199.232.68.133     avatars5.githubusercontent.com
151.101.184.133    avatars6.githubusercontent.com
199.232.68.133     avatars6.githubusercontent.com
151.101.184.133    avatars7.githubusercontent.com
199.232.68.133     avatars7.githubusercontent.com
151.101.184.133    avatars8.githubusercontent.com
199.232.68.133     avatars8.githubusercontent.com
199.232.96.133     avatars9.githubusercontent.com

# GitHub End
```

