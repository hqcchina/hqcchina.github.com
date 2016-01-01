--- 
layout: post
title: "Shell环境下生成文件并写内容"
wordpress_id: 10
wordpress_url: http://www.yunjing.me/?p=10
date: 2016-01-01 10:24:00 +08:00
category: linux
tags: 
 - linux
 - shell
---

### 楔子
在Linux的脚本中经常会碰到需要生成一个配置文件并写入指定的内容，如环境变量、本机IP地址等。本文描述两种方法用于CentOS下生成并写入配置文件的方式。

使用cat的方法

    ```
        $ mv /etc/php-fpm.d/www.conf /etc/php-fpm.d/www.conf.orig
        $ cat <<EOFFPMD> /etc/php-fpm.d/www.conf
        [www]
        listen = 127.0.0.1:9000
        listen.allowed_clients = 127.0.0.1
        user = groot
        group = groot 
        pm = dynamic
        pm.max_children = 50
        pm.start_servers = 6
        pm.min_spare_servers = 4
        pm.max_spare_servers = 8
        pm.max_requests = 200
        pm.status_path = /php-status
        request_terminate_timeout = 2m
        request_slowlog_timeout = 1m
        slowlog = /var/log/php-fpm/www-slow.log
        rlimit_files = 5000
        security.limit_extensions = .php .php3 .php4 .php5
        env[MYSQL_HOST]=$MYSQL_HOST
        env[MYSQL_PORT]=$MYSQL_PORT
        env[MYSQL_LOGIN]=$MYSQL_LOGIN
        env[MYSQL_PASSWORD]=$MYSQL_PASSWORD
        env[MYSQL_DBNAME]=$MYSQL_DBNAME
        php_admin_value[error_log] = /home/groot/logs/php_errors.log
        php_admin_flag[log_errors] = on
        php_admin_value[memory_limit] = 800M
        php_value[session.save_handler] = files
        php_value[session.save_path] = /tmp
        EOFFPMD
    ```

使用tee的方法

    ```
        $ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
        [dockerrepo]
        name=Docker Repository
        baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
        enabled=1
        gpgcheck=1
        gpgkey=https://yum.dockerproject.org/gpg
        EOF
    ```

