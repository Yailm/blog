title: 在archlinux下搭建lnmp环境
date: 2015-09-18 23:21:31
tags:
  - Webserver
  - Archlinux
---
本文主要根据[张宴的BLOG](http://zyan.cc)，我在centos虚拟机搭建成功后，想想也在archlinux练练，把搭建的笔记整理了一下。

这里lnmp是linux+nginx+mariadb+PHP，目前截止使用的软件都是最新的，archlinux也是保持滚动到最新

1.从源里直接安装一些软件

    $ sudo -s
    # pacman -S --noconfirm gcc autoconf libjpeg libpng freetype libxml2 zlib glibc glib2 bzip2 ncurses curl e2fsprogs krb5 libidn openssl pcre mhash openldap nss_ldap libmcrypt mariadb

这里`mariadb`数据库直接用源里的了

    # mkdir -p /data0/soft
    # cd /data0/soft
    # wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
    # wget http://nginx.org/download/nginx-1.8.0.tar.gz
    # wget "http://downloads.sourceforge.net/mcrypt/mcrypt-2.6.8.tar.gz?modtime=1194463373&big_mirror=0"
    # wget http://cn2.php.net/get/php-5.6.13.tar.gz/from/this/mirror
    # wget http://pecl.php.net/get/memcache-2.2.7.tgz

2.接下来就开始编译PHP的支持库

    # tar zxvf libiconv[TAB]
    # cd libiconv[TAB]
    # ./configure --prefix=/usr/local
    # make
    # make install
    # cd ..

    # tar zxvf mcrypt[TAB]
    # cd mcrypt[TAB]
    # /sbin/ldconfig
    # ./configure
    # make
    # make install
    # cd ..

`mcrypt`的编译有问题的，看下`/usr/lib/`中有没有libmhash和libmcrypt的库文件。

3.配置mariadb

创建mariadb的用户和数据库目录

    # groupadd mysql
    # useradd -g mysql mysql
    # mkdir -p /data0/data/mysql
    # mkdir -p /data0/logs/mysql
    # mkdir /data0/etc

创建数据表和配置文件

    # mysql_install_db --user=mysql --basedir=/usr --datadir=/data0/data/mysql
    # vim /data0/etc/my.cnf

my.cnf配置文件

    [client]
    character-set-server=utf8
    port=3306
    socket=/run/mysqld/mysqld.sock

    [mysqld]
    character-set-server=utf8
    user=mysql
    port=3306
    socket=/run/mysqld/mysqld.sock
    basedir=/usr
    datadir=/data0/data/mysql
    log-error=/data0/logs/mysql/err.log
    pid-file=/data0/data/mysql/mysql.pid
    open_files_limit=8192
    back_log=600
    max_connections=5000
    max_connect_errors=6000
    table-cache=4096
    external-locking=FALSE
    max_allowed_packet=32M
    sort_buffer_size=1M
    join_buffer_size=1M
    thread_cache_size=256
    #thread_concurrency=8
    query_cache_size=512M
    query_cache_limit=2M
    query_cache_min_res_unit=2K
    default_storage_engine=MyISAM
    thread_stack=192k
    transaction_isolation=READ-COMMITTED
    tmp_table_size=246M
    max_heap_table_size=246M
    long_query_time=3
    log_slave_updates
    log-bin=/data0/logs/mysql/binlog
    binlog_cache_size=4M
    binlog_format=MIXED
    #log
    #log_warngins
    max_binlog_size=1G
    relay-log-index=/data0/logs/mysql/relaylog
    relay-log-info-file=/data0/logs/mysql/relaylog
    relay-log=/data0/log/mysqlrelaylog
    expire_logs_days=30
    key_buffer_size=256M
    read_buffer_size=1M
    read_rnd_buffer_size=16M
    bulk_insert_buffer_size=64M
    myisam_sort_buffer_size=128M
    myisam_max_sort_file_size=10G
    myisam_repair_threads=1
    myisam_recover

    interactive_timeout=120
    wait_timeout=120

    skip-name-resolve
    #master-connect-retry=10
    slave-skip-errors=1032,1062,126,1114,1146,1048,1396

    #master-host=192.168.1.2
    #master-user=username
    #master-password=password
    #master-port=3306

    server-id=1

    innodb_additional_mem_pool_size=16M
    innodb_buffer_pool_size=512M
    innodb_data_file_path=ibdata1:512M:autoextend
    innodb_file_io_threads=4
    innodb_thread_concurrency=8
    innodb_flush_log_at_trx_commit=2
    innodb_log_buffer_size=16M
    innodb_log_file_size=128M
    innodb_log_files_in_group=3
    innodb_max_dirty_pages_pct=90
    innodb_lock_wait_timeout=120
    innodb_file_per_table=0

    #log-slow-queries=/data0/logs/mysql/slow.log
    #long_query_time=10

    [mysqldump]
    quick
    max_allowed_packet=32M

创建管理mariadb数据库的shell脚本

    # mkdir -p /data0/bin
    # vim /data0/bin/mysql

    #!/bin/sh
    mysql_username="admin"
    mysql_password="nimda"          //账号等下再添加
    mysql_port=3306

    function_start_mysql()
    {
        printf "Starting MySQL..."
        /bin/sh /usr/bin/mysqld_safe --defaults-file=/data0/etc/my.cnf 2>&1 >/dev/null &
        printf "Done!\n"
    }

    function_stop_mysql()
    {
        printf "Stoping MySQL..."
        /usr/bin/mysqladmin -u ${mysql_username} -p${mysql_password} -S /run/mysqld/mysqld.sock shutdown
        printf "Done!\n"
    }

    function_restart_mysql()
    {
        printf "Restarting MySQL..."
        function_stop_mysql
        sleep 5
        function_start_mysql
        printf "Done!\n"
    }

    function_kill_mysql()
    {
        printf "Killing MySQL..."
        kill -9 $(ps -ef | grep 'bin/mysqld' | grep ${mysql_port} | awk '{printf $3 " " $2}')
        printf "Done!\n"
    }

    if [ "$1" = "start" ]; then
        function_start_mysql
    elif [ "$1" = "stop" ]; then
        function_stop_mysql
    elif [ "$1" = "restart" ]; then
        function_start_mysql
    elif [ "$1" = "kill" ]; then
        function_kill_mysql
    else
        printf " Usage: /data0/bin/mysql {start|stop|restart|kill}\n"
    fi

设置好权限

    # chmod +x /data0/bin/mysql
    # chown -R mysql:mysql /data0/data/mysql

启动，确认没什么错误

    # /data0/bin/mysql start
    # cat /data0/logs/mysql/err.log

连接数据库，提示password直接回车

    # mysql -u root -p

输入SQL语句，创建刚才脚本中设置的admin用户

    > GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' IDENTIFIED BY 'nimda';
    > GRANT ALL PRIVILEGES ON *.* TO 'admin'@'127.0.0.1' IDENTIFIED BY 'nimda';

停止MySQL

    # /data0/bin/mysql stop         //没有错误说明用户设置成功了

4.编译安装PHP


    # tar zxvf php[TAB]
    # cd php[TAB]
    # ./configure 
        --prefix=/usr --with-config-file-path=/data0/etc 
        --with-mysql=/usr --with-mysqli=/usr/bin/mysql_config 
        --with-iconv-dir=/usr/local --with-freetype-dir 
        --with-jpeg-dir --with-png-dir 
        --with-zlib --with-libxml-dir=/usr 
        --enable-xml --disable-rpath 
        --enable-bcmath --enable-shmop 
        --enable-sysvsem --enable-inline-optimization 
        --with-curl --enable-mbregex 
        --enable-fpm --enable-mbstring 
        --with-mcrypt --with-gd --enable-gd-native-ttf 
        --with-openssl --with-mhash 
        --enable-pcntl --enable-sockets 
        --with-ldap --with-ldap-sasl 
        --with-xmlrpc --enable-zip 
        --enable-soap --enable-opcache 
        --with-pdo-mysql --enable-maintainer-zts
    # make ZEND_EXTRA_LIBS='-liconv'
    # make install
    # cp php.ini-development /data0/etc/php.ini
    # cd ..

编译memcache扩展模块

    # tar zxvf memcache[TAB]
    # cd memcache[TAB]
    # /usr/bin/phpize
    # ./configure   //我这里可以直接发现php-config，其他路径需加上--with-php-config=/path/to/php-config
    # make
    # make install

修改php.ini文件

查找/data0/etc/php.ini中的extension_dir="./"，修改为

    extension_dir="/usr/lib/php/extensions/no-debug-zts-20131226/"

再其后一行添加

    extension="memcache.so"

PHP现在已经不用eaccelerator来加速，而是使用内置的opcache，这里在php.ini中搜索opcache，直接修改相应的配置就行，这里我设置成这样

    [opcache]

    zend_extension="/usr/lib/php/extensions/no-debug-zts-20131226/opcache.so"

    ; Determines if Zend OPCache is enabled
    opcache.enable=1

    ; Determines if Zend OPCache is enabled for the CLI version of PHP
    opcache.enable_cli=1

    ; The OPcache shared memory storage size.
    opcache.memory_consumption=128

    ; The amount of memory for interned strings in Mbytes.
    opcache.interned_strings_buffer=8

    ; The maximum number of keys (scripts) in the OPcache hash table.
    ; Only numbers between 200 and 100000 are allowed.
    opcache.max_accelerated_files=4000

    ; How often (in seconds) to check file timestamps for changes to the shared
    ; memory storage allocation. ("1" means validate once per second, but only
    ; once per request. "0" means always validate)
    opcache.revalidate_freq=60

    ; If enabled, a fast shutdown sequence is used for the accelerated code
    opcache.fast_shutdown=1

创建www用户和组


    # mkdir /data0/www
    # groupadd www
    # useradd -g www www
    # chown -R www:www /data0/www

配置php-fpm.conf文件（可以平滑变更php.ini而不用重启cgi)，参照了张宴的设置


    # vim /usr/etc/php-fpm.conf

    [global]
    pid = /data0/logs/php-fpm/php-fpm.pid
    error_log = /data0/logs/php-fpm/err.log
    log_level = notice
    emergency_restart_threshold = 10
    emergency_restart_interval = 1m
    process_control_timeout = 5s
    daemonize = yes

    [www]
    user = www
    group = www
    listen = 127.0.0.1:9000
    listen.backlog = 65535
    listen.mode = 0660
    listen.allowed_clients = 127.0.0.1
    pm = dynamic
    pm.max_children = 128
    pm.start_servers = 20
    pm.min_spare_servers = 5
    pm.max_spare_servers = 35
    pm.max_requests = 1024
    slowlog = log/$pool.log.slow
    request_slowlog_timeout = 0s
    request_terminate_timeout = 0
    rlimit_files = 65535
    rlimit_core = 0
    catch_workers_output = yes
    env[HOSTNAME] = $HOSTNAME
    env[PATH] = /usr/local/bin:/usr/bin:/bin
    env[TMP] = /tmp
    env[TMPDIR] = /tmp
    env[TEMP] = /tmp

尝试启动php-fpm
    # ulimit -SHn 65535
    # php-fpm

5.安装Nginx

    # mkdir /data0/etc/nginx
    # mkdir /data0/logs/nginx
    # mkdir -p /data0/tmp/nginx
    # tar zxvf nginx[TAB]
    # cd nginx[TAB]
    # ./configure 
        --prefix=/usr --conf-path=/data0/etc/nginx/nginx.conf 
        --error-log-path=/data0/logs/nginx/err.log --http-log-path=/data0/logs/nginx/access.log 
        --pid-path=/data0/logs/nginx/nginx.pid --lock-path=/tmp/nginx.lock 
        --user=www --group=www 
        --with-ipv6 --with-http_ssl_module 
        --with-http_spdy_module --with-http_gzip_static_module 
        --with-http_stub_status_module --http-client-body-temp-path=/data0/tmp/nginx/client_body 
        --http-proxy-temp-path=/data0/tmp/nginx/proxy --http-fastcgi-temp-path=/data0/tmp/nginx/fastcgi 
        --http-uwsgi-temp-path=/data0/tmp/nginx/uwsgi --http-scgi-temp-path=/data0/tmp/nginx/scgi
    # make & make install
    # cd ..

加了一些没用到的参数，但我相信以后可以用到

如果nginx已经安装了，就不要make install，执行下面语句升级，配置文件也不会丢失

    # rm -rf /usr/sbin/nginx
    # cp objs/nginx /usr/sbin/nginx
    # make upgrade

配置目录权限


    # chmod +w /data0/logs/nginx
    # chown -R www:www /data0/logs/nginx
    # chown -R www:www /data0/tmp/nginx

修改配置文件nginx.conf


    # vim /data0/etc/nginx/nginx.conf

    user    www www;
    worker_processes  8;

    error_log   /data0/logs/nginx/err.log   crit;
    worker_rlimit_nofile    65535;

    events {
        use     epoll;
        worker_connections  65535;
    }

    http {
        include       mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log /data0/logs/nginx/access.log  main;

        server_names_hash_bucket_size    128;
        client_header_buffer_size   32k;
        large_client_header_buffers     4 32k;
        client_max_body_size    8m;

        sendfile        on;
        tcp_nopush     on;

        keepalive_timeout  60;

        tcp_nodelay on;

        fastcgi_connect_timeout 300;
        fastcgi_send_timeout    300;
        fastcgi_read_timeout    300;
        fastcgi_buffer_size     64k;
        fastcgi_buffers     4   64k;
        fastcgi_busy_buffers_size   128k;
        fastcgi_temp_file_write_size    128k;

        gzip  on;
        gzip_min_length     1k;
        gzip_buffers    4   16k;
        gzip_http_version   1.0;
        gzip_comp_level     2;
        gzip_types      text/plain application/x-javascript text/css application/xml;
        gzip_vary       on;

        server {
            listen       80;
            server_name  localhost;
            index index.html index.htm index.php;
            root /data0/www;

            #error_page  404              /404.html;

            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            location ~ .*\.(php|php5)?$ {
                fastcgi_pass   127.0.0.1:9000;
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
                include        fastcgi_params;
            }

            location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
                expires 30d;
            }

            location ~ .*\.(js|css)?$ {
                expires 1h;
            }

            location /status/ {
                stub_status on;
                access_log  off;
            }
        }
    }

最后启动nginx

    # ulimit -SHn 65535
    # nginx

