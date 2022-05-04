## Window下建立ip地址连接
2. 1. Window下建立ip，port连接

   ```shell
   curl 127.0.0.1:8888
   ```

   ```shell
   telnet 127.0.0.1 8888
   ```

   

2. Window下查看当前ip，port信息

   ```shell
   netstat -aon|findstr "8888"
   ```

3. Window下终止进程

   ```shell
   taskkill -F -PID 25088
   ```

   