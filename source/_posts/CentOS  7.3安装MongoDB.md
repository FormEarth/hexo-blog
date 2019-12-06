---
title: CentOS  7.3安装MongoDB
date: 2019-12-04 21:53:15
index_img: https://i.loli.net/2019/12/04/BfmYrzd4xu1SK6h.jpg
banner_img: https://i.loli.net/2019/12/04/BfmYrzd4xu1SK6h.jpg
tags:
- Linux 
- mongoDB
---
## 下载
去mongoDB[官网](https://www.mongodb.com/download-center/community)，选择对应的版本，RHEL是redhat的linux发行版本，虽然不知道6.0和7.0有什么区别。这里选了gtz![我选择的版本](https://img-blog.csdnimg.cn/20191113114451281.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpbWVMZW5ndGhGb3JtYXQ=,size_16,color_FFFFFF,t_70)
## 安装
下载完成后将 `mongodb-linux-x86_64-rhel70-4.2.1.tgz`上传至服务器指定目录，
然后解压 `tar -zxvf  mongodb-linux-x86_64-rhel70-4.2.1.tgz`
重命名下 `mv mongodb-linux-x86_64-rhel70-4.2.1 mongodbServer`
将bin目录添加到环境变量中：
编辑环境变量 `vi /etc/profile`
添加以下内容
```shell
    #mongodb
    export PATH=$PATH:/usr/tools/mongodbServer/bin/
```
刷新环境变量 `source /etc/profile`
查看是否已经加入 `echo $PATH`

OK，安装完成，我们用 `mongod` 试图去启动服务，报错了
![启动报错](https://img-blog.csdnimg.cn/20191113122425930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpbWVMZW5ndGhGb3JtYXQ=,size_16,color_FFFFFF,t_70)
报的是`/data/db`没有找到，原来启动是需要参数的，这个路径是它默认的数据保存的地方，但它不会自己创建。我们可以自己创建这个目录，但是我们可以指定自定义的目录。
mongoDB启动时可以指定很多参数，也可以指定一个配置了指定参数的配置文件，以下是两种方式的命令，这些参数可以配置mongoDB的工作方式，参考文章[这里](https://www.cnblogs.com/seasonzone/p/9359425.html)

- `mongod --fork --dbpath=<自定义的数据存放路径> --logpath=<自定义的日志数据存放路径> --logappend` 
- `mongod --config <自定义的配置文件路径>`

我们自定义一个conf文件:
`echo /usr/tools/mongodbServer/mongo_server.conf`
内容如下
```
#端口号(默认为27017)
#port = 27017
#数据存放目录
dbpath =/usr/tools/mongodbServer/data/db/
#日志目录
logpath = /usr/tools/mongodbServer/data/log/mongo_server.log
#设置后台运行
fork = true
#日志输入方式
logappend = true
#开启认证（因为本地访问，所以关闭了，若是需要远程，则应该开启）
#auth = false
#绑定IP 默认的为 127.0.0.1，即只允许本地访问，0.0.0.0允许任何访问
bind_ip = 0.0.0.0 
```
OK,再启动  `mongod --config /usr/tools/mongodbServer/mongo_server.conf`
tip:关闭mongoDB `mongod  --shutdown --config /usr/tools/mongodbServer/mongo_server.conf`
![成功](https://img-blog.csdnimg.cn/20191113125208644.png)
看起来是成功了
## 创建账号
因为默认是本地访问，无需验证，使用命令 `mongo` 进入 mongoDB数据库
一堆的警告出现了，先不管
`show dbs` 查看所有数据库
`use admin` 切换数据库
`db.createUser({user:"admin",pwd:"<你的密码>",roles:["root"]})`创建一个 admin 用户，权限是 root，有关于权限即 roles，请参考以下内容，可以指定一个账户在某个具体数据库的具体权限。
```
read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向 system.users 集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在 admin 数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在 admin 数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在 admin 数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在 admin 数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在 admin 数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```

创建一个application_user用户，在demo库上是可读写的、在admin上是只读的`show users` 查看当前库的用户
```shell
db.createUser( { user: "application_user", pwd: "<你的密码>", roles: [ { role: "readWrite", db: "demo" },  { role: "read", db: "admin" } ] })
```
查看当前库的用户 `show users `
查看系统内所有用户 `db.system.users.find().pretty()`  
有了用户之后我们就可以开启验证，允许远程连接了,关闭mongoDB，修改 mongo_server.conf
取消 `auth = true`，保存，重启mongoDB。
进入mongoDB，可以进来，但是操作什么的都没有反应，甚至报错了
![报错了](https://img-blog.csdnimg.cn/20191113135505875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpbWVMZW5ndGhGb3JtYXQ=,size_16,color_FFFFFF,t_70)
该命令需要认证，OK，认证一下 `db.auth("application_user","<你的密码>")`
接下来我们访问下我们的域名＋27017，出现下面这个就说明能够访问了，需要开放27017端口![访问](https://img-blog.csdnimg.cn/20191113140303209.png)
使用studion 3T连接![成功，ok](https://img-blog.csdnimg.cn/20191113144510810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RpbWVMZW5ndGhGb3JtYXQ=,size_16,color_FFFFFF,t_70)






