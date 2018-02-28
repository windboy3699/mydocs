### 所需程序安装
sudo -s  
LANG=C  
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers

### 下载相关源码
mkdir -p /data0/software  

wget http://nginx.org/download/nginx-1.8.1.tar.gz  
wget http://am1.php.net/distributions/php-7.0.11.tar.gz  
wget https://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.39.tar.gz  (官网下载source code)  
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz  
wget http://nchc.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz  
wget http://nchc.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz  
wget http://pecl.php.net/get/memcache-3.0.8.tgz  
wget http://nchc.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz  
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz  
wget https://github.com/eaccelerator/eaccelerator/tarball/master  
wget http://pecl.php.net/get/PDO_MYSQL-1.0.2.tgz  
wget http://blog.zyan.cc/soft/linux/nginx_php/imagick/ImageMagick.tar.gz  
wget http://pecl.php.net/get/imagick-3.4.2.tgz  

wget http://download.redis.io/releases/redis-4.0.7.tar.gz


tar zxvf libiconv-1.14.tar.gz  
cd libiconv-1.14/  
./configure --prefix=/usr/local  
make  
make install  

```
如果报错：
_GL_WARN_ON_USE (gets, “gets is a security hole - use fgets instead”); 这一行替换成  
 #if defined(__GLIBC__) && !defined(__UCLIBC__) && !__GLIBC_PREREQ(2, 16)  
_GL_WARN_ON_USE (gets, "gets is a security hole - use fgets instead");  
 #endif  
```

tar zxvf libmcrypt-2.5.8.tar.gz  
cd libmcrypt-2.5.8/  
./configure  
make  
make install  
/sbin/ldconfig  
cd libltdl/  
./configure --enable-ltdl-install  
make  
make install  

tar zxvf mhash-0.9.9.9.tar.gz  
cd mhash-0.9.9.9/  
./configure  
make  
make install  

ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la  
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so  
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4  
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8  
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a  
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la  
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so  
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2  
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1  
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config  
ln -s /usr/lib64/libldap.so /usr/lib/libldap.so  
ln -s /usr/local/webserver/mysql/lib/libmysqlclient.so /usr/lib/libmysqlclient.so  
ln -s /usr/local/webserver/mysql/lib/libmysqlclient.so.20 /usr/lib/libmysqlclient.so.20
ln -s /usr/lib64/liblber* /usr/lib/

tar zxvf mcrypt-2.6.8.tar.gz  
cd mcrypt-2.6.8/  
/sbin/ldconfig  
./configure  
make  
make install  
cd ../  

### 安装cmake  
yum install -y gcc gcc-c++ make automake  
yum install -y wget  
wget http://www.cmake.org/files/v2.8/cmake-2.8.10.2.tar.gz  
tar -zxvf cmake-2.8.10.2.tar.gz  
cd cmake-2.8.10.2  
./bootstrap  
gmake  
gmake install  

### 安装MySQL

5.7以上版本在阿里云主机上会因为内存不足通不过，低配建议使用5.7以下版本

/usr/sbin/groupadd mysql  
/usr/sbin/useradd -g mysql mysql  

tar zxvf mysql-5.6.39.tar.gz  
cd mysql-5.6.39  
cmake \  
-DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql \  
-DMYSQL_DATADIR=/data0/mysql/3306/data \  
-DWITH_MYISAM_STORAGE_ENGINE=1 \  
-DWITH_INNOBASE_STORAGE_ENGINE=1 \  
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \  
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \  
-DENABLED_LOCAL_INFILE=1 \  
-DDEFAULT_CHARSET=utf8 \  
-DDEFAULT_COLLATION=utf8_general_ci \  
-DEXTRA_CHARSETS=all \  
-DMYSQL_TCP_PORT=3306 \  
-DMYSQL_UNIX_ADDR=/tmp/mysqld.sock \  
-DWITH_DEBUG=0

make  
make install  

```
如果报错：CMake Error at cmake/boost.cmake:81 (MESSAGE)    
1.在/usr/local下创建一个名为boost的文件夹  
    mkdir -p /usr/local/boost  
2.进入这个新创建的文件夹然后下载boost  
    wget http://www.sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz  
3.解压  
    tar -xvzf boost_1_59_0.tar.gz  
4.继续cmake，添加上红色部分  
    -DWITH_BOOST=/usr/local/boost  
```
chmod +w /usr/local/webserver/mysql  
chown -R mysql:mysql /usr/local/webserver/mysql

mkdir -p /data0/mysql/3306/data/  
mkdir -p /data0/mysql/3306/binlog/  
mkdir -p /data0/mysql/3306/relaylog/  
chown -R mysql:mysql /data0/mysql/  

cp /usr/local/webserver/mysql/support-files/mysql.server /etc/init.d/mysqld  
chmod +x /etc/init.d/mysqld   
chkconfig --add mysqld   
chkconfig mysqld on  

vim /etc/my.cnf 

```
[client]
default-character-set = utf8
port = 3306
socket = /tmp/mysql.sock

[mysqld]
user = mysql
port = 3306
character-set-server = utf8
default_storage_engine = InnoDB
basedir = /usr/local/webserver/mysql
datadir = /data0/mysql/3306/data
socket = /tmp/mysql.sock
pid-file = /data0/mysql/3306/mysql.pid
log_error = /data0/mysql/3306/mysql-error.log
slow_query_log_file = /data0/mysql/3306/mysql-slow.log
```
/usr/local/webserver/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/webserver/mysql --datadir=/data0/mysql/3306/data

/etc/init.d/mysqld start

修改密码  
/usr/local/webserver/mysql/bin/mysql -u root -p -S /tmp/mysql.sock  
GRANT ALL PRIVILEGES ON \*\.\* TO 'admin'@'localhost' IDENTIFIED BY '12345678';  
GRANT ALL PRIVILEGES ON \*\.\* TO 'admin'@'127.0.0.1' IDENTIFIED BY '12345678';

### PHP安装

vim /etc/ld.so.conf
增加 /usr/local/lib64
ldconfig -v 

tar zxvf php-7.0.11.tar.gz  
cd php-7.0.11  
./configure --prefix=/usr/local/webserver/php --with-config-file-path=/usr/local/webserver/php/etc --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv-dir=/usr/local --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-cgi --enable-fpm --enable-mbstring --with-mcrypt --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-ldap=shared --with-ldap-sasl --with-xmlrpc --enable-zip --enable-soap  
make ZEND_EXTRA_LIBS='-liconv'  
make install  
cp php.ini-development /usr/local/webserver/php/etc/php.ini  

#### 安装PHP扩展

memcache 由于php7兼容问题需要下载memcache分支安装  
wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz  
tar -zxf libmemcached-1.0.18.tar.gz  
cd libmemcached-1.0.18/  
./configure --prefix=/usr/local/libmemcached  
make && make install  

git clone https://github.com/php-memcached-dev/php-memcached.git  
cd php-memcached/  
git fetch origin php7:php7  
git checkout php7  

/usr/local/webserver/php/bin/phpize  
./configure --disable-memcached-sasl --with-libmemcached-dir=/usr/local/libmemcached --with-php-config=/usr/local/webserver/php/bin/php-config  
make && make install

memcache & redis扩展参考：http://www.cnblogs.com/zqifa/p/linux-php-2.html

yum -y install ImageMagick

yum install ImageMagick-devel

tar zxvf imagick-3.4.2.tgz  
cd imagick-3.4.2  
/usr/local/webserver/php/bin/phpize  
./configure --with-php-config=/usr/local/webserver/php/bin/php-config  
make  
make install


修改php.ini文件  
查找/usr/local/webserver/php/etc/php.ini中的extension_dir = "./"  
修改为extension_dir = "/usr/local/webserver/php/lib/php/extensions/no-debug-non-zts-20151012/"  
并在此行后增加以下几行，然后保存  
extension = memcached.so  
extension = imagick.so  

再查找output_buffering = Off  
修改为output_buffering = On  

再查找; cgi.fix_pathinfo=0  
修改为cgi.fix_pathinfo=0，防止Nginx文件类型错误解析漏洞。

/usr/sbin/groupadd www  
/usr/sbin/useradd -g www www

新建www目录  
mkdir -p /data0/htdocs/www  
chmod +w /data0/htdocs/www  
chown -R www:www /data0/htdocs/www

cp /usr/local/webserver/php/etc/php-fpm.conf.default /usr/local/webserver/php/etc/php-fpm.conf

include=/usr/local/webserver/php/etc/php-fpm.d/*.conf 改成  
include=/usr/local/webserver/php/etc/php-fpm.d/www.conf.default

修改/usr/local/webserver/php/etc/php-fpm.d/www.conf.default 找到security.limit_extensions把他修改为： 
security.limit_extensions=.php .html .js .css .jpg .jpeg .gif .png .htm #常用的文件扩展名，否则这些后缀文件无法访问

启动php-fpm  
ulimit -SHn 65535  
/usr/local/webserver/php/sbin/php-fpm

### 安装Nginx

tar zxvf pcre-8.39.tar.gz  
cd pcre-8.39  
./configure  
make  
make install  

tar zxvf nginx-1.8.1.tar.gz  
cd nginx-1.8.1  
./configure --user=www --group=www --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module  
make  
make install

mkdir -p /data1/logs  
chmod +w /data1/logs  
chown -R www:www /data1/logs  

vim /usr/local/webserver/nginx/conf/nginx.conf

```
user  www www;

worker_processes  8;

error_log  /data1/logs/nginx_error.log;

pid        /usr/local/webserver/nginx/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    #gzip  on;

    server {
        listen       80;
        server_name  shop.flowphp.com;
        charset utf-8;
        access_log  /data1/logs/www.shop.access.log;

        location / {
            fastcgi_pass  127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
            root /data0/htdocs/demo/sites/shop-web;
            index  index.html index.htm index.php;

            if (!-e $request_filename){
                rewrite ^/(.*) /index.php last;
            }
        }

        location ~ .*\.(js|css|gif|jpg|jpeg|png|bmp|swf)$ {
            root /data0/htdocs/demo/sites/shop-cms;
            if (-f $request_filename) {
                expires 1d;
                break;
            }
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }

}

```

启动Nginx  
ulimit -SHn 65535  
/usr/local/webserver/nginx/sbin/nginx

测试配置文件  
/usr/local/webserver/nginx/sbin/nginx -t

重启  
/usr/local/webserver/nginx/sbin/nginx -s reload

### 安装Redis

cp ./redis-4.0.7.tar.gz /usr/local/webserver  
tar zxvf redis-4.0.7.tar.gz  
cd redis-4.0.7  
make  
make install  
vim redis.conf  daemonize no 改成 daemonize yes  
启动redis-server /usr/local/webserver/redis/redis.conf

### 参考资料
http://blog.zyan.cc/nginx_php_v6  
http://blog.csdn.net/wulantian/article/details/50832988  
http://www.php.net/manual/zh/configure.about.php  
http://blog.csdn.net/zgl_dm/article/details/8363843

---

git安装的时候，如果报错not found libcharset.so.1需要把/usr/local/lib下ln到/lib64下
