# 打包服务器中数据库数据迁移到本地服务中


## 1. 打包对应数据库数据下载到本地服务器中

```sh

<db_conf>
      <hys_import name="hys_import_disk23_cdn" ip="192.168.52.250" port="3306" username="homed" password="iPanel_Meizi@Shandong_201808"/>
      <homed_dtvs name="homed_dtvs" ip="192.168.52.249" port="3306" username="homed" password="iPanel_Meizi@Shandong_201808"/>
      <homed_material name="homed_material" ip="192.168.52.247" port="3306" username="homed" password="iPanel_Meizi@Shandong_201808"/>
</db_conf>

mysqldump -h 192.168.52.250 -u homed -p hys_import_disk23_cdn > /r2/z_homed/hys_import_disk23_cdn.sql

```


## 2. 将文件转移到服务器中

```sh

scp localfile user@server:/destination

# localfile为本地文件名，user为连接Linux服务器的用户名，server为Linux服务器的IP地址或域名，destination是文件在Linux服务器中的目标路径。

scp /r2/z_homed/hys_import_disk23_cdn.sql root@192.168.49.210:/r2/z_home/upload_model

```


## 3. 将sql脚本中数据导入本地数据库

```sh

mysql -h 192.168.24.14 -u username -p database_name < /path/to/backup_file.sql

```


## 4. 导表失败

```sh

[root@disk23 data_sql] mysqldump -h 192.168.52.247 -u homed -p homed_material > /r2/export_server/data_sql/homed_material.sql
Enter password:
mysqldump: Got error: 1045: "Access denied for user 'homed'@'disk23' (using password: NO)" when trying to connect

# 一般为对应用户权限不够，没有导出权限, 换用root权限导表，或者对对应用户添加权限

GRANT ALL PRIVILEGES ON homed_material.* TO 'homed'@'%' WITH GRANT OPTION;

```
