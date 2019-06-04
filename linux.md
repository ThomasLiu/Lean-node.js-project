创建脚本
```
$ vi test.sh
```

test.sh
```
#! /bin/bash
# test.sh

echo hi
```

保存后
```
$ chmod +x ./test.sh  #给脚本添加执行权限
./test.sh           #执行脚本文件
```




## 常用
#### 自定义方法及方式调用、参数

例子：重启某个目录下的所有项目
```
#!/bin/bash
#folder="./egg"

function readfile ()
{
  for file in `ls $1`
  do
  if [ -d $1"/"$file ]
  then
   echo $1"/"$file
   cd $1"/"$file
   npm run restart
   cd /home

  fi
  done
}

folder="./egg"
readfile $folder
```


#### 定时任务
```
$ systemctl enable crond #（设为开机启动） 
$ systemctl start crond #（启动crond服务） 
$ systemctl status crond #（查看状态） 
$ systemctl stop crond #（暂停） 
$ systemctl restart crond #（重启） 
```
如果没有 crond 的话使用 yum 安装
```
$ yum install crontabs
```
设置用户自定义定时任务
```
$ vi /etc/crontab 
```
按照文件内的提示设置你需要的定时任务

保存文件后
```
$ crontab /etc/crontab # 加载任务,使之生效
$ crontab -l # 查看任务
```
##### 拿一个常见需求来举例，每天凌晨定时把30天前的日志删除。
```
$ cd /home
$ vi cleanlog.sh
```
cleanlog.sh 内容
```
#!/bin/bash
# cleanlog.sh

echo clean start... find /root/logs/ -mtime +30 -exec rm -rf {} \;

find /root/logs/ -mtime +30 -exec rm -rf {} \;
echo clean finish~
```
保存后
```
$ vi /etc/crontab 
```
在最后加上
```
$ 0 0 * * * root /home/cleanlog.sh
```
保存文件
```
$ crontab /etc/crontab # 加载任务,使之生效
$ crontab -l # 查看任务
```

更多 [linux 脚本学习](http://c.biancheng.net/shell/)
