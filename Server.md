# Server for CentOS 7.2 64位

### 安装git
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

### 安装 [n](https://github.com/tj/n) 来管理node.js 的版本


