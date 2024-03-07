# ftplib的常用方法


## ftp密码有特殊符号
```sh

# 将特殊符号转化为ascii字符，然后再使用下面语句将密码在登录前转换过来
ftp_passwd = ftp_passwd.encode('utf-8').decode('unicode_escape')  # 解码密码

```


## ftp登录连接

```sh

from ftplib import FTP                                          # 加载ftp模块
ftp=FTP()                                                       # 设置变量
ftp.connect("IP", port)                                         # 连接的ftp sever和端口
ftp.login("user","password")                                    # 连接的用户名，密码

ftp.set_debuglevel(2)                                           # 打开调试级别2，显示详细信息
print ftp.getwelcome()                                          # 打印出欢迎信息
ftp.set_debuglevel(0)                                           # 关闭调试模式


bufsize=1024                                                    # 设置的缓冲区大小
filename="filename.txt"                                         # 需要下载的文件
file_handle=open(filename,"wb").write                           # 以写模式在本地打开文件
ftp.storbinaly("STOR filename.txt",file_handle,bufsize)         # 上传目标文件
ftp.retrbinary("RETR filename.txt",file_handle,bufsize)         # 下载FTP文件

```


## ftp相关命令操作

```sh

ftp.cwd('pathname')                                             # 设置FTP当前操作的路径
ftp.dir()                                                       # 显示目录下所有目录信息
ftp.nlst()                                                      # 获取目录下的文件
ftp.mkd(pathname)                                               # 新建远程目录
ftp.pwd()                                                       # 返回当前所在位置
ftp.rmd(dirname)                                                # 删除远程目录
ftp.delete(filename)                                            # 删除远程文件
ftp.rename(fromname, toname)                                    # 将fromname修改名称为toname。
ftp.quit()                                                      # 退出ftp

```
