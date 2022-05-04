## MYSQL 初始密码修改

### 一、Linux MYSQL-5.9 初始密码修改

#### 1.配置mysql启动时不对密码进行验证

```shell
# my.cnf路径
# /etc/mysql/my.cnf
# 添加如下字段
[mysqld]
skip-grant-tables=1
```



#### 2. 重启mysql服务

```shell
systemctl restart mysqld
```

或者

```shell
/etc/init.d/mysql stop
/etc/init.d/mysql start
```



#### 3. 使用root用户登录到mysql

```shell
mysql -u root
```



#### 4. 切换到mysql数据库，更新user表

```shell
use mysql;
update user set authentication_string = password('123456'), password_expired = 'N', password_last_changed = now() where user = 'root';
```



#### 5.推出mysql，编辑my.cnf文件

```shell
quit;
sudo vi /etc/mysql/my.cnf
# 删除 skip-grant-tables=1 字段
```



#### 6.重启服务，重新登录密码



### 二、Window mysql-8.0 初始密码修改

#### 1. 以**管理员身份**运行cmd，关闭mysql服务

```shell
net stop mysql
```



#### 2.跳过密码输入授权

```shell
mysqld --console --skip-grant-tables --shared-memory 
```



#### 3.重新打开另一个cmd，重置密码

```shell
mysql #直接进入不用输入密码
use mysql; #进入mysql数据库
select use,authentication_string from user; #查看数据
update user set authentication_string='' where user='root'; #设置无密码
flush privileges; #保存更新

alter user 'root'@'localhost' identified with mysql_native_password by "123456"
flush privileges; #保存更新
```



#### 4.重启mysql服务，连接登录

```shell
net start mysql
mysql -u root -p
```

