## Linux MYSQL-5.9 初始密码修改

### 1.配置mysql启动时不对密码进行验证

```shell
# my.cnf路径
# /etc/mysql/my.cnf
# 添加如下字段
[mysqld]
skip-grant-tables=1
```



### 2. 重启mysql服务

```shell
systemctl restart mysqld
```

或者

```shell
/etc/init.d/mysql stop
/etc/init.d/mysql start
```



### 3. 使用root用户登录到mysql

```shell
mysql -u root
```



### 4. 切换到mysql数据库，更新user表

```shell
use mysql;
update user set authentication_string = password('123456'), password_expired = 'N', password_last_changed = now() where user = 'root';
```



### 5.推出mysql，编辑my.cnf文件

```shell
quit;
sudo vi /etc/mysql/my.cnf
# 删除 skip-grant-tables=1 字段
```



### 6.重启服务，重新登录密码