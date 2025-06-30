 在生产环境或长期运行的老旧机器上**直接升级 Python 版本是一项非常危险的操作**。  
不仅可能导致依赖不兼容、老旧项目无法运行，还可能破坏系统自带工具（例如 `yum`、`apt`、`ansible` 等都依赖系统 Python）。  

[https://www.cnblogs.com/MrHSR/p/16469035.html](https://www.cnblogs.com/MrHSR/p/16469035.html)



```bash
 # 下载mysql源
 wget https://dev.mysql.com/get/mysql80-community-release-el8-1.noarch.rpm
 # 安装mysql数据源
 yum install mysql80-community-release-el8-1.noarch.rpm
 # 检测mysql源是否安装成功
 yum repolist enabled | grep "mysql.*-community.*"
 # 禁用centos自带的mysql模块
 yum module disable mysql
 # 安装mysql
 yum install mysql-community-server
 service mysqld status
 service mysqld start
 # 查找随机密码
 grep 'temporary password' /var/log/mysqld.log
 mysql -uroot -p

# 修改默认密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root_21root'
# 查看密码策略
SHOW VARIABLES LIKE 'validate_password%';
# 修改密码长度
set global validate_password.length=1;
# 修改密码等级
set global validate_password.policy=0;
# 设置自己想要的密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'xxxxxxxx'




 create user 'root'@'%' identified by 'xxxxxxx';
grant all privileges on *.* to 'root'@'%' with grant option;
FLUSH PRIVILEGES;
```

