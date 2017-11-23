# Server for CentOS 7.2 64位

### 安装 [git](https://github.com/git/git) 管理项目版本
```
sudo yum update
sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

cd /home
wget https://github.com/git/git/archive/v2.9.2.zip
yum install -y unzip zip
unzip v2.9.2.zip

cd git-2.9.2
make prefix=/usr/local/git all
sudo make prefix=/usr/local/git install

git --version
//如果这个时候 git 的版本还是2.9.2 继续执行下面的

whereis git
//正常来说会输出 git: /usr/bin/git /usr/local/git /usr/share/man/man1/git.1.gz

sudo vim /etc/profile
//然后在文件的最后一行，添加下面的内容，然后保存退出。 按i进行编辑，按esc退出编辑，按shift+;再输入wq保存并退出
//export PATH=/usr/local/git/bin:$PATH

source /etc/profile
//刷新配置

git --version
//这个时候就应该是 2.9.2 了

```


### 安装 g++
```
yum list gcc-c++
yum install gcc-c++.x86_64
```

### 将gcc 升级到支持C++11的模式
```
sudo yum install centos-release-scl 
sudo yum-config-manager --enable rhel-server-rhscl-7-rpms
sudo yum install devtoolset-3-gcc-c++
scl enable devtoolset-3 bash
echo "source /opt/rh/devtoolset-3/enable" >>/etc/profile
```

### 安装 [n](https://github.com/tj/n) 管理node.js 的版本
```
export N_PREFIX=/usr/local/n
curl -L https://git.io/n-install | bash

. ~/.bashrc
```
#### 使用n来安装最新的node.js
```
n latest
node -v
//检查安装后的node 的版本
npm -v 
//检查安装后的npm 的版本
```

### 安装 [redies](https://redis.io) 管理缓存
[教程](http://www.cnblogs.com/_popc/p/3684835.html)
先在[redis的官网](https://redis.io/download)查看最新的redis版本，以下以3.2.8版为例子
```
mkdir /usr/local/redis
cd /usr/local/src 
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar xzf redis-3.2.8.tar.gz
cd redis-3.2.8 redis
cd redis
make PREFIX=/usr/local/redis install
cd /usr/local/redis/bin
ls
//看到这些文件代表安装成功 redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-server
```
#### 将redis做成一个服务
```
cp /usr/local/src/redis/utils/redis_init_script /etc/rc.d/init.d/redis
vim /etc/rc.d/init.d/redis
```
##### 修改里面的内容
* 第2行改成 ```#chkconfig: 2345 80 90 ```
* 第7行改成 ```EXEC=/usr/local/redis/bin/redis-server ```
* 第8行改成 ```CLIEXEC=/usr/local/redis/bin/redis-cli ```
* 第20行改成 ```$EXEC $CONF & ```

##### 最后的内容大致为
```
#!/bin/sh
#chkconfig: 2345 80 90
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF &
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```
将redis配置文件拷贝到/etc/redis/${REDISPORT}.conf
```
mkdir /etc/redis
cp /usr/local/src/redis/redis.conf /etc/redis/6379.conf
chkconfig --add redis
service redis start 
```
将Redis的命令所在目录添加到系统参数PATH中 
```
vi /etc/profile
//在最后行追加: export PATH="$PATH:/usr/local/redis/bin"
. /etc/profile 
//之后就可以直接运行 redis-cli 来查看redis 的缓存情况了
```

### 安装 [Nginx](http://nginx.org/) 高性能的 HTTP 和 反向代理 服务器
```
yum install nginx
```
##### 启动
```
service nginx restart
```
##### 重启
```
sudo service nginx reload
```
##### 在浏览器输入服务器的ip可以访问到nginx的页面就是成功了
##### 修改配置
```
vi /etc/nginx/nginx.conf
```

### 安装 [GraphicsMagick](http://www.graphicsmagick.org/INSTALL-unix.html) 用来做图片处理
```
cd /usr/local/src 
wget https://downloads.sourceforge.net/project/graphicsmagick/graphicsmagick/1.3.25/GraphicsMagick-1.3.25.tar.gz
tar xzf GraphicsMagick-1.3.25.tar.gz
cd GraphicsMagick-1.3.25.tar.gz
./configure
make
make install
gm -version
```

### 安装 [ImageMagick](https://www.imagemagick.org) 用来做图片处理
```
yum install ImageMagick
```

### 安装 [PM2](https://github.com/Unitech/pm2) 进行node.js的项目管理
```
npm install -g pm2
```
##### 进行需要发布的项目
```
NODE_ENV=production pm2 start bin/www -i 0 --name "projectName"
```
##### 进行需要发布测试的项目
```
NODE_ENV=test pm2 start bin/www -i 0 --name "projectName"
```
##### 日志分割
```
pm2 install pm2-logrotate
```


### 常用Linux 命令
```
find / -type f -name "*.log" | xangs grep "ERROR" //从根目录开始查找所有扩展名为.log的文本文件，并找出包含”ERROR”的行
rm -rf test //删除test 文件夹及内部的所有文件

//Copy 本地文件 /etc/eva.log, 到远程机器 sysB, 用户
//user 的家目录下 
scp /etc/eva.log user@sysB:/home/user
//copy 远程机器 sysB 上的文件 /home/uesr/eva.log, 到本地的 /etc 目录下 , 并保持文件属性不变
scp -p user@sysB:/home/uesr/eva.log /etc
//copy sysB 上的目录 /home/user， 到本地 /home/user/tmp, <new dir,/home/user/tmp/user>
scp -r user@sysB:/home/user /home/user/tmp

更改文件归宿者
chown 用户名 文件名
chown -R 用户名 文件名    全部文件

更改文件权限组
chgrp 组名 文件名

添加用户
adduser www

查看进程
ps aux | grep 进程名

修改权限
chmod 777 文件名

修改ssh的链接时长
修改server端的etc/ssh/sshd_config
ClientAliveInterval 60 ＃server每隔60秒发送一次请求给client，然后client响应，从而保持连接
ClientAliveCountMax 300 ＃server发出请求后，客户端没有响应得次数达到300，就自动断开连接
然后保存后执行
service sshd reload
再
vim .bash_profile
添加
export TMOUT=1000000
到最后
```


