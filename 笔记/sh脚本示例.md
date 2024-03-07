## 脚本1

```sh
#!/bin/sh
echo 启动程序
#    查找进程         正则筛选             反转查找     统计个数
count=`ps -ef | grep video_upload.py | grep -v "grep" | wc -l`

# if语句中的 -gt 表示大于的意思
# shell 脚本中进行比较时，if语句根据[]表达式执行的退出状态码进行判断，0表示真，非0表示假，但不能作为if[]的判断条件来用

if [ $count -gt 0 ]; then
    echo "video_upload already runing....."
    exit
else
   echo "start video_upload.py....."
fi

cd /r2/videoUpload
# nohup 执行程序 &
nohup python3 video_upload.py >/r2/ReleaseMediaShow_lite/stdout.log 2>&1 &
echo "finish start video_upload.py server"
echo 程序已启动
```


## 脚本2

```sh
#!/bin/sh

# awk 逐行读取输入文本   $n 表示当前处理行的第n个字段

kill -9 `ps -ef | grep video_upload.py | grep -v "grep" | awk ' { print ( $2 ) } '`
echo 程序已终止...
```

## 注释

```sh
将目录赋值给AIROOT 使用$()执行命名，即获取当前目录的上两层的目录，并进行赋值 
AIROOT=$(dirname $(dirname $(pwd)))
```


## 脚本3

```sh
#!/bin/sh

echo "enter start video upload..."
count=`ps -ef | grep video_upload.exe | grep -v "grep" | wc -l`

if [ $count -gt 0 ]; then
    echo "video_upload already running..."
    exit
else
    echo "start video_upload.exe..."
fi

cd /r2/videoUpload

# 判断是否为常规文件
if [ -f ./video_upload.exe ]; then
    rm -rf ./video_upload.exe
fi

cp ./bin/video_upload.exe ./ && chmod +x video_upload.exe

nohup ./video_upload.exe >/r2/ReleaseMediaShow_lite/stdout.log 2>&1 &
```


## 注释

```sh
echo ${BASH_SOURCE[0]}
echo ${BASH_SOURCE}
echo $( dirname ${BASH_SOURCE[0]})
DIR=$( cd $(dirname ${BASH_SOURCE[0]}) && pwd)
echo $DIR
```


## 脚本4

```sh
#!/bin/sh

start_time=$(date "+%Y-%m-%d %H:%M:%S")
echo "start at $start_time"

# python3.x 环境
myPip=/usr/local/python3/python3812/bin/pip3.8
myPyInstaller=/usr/local/python3/python3812/bin/pyinstaller

# $()是作为脚本中执行函数命令的，$与括号之间不能有空格
# 得到shell脚本文件所在完整路径(绝对路径)及文件名，且不改变shell的当前目录
buildpath=$(cd $(dirname ${BASH_SOURCE[0]} ) && pwd)
echo "[Python build] buildpath=$buildpath"

# 代码根目录
CODE_ROOT="$( cd -P "$( dirname "$buildpath" )" && pwd)"
echo "[Pyhton build] CODE_ROOT=$CODE_ROOT"

# 工程生成名称
exeFileName="video_upload.exe"

# 目标代码文件
codeMain="$CODE_ROOT/video_upload.py"
# 打包前先clean
exepath="$CODE_ROOT/bin/$exeFileName"

echo "[Python build] clean start"
cleancmd="rm -rf "$buildpath"/build "$buildpath"/*.spec"

if [ -f "$exepath" ]; then
    cleancmd="$cleancmd $exepath"
fi
echo "exec cmd:"$cleancmd
$cleancmd
echo "[Python build] clean end"

# 自动下载依赖包
echo "[Python build] check requirements.txt"
filename="$buildpath""/requirements.txt"
if [ ! -f "$filename" ]; then
    echo "ERROR! no requests.txt"
    echo -e "需要先执行命令: \033[41;37mpip freeze > requirements.txt\033[0m"
    echo "由于每个程序依赖的三方包不一样, 编译机上不一定有所需的依赖包, 所以新增依赖文件 './requirements.txt'. \
开发人员测试程序正常后, 执行命令: 'pip freeze > requirements.txt' 将程序所需的依赖包以及版本信息全部导出到文件\
 'requirements.txt' 中, 并将 'requirements.txt' 上传到和 'make.sh' 同一路径, 'make.sh' 会执行 pip 下载文件中的所有包, \
否则打包出来的exe会因为缺少依赖而运行异常。"
    # exit 1 就是脚本执行结束后返回一个错误的值给操作系统，表示检查出失败的信息。exit 0 表示成功，也可以通过echo $? 查看返回的结果
    exit 1
else
    # --use-deprecated=legacy-resolver 因为有版本冲突，如果没有加入这个参数会进入死循环找版本
    pip_cmd="$myPip install -r $filename --use-deprecated=legacy-resolver --user"
    echo "[Python build] exec cmd: $pip_cmd"
    $pip_cmd
fi

# -F： 告诉pyinstaller将打包结果放在一个exe文件中， -w：隐藏执行窗口， --clean：删除缓存和临时文件，--dispath：打包到哪个目录下
pyinstallerCmd="$myPyInstaller -F -w --clean  --dispath=$CODE_ROOT/bin --name=$exeFileName $codeMain"
echo "[Python build] pyinstaller start, cmd:$pyinstallerCmd"
$pyinstallerCmd

echo "[Python build] pyinstaller end"
```


## 脚本5

```sh
#!/bin/bash

#为了最小化打包，最好起一个docker 容器去打包
#启动容器时，做好目录映射
# src目录
ROOT=$(dirname $(pwd))

# 选做的镜像
IMAGE=image.registry.ai.ipanel.cn:5000/video-upload
NAME=videoupload_builder

chmod +x *

#启动容器并自动启动脚本去执行编译
# --privileged=true：允许开启特权功能， -i：以交互模式运行容器，通常与-t同时使用，-v：绑定一个卷，--name：为容器指定一个名称，-c：使用Dockerfile指令来创建镜像
docker run -i -v $ROOT:/code  --privileged=true --name=$NAME ${IMAGE} /bin/bash -c "source /etc/profile && /code/build/make.sh"
echo $?
# 删除容器
docker rm -f $NAME

#启动一个容器并进入后台
#docker run -itd --name $NAME --privileged=true -v $ROOT:/code $IMAGE /bin/bash

#进入该容器
#docker exec -it $NAME /bin/bash

# 进入容器后，再进到这个目录，执行 bash make.sh
```


## 脚本6

```sh
#!/bin/bash
# clean data

ROOT="/r2/DataFtp"
TMP="$ROOT/tmp"

DAYS=3

echo "clean dirname: $ROOT"
# find ./ -mtime +3 ：将目录下前三天的那一天被更改过的内容的文件列出，-type：按照类型查找文件，后接 f -name ""表示类型
# -exec参数后面跟的是command命令，它的终止是以 ; 为结束标志，考虑到各个系统中分号会有不同的意义，所以在前面加反斜杠
# {} 代表前面find 查找出来的文件名
find $ROOT -mtime +$DAYS -type f -name "*" -exec rm -rc {} \;
find $TMP -mtime +$DAYS -type f -name "*" -exec rm -rc {} \;
echo "clean end."
```


## 脚本7

```sh
#!/bin/bash

# 为了最小化打包，最好起一个docker 容器去打包
# 启动容器时，做好目录映射
# src目录
AIROOT=$(dirname $(dirname $(pwd)))

# 选做的镜像
# 选做的镜像就是docker要启动的容器
IMAGE=image.registry.ai.ipanel.cn:5000/centos:1.1
# 容器名称
CONTAINER_NAME=ReleaseMediaShow_lite
# 映射的磁盘
DATAROOT=/data

'''
启动一个容器并进入后台
-i选项指示docker要在容器上打开一个标准的输入接口，-t 指示docker要创建一个伪tty终端，连接容器的标准输入接口，之后用户就可以在终端进行输入，-d 在后台运行容器
/bin/bash 表示载入容器后运行bash，防止容器终止
CONTAINER_NAME是为容器命名的名称
-v：挂载宿主机的一个目录，冒号 ":" 前面的目录是宿主机目录，后面的目录是容器内目录
容器目录不可以为相对路径，宿主机目录如果不存在，则会自动生成
这样宿主机目录和容器内目录就互通，可以互相传输文件
'''

# 启动一个 $IMAGE 容器，宿主机的 $DATAROOT目录挂载到容器的 /data目录
docker run -itd --name CONTAINER_NAME -V $DATAROOT:/data $IMAGE /bin/bash
# 查看命令执行成功与否，0表示正常
echo $?

# 进入该容器
# docker exec：在运行的容器中执行相关命令
docker exec -it $CONTAINER_NAME /bin/bash
```


## 脚本8

```sh

#!/bin/bash

# 服务使用python版本启动
runPython='/usr/local/python3/python3812/bin/python3.8'
# 服务名称
projectName='z_home'
# 服务程序入口
pythonMain='main.py'
# 日志路径及名称
logFilePath='/homed/iactivity/src/z_home/log/performance_89000.log'
# 服务端口
serverPort=8000

# echo
echo "start to $projectName server..."

'''
查找进程时用到了grep命令，而这个命令执行的时候带有查找的进程参数，同时这也是一个进程，所以需要先使用grep作为关键字将这个进程过滤
ps -ef 显示数据格式；
用户名  进程ID  父进程ID  进程占CPU百分比 该进程启动到现在的时间  该进程在哪个终端上运行(若与终端无关，则显示 ？  若为pts/0等，表示由网络连接主机进程)  命令的名称和参数
'''
processId=`ps -ef | grep -v grep | grep $projectName | awk '{print $2}'`
# processId=`ps -ef | grep $projectName | grep -v "grep" | awk '{print $2}'`

if [ "$processId" != "" ]; then
    echo "Server is running! check server: $processId"
    # 退出脚本，返回成功退出码
    exit 0
fi

# 获取进程
process=`netstat -anpl | grep LISTEN | grep $serverport`
# [-z 内容] 判断空，则为真
# -z 用来判断进程是否存在， 如果不存在，则继续执行if，如果存在，则执行else
if [ -z "$process" ]; then
    # -u：表示不启用缓存，实时输出打印信息到日志文件  2>&1：将标准错误2重定向到标准输出&1，再被重定向输入到 .log 文件中
    nohup $runPython -u $pythonMain -d 89000 $projectName > $logFilePath 2>&1 &
    echo "finish $projectName server..."
else
    # 判断端口是否被占用  -F后接分隔符号，获取其中的一部分
    processPort=`echo $process | awk '{print $4}' | awk -F ':' '{print $2}'`
    # -eq：相等比较
    if [ $processPort -eq $serverPort ]; then
        processId=`echo $process | awk '{print $7}' | awk -F '/' '{print $1}'`
        echo "$serverPort is used by processId:$processId, please check it..."
    exit 0
    fi
fi
```
