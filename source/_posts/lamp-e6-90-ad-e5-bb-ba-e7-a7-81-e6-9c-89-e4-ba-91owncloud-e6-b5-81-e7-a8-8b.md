---
title: LAMP搭建私有云OwnCloud流程
url: 202.html
id: 202
categories:
  - 应用
date: 2018-10-13 13:03:58
tags:
---

STEP1: 在Ubuntu 16.04搭建LAMP
--------------------------

### Apache installation

To install webserver i.e. Apache, open terminal & execute the following command,

`$ sudo apt-get install apache2`

Once the package has been installed, start the apache service & enable it for boot,

`$ sudo systemctl start apache2`

`$ sudo systemctl enable apache2`

Now we can test the apache installation. To test the apache installation, open web browser & in the address bar enter the following url,

`http://localhost`

### MySQL installation

Next thing to install is database, we are using MySql as our choice of database server. To install MySql on Ubuntu machine, run the following command from terminal

`$ sudo apt-get install mysql-server mysql-client`

During the MySql installation, we will be asked to provide ‘root’ password, provide a suitable password & than confirm it to complete the installation. Once the installation has been completed, start the MySql service & enable it for boot,

`$ sudo systemctl start mysql`

`$ sudo systemctl enable mysql`

We can now connect to the database to check the installation, to connect to MySql, use the following command,

`$ mysql -u root -p`

And than enter the root password that we provided at the time of installation to connect to mysql,

PHP installation
----------------

Dynamic content of a website is processed by PHP, its a general purpose programming language. To install php on the system, execute the following command from terminal,

`$ sudo apt-get install php7.0-mysql php7.0-curl php7.0-json php7.0-cgi libapache2-mod-php7.0 php7.0`

Once the isntallation has been complete, we will test the php. To do that, open

`$ sudo vim /var/www/html/test.php`

& enter the following content to it,

`<?  
phpinfo();  
?>`

Save the file & restart the apache service,

`$ sudo systemctl restart apache2`

Now open the browser & enter the following URL,

`http://localhost/test.php`

We should now see the following page with details about the installed PHP,

We know have our LAMP stack on Ubuntu OS ready to use, we can now deploy the dynamic websites on this stack.

STEP2: 搭建OwnCloud
-----------------

我们需要在自己的设备里安装这些包：

    $ sudo yum install php-mysql php-json php-xml php-mbstring php-zip php-gd curl php-curl php-pdo

安装 OwnCloud，我们现在需要在服务器上下载 OwnCloud 安装包。使用下面的命令从官方网站下载安装包（10.0.4）：

    $ wget https://download.owncloud.org/community/owncloud-10.0.4.tar.bz2

使用下面的命令解压：

    $ tar -xvf owncloud-10.0.4.tar.bz2

现在，将所有解压后的文件移动至 `/var/www/html`：

    $ mv owncloud/* /var/www/html

注意还有两个隐藏文件需要分别移动：

    $ mv owncloud/.htaccess /var/www/html
    $ mv owncloud/.user.ini /var/www/html

下一步，我们需要在 Apache 的配置文件 `apache2.conf` 上做些修改：

    $ sudo vim /etc/apache2/apache2.conf

更改下面的选项：

    AllowOverride All

保存该文件，并修改 OwnCloud 文件夹的文件权限：

    $ sudo chown -R www-data:www-data /var/www/html/
    $ sudo chmod 777 /var/www/html/config/

然后重启 Apache 服务器执行修改：

    $ sudo systemctl restart apache2

现在，我们需要在 MySQL 上创建一个数据库，保存来自 OwnCloud 的数据。使用下面的命令创建数据库owncloud和数据库用户ocuser：

    $ mysql -u root -p
    mysql > create database owncloud;
    mysql > GRANT ALL ON owncloud.* TO ocuser@localhost IDENTIFIED BY 'owncloud';
    mysql > flush privileges;
    mysql > exit

修改ocuser用户密码：

    $ mysql -u root -p
    mysql > use mysql; 
    mysql > UPDATE user SET authentication_string=PASSWORD("your password") WHERE user='ocuser';
    mysql > FLUSH PRIVILEGES;
    mysql > quit;

在浏览器中输入你的服务器IP地址，显示如下界面，填写配置，服务器端搭建，完成！

![](http://diaryfun.info/wp-content/uploads/2018/10/234816ylnpqg2azlxpnlqz.jpg)

去官网下载并安装客户端：https://owncloud.org/download/。

ownCloud，启动！

![](http://diaryfun.info/wp-content/uploads/2018/10/Snipaste_2018-10-13_12-49-40.png)

参考链接：

https://linuxtechlab.com/install-lamp-stack-on-ubuntu

https://linux.cn/article-9418-1.html

若有大佬对此文存在修改意见，或是配置过程中存在错误，请在评论区留言。