# ssbc
手撕包菜网站

## 网站说明
这是 www.shousibaocai.com 的网站源代码。
开源的目的是为了促进技术交流和相互学习，把DHT与搜索引擎技术应用到更广泛的领域去。

本站于2015年5月使用django改写。
与爬虫相关的代码都在目录workers下。

相关文章请查看作者博客：
http://xiaoxia.org/2015/05/15/shousibaocai-opensource/

## 安装说明
**环境：**

服务器商 | 内存 | 系统 | 网络
---|---|---|---
Vultr日本 | 2G | Centos7 x64 | 有公网IP
DO | 1G | Centos7 x64 | 有公网IP
Linode(推荐) | 1G | Centos7 x64 | 有公网IP

1. 环境检测（*ssbc当前版本是基于django1.8.1开发，所需python环境为python2.7.5以上*）  
```Shell
[root@localhost ~]# python -V  //执行python -V即可获取当前版本    
Python 2.7.5   
[root@localhost ~]# systemctl stop firewalld.service  //关闭firewall防火墙  
[root@localhost ~]# systemctl disable firewalld.service    //禁止firewall防火墙开机启动  
[root@localhost ~]# systemctl stop iptables.service  //关闭iptables防火墙  
[root@localhost ~]# systemctl disable iptables.service  //禁止iptables防火墙开机启动
```  
2. 获取ssbc安装包并解压  
[github源码](https://github.com/aystshen/ssbc)  

```Shell
[root@localhost ~]# wget https://github.com/78/ssbc/archive/master.zip  
[root@localhost ~]# yum -y install unzip  
[root@localhost ~]# unzip master.zip  
```
解压后你会发现在/root目录下有个文件夹ssbc-master  

3. 安装MariaDB数据库及所需环境
```Shell
[root@localhost ~]# yum -y install gcc  
[root@localhost ~]# yum -y install gcc-c++  
[root@localhost ~]# yum -y install python-devel  
[root@localhost ~]# yum -y install mariadb  
[root@localhost ~]# yum -y install mariadb-devel  
[root@localhost ~]# yum -y install mariadb-server  
[root@localhost ~]# cd ssbc-master  
[root@localhost ssbc-master]# wget https://raw.github.com/pypa/pip/master/contrib/get-pip.py   
[root@localhost ssbc-master]# python get-pip.py  
[root@localhost ssbc-master]# pip install -r requirements.txt  
```

4. 创建ssbc数据库  
```Shell
[root@localhost ssbc-master]# systemctl start  mariadb.service  //启动数据库  
[root@localhost ssbc-master]# mysql -uroot -p  
Enter password: (回车即可)  
MariaDB [(none)]> create database ssbc default character set utf8;  
MariaDB [(none)]> quit;  //创建成功后退出  
```

5．Web服务器设置并启动
```Shell
[root@localhost ssbc-master]# python manage.py makemigrations  
[root@localhost ssbc-master]# python manage.py migrate  
[root@localhost ssbc-master]# nohup python manage.py runserver 0.0.0.0:80 >/dev/zero &          //启动网站并在后台运行  
```
按回车键继续  
浏览器输入http://IP，网站能打开  

6. 安装Sphinx  
```Shell
[root@localhost ssbc-master]# yum -y install unixODBC unixODBC-devel postgresql-libs  
[root@localhost ssbc-master]# wget http://sphinxsearch.com/files/  sphinx-2.2.9-1.rhel7.x86_64.rpm  
[root@localhost ssbc-master]# rpm -ivh sphinx-2.2.9-1.rhel7.x86_64.rpm  
```
6. 创建以下文件夹并赋予755权限 
```Shell
[root@localhost ssbc-master]# mkdir  -p  /data/bt/index/db /data/bt/index/binlog  /tem/downloads  
[root@localhost ssbc-master]# chmod  755 -R /data  
[root@localhost ssbc-master]# chmod  755 -R /tem  
```

7. 生成索引  
```Shell
[root@localhost ssbc-master]# systemctl restart mariadb.service  //重新启动Mariadb 
[root@localhost ssbc-master]# systemctl enable mariadb.service  //设置mariadb开启自启动  
[root@localhost ssbc-master]# indexer -c sphinx.conf --all //all 前面是空格减号减号
[root@localhost ssbc-master]# searchd --config ./sphinx.conf  //config前是空格减号减号
```
确定没有报错，继续下一步  

8. 开启爬虫(*workers目录下*) 
```Shell
[root@localhost ssbc-master]# cd workers  
[root@localhost workers]# python simdht_worker.py  //爬虫运行，等2分钟出现数据之后CTRL+C停止  
[root@localhost workers]# nohup python simdht_worker.py >/dev/zero & //让爬虫在后台运行，按回车键继续  
[root@localhost workers]# python index_worker.py      //入库索引 ，等待10分钟出现数据后CTRL+C停止  
[root@localhost workers]# nohup python index_worker.py >/dev/zero &   //让索引在后台运行，按回车键继续  
[root@localhost workers]#cd ..  
[root@localhost ssbc-master]# python manage.py createsuperuser  //增加后台管理员
//输入管理员用户名  
//输入管理员邮箱  
//输入管理员密码  
//确认密码，完成  
//管理员登录地址：http://IP/admin 
```

9. 测试效果：  
浏览器输入http://IP 

## 附：

**去除搜索页右下角广告：**
```Shell
[root@localhost ssbc-master]# cd web/static/js  
[root@localhost js]# vi ssbc.js  //找到如下3行，在前面添加//进行注释，保存  
//        document.write('<script src="http://v.6dvip.com/ge/?s=47688"><\/script>');  
//            document.writeln("<script language=\"JavaScript\" type=\"text/ javascript\" src=\"http://js.6dad.com/js/xiaoxia.js\"></script>");  
//           document.writeln("<script language=\"JavaScript\" type=\"text/javascript\" src=\"http://js.ta80.com/js/12115.js\"></script>");  
```

**可能出现的故障：**  
1. 搜索中文报错  
```
'ascii' codec can't encode characters in position 42-43: ordinal not in range(128)
```
解决办法：  
如果是centos7系统，修改/usr/lib64/python2.7/site.py  
```
vi  /usr/lib64/python2.7/site.py  
```
在import sys下添加2行： 
```
reload(sys)  
sys.setdefaultencoding('utf8')  
```

2. 爬虫运行时 可能会遇到如下问题：
```
Python and Django OperationalError (2006, 'MySQL server has gone away')  
```
解决方法：   
<http://stackoverflow.com/questions/14163429/python-and-django-operationalerror-2006-mysql-server-has-gone-away>  
/etc/my.cnf 在最后一行的下面添加
```
wait_timeout=2880000  
interactive_timeout = 2880000  
max_allowed_packet = 512M  
```

**常见问题：**  
1. 必须centos7吗？  
非常建议使用centos7，centos6可能会有意想不到的错误  
2. 怎么查看入库的文件？  
登录管理员后台，点击 Hashs   
3. 怎么查看每天入库了多少文件，以便清楚入库效率？  
登录管理员后台，点击 Status reports   
4. 如何确认web服务器、采集、入库正在运行？  
ps -ef|grep python  
结果里面有  
python manage.py runserver 0.0.0.0:80    
python simdht_worker.py    
python index_worker.py  
即表示正在运行。  