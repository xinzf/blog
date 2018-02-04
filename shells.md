上传h5文件
```
#!/usr/bin/env bash

file=$1
filename=${file##*/}
filenamenoext=${filename%.*}
target_root="/var/www/spread"
target_dir="$target_root/releases/$filenamenoext"

scp -P 30218 $file www@127.0.0.1:/tmp

ssh -p 30218 www@127.0.0.1 "mkdir $target_dir && unzip /tmp/$filename -d $target_dir && mv $target_dir/*/* $target_dir && rm $target_root/current && ln -s $target_dir $target_root/current"

#echo $target_dir
```
