## 下载mysql安装包

```sh

wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz

```


## 解压压缩包

```sh

tar -xvf mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz

# 移动文件夹
mv mysql-5.7.27-linux-glibc2.12-x86_64 /usr/local/mysql

```


## mysql用户组和用户操作

```sh

# 创建mysql用户组和用户并修改权限
groupadd mysql
useradd -r -g mysql mysql

```


## 创建数据目录并赋予权限

```sh

# 创建目录
mkdir -p  /data/mysql

# 赋予权限            
chown mysql:mysql -R /data/mysql  

```


## 配置my.cnf

```sh

vim /etc/my.cnf

# 把my.cnf的内容删除了，将下面的内容放到里面

[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
max_connections=10000
max_user_connections=2000
wait_timeout=200
#character config
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
lower_case_table_names=1


# lower_case_table_names=1 是指忽略大小写，max_connections最大连接线程数
```


## 初始化数据库

```sh

# 进入mysql的bin目录

cd /usr/local/mysql/bin/

./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize

```


## 查看密码

```sh

cat /data/mysql/mysql.err

```


## 启动mysql，更改root 密码

```sh

# 先将mysql.server放置到/etc/init.d/mysql中

cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

# 启动
service mysql start

# 停止
service mysql stop

# 重启
service mysql restart

```


## 启动成功，查看进程

```sh

ps -ef|grep mysql

```


## 修改密码

```sh

cd /usr/local/mysql/bin/
./mysql -u root -p

# 输入刚才获取的密码  成功进入后，开始修改密码

SET PASSWORD = PASSWORD('123456');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
FLUSH PRIVILEGES;

# 退出mysql使用新密码登录试试

```


## 开启远程登录

```sh

# 一般操作数据库都是在本地的系统数据库操作工具来操作的，但是mysql初次安装后需要设置允许远程登录操作
# 在linux系统中登录mysql
# 逐条输入以下命令

# 登录mysql
./mysql -u root -p

# 输入新密码
123456

# 访问mysql库
use mysql

# 使root能再任何host访问                                           
update user set host = '%' where user = 'root';   

# 刷新   
FLUSH PRIVILEGES;

# 退出   
exit                                    

# 使用操作工具试试看，应该可以正常连接了，如果还不能连接，请确保linux的防火墙是否关闭，3306端口是否开启

```


## 环境变量配置

```sh

## 要将 MySQL的可执行文件路径添加到linux服务器的环境变量中，可以按如下步骤进行

# a. 在linux服务器上登录到root管理员账户。

# b. 打开/etc/profile文件。这是系统级别的bash环境变量设置文件。

# c. 在该文件末尾添加如下语句：


export PATH=$PATH:/usr/local/mysql/bin

# d. 保存并关闭编辑器。

# e. 运行以下命令使环境变量设置生效：

source /etc/profile

# 现在，您可以在终端中运行mysql命令，而不必指定完整的可执行文件路径。如果MySQL安装在非标准目录下，请将上述路径更改为相应的目录路径。

```


## 安装mysql相关包出错


```sh

# 安装mysql相关的python包时，都有可能出现以下错误：   EnvironmentError: mysql_config not found

# For MacOS 最快的临时解决方法：

yum install python-devel mysql-devel

# For CentOS:  

sudo apt-get install python-dev
sudo apt-get install python-MySQLdb

# For Ubuntu:

sudo apt-get install python-dev
sudo apt-get install python-MySQLdb

```