PostgreSQL常用命令
===

## 登陆数据库
### 1 数据库用户登陆
```
/* 切换到数据库用户 */
su - postgres

/* 登录 */
psql
```
### 2 远程登陆
```
psql -h 192.168.200.163 -p 5432 -U <用户名> -d <数据库名>
```

## 数据库操作

### 列举数据库
```
\l
```

### 切换数据库
```
\c database_name
```

## 用户管理

### 创建角色
```
CREATE ROLE rolename;
```

### 创建用户
```
CREATE USER username WITH PASSWORD '********';
```

### 显示所有用户
```
\du
```

### 修改用户权限
```
ALTER ROLE ussername WITH privileges;
```

### 赋给用户表的所有权限
```
GRANT ALL ON tablename TO user;
```

### 赋给用户数据库的所有权限
```
GRANT ALL PRIVILEGES ON DATABASE database_name TO user;
```

### 撤销用户权限
```
REVOKE privileges ON table_name FROM user;
```

## 数据库操作

### 创建数据库
```
create database dbname;
```

### 删除数据库
```
drop database dbname;
```

## 表操作

### 增加让主键自增的权限
```
grant all on sequence tablename_keyname_seq to webuser;
```

### 重命名一个表
```
alter table table_name rename to new_table_name;
```

### 删除一个表
```
drop table table_name;
```

### 在已有的表里添加字段
```
ALTER TABLE table_name ADD COLUMN column_name column_type;
```

### 删除表中的字段
```
ALTER TABLE table_name DROP COLUMN column_name;
```

### 重命名一个字段
```
ALTER TABLE table_name RENAME COLUMN column_name TO new_column_name;
```

### 给一个字段设置缺省值
```
```

### 去除缺省值
```
```

### 插入数据
```
```

### 修改数据
```
```

### 删除数据
```
```

### 删除表
```

```

### 查询
```
```

### 创建表
```
```

## 退出

```
\q
quit
```