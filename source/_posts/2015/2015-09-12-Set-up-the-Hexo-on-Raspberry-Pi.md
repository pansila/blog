---
title: 在树莓派上安装hexo
abbrlink: ada4fde5
date: 2015-09-12 09:01:19
tags:
category:
---
之前烧录的Raspberry image已经自带了很多工具，其中包括git，但是hexo还不在里面。用`apt-get install nodejs`装了一个，版本非常老0.6.xx。想要更新到最新？没问题，官网直接为树莓派的老旧CPU编译好了。
```bash
wget http://nodejs.org/dist/latest/node-v4.0.0-linux-armv6l.tar.gz
tar xf node-v4.0.0-linux-armv6l.tar.gz
mv node-v4.0.0-linux-armv6l/ node
mv node /usr/local/
chown -R pi.pi /usr/local/node
```
如果后面有更新版本，上面的版本号注意替换掉。
设置环境变量
```bash
echo "PATH=$PATH:/usr/local/node/bin" >> ~/.profile
echo "export PATH" >> ~/.profile
source ~/.profile
node -v  #检查路径是否设置正确
```
安装hexo
```bash
npm install -g hexo
mkdir -p /home/hexo
cd /home/hexo
hexo init #添加一个博客站点
npm install
```
clone blog hexo源文件到本地
```bash
git clone -b blog_code https://github.com/pansila/pansila.github.io.git
```
执行后会将git上blog_code分支的文件放到pansila.github.io目录，这个目录放置了hexo源代码（网页静态文件放在了master branch），因为之前在hexo目录建立hexo环境，所以要让hexo在git的文件上跑起来，需要将pansila.github.io目录下的所有东西复制到hexo目录
```
cp pansila.github.io .. -R
rm pansila.github.io -R
```
试了下速度，rapberry pi上跑hexo性能不如乐观。。。
同样的文件raspberry pi上，load files要24s，generate files要23s
PC上load files只要1s，generate files 580ms

参考文章：[树莓派安装node.js来跑Hexo静态博客](http://rpi.linux48.com/Hexo.html)
