
## SVN客户端安装

```sh

sudo apt-get install subversion

```


## 下载代码

```sh

svn checkout SVN_URL LOCAL_PATH

# 其中，<SVN_URL>是代码仓库的URL地址，<LOCAL_PATH>是您希望代码下载到的本地路径。

```


## 更新代码

```sh

svn update LOCAL_PATH

```


## 提交更改

```sh

svn commit -m "commit message" LOCAL_PATH

# 其中，-m参数是用来提供提交的注释信息，<LOCAL_PATH>是要提交更改的本地路径。

```


## 查看代码状态

```sh

svn status LOCAL_PATH

# 这将显示文件的状态（修改，新增，删除等）。

# 注意：在下载和更新代码之前，确保已经具备访问代码仓库的权限和URL地址。

```
