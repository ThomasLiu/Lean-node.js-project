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



更多 [linux 脚本学习](http://c.biancheng.net/shell/)
