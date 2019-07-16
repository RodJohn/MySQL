

# 安装


下载yum源

    1现在centos上默认是没有yum源的，yum安装的是 MariaDB。
    所以我们需要自己先配置yum源
    wget 'https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm'

安装yum源

    rpm -Uvh mysql57-community-release-el7-11.noarch.rpm

查看mysql:

    yum repolist all | grep mysql

安装

    yum install -y mysql-community-server

# 控制

centos7

    systemctl start mysqld
    systemctl status mysqld
    
    而在centos6中，使用service mysqld start


# 初始化

查看数据库密码

    mysql5.7在初始化的时候会生成一个自定义的密码，
    grep 'temporary password' /var/log/mysqld.log

登录

    mysql -uroot -p

修改数据库密码

    SET PASSWORD = PASSWORD('Admin123!')




