

# CentOS 部署 MongoDB

## **安装过程**

1、添加 MongoDB 的源：

`mongodb-org` 这个包默认不存在 CentOS 的源里，所以要先添加到我们服务器中：

```text
 vi /etc/yum.repos.d/mongodb-org.repo
```



然后访问 [Install on Red Hat](http://link.zhihu.com/?target=https%3A//docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/%23configure-the-package-management-system-yum) 找到最新的 MongoDB 稳定版本并添加到上面打开的文档中，类似这样：

```text
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

```shell
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

```shell
[mongodb-org-3.4]
name=MongoDB 3.4 Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=0
enabled=1
```

编辑并保存文件，查看服务器源列表中是否已添加成功（留意 mongodb-org**-版本**）：

```text
$ yum repolist 
# 输出一般如下
. . .
repo id                          repo name
base/7/x86_64                    CentOS-7 - Base
extras/7/x86_64                  CentOS-7 - Extras
mongodb-org-3.2/7/x86_64         MongoDB Repository
updates/7/x86_64                 CentOS-7 - Updates
. . .
```



2、安装 MongoDB：

```text
 yum install mongodb-org
```



3、启动 MongoDB：

```text
$ sudo systemctl start mongod

```



如有需要重新解析改动后的 /etc/mongod.conf 配置文件，可以执行：

```text
$ sudo systemctl reload mongod
```



4、因为 systemctl 并不返回启动结果，所以可以通过以下命令查看是否启动：

```text
$ sudo tail /var/log/mongodb/mongod.log
```



查找是否包含该日志，若出现则表示服务已启动，可以通过 `mongo` 来开启命令：

```text
. . .
[initandlisten] waiting for connections on port 27017
```



5、开机自启动

首先查看是否已启用：

```text
$ systemctl is-enabled mongod; echo $?


# 查看输出是否包含 enabled 字样
. . .
enabled
0
```



若无，可以手动启动：

```text
$ sudo systemctl enable mongod
```



6、导入 example 数据

```
mongorestore -h 127.0.0.1:27017 -d doracms2 --drop #centos上要引入的数据的目录
```

7 创建管理员 和操作员

 ```shell
mongo ...
...
use admin
# 创建管理员
db.createUser({user: "admin",pwd: "admin",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]})
退出  然后使用管理员账号重新登录 创建操作员
# 创建数据操作员 
db.createUser({user: "leiyuyang",pwd: "leiyuyang",roles: [ { role: "readWrite", db: "doracms2" } ]})
 ```

8 修改配置文件 mongdb.conf 

 添加在

```js
security:
   auth：true
```

如果是在3.x 版本以上添加

```js
security:
   authorization: "enabled"
```



查看数据：

```text
$ mongo
$ db.restaurants.find().limit(1).pretty()

# 输出
{
    "_id" : ObjectId("57e0443b46af7966d1c8fa68"),
    "address" : {
        "building" : "1007",
        "coord" : [
            -73.856077,
            40.848447
        ],
        "street" : "Morris Park Ave",
        "zipcode" : "10462"
    },
    "borough" : "Bronx",
    "cuisine" : "Bakery",
    "grades" : [
        {
            "date" : ISODate("2014-03-03T00:00:00Z"),
            "grade" : "A",
            "score" : 2
        },
        {
            "date" : ISODate("2013-09-11T00:00:00Z"),
            "grade" : "A",
            "score" : 6
        },
        {
            "date" : ISODate("2013-01-24T00:00:00Z"),
            "grade" : "A",
            "score" : 10
        },
        {
            "date" : ISODate("2011-11-23T00:00:00Z"),
            "grade" : "A",
            "score" : 9
        },
        {
            "date" : ISODate("2011-03-10T00:00:00Z"),
            "grade" : "B",
            "score" : 14
        }
    ],
    "name" : "Morris Park Bake Shop",
    "restaurant_id" : "30075445"
}
```



删除数据库：

```text
$ db.restaurants.drop()
```



## **数据库操作**

1、添加数据库用户

```text
$ mongo --port 27017
$ use admin
$ db.createUser(
   {
     user: "root",
     pwd: "xxxxxx",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  })

# 现在我们为 MongoDB 的 admin 数据库添加一个用户 root，MongoDB 可以为每个数据库
# 都建立权限认证，也就是你可以指定某个用户可以登录到哪个数据库。上面的代码，我们为
# admin 数据库添加了一个 root 用户，在 MongoDB 中 admin 数据库是一个特别的数据库，
# 这个数据库的用户，可以访问 Mongodb 中的所有数据库。
```



2、用户登录

```text
# 方式一
$ mongo --port 27017 -u "root" -p "xxxxxx" --authenticationDatabase "admin"

# 方式二
$ mongo --port 27017
$ use admin
$ db.auth("root", "xxxxxx")

// 输出 1 表示验证成功，并且不会报错了
```



3、账户相关操作：

```text
# 查询某个数据库下用户（每个数据库的用户账号都是以文档形式存储在 system.users 集合里面的）
$ db.system.users.find()

# 删除某个数据库下的所有用户
$ db.system.users.remove()

# 删除指定用户
$ db.system.users.remove({'user':'用户名'})

# 修改账户密码
db.changeUserPassword("username", "newpassword")
```



4、数据库操作

```text
$ mongo --port 27017
$ use admin
$ db.auth("root", "xxxxxx")

$ show dbs         # 查看数据库
$ use <db name>    # 选择数据库

$ show collections           # 查看 collections
$ db.collectionName.find()   # 查看 documents

$ use <db name>     # 创建数据库
$ db.test.insert({"name":"tutorials point"})    # 随便插入点数据（才能出现）
```



5、数据的导入与导出

```text
# mongoexport导出表，或者表中部分字段（一般可以简写）
$ mongoexport -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -c 表名 --csv -q 条件 -f 字段 -o newdbexport.json/dat/csv

-h：数据库宿主机的IP
-u：数据库用户名
-p：数据库密码
-d：数据库名字
-c：集合的名字
-f：导出的列名
-q：导出数据的过滤条件
-o：导出文件的目录及文件名（/xx/xx/xx.json）
--type：json 或 csv（默认是 json）

比如导出整张表：
$ mongoexport -d test -c users -o allusers.dat

比如导出部分字段：
$ mongoexport -d test -c users --csv -f uid,name,sex -o test/users.csv

比如根据条件敢出数据：
$ mongoexport -d test -c users -q '{uid:{$gt:1}}' -o test/users.json 
```



```text
# mongoimport 导入表，或者表中部分字段
$ mongoimport -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -c 表名 --upsert --drop 文件名

比如还原导出的表数据（--upsert 插入或者更新现有数据）
$ mongoimport -d test -c users --upsert test/users.dat

比如部分字段的表数据导入（--upsertFields根--upsert一样）
$ mongoimport -d test -c users --upsertFields uid,name,sex test/users.dat

比如还原csv文件
$ mongoimport -d test -c users --type csv --headerline --file test/users.csv  
```



```text
# mongodump 备份数据库（如果想导出所有数据库，可以去掉-d）
$ mongodump -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 -o 文件存在路径

比如：
$ mongodump -h 127.0.0.1 -p 30216 -d test -uxxxx -pxxxxx -o home/mongodb/
```



```text
# mongorestore 还原数据库
$ mongorestore -h IP --port 端口 -u 用户名 -p 密码 -d 数据库 --drop 文件存在路径

比如：
$ mongorestore -d test /home/mongodb/test
```



原

# yum 升级 mongodb

2016年10月24日 14:12:29 [huaism](https://me.csdn.net/huaishuming) 阅读数：1782



版权声明：本文为博主原创文章，未经博主允许不得转载。	https://blog.csdn.net/huaishuming/article/details/52911159

**名为升级实际是卸载重装。**

**主要流程**

**1. 备份**

**2. 卸载**

**3.安装**

**4.配置**

**5.还原**



1.备份 不做解释



```html
 ./mongodump --host 127.0.0.1 --port 27017 --username root --password xxxx --out /application/mongodb_backup 
```







\2. 卸载[yum卸载mongodb及后续问题的解决](http://blog.csdn.net/sodino/article/details/52368853)



![error.mongodb](http://ww2.sinaimg.cn/mw690/e3dc9ceagw1f7bmmtwmjtj20g107z0vm.jpg)

卸载过程

``// 找出mongodb相关的安装包yum list installed | grep mongo// 删除指定的安装包,包名由上面的list命令获得yum erase mongodb.x86_64yum erase mongodb-server.x86_64

详细如下图：
![uninstall.mongodb](http://ww2.sinaimg.cn/mw690/e3dc9ceagw1f7bmmuloxhj213z0f344l.jpg)

确认删除了，查看`which`命令发现`mongo`指向了3.2.9高版本的程序了（之前已经有将高版本mongo路径添加到系统的环境变量）。
![which.mongo](http://ww1.sinaimg.cn/mw1024/e3dc9ceagw1f7bmmv5ywaj20ca045ab0.jpg)

但下一步直接使用`mongo`却发现出错了

``# mongo-bash: /usr/bin/mongo: No such file or directory

![error.mongodb](http://ww2.sinaimg.cn/mw1024/e3dc9ceagw1f7bmmwgc35j20sf03mwgm.jpg)

好吧，`yum`只删除了安装包，但没有把快捷方式一并清除。
那就把快捷方式指向高版本的mongodb的可执行文件吧。

``ln -s /root/soft/mongodb/bin/mongo /usr/bin/mongo

好了，解决了。



如果没有设置软连接，则出现这个情况可能是，没启动服务









3.安装：

参考：

选中自己的版本进行安装

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/





### Configure the package management system (`yum`).



Create a `/etc/yum.repos.d/mongodb-org-3.2.repo` file so that you can install MongoDB directly, using `yum`.

Changed in version 3.0: MongoDB Linux packages are in a new repository beginning with 3.0.

#### For the latest stable release of MongoDB

Use the following repository file:





```
$releasever 是系统版本
```



```
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
//执行安装
sudo yum install -y mongodb-org-3.2.10 mongodb-org-server-3.2.10 mongodb-org-shell-3.2.10 mongodb-org-mongos-3.2.10 mongodb-org-tools-3.2.10
//启动服务
service mongod start
```

4.还原：

mongorestore /application/data_backup/mongodb



\5. 配置；

安装后 输入 mongo 会出现警告：

\1. 数量限制

解决办法



修改配置文件 /etc/security/limits.conf，添加配置信息：

[root@localhost ~]# vi /etc/security/limits.conf

 

[?](http://www.2cto.com/database/201505/398290.html#)

```
`mongod soft nofile 64000``mongod hard nofile 64000``mongod soft nproc 32000``mongod hard nproc 32000`
```

 

重启 mongod 服务：

转载：





# 系统环境

```
Distributor ID: CentOS



Description:    CentOS release 6.7 (Final)



Release:    6.7



Codename:   Final
```

# 问题描述



在系统上安装mongodb之后报错。 
(安装教程地址: <https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/>)

错误信息: 
WARNING: /sys/kernel/mm/transparent_hugepage/enabled is ‘always’.We suggest setting it to ‘never’ 
WARNING: /sys/kernel/mm/transparent_hugepage/defrag is ‘always’.We suggest setting it to ‘never’ 
WARNING: soft rlimits too low. rlimits set to 1024 processes, 65535 files. Number of processes should be at least 32767.5 : 0.5 times number of files. 
如图： 
![mongodb报错](https://img-blog.csdn.net/20160521162126599)

# 解决方案

## 前两个warning

```
sudo echo "never" > /sys/kernel/mm/transparent_hugepage/enabled



sudo echo "never" >  /sys/kernel/mm/transparent_hugepage/defrag
```

## 第三个warning

```
vim /etc/security/limits.conf



添加一下几行



mongod  soft  nofile  64000



mongod  hard  nofile  64000



mongod  soft  nproc  32000



mongod  hard  nproc  32000
```

## 重启mongod

```
sudo service mongod restart
```

## 成功

重启成功之后，所有报错都没啦，如下

```
➜  ~ git:(master) mongo



MongoDB shell version: 3.2.6



connecting to: test



>
```