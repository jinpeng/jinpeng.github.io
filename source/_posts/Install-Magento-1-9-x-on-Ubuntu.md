title: Install Magento 1.9.x on Ubuntu
date: 2016-04-21 18:07:01
tags:
---
##Install Magento 1.9.x on Aliyun##

###Environment###

Aliyun, Ubuntu 14.04LTS, 101.200.167.187

###Install LNMP###

```bash
$ ssh root@101.200.167.187
# wget http://soft.vpser.net/lnmp/lnmp1.2-full.tar.gz
# tar zxf lnmp1.2-full.tar.gz
# cd lnmp1.2-full
# vim include/mysql.sh
```

Add InnoDB engine to MySQL installation script:

```
-DWITH_INNOBASE_STORAGE_ENGINE=1
```

Please notice the section, looks like:

```bash
Install_MySQL_56()
{
...
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_READLINE=1 -DWITH_SSL=bundled -DWITH_ZLIB=system -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 ${MySQL55MAOpt}
...
}
```

```bash
# ./install.sh lnmp
```

MySQL root password: xxxxx
MySQL 5.6.23
PHP 5.6.9
Memory Allocator TCMalloc
(after about 1 hour build and installation)

```
+------------------------------------------------------------------------+
|          LNMP V1.2 for Ubuntu Linux Server, Written by Licess          |
+------------------------------------------------------------------------+
|         For more information please visit http://www.lnmp.org          |
+------------------------------------------------------------------------+
|    lnmp status manage: lnmp {start|stop|reload|restart|kill|status}    |
+------------------------------------------------------------------------+
|  phpMyAdmin: http://IP/phpmyadmin/                                     |
|  phpinfo: http://IP/phpinfo.php                                        |
|  Prober:  http://IP/p.php                                              |
+------------------------------------------------------------------------+
|  Add VirtualHost: lnmp vhost add                                       |
+------------------------------------------------------------------------+
|  Default directory: /home/wwwroot/default                              |
+------------------------------------------------------------------------+
|  MySQL/MariaDB root password: xxxxx                          |
```

###Create Database###

Open browser http://101.200.167.187/phpmyadmin/index.php
Create MySQL user and database:
kiwitao / xxxxx (for starting from scratch)
magento / xxxxx (for Sample Data)

###Create VHost###

```bash
# lnmp vhost add
```

Virtualhost infomation:
Your domain: kiuitao.com
Home Directory: /home/wwwroot/kiuitao.com
Rewrite: other
Enable log: yes
Create database: no
Create ftp account: no

###Install Magento 1.9.2.4###

```
# wget http://www.magentocommerce.com/downloads/assets/1.9.2.4/magento-1.9.2.4.tar.gz
# wget http://www.magentocommerce.com/downloads/assets/1.9.1.0/magento-sample-data-1.9.1.0.tar.gz
# tar zxf magento-1.9.2.4-2016-02-23-06-04-07.tar.gz -C /home/wwwroot/kiuitao.com
# cd /home/wwwroot/kiuitao.com
# mv magento/* .
# mv magento/.htaccess .
# mv magenta/.htaccess.sample .
# rmdir magento
# chown -R www:www /home/wwwroot/kiuitao.com
```

Open browser http://kiuitao.com/
Follow the guide on pages to finish setup, (skip Base URL check if it failed.)

Admin Account:
kvtadmin / xxxxx

encryption key:
xxxxxxxxxx

###Install Magento Sample Data 1.9.1.0###

```bash
# tar jxf magento-sample-data-1.9.1.0.tar-2015-02-11-09-56-42.bz2
# cd magento-sample-data-1.9.1.0
# mysql -u magento -p -h localhost magento < magento_sample_data_for_1.9.1.0.sql
# cp -R media /home/wwwroot/kiuitao.com/
# cp -R skin /home/wwwroot/kiuitao.com/
```

Open browser http://kiuitao.com/
Follow the guide on pages to finish setup, (skip Base URL check if it failed.)

Admin Account:
kvtadmin / xxxxx

encryption key:
xxxxxxxxxx

