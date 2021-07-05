### 连接Jack server发生SSL错误

Android 7.0源码编译jack-server无法起来编译失败

一、报错内容：

```shell
/bin/bash -c "(prebuilts/sdk/tools/jack-admin install-server prebuilts/sdk/tools/jack-launcher.jar prebuilts/sdk/tools/jack-server-4.11.ALPHA.jar  2>&1 || (exit 0) ) && (JACK_SERVER_VM_ARGUMENTS=\"-Dfile.encoding=UTF-8 -XX:+TieredCompilation\" prebuilts/sdk/tools/jack-admin start-server 2>&1 || exit 0 ) && (prebuilts/sdk/tools/jack-admin update server prebuilts/sdk/tools/jack-server-4.11.ALPHA.jar 4.11.ALPHA 2>&1 || exit 0 ) && (prebuilts/sdk/tools/jack-admin update jack prebuilts/sdk/tools/jacks/jack-4.32.CANDIDATE.jar 4.32.CANDIDATE || exit 47 )"
Jack server already installed in "/home/system1/.jack-server"
Communication error with Jack server (35), try 'jack-diagnose' or see Jack server log
SSL error when connecting to the Jack server. Try 'jack-diagnose'
SSL error when connecting to the Jack server. Try 'jack-diagnose'
```

修改方案如下：

```shell
cd  /etc/java-8-openjdk/security/
```

到该目录下；

```shell
sudo vim java.security
```

(注意需要用root用户去打开并修改)；

remove "TLSv1, TLSv1.1"这两个配置；

保存后重启电脑；

在服务器上修改，经测试修改完配置文件后可不重启服务器即可生效。



二、报错原因

jack不支持多用户同时编译，所以经常出现jack server报错的现象

解决方法：

编辑 $home/.jack，$home/.jack-settings 和 $home/.jack-server/config.properties,修改SERVER_PORT_SERVICE和SERVER_PORT_ADMIN的端口号，然后保存。

如果权限不对可以执行下面的命令修改权限：

```shell
chmod 600 .jack
chmod 600 .jack-settings
chmod 700 .jack-server
```

查看jack server是否启动

```shell
cd prebuilts/sdk/tools ./jack-admin start-server/stop-server
jack-admin server-log
```

在Android 7.0的工程中进行编译之前，运行如下命令，进行交互
 bule@sky:~/workspace/jianwen.fu/jianwen.fu/V65_An7/prebuilts/sdk/tools$
 jack-admin start-server
 jack-admin kill-server
 jack-admin list-server
 jack-admin uninstall-server
 mm -j32 showcommands &> mm.out
 jack-admin install-server jack-launcher.jar  jack-server-4.8.ALPHA.jar
 jack-admin dump-report
 jack-admin dump-re



三、问题：

```shell
No Jack server running. Try 'jack-admin start-server'
```

解决方案：
通过查看文件 $HOME/.jack-server/logs/jack-server-0-0.log：

com.android.jack.server.api.v01.ServerException: './config.properties' musthave permission rw------- but have rwx------

Caused by: java.io.IOException: './config.properties' must have permissionrw------- but have rwx------

... 2 more

发现是配置文件的权限不对造成的，把文件$HOME/.jack-server/config.properties的权限由rwx改为rw即可解决问题。




