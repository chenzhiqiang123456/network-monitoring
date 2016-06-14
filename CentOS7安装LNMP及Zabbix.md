#CentOS7安装编译安装LNMP及Zabbix

##一、CentOS7安装LNMP
###**1.1.nginx编译安装**
<pre>
##创建用户
useradd -u 600 www  -s /sbin/nologin  -M
##安装依赖包
yum install pcre-devel zlib-devel openssl-devel   gcc gcc-c++  -y
##下载nginx1.8的gz包
wget http://nginx.org/download/nginx-1.8.0.tar.gz
解压nginx包
tar xf nginx-1.8.0.tar.gz
编译
cd nginx-1.8.0
./configure --user=www --group=www --prefix=/usr/local/nginx-1.8.0 --with-http_stub_status_module --with-http_ssl_module --with-http_spdy_module --with-http_gzip_static_module --with-ipv6 --with-http_sub_module --without-http_geo_module --without-http_map_module
####
make && make install
####制作软连接
ln -s /usr/local/nginx-1.8.0/ /usr/local/nginx
####启动nginx
/usr/local/nginx/sbin/nginx 
####修改nginx的配置文件
cp nginx.conf nginx.conf.default
egrep -v "#|^$" nginx.conf.default  >nginx.conf
</pre> 

###**1.2 mysql数据库的安装**
<pre>
添加用户
useradd -s /sbin/nologin -M mysql
安装依赖包及perl模块
yum install make gcc-c++ cmake bison-devel nucurses-devel perl-Module-Install.noarch -y
下载mysql-5.6.23的gz包
wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.23.tar.gz  
解压mysql的包
tar xf mysql-5.6.23.tar.gz 
编译
<pre>
   cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DMYSQL_UNIX_ADDR=/var/lib/mysql/mysqld.sock -DENABLED_LOCAL_INFILE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci
make  &&  make install
</pre>
创建 mysql的数据包目录及授权
mkdir /mnt/java/mysql/data -p  
chown  -R mysql.mysql /mnt/java/mysql/data

[root@chen-mysql scripts]# pwd
/usr/local/mysql/scripts
<pre>
[root@chen-mysql scripts]# ./mysql_install_db  --basedir=/usr/local/mysql/ --datadir=/mnt/java/mysql/data --user=mysql
</pre>
vim /etc/profile
PATH=/usr/local/mysql/bin:$PATH
source /etc/profile
cp -r  /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
vi /etc/init.d/mysqld
=========================
basedir=/usr/local/mysql
datadir=/mnt/java/mysql/data

vi /etc/my.cnf 
datadir=/mnt/java/mysql/data
socket=/var/lib/mysql/mysqld.sock
/etc/init.d/mysqld start

mysqladmin -uroot password "123456" 
</pre> 

###**1.3 PHP安装**
<pre>
php-5.5.27包下载
wget http://cn2.php.net/distributions/php-5.5.27.tar.xz
使用epel源
 wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
安装依赖包
yum install gcc bison bison-devel zlib-devel libmcrypt-devel mcrypt mhash-devel openssl-devel libxml2-devel libcurl-devel bzip2-devel readline-devel libedit-devel sqlite-devel -y
 yum install libxslt libxslt-devel freetype-devel  libpng-devel libjpeg-devel -y  
解压php包
tar xf php-5.5.27.tar.gz
cd php-5.5.27
编译
 ./configure \
--prefix=/application/php5.5.27 \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-iconv-dir=/usr/local/libiconv \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib \
--with-libxml-dir=/usr \
--enable-xml \
--disable-rpath \
--enable-bcmath \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--with-curl \
--enable-mbregex \
--enable-fpm \
--enable-mbstring \
--with-mcrypt \
--with-gd \
--enable-gd-native-ttf \
--with-openssl \
--with-mhash \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-soap \
--enable-short-tags \
--enable-static \
--with-xsl \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-ftp

make && make install

php配置
<pre>
 cp php.ini-development /application/php5.5.27/etc/php.ini
 cp /application/php5.5.27/etc/php-fpm.conf.default /application/php5.5.27/etc/php-fpm.conf
 cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod +x /etc/init.d/php-fpm
 /etc/init.d/php-fpm start

在nginx server中添加
location ~ .*\.(php|php5)?$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            include  fastcgi.conf;
        }

</pre>
php测试

<?php
phpinfo();
?>

mysql 测试
[root@web1 bbs]# cat oldboy_mysql.php 
<?php
$link_id=mysql_connect('localhost','root','123456') or mysql_error();

if($link_id){
   echo "mysql successful by chen !\n";
}else{
   echo "mysql_error()";

}
?>
添加gettext.so模块
进入/application/tools/php-5.5.27/ext/gettext目录
/application/php5.5.27/bin/phpize 
 ./configure --with-php-config=/application/php5.5.27/bin/php-config
make && make install
在到php.ini 添加
extension=gettext.so
</pre>

##二、zabbix 安装
###**3.1 mysql配置**
<pre>
wget http://120.52.73.47/nchc.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.3/zabbix-3.0.3.tar.gz
tar xf  zabbix-3.0.3.tar.gz
mysql> create database zabbix character set utf8; 
mysql> grant all on  zabbix.* to zabbix@'192.168.56.%' identified by '123456' with grant option; 
mysql> flush privileges;

[root@zabbix-server www]# mysql -uzabbix -p123456 -h192.168.56.12 zabbix< /application/tools/zabbix-3.0.3/database/mysql/schema.sql 
Warning: Using a password on the command line interface can be insecure.
[root@zabbix-server www]# mysql -uzabbix -p123456 -h192.168.56.12 zabbix< /application/tools/zabbix-3.0.3/database/mysql/images.sql 
Warning: Using a password on the command line interface can be insecure.
[root@zabbix-server www]# mysql -uzabbix -p123456 -h192.168.56.12 zabbix< /application/tools/zabbix-3.0.3/database/mysql/data.sql 
Warning: Using a password on the command line interface can be insecure.
</pre>

###3.2 **安装zabbix**
<pre>
useradd -u 700 -s /sbin/nologin -M zabbix
yum install gcc gcc-c++ glibc  libxml2-devel curl curl-devel net-snmp net-snmp-devel libssh2-devel OpenIPMI-devel  -y

cd /application/tools/zabbix-3.0.3/
./configure --prefix=/usr/local/zabbix --enable-server --enable-agent --with-net-snmp --with-libcurl --enable-proxy --with-mysql=/usr/local/mysql/bin/mysql_config --enable-java --enable-ipv6 --with-libxml2

make install

[root@zabbix-server zabbix-3.0.3]# ln -s /usr/local/zabbix/sbin/* /usr/local/sbin/
[root@zabbix-server zabbix-3.0.3]# ln -s /usr/local/zabbix/bin/*  /usr/local/bin/
2.6 添加zabbix服务对应的端口
cat >>/etc/services << EOF
zabbix-agent    10050/tcp        #ZabbixAgent
zabbix-agent    10050/udp        #Zabbix Agent
zabbix-trapper  10051/tcp        #ZabbixTrapper
zabbix-trapper  10051/udp        #Zabbix TrapperEOF

 添加开机启动脚本
cp -a ./misc/init.d/fedora/core/zabbix_server /etc/init.d/
cp -a ./misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
sed -i 's@BASEDIR=/usr/local@&/zabbix@' /etc/rc.d/init.d/zabbix_server   就是把/usr/local 改为/usr/local/zabbix
sed -i 's@BASEDIR=/usr/local@&/zabbix@' /etc/rc.d/init.d/zabbix_agentd
chkconfig zabbix_server on
chkconfig zabbix_agentd on

chown -R zabbix.zabbix /usr/local/zabbix/
mkdir /logs/zabbix -p   日志地址
 chown -R zabbix.zabbix /logs/zabbix/
</pre>

###**3.3 配置zabbix**
<pre>
vim /usr/local/zabbix/etc/zabbix_server.conf
LogFile=/logs/zabbix/zabbix_server.log
 DebugLevel=3
PidFile=/usr/local/zabbix/zabbix_server.pid
DBHost=192.168.56.12
DBName=zabbix
DBUser=zabbix
DBPassword=123456
DBSocket= /var/lib/mysql/mysqld.sock
AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts
</pre>
###**3.4 配置zabbix agent**
<pre>
vim /usr/local/zabbix/etc/zabbix_agentd.conf
Include=/usr/local/zabbix/etc/zabbix_agentd.conf.d/
UnsafeUserParameters=1
</pre>

###**3.5 配置WEB站点**
<pre>
cd /application/tools/zabbix-3.0.3/
 cp -r frontends/php/* /data/www/
chown -R www.www /data/www/
wget https://www.dwhd.org/wp-content/uploads/2015/05/simkai.ttf -O /data/www/fonts/simkai.ttf  字体
</pre>

###**3.6 配置php.ini**
<pre>
################php.ini##########
vim /usr/local/php/etc/php.ini
max_execution_time =600
max_input_time =300
memory_limit =128M
post_max_size =32M
date.timezone =Asia/Shanghai
如果加载不了php.ini 是因为php.ini应该放在/application/php/lib下
[root@zabbix-server www]# /etc/init.d/php-fpm restart
Gracefully shutting down php-fpm . done
Starting php-fpm  done
[root@zabbix-server www]# /usr/local/nginx/sbin/nginx  -s reload
</pre>

###**3.7、zabbix agent 配置检查网站的活动连接数**
<pre>
[root@zabbix-server zabbix_agentd.conf.d]# pwd
/usr/local/zabbix/etc/zabbix_agentd.conf.d
[root@zabbix-server zabbix_agentd.conf.d]# cat a.conf 
UserParameter=nginx.active,/usr/bin/curl -s "http://192.168.56.12:8080/nginx_status" | grep 'Active' | awk '{print $NF}'

[root@zabbix-server ~]# zabbix_get -s 192.168.56.12 -p 10050 -k "nginx.active"
1
</pre>
