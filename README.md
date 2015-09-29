# docker-mysql


[![mysql](https://dn-daoweb-resource.qbox.me/image-icon/mysql.svg)](http://www.mysql.com)

mysql数据库docker基础镜像

mysql是一个关系型数据库管理系统，由瑞典 mysql ab 公司开发，目前属于 oracle 旗下公司。mysql 最流行的关系型数据库管理系统，在 web 应用方面 mysql 是最好的 rdbms (relational database management system，关系数据库管理系统) 应用软件之一。mysql 是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。mysql 所使用的 sql 语言是用于访问数据库的最常用标准化语言。mysql 软件采用了双授权政策（本词条“授权政策”），它分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 mysql 作为网站数据库。由于其社区版的性能卓越，搭配 php 和 apache 可组成良好的开发环境。


## mysql版本


不同版本由不同的文件夹创建

* 5.5

* 5.6


## 使用


执行下面命令创建不同版本的mysql镜像:

```
docker build -t dockerxman/mysql 5.5/
```

启动镜像绑定3306端口

```
docker run -d -p 3306:3306 dockerxman/mysql
```

第一次你运行你的容器,将在mysql中创建一个随机密码的一个拥有所有特权的新用户“admin”。在运行的容器查看日志获得密码
:

```
docker logs <CONTAINER_ID>
```

您将看到如下输出:

```
You can now connect to this MySQL Server using:

mysql -uadmin -p47nnf4FweaKu -h<host> -P<port>

Please remember to change the above password as soon as possible!

MySQL user 'root' has no password but only allows local connections.
```       

在这种情况下, `47nnf4FweaKu` 就是`admin`的密码。


记住,`root`用户没有密码,只有从容器内访问

测试部署

```
mysql -uadmin -p
```

完成!

## 通过额外的配置来启动mysql服务器

你可以通过使用环境变量`EXTRA_OPTS`附加到`mysqld`设置

例如,使用小写运行mysql表名:

```
docker run -d -p 3306:3306 -e EXTRA_OPTS="--lower_case_table_names=1" dockerxman/mysql
```


## 设置一个特定的管理员账户密码

如果你想使用一个预设的密码,而不是一个随机生成的一个,你可以
在容器运行时设置环境变量“MYSQL_PASS”来设定一个密码`mypass`:

```
docker run -d -p 3306:3306 -e MYSQL_PASS="mypass" dockerxman/mysql
```

现在,你可以测试你的部署:

```
mysql -uadmin -p"mypass"
```

管理员用户名也可通过环境变量`MYSQL_USER`设置。



## 创建一个数据库在容器创建时


如果你想在第一次运行容器时在容器内创建一个数据库
你可以通过设置环境变量“ON_CREATE_DB”来创建一个自定义
名称的数据库

```
docker run -d -p 3306:3306 -e ON_CREATE_DB="newdatabase" dockerxman/mysql
```

如果结合导入SQL文件,这些文件将被导入到
创建的数据库

## 挂载数据库文件卷

为了保存数据库数据,你可以从主机挂载一个本地文件夹
到容器来存储数据库文件:

```
docker run -d -v /path/in/host:/var/lib/mysql dockerxman/mysql /bin/bash -c "/usr/bin/mysql_install_db"
```

这将挂载本地文件夹 `/path/in/host` 到docker容器内部的 `/var/lib/mysql` (mysql文件存储默认情况下)。`mysql_install_db` 创建数据库结构。

之后你就可以使用`/path/in/host`作为数据库目录运行你的mysql镜像:

```
docker run -d -p 3306:3306 -v /path/in/host:/var/lib/mysql dockerxman/mysql
```

## 从其他的容器挂载数据库文件卷

数据库数据持久化的另一种方法是将数据库文件存储在另一个容器。

要做到这一点,首先创建一个容器保存数据库文件:

```
docker run -d -v /var/lib/mysql --name db_vol -p 22:22 dockerxman/ubuntu
```

这将创建一个新的运行ssh的容器和使用 `/var/lib/mysql` 来存储mysql数据库数据

你可以通过使用`--name`来指定容器的名称,将用于下一个步骤

之后你就可以运行mysql镜像在数据卷上面创建容器(数据卷容器的名称通过`--volumes-from`指定)

```
docker run -d --volumes-from db_vol -p 3306:3306 dockerxman/mysql
```
## 迁移现有的mysql服务器

为了迁移你当前的mysql服务器,从你当前的服务器执行以下命令:

备份你的数据库结构:

```
mysqldump -u<user> -p --opt -d -B <database name(s)> > /tmp/dbserver_schema.sql
```

备份你的数据库数据:

```
mysqldump -u<user> -p --quick --single-transaction -t -n -B <database name(s)> > /tmp/dbserver_data.sql
```

例如sql数据库数据备份存储在主机文件夹`/tmp`,运行以下命令导入sql备份:

```
sudo docker run -d -v /tmp:/tmp dockerxman/mysql /bin/bash -c "/import_sql.sh <user> <pass> /tmp/<dump.sql>"
```

同样,你可以用sql文件运行新的数据库初始化:

```
sudo docker run -d -v /path/in/host:/var/lib/mysql -e STARTUP_SQL="/tmp/<dump.sql>" tutum/mysql
```

`<user>`和`<pass>`是之前设置的数据库用户名和密码, `<dump.sql >`是导入的sql文件的名称。


## 复制-主/从
要使用mysql复制, 需要设置环境变量 `REPLICATION_MASTER`/`REPLICATION_SLAVE`为`true`。同样, 主这边, 你可能希望指定 `REPLICATION_USER` 和 `REPLICATION_PASS`帐户和密码进行复制, 默认值是`replica:replica`

列子:

* 主

```
docker run -d -e REPLICATION_MASTER=true -e REPLICATION_PASS=mypass -p 3306:3306 --name mysql dockerxman/mysql
```

* 从

```
docker run -d -e REPLICATION_SLAVE=true -p 3307:3306 --link mysql:mysql dockerxman/mysql
```

现在你可以通过端口 `3306` 和 `3307` 访问主/从 mysql数据库

## 环境变量

`MYSQL_USER`: 设置一个特定的管理帐户的用户名 (默认 'admin')。

`MYSQL_PASS`: 设置一个特定的管理员帐户密码。

`STARTUP_SQL`: 定义一个或多个sql脚本用空格分开来初始化数据库。注意,脚本必须在容器内,所以你可能需要挂载它们

## 兼容性问题

* 由mysql 5.6创建的数据卷不能用于mysql 5.5镜像

## 代码创建和维护

* QQ: 479608797

* 邮件: fenyunxx@163.com

* [github](https://github.com/xiongjungit/docker-mysql)

* [dockerhub](https://hub.docker.com/r/dockerxman/)
