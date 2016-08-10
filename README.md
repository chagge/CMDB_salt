# CMDB_salt
基于saltstack的CMDB

管理软件版本是基于saltstack的接口调用来实现，后期会更新，推送和执行的后台日志记录，和任务编排等系统，版本去掉了监控部分，后期应该会结合zabbix自定义时间出图表：
安装步骤（基本环境，需要安装好（Django、south、MySQLdb模块）安装过程报错就继续安装模块即可）：
```sh
可以参照http://sofar.blog.51cto.com/353572/1596960/ 装环境
pip install 'django==1.6.5'
pip install south
pip install MySQL-python
```
1、在服务端建立/web目录。把项目拷贝到目录下。

2、在服务起端执行脚本:(在/web/CMDB/app/backend目录存放)
```sh
install_server.sh  


[root@nba_statistics2 backend]# sh install_server.sh 
Retrieving http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
warning: /var/tmp/rpm-tmp.sXNnZn: Header V3 RSA/SHA256 Signature, key ID 0608b895: NOKEY
Preparing...                ########################################### [100%]
        package epel-release-6-8.noarch is already installed
Loaded plugins: fastestmirror, security
Loading mirror speeds from cached hostfile
Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again

解决方法：yum upgrade ca-certificates --disablerepo=epel
```

3、在cp 客户端脚本到client执行，注意执行格式：
```sh
./install_client.sh client 192.168.63.239    
#####client表示客户端主机ID，建议跟主机的Hostname一致，，后面的IP表示server端的IP地址。
```
4、安装完成之后；测试是否成功：在server执行命令如下：
```sh
[root@master backend]# salt '*' test.ping
client:
    True
有这个返回值说明成功安装了，saltstack的master 和客户端。
```
5、在server端同步client脚本到client：
```sh
salt '*' saltutil.sync_all
```
=====================

6、在server端安装salt-api，因为大部分操作都是调用api,可以参考博客（因为需要手工指定证书，所以没有做成脚本的形式）：

http://xiaoluoge.blog.51cto.com/9141967/1613353

https://www.xiaomastack.com/2014/11/18/salt-api/

7、建立mysql 数据库并且授权账号登录：
```
create database cmdb default charset=utf8; 
8、修改配置文件config.ini(所在目录：/web/CMDB/app/backend/):
[db]
db_host = 127.0.0.1  
db_port = 3306
db_user = root
db_pass = 123456
db_name = cmdb
[saltstack]
url = https://192.168.63.89:8888
user = xiaoluo
pass = 123456
[network]
device = eth0 ####因为不确定有些系统用的是eth0.有些用的是em。根据自己的需求填
****备注上面是数据库的账号管理密码等，下面是salt-api的账号密码：
新建 /etc/salt/master.d/mysql.conf （修改好后需重启salt-master和salt-master）
mysql.port: 3306
mysql.user: root
mysql.pass: 123456
mysql.db: cmdb
mysql.host: 127.0.0.1 
/etc/init.d/salt-master restart
/etc/init.d/salt-api restart
```
8、数据库创建：
```
manage.py syncdb
manage.py migrate app
备注：输入的账号密码是登录网站的账号密码：
```
9、安装成功启动登录：
```
启动步骤：
启动/web/CMDB/app/backend
nohup python salt_event_to_mysql.py &   ###事件监听返回日志
nohup ./manage.py runserver 0.0.0.0:80 &
```
