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

### 安装 [n](https://github.com/tj/n) 管理node.js 的版本
```
curl -L https://git.io/n-install | bash

. ~/.bashrc
```
##### 使用n来安装最新的node.js
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
ln -s redis-3.2.8 redis
cd redis
make PREFIX=/usr/local/redis install
cd /usr/local/redis/bin
ls
//看到这些文件代表安装成功 redis-benchmark  redis-check-aof  redis-check-dump  redis-cli  redis-server
```
##### 将redis做成一个服务
```
cp /usr/local/src/redis/utils/redis_init_script /etc/rc.d/init.d/redis
vim /etc/rc.d/init.d/redis
```
修改里面的内容
* 第2行 ```#chkconfig: 2345 80 90 ```


