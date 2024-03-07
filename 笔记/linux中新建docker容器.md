# 基于centos 7系统创建Python3环境新镜像


## 1. 执行下载Python3环境包

```sh

wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz

```


## 2. 新建 Dockerfile 文件，填充内容

```sh
FROM centos:7
LABEL maintainer="zyanwei2011@163.com"
LABEL description="centos7 & python3.8"
COPY ./Python-3.8.0.tgz ./Python-3.8.0.tgz

RUN set -ex \
    # 预安装所需组件
    && yum update -y \
    && yum install -y wget tar libffi-devel zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make initscripts \
    && yum clean all \
    # && wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tgz \
    && tar -zxvf Python-3.8.0.tgz \
    && cd Python-3.8.0 \
    # --enable-shared 不加这个的话，编译Python3环境时不会产生 libpython3.8.so.1.0, libpython3.8.so 文件
    && ./configure --prefix=/usr/local/python3 --enable-shared \
    && make \
    && make install \
    && make clean \
    && rm -rf /Python-3.8.0* \
    && yum install -y epel-release \
    && yum install -y python-pip

# 添加 --enable-shared，报错 ImportError: libpython3.7m.so.1.0: cannot open shared object file: No such file or directory，执行
RUN echo "/usr/local/python3/lib/" >> /etc/ld.so.conf
RUN ldconfig

# 设置默认为python3
RUN set -ex \
    # 备份旧版本python
    && mv /usr/bin/python /usr/bin/python27 \
    && mv /usr/bin/pip /usr/bin/pip-python2.7 \
    # 配置默认为python3
    && ln -s /usr/local/python3/bin/python3.8 /usr/bin/python \
    && ln -s /usr/local/python3/bin/pip3 /usr/bin/pip

# 修复因修改python版本导致yum失效问题
RUN set -ex \
    && sed -i "s#/usr/bin/python#/usr/bin/python2.7#" /usr/bin/yum \
    && sed -i "s#/usr/bin/python#/usr/bin/python2.7#" /usr/libexec/urlgrabber-ext-down \
    && yum install -y deltarpm

# 基础环境配置
RUN set -ex \
    # 修改系统时区为东八区
    && rm -rf /etc/localtime \
    && ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && yum install -y vim \
    # 安装定时任务组件
    && yum -y install cronie

# 支持中文
RUN yum install kde-l10n-Chinese -y
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8

# 更新pip版本
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip

# 下载pyinstaller模块
RUN pip install pyinstaller
ENV LC_ALL zh_CN.UTF-8
ENV TZ Asia/Shanghai
ENV PATH /usr/local/python3/bin/:$PATH
RUN ln -sf /usr/share/zoneinfo/Asia/ShangHai /etc/localtime

```


## 3. 执行创建docker新镜像

```sh

# 生成名为 python3-image 的镜像
docker build -t python3-image .

# 进入 python3-image 镜像中
docker run -it python3-image /bin/bash

# python3 相关文件位置
/usr/local/python3/bin/python3
/usr/local/python3/bin/pip3
/usr/local/python3/bin/pyinstaller

```


## 4. docker容器相关指令

```sh

# 查看正在运行的容器
docker ps

# 查看docker镜像列表
docker image ls

# 删除一个docker镜像
docker rmi -f python3-image

```
