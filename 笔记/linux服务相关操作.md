##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## scp上传本地文件到目标服务器

```sh

# 执行命令将本地文件上传到目标Linux服务器，输入指令：

scp localfile user@server:/destination

localfile为本地文件名，user为连接Linux服务器的用户名，server为Linux服务器的IP地址或域名，destination是文件在Linux服务器中的目标路径。

# 范例  scp /r2/z_home/media_backup/bin/mediabackup.exe root@192.168.49.210:/r2/z_home/upload_model

```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## curl 测试用例

```sh

curl -X POST "http://localhost:18558/media/search" -H  "accept: application/json" -H  "Content-Type: application/json" -d '{"pageidx":1,"pagenum":20,"origsourceid":2000,"contenttype":0,"name":"恐龙","mediaid":0,"target":1,"accesstoken":"TOKEN3590"}'


curl -X GET "http://localhost:18558/media/series/get_list?pageidx=1&pagenum=20&origsourceid=2000&contenttype=0&sortby=2&asc=1&accesstoken=TOKEN3590" -H  "accept: application/json"

```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## 打包服务器的环境

```sh

# 输出本地包环境至文件
./pip freeze | tee requirements.txt

# 根据文件进行包安装
./pip install -r requirements.txt

# linux中下载Python模块
pip install pyyaml

# 下载速度慢的话加上清华镜像源
pip install pyyaml -i https://pypi.tuna.tsinghua.edu.cn/simple

```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## 查看linux服务器内存大小相关指令

```sh

在Linux中，可以使用以下命令来查看一个文件夹下占用的空间大小：

du -sh 文件夹路径
du -sh ./ 读取当前文件夹下的路径大小
df -h  读取整个磁盘所占内存空间

```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## 服务器账号密码

```sh

ai服务器        192.168.35.245      zhougx      ai@ipanel.cn
反向机          192.168.36.100      root        Hds654321
开发集群        192.168.49.210      root        iPanel2.0dev@2018
堡垒机          192.168.35.3        zhougx      201114Hyc@
服务器          192.168.101.96      root        ipanel123
运营集群        192.168.53.13       vedio       Vedio_iPanel@2020
阿里云服务      114.55.130.88       root        iPanel2022!@#456


# 运营集群服务器root权限
su root
Vedio_Root@987^%$

# ai集群服务器root权限
su root
root.ai@ipanel.cn

# github账号密码
2278857688@qq.com
201114hyc

# 数据库相关
ip：192.168.101.9
user: root
passwd: 123456
port: 3306

cat /homed/config_comm.xml | grep iactivity
这样搜 他们配置的是那个 数据库


```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## 文档相关
```sh

# 重启增值应用活动
/homed/homed restart iactivity

# 开发集群
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_0_0_dev

# Homed API文档
http://openapi.homed.ipanel.cn/homed_docs/Activity.backend

# OpenApi文档
http://svn1.ipanel.cn:18080/svn/homed/docs/openapi

# python work
http://svn1.ipanel.cn:18080/svn/homed/pythonwork/

# 注册服务中心，需要切换dns进入
http://configmanager.homedfm.me/#/resource/version

# 部署版本
http://svn1.ipanel.cn:18080/svn/homed_2_0/app_release

# 分享ppt
https://svn.eis/svn/AI/main/doc/share

# 增值应用
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_0_2/iactivity
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_6_0/iactivity
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_10_guangxin/iactivity
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_11_0/iactivity
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_10_jilin/iactivity
http://svn1.ipanel.cn:18080/svn/homed_2_0/code_v_2_10_0/iactivity

```


##  ---------------------------------------------------------------------------------------------------------------------------------------------------------
## Linux服务器常规指令

```sh

ps -aux | grep iactivity            # 查找指定文件的进程
ps -ef | grep iactivity

netstat -tunlp | grep 18990         # 查找指定端口的进程
netstat -antp | grep 18990

cd ../../                           # 逐层退回上一级目录

kill -9 port                        # 查看端口占用并杀死对应进程

cat ./1.xml | grep a -C 20          # 查找固定字符上下20行的信息
cat -n file1                        # 标示出文件的行数
more file1                          # 查看一个长文件的内容

history                             # 查看历史命令

head -n 2 file1                     # 查看一个文件的前两行
tail -n 2 file1                     # 查看一个文件的最后两行

find / -name file1                  # 从'/'开始进入根文件系统搜索文件和目录

ls -lh                              # 显示权限

grep Aug /var/log/messages          # 在文件'/var/log/messages'中查找关键词Aug

paste file1 file2                   # 合并两个文件或两栏的内容
paste -d '+' file1 file2            # 合并两个文件或两栏的内容，中间用"+"区分

# sort命令

sort file1 file2                    # 排序两个文件的内容
sort file1 file2 | uniq             # 取出两个文件的并集（重复的行只保留一份）
sort file1 file2 | uniq -u          # 删除交集，留下其它的行
sort file1 file2 | uniq -d          # 取出两个文件的交集

# comm命令

comm -1 file1 file2                 # 比较两个文件的内容，只删除'file1'所包含的内容
comm -2 file1 file2                 # 比较两个文件的内容，只删除'file2'所包含的内容

ps aux                              # 查看系统所有的进程数据

wget url                            # 下载链接内容                                

```
