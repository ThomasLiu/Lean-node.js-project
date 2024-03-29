# Server for CentOS 7.2 64位

### 安装 [git](https://github.com/git/git) 管理项目版本
```
sudo yum update
sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker

sudo yum install git
```

如果上面的不可以，可以试用通过仓库来安装

```
cd /home
wget https://github.com/git/git/archive/v2.9.2.zip
yum install -y unzip zip
unzip v2.9.2.zip

cd git-2.9.2
make prefix=/usr/local/git all
sudo make prefix=/usr/local/git install

git --version
//如果这个时候 git 的版本还不是2.9.2 继续执行下面的

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

### 生成git用的 SSH密钥

```bash
cd ~/.ssh # 看是否有密钥

ssh-keygen -t rsa -C "abc@mail.com" # 这个邮箱是你的github 账号邮箱

ssh-keygen -t rsa -C "liurongliang@belloai.com" # 会生成 id_rsa.pub

cat id_rsa.pub # 复制到 github 的 https://github.com/settings/keys
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

#### 使用nrm管理npm的源
```
npm install -g nrm
nrm ls
nrm use taobao
```

### 安装 [redies](https://redis.io) 管理缓存
[教程](http://www.cnblogs.com/_popc/p/3684835.html)
先在[redis的官网](https://redis.io/download)查看最新的redis版本，以下以3.2.8版为例子
```
mkdir /usr/local/redis
cd /usr/local/src 
wget http://download.redis.io/releases/redis-3.2.8.tar.gz
tar xzf redis-3.2.8.tar.gz
mv redis-3.2.8 redis
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
```

设置Redis的最大内存
```
vi /etc/redis/6379.conf
//在最后行追加:
# In short... if you have slaves attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for slave
# output buffers (but this is not needed if the policy is 'noeviction').
#
# maxmemory <bytes>
maxmemory 268435456
```
一般推荐Redis设置内存为最大物理内存的四分之三
如果设置了maxmemory，一般都要设置过期策略
```
vi /etc/redis/6379.conf
//在最后行追加:
# LRU是Least Recently Used 近期最少使用算法
# volatile-lru -> remove the key with an expire set using an LRU algorithm 根据LRU算法生成的过期时间来删除。
# allkeys-lru -> remove any key accordingly to the LRU algorithm 根据LRU算法删除任何key
# volatile-random -> remove a random key with an expire set 根据过期设置来随机删除key。
# allkeys-random -> remove a random key, any key 无差别随机删。
# volatile-ttl -> remove the key with the nearest expire time (minor TTL) 根据最近过期时间来删除（辅以TTL
# noeviction -> don't expire at all, just return an error on write operations 谁也不删，直接在写操作时返回错误。
maxmemory-policy volatile-lru
```
启动 redis
```
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

### 配置 [gzip](https://blog.csdn.net/baidu_35407267/article/details/77141871) 来优化你的网站
[相关资料](https://blog.csdn.net/huangbaokang/article/details/79931429)
```
$ vi /etc/nginx/nginx.conf
```
大概配置如下
```
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    gzip  on;   #开启gzip
    gzip_min_length 1k; #低于1kb的资源不压缩
    gzip_comp_level 3; #压缩级别【1-9】，越大压缩率越高，同时消耗cpu资源也越多，建议设置在4左右。
    gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;  #需要压缩哪些响应类型的资源，多个空格隔开。不建议压缩图片，下面会讲为什么。
    gzip_disable "MSIE [1-6]\.";  #配置禁用gzip条件，支持正则。此处表示ie6及以下不启用gzip（因为ie低版本不支持）
    gzip_vary on;  #是否添加“Vary: Accept-Encoding”响应头

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
```
针对单个 配置的话如下 （以下配置还包含，服务器缓存 ）
```
server {
       server_name www.hiredchina.com;
       location /{
            proxy_pass http://127.0.0.1:3103;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       }
       location  ~ .*\.(?i)(woff|ttf|js|css|gif|jpg|jpeg|png|bmp|ico|html)$ {
            proxy_pass http://127.0.0.1:3103;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_cache cache_one;
            proxy_cache_valid 200 7d;
            proxy_cache_valid 301 1d;
            proxy_cache_valid any 1m;
            add_header Cache-by "nginx";
            gzip on;
            gzip_comp_level 9;
            gzip_min_length 1024;
            gzip_types application/json text/plain application/x-javascript text/css text/javascript application/javascript;
            expires 7d;
      }
}
```

### 配置[SSL](https://baike.baidu.com/item/ssl/320778?fr=aladdin) 使用[https](https://baike.baidu.com/item/https)
相关资料 

[八大免费SSL证书-给你的网站免费添加Https安全加密](https://www.cnblogs.com/lxwphp/p/8289337.html)

[https网页加载http资源导致的页面报错及解决方案](http://www.cnblogs.com/yougewe/p/7440008.html)

其中我选用的是 [Let’s Encrypt](https://letsencrypt.org) 的证书，因为安装和使用都非常简单。

首先下载 [letsencrypt](https://github.com/letsencrypt/letsencrypt), 可以在服务器直接用`git clone`来把整个项目下载。
```
$ sudo yum install python2-certbot-nginx
$ cd /home
$ mkdir ssl
$ cd ssl
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./certbot-auto --help # 查看可用的命令
```
因为刚才我们配置好了 nginx，所以这里选择 nginx 的配置
```
$ ./certbot-auto --nginx -d example.com -d www.example.com -d other.example.net
```
命令运行过程会需要你填写一个邮箱地址和同意协议。

如果安装失败会有相应的提示。按照提示来修改就好了。比如它会把证书安装在某一个路径，如果这个路径不存在，就会安装失败。需要你先手动创建对应的路径。再重新运行上面的命令就可以了。

最后还会引导你选择是只保留https请求，还是http和https兼容，所有的http都会自动重定向到https。根据你具体需求选择就好了。

配置成功后，记得开放 443 端口。
```
$ firewall-cmd --zone=public --list-ports # 查看防火墙目前开的端口
$ firewall-cmd --zone=public --add-port=443/tcp --permanent    # --permanent永久生效，没有此参数重启后失效
```

证书到期之后需要
```
$ ./certbot-auto certificates # 查看所有证书的状态
```
```
$ ./certbot-auto renew # 更新所有的证书
```

#### 如果是使用在前端就处理完图片的话 就可以忽略 GraphicsMagick 和 ImageMagick

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

#### 如果是使用egg.js 就可以忽略 PM2

### 安装 [PM2](https://github.com/Unitech/pm2) 进行node.js的项目管理
pm2默认运行目录是取当前用户的 $HOME/.pm2 ，这就造成每个用户都会启动一个新的 pm2 守护进程，也无法看到别的用户运行的node进程。
可以通过指定一个环境变量 PM2_HOME 来指定 .pm2 的位置，把它指定到 /var/run/pm2 方便所有用户访问。
```
vi /etc/profile 
#在文件最后加上: export PM2_HOME="/var/run/pm2"

source /etc/profile
# 保存修改后刷新配置
```
然后在安装pm2
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

##### 常见问题
```
$ pm2 list
[PM2] Spawning PM2 daemon with pm2_home=/home/sankuai/.pm2
```
一般是Node服务因为死循环而日志把机器的磁盘给打满了，导致了PM2的守护进程无法启动。可以通过先检查机器的磁盘空间是否给占满。如果是占满了，可以考虑对PM的部分Log日志进行删除。

### 常用Linux 命令
```
find / -type f -name "*.log" | xangs grep "ERROR" //从根目录开始查找所有扩展名为.log的文本文件，并找出包含”ERROR”的行

grep '\[ERROR\] \/' position-out-3.log //匹配对应内容的行

tail -n 100 position-out-3.log //显示最后100行
 
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

查看磁盘空间
df -hl

# 对某个文件添加写权限
chmod u+w /home/test

# 对某个文件移除写权限
chmod u-w /home/test

# 查看环境变量
echo $PATH
# 设置临时环境变量
export PATH="xxxxxx"
# 清除环境变量
unset $PATH

```


