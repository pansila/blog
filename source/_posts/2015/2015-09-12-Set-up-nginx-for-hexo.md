title: 设置nginx运行Hexo生产的静态网站
date: 2015-09-12 11:00:36
tags:
category:
---
hexo基于nodejs实现，本质上都是V8引擎解释js语句执行，效率必然比不上本地代码。在raspberry pi资源这么有限的环境下，hexo server不是个好主意，那么以速度著称的nginx就是个必然选择。
```
sudo apt-get install nginx
sudo /etc/init.d/nginx start
sudo /usr/sbin/update-rc.d -f nginx defaults
sudo vi /etc/nginx/sites-enabled/default
```
找到server的配置，修改root目录
```
server {
    ...
    server_name localhost;
    index index.html index.htm;
    root  /home/pi/hexo/public;
    ...
}
```
修改完后别忘了让nginx重新读取配置
```
sudo /etc/init.d/nginx reload
```
为了能根据blog源代码的改变及时更新public，可以让hexo在后台进行监视
```
hexo generate --watch
```
