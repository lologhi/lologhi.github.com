---
layout:     post
title:      Install MySQL 5.7 on OSX El Capitan
date:       2015-10-25
summary:    Homebrew only allow you to install MySQL 5.6, but the new RC version give you access to Spatial features, let's upgrade!
categories: symfony2
tags: symfony2 form choicelist entity
---

Get the last version from Oracle website. As I'm writing, it's [`mysql-5.7.9-rc-osx10.10-x86_64.dmg`](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.9-osx10.10-x86_64.dmg) (it's a direct download link!). Then, install from the `.pkg` file located in the `.dmg` you downloaded.

{% include figure.html src="/images/MySQL-is-fat.png" caption="Yes 1.32GB once installed (and wrong OSX version naming)" %}

Once installed, if you try to connect to MySQL with `mysql -u root -p`, you'll have this error message: `ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.` You need to set a new password:

`SET PASSWORD = PASSWORD('my_super_complicated_password');`

No more error message, but still nothing if you try to connect to your localhost with `mysql -h 127.0.0.1 -u root -p` or if you listen with `telnet 127.0.0.1 3306`. To fix that you'll need to create a configuration file: `/etc/my.conf`. Here is a basic one :

{% highlight properties %}
[client]
port = 3306
socket = /tmp/mysql.sock

[mysqld]
bind-address = 127.0.0.1
port = 3306
socket = /tmp/mysql.sock
datadir = /usr/local/var/mysql
key_buffer_size = 16M
max_allowed_packet = 8M
log-error = /var/log/mysql/error.log
symbolic-links = 1
lower_case_table_names = 2

[mysqldump]
quick
{% endhighlight %}

Hope this can be help. Details about what you're now allowed to do on [MySQL Server blog](http://mysqlserverteam.com/mysql-5-7-and-gis-an-example/).
