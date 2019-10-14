---
title: nginx 403错误
---

### nginx 403错误
1. 权限配置不正确

 nginx有2个进程，一个主进程，一个工作进程ps -ef|grep nginx 如下(第三行是grep进程)
```
root     10181     1  0 11:04 ?        00:00:00 nginx: master process nginx -c /var/wwwnginx/nginx/conf/nginx_temp.conf
nginx    10182 10181  0 11:04 ?        00:00:00 nginx: worker process
root     16250 32426  0 11:20 pts/8    00:00:00 grep --color=auto nginx
```

* 主进程跟nginx执行的当前用户有关
* 工作进程跟nginx.conf配置相关(默认为nginx或者nobody) 使用use nginx(角色名); 配置

 工作进程角色的权限不正确是nginx出现403 forbidden最常见的原因。
 为了保证文件能正确执行，nginx的工作进程角色既需要文件的读权限,又需要文件所有父目录的可执行权限。

* 例如，当访问/usr/local/nginx/html/image.jpg时，nginx既需要image.jpg文件的可读权限，也需要/,/usr,/usr/local,/usr/local/nginx,/usr/local/nginx/html的可以执行权限。

* 1 解决办法:设置所有父目录为755权限，设置文件为644权限可以避免权限不正确。
* 2 如果必需要把目录放置在home目录，可以使用/var(root,所有用户可读可执行r-x),使用符号链接的方式