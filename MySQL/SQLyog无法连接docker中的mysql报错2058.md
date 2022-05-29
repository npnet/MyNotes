## SQLyog无法连接docker中的mysql报错2058

**原因：密码加密算法更新导致**

1.查看用户

```mysql
select HOST,USER,PLUGIN from mysql.user;
```



2修改加密规则

```mysql
alter user 'root'@'%' identified by 'root' password expire never;
```



3.将这个root用的密码加密算法修改为"mysql_native_password"，并将密码修改为 “123456”

```mysql
alter user 'root'@'%' identified with mysql_native_password by 'root' ;
```



4.刷新权限

```mysql
flush privileges;
```