# Nginx + PHP + MariaDB

## Nginx에서 PHP-fpm로의 PHP파일 전달 과정

NGINX와 PHP 연동하기전 NGINX가 PHP파일을 사용자에게 요청받았을때 어떻게 처리하고 PHP-FPM에게 전달하는 과정을 알아야 한다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/996813445EFDA68F14.png](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/996813445EFDA68F14.png)

1. NGINX에 PHP파일을 요청받게되면 PHP 처리가 정의 되어 있는 LOCATION 안에 제어문을 처리하게 된다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/9962E5425EFDA8BD13.png](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/9962E5425EFDA8BD13.png)

2. LOCATION 제어문 안에 있는 fastcgi_param 지시어로 $document_root(웹사이트 경로)변수와 $fastcgi_script_name(사용자가 요청한 파일)변수를 SCRIPT_FILENAME 변수에 넣어주게 된다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/99542A4C5EFDAA7E16.png](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/99542A4C5EFDAA7E16.png)

3. 처리해야될 파일에 위치 담아있는 SCRIPT_FILENAME 변수에 fastcgi_pass 지시어가 있는 접속경로로 PHP-FPM에게 전달하게 된다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/9923814A5EFDAC471F.png](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/9923814A5EFDAC471F.png)

fastcgi_pass 지시어로 SCRIPT_FILENAME 에 처리해야될 파일을 전달 받은 PHP-FPM는 PHP가 실행할 수 있도록 한다.

## NGINX에서 FastCGI를 통해 PHP-FPM 연동

# **MariaDB 10.1 설치**

**MariaDB Repo 설치**

```
# vim /etc/yum.repos.d/MariaDB.repo

[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=0
enabled=1
```

MariaDB 설치

```
# yum install -y mariadb mariadb-server
# systemctl start mariadb
```

Mariadb 보안설정

```
# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:            <-- MariaDB의 root계정 비밀번호 생성
Re-enter new password:   <-- 비밀번호 재입력
Password updated successfully!
Reloading privilege tables..
 ... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

부팅시 자동으로 실행

```
# systemctl enable mariadb
# systemctl status mariadb
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since 금 2017-06-02 19:37:14 KST; 3min 13s ago
 Main PID: 2933 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─2933 /usr/sbin/mysqld

 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986504....
 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986504....
 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986504...t
 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986504...9
 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986504....
 6월 02 19:37:13 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:13 13986428...d
 6월 02 19:37:14 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:14 13986504....
 6월 02 19:37:14 blog.dazzii.com mysqld[2933]: 2017-06-02 19:37:14 13986504....
 6월 02 19:37:14 blog.dazzii.com mysqld[2933]: Version: '10.1.24-MariaDB'  ...r
 6월 02 19:37:14 blog.dazzii.com systemd[1]: Started MariaDB database server.
Hint: Some lines were ellipsized, use -l to show in full.
```

Nginx 1.13.0 설치

**Nginx Repo 설치**

```
# vim /etc/yum.repos.d/nginx.repo

[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
```

**Nginx 설치**

```
# yum update
# yum install nginx
```

**부팅시 자동으로 실행**

```
# systemctl start nginx
# systemctl enable nginx
# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since 금 2017-06-02 22:14:50 KST; 18s ago
     Docs: http://nginx.org/en/docs/
 Main PID: 29892 (nginx)
   CGroup: /system.slice/nginx.service
           ├─29892 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx....
           └─29893 nginx: worker process

```

## PHP7.1 설치

**EPEL 저장소 구성**

```
# yum -y install wget
# wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
# rpm -Uvh remi-release-7.rpm epel-release-latest-7.noarch.rpm
```

**EPEL 저장소 활성화**

```
# yum -y install yum-utils
# yum --enablerepo=remi update remi-release
# yum-config-manager --enable remi-php71
```

**php7.1 설치**

```
# yum --enablerepo=remi-php71 install -y php php-fpm php-mysql
# yum --enablerepo=remi-php71 install -y php-mbstring php-common php-cli php-gd php-json php-devel php-imageic php-mcrypt php-xml php-xmlrpc php-soap php-bcmatch php-pear php-dba php-pdo php-ldap php-mysqlnd php-opcache
# yum --enablerepo=remi-php71 install -y zip unzip php-zip
```

**부팅시 자동으로 실행**

```
# systemctl start php-fpm
# systemctl enable php-fpm
# systemctl status php-fpm
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; enabled; vendor preset: disabled)
   Active: active (running) since 금 2017-06-02 22:44:37 KST; 38s ago
 Main PID: 30393 (php-fpm)
   Status: "Processes active: 0, idle: 5, Requests: 0, slow: 0, Traffic: 0req/sec"
   CGroup: /system.slice/php-fpm.service
           ├─30393 php-fpm: master process (/etc/php-fpm.conf)
           ├─30394 php-fpm: pool www
           ├─30395 php-fpm: pool www
           ├─30396 php-fpm: pool www
           ├─30397 php-fpm: pool www
           └─30398 php-fpm: pool www
```

# **PHP, NGINX 설정파일 수정**

**php.ini 파일 수정**

```
# vim /etc/php.ini

cgi.fix_pathinfo = 0       <-- 주석(;)을 제거하고 1을 0으로 변경
allow_url_fopen = On
expose_php = Off
display_errors = Off
date.timezone = Asia/Seoul <-- 주석(;)을 제거하고 Asia/Seoul 입력
```

**php-fpm 설정**

```
# vim /etc/php-fpm.d/www.conf

user = nginx          <-- apache를 nignx로 변경
group = nginx         <-- apache를 nignx로 변경

listen.owner = nginx  <-- 주석(;) 제거하고 nobody를 nginx로 변경
listen.group = nginx  <-- 주석(;) 제거하고 nobody를 nginx로 변경
listen = /var/run/php-fpm/php-fpm.sock   <-- listen = 127.0.0.1:9000 찾아서변경
```

**nginx.conf 수정**

```
# vim /etc/nginx/nginx.conf

user nginx;
worker_processes auto;
include /etc/nginx/conf.d/default.conf;
```

nginx 설정 파일 구조는 events{ }과 http { } 구조로 되어있고, http { } 안에서 Server Block 설정 부분인 /etc/nginx/conf.d/default.com 파일을 include 하여 읽어 들여는 구조로 되어 있다.

**웹서버 루트 디렉토리 사용자 변경**

```
# ls -ld /var/www/html
drwxr-xr-x. 2 root root 4096  6월  3 01:48 /var/www/html

# chown -R nginx:nginx /var/www/html
# ls -ld /var/www/html
drwxr-xr-x. 2 nginx nginx 4096  6월  3 01:48 /var/www/html
```

# **Server 구성**

**Server Block 설정파일 수정**

Server Block 파일 원본에 주석처리된 내용을 삭제 하고 난 원본 파일(/etc/nginx/conf.d/default.conf)

```
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

아래와 같이 수정

```
# cp /etc/nginx/conf.d/default.conf default.conf.bak
 ; server block 설정 파일 백업

# vim /etc/nginx/conf.d/default.conf
 ; 아래와 같이 수정
server {
    listen       80;
    server_name  localhost;  <-- 자신의 서버 도메인 주소
    root /var/www/html;            <-- 기존 루트 폴더(CentOS 7의 Default)
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
    }
}
```

CentOS의 Default 루트디렉토리는 /var/www/html 이다. 만약 루트 디렉토리를 변경 할려면 예들들어 루트 디렉토리를 /home/www/html로 설정하고 싶으면 root /home/www/html로 변경하면된다. 단 루트 디렉토리를 변경하게 되면 CentOS의 보안 시스템인 SELinux 때문에 외부에서 웹 서비스 접속이 불가능하다. 이것을 해결하는두가지 방법이 있는데 하나는 SELinux 기능을 해제 하거나 SELinux 보안 켁스트를 변경 하면 된다. 이 방법은 다음 포스트를 참고 해서 수정 하면 된다.

수정이 완료 되면 아래와 같이 nginx와 php-fpm 을 재시작 한다.

```
# systemctl restart nginx
# systemctl restart php-fpm
```

이제 웹브라우저에서 접속해보면 아래와 같이 nginx 웹 서버에 정상적으로 접속이 된다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9.jpg](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9.jpg)

이제 php 파일을 정상적으로 처리 하는지 확인하보기 위해 아래와 같이 파일을 만든다.

```
vi /var/www/html/info.php
 ; 아래 입력

<?php phpinfo(); ?>
```

이제 웹브라우저에서 http://도메인주소/info.php 를 입력

정상적으로 실행이 되면 아래와 같이 출력 된다.

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/1.jpg](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9/1.jpg)

mariadb 연동 확인

![Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9%201.jpg](Nginx%20+%20PHP%20+%20MariaDB%20d5ca2581d3e340238b20e5ca053d1aa9%201.jpg)