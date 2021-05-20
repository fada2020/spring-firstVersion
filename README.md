www.naver.com/jinsoo55
#goolge -> centos sudo 설정
#https://info-lab.tistory.com/163
su - root
visudo -f /etc/sudoers
root    ALL=(ALL)       ALL
bricks  ALL=(ALL)       ALL

#useradd bricks
#passwd bricks
vi /etc/group
root:x:0:bricks (bricks 계정추가)

#google->centos 자동꺼짐
setterm -blank 0  -powerdown 0


#google -> All mirror URLs are not using ftp, http[s] or file.
#https://chhanz.github.io/linux/2020/12/10/centos-6-yumrepo-error/

echo "https://vault.centos.org/6.10/os/x86_64/" > /var/cache/yum/x86_64/6/base/mirrorlist.txt
echo "https://vault.centos.org/6.10/extras/x86_64/" > /var/cache/yum/x86_64/6/extras/mirrorlist.txt
echo "https://vault.centos.org/6.10/updates/x86_64/" > /var/cache/yum/x86_64/6/updates/mirrorlist.txt

# install apache (2.2.15-69.el6.centos.x86_64)
yum -y install httpd

#google->how to open http port 80 in linux
#https://www.binarytides.com/open-http-port-iptables-centos/
#google->Error:Connection timed out: connect
#https://okky.kr/article/653163
#connection refused는 IP는 연결되나 port 연결 안된경우이고
#Connection timed out인경우는 대부분 서버 방화벽을 막아 버린 경우가 대부분입니다.
iptables -I INPUT 5 -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT 5 -i eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT 5 -i wlan0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT 5 -i wlan0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
service iptables save
iptables: 방화벽 규칙을 /etc/sysconfig/iptables에 저장 중: [  OK  ]

#install mysql-server (5.1.73-8.el6_8) dependencies:mysql, perl-DBD-MySQL(4.013-3.el6), perl-DBI(1.609-4.el6)
yum -y install mysql-server

#start apache, mysql 
service httpd start
service mysqld start

#mysql root password
/usr/bin/mysql_secure_installation
Set root password? [Y/n] Y
New password: YourDesiredPassword
Re-enter new password: YourDesiredPassword
Remove anonymous user? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y

#CodeIgniter
cd /var/www/html
wget https://github.com/bcit-ci/CodeIgniter/archive/3.0.5.zip
unzip 3.0.5.zip
http://127.0.0.1/CodeIgniter-3.0.5/

#php php-mysql(5.3.3-50.el6_10) dependencies(php-cli, php-common, php-pdo)
yum -y install php php-mysql
yum search php-

##Usually, you will need two modules: "php-mcrypt" and "php-mbstring". Install them with the following commands:
yum -y install php-mcrypt.x86_64
yum -y install php-mbstring.x86_64

##To get the stack functioning, you also need to set Apache and MySQL to run automatically when your VPS boots (PHP will run automatically with Apache):
chkconfig httpd on
chkconfig mysqld on

#check php
rpm -qa | grep php
php -v

##download source
yum -y install http://opensource.wandisco.com/centos/6/git/x86_64/wandisco-git-release-6-1.noarch.rpm
yum -y install git
cd /home/bricks
git config --global user.name 'jinsoo4ever'
git config --global user.email jinsoo4ever@gmail.com
git clone https://gitlab.com/jinsoo4ever/ebricks_php.git 
cp -R /home/bricks/ebricks_php/* /var/www/html/

#configure conf file
cd /var/www/html/

#/var/www/html 에 .htaccess 파일 추가
vi .htaccess
<IfModule mod_rewrite.c>
    RewriteEngine On
 RewriteBase /
 RewriteCond $1 !^(index\.php|images|captcha|data|include|uploads|robots\.txt)
 RewriteCond %{REQUEST_FILENAME} !-f
 RewriteCond %{REQUEST_FILENAME} !-d
 RewriteRule ^(.*)$ /index.php/$1 [L]
</IfModule>

#configure conf file
vi /etc/httpd/conf/httpd.conf
#ServerName www.example.com:80
ServerName 127.0.0.1:80
<Directory "/var/www/html">
 Options Indexes FollowSymLinks
 #AllowOverride None
 AllowOverride All
 Order allow,deny
 Allow from all
</Directory>
#DirectoryIndex index.html index.html.var
DirectoryIndex index.php

#change file mode
chmod -R 777 /var/www/html/

#restart apache
service httpd restart

#modifiy src
find /var/www/html/ -name "Main*.*"
vi /var/www/html/application/controllers/Main.php
find /var/www/html/ -name "Member*.*"
vi /var/www/html/application/controllers/Member.php

##google -> centos 6 apache enable https
#https://www.digitalocean.com/community/tutorials/how-to-create-a-ssl-certificate-on-apache-for-centos-6
yum -y install mod_ssl
mkdir /etc/httpd/ssl 

#Create a Self Signed Certificate
cd /etc/httpd/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/httpd/ssl/apache.key -out /etc/httpd/ssl/apache.crt
1) Enter PEM pass phrase: 1234
2) Country Name : KR
3) State or Province Name : Seoul
4) Locality Name : Mapo-gu  (Yangcheon-gu)
5) Organization Name : Sahoipyoungnon
6) Organizational Unit Name : Digital Business Team
7) Common Name : apache
8) Email Address : jinsoo@sapyoung.com
skipp 'extra' attritutes

#Set Up the Certificate
#google -> apache 2.2 ssl.conf download
#https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html
vi /etc/httpd/conf.d/ssl.conf
LoadModule ssl_module modules/mod_ssl.so
Listen 443
#<VirtualHost _default_:443>
<VirtualHost *:443>
    #ServerName www.example.com:443
    ServerName 127.0.0.1:443
    SSLEngine on
    #SSLCertificateFile /etc/pki/tls/certs/localhost.crt
    SSLCertificateFile /etc/httpd/ssl/apache.crt
    #SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
    SSLCertificateKeyFile /etc/httpd/ssl/apache.key
</VirtualHost> 

#Restart Apache
service httpd restart

#google -> php mysql 한글 인코딩
#https://luckyyowu.tistory.com/279
#google -> mysql 한글 깨짐
#https://nesoy.github.io/articles/2017-05/mysql-UTF8
#show variables;
vi /etc/my.cnf
[mysqld]
lower_case_table_names=2
collation-server = utf8_general_ci
init-connect='SET NAMES utf8'
character-set-server = utf8

#기존에 만든 데이터베이스가 있었다면 utf로 변경
ALTER DATABASE dbebricks_new DEFAULT CHARACTER SET utf8;  
show variables like 'c%';

#restart mysql
service mysqld restart

#google -> phpmailer gmail smtp connect() failed
#https://netcorecloud.com/tutorials/phpmailer-smtp-error-could-not-connect-to-smtp-host/#:~:text=There%20are%20many%20popular%20cases,you%20enable%20the%20extension%3Dphp_openssl.
getsebool httpd_can_sendmail
getsebool httpd_can_network_connect
sudo setsebool -P httpd_can_sendmail 1
sudo setsebool -P httpd_can_network_connect 1

#java
cd /home/bricks
mkdir jdk_1.8
wget https://cdn.azul.com/zulu/bin/zulu8.54.0.21-ca-jdk8.0.292-linux_x64.tar.gz
tar -zxvf zulu8.54.0.21-ca-jdk8.0.292-linux_x64.tar.gz -C /home/bricks/jdk_1.8/
mkdir -p /usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64
mv /home/bricks/jdk_1.8/zulu8.54.0.21-ca-jdk8.0.292-linux_x64/* /usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64/
cat /etc/profile
cd /etc/profile.d
vi custom.sh
JAVA_HOME=/usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64
export JAVA_HOME
CLASSPATH=.:$JAVA_HOME/lib/tools.jar
export CLASSPATH
PATH=$PATH:$JAVA_HOME/bin
export PATH
:x

#쉘스크립트(sh) 파일 실행
echo $PATH
/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
chmod +x custom.sh
./custom.sh
source ./custom.sh
echo $PATH

#java 버전 확인
which java
sudo alternatives --install /usr/bin/java java /usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64/bin/java 100
sudo alternatives --config java
java -version

#make logs directory
mkdir -p /var/www/html/logs
chmod -R 777 /var/www/html/logs

#php writeLog
function writeLog($msg)
{
    $file = "./logs/Member_".date("Ymd").".log";
    
    
    if(!($fp = fopen($path.$file, "a+"))) return 0;
    
    ob_start();
    print_r($msg);
    $ob_msg = ob_get_contents();
    ob_clean();

    $date = new DateTime();
    
    if(fwrite($fp, "[".date_format($date, 'Y-m-d H:i:s')."]".$ob_msg."\n") === FALSE)
    {
        fclose($fp);
        return "0";
    }
    fclose($fp);
    return "1";
}
public function func_1() {
    $data=array();
    $params = $this->input->post();
    $name = $params['name'];
    $this->writeLog($name);
}

#compile java
vi sh_compile_java.sh
#!/bin/sh
# A shell script
get_property()
{
  echo $(sed -n "s/.*<entry key=\"$1\">\([^<]*\)<\/entry>.*/\1/p" config/db_properties.xml)
}

root_dir=$(get_property "root_dir")

export CLASSPATH=.:/usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64/lib/tools.jar
export CLASSPATH=${CLASSPATH}:$root_dir/lib/activation-1.1.jar
export CLASSPATH=${CLASSPATH}:$root_dir/lib/log4j-api-2.8.2.jar
export CLASSPATH=${CLASSPATH}:$root_dir/lib/log4j-core-2.8.2.jar
export CLASSPATH=${CLASSPATH}:$root_dir/lib/mail-1.4.7.jar
export CLASSPATH=${CLASSPATH}:$root_dir/lib/mariadb-java-client-2.7.2.jar

export CLASSPATH=${CLASSPATH}:$root_dir/config
## log4j.profile 도 classpath 에 있어야 함
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/common/CommonException.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/common/CommonUtil.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/common/StaticConstant.java

javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/connector/ConnectorException.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/connector/DBConnector.java

javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/mailsender/AutomaticWithdrawal.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/mailsender/MailSenderException.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/mailsender/ScheduledToSwitchToDomantAccount.java
javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/mailsender/SwitchToDormantAccount.java

javac -cp $CLASSPATH $root_dir/kr/co/ebricks/dormant/controller/Manager.java
:x

chmod +x sh_compile_java.sh

#make jar file(1)
vi manifest.txt
Main-class: kr.co.ebricks.dormant.controller.Manager
Class-path: /html_docs/ebricks_2018/mail_to_dormant_account/lib/activation-1.1.jar /html_docs/ebricks_2018/mail_to_dormant_account/lib/log4j-api-2.8.2.jar /html_docs/ebricks_2018/mail_to_dormant_account/lib/log4j-core-2.8.2.jar /html_docs/ebricks_2018/mail_to_dormant_account/lib/mail-1.4.7.jar /html_docs/ebricks_2018/mail_to_dormant_account/lib/mariadb-java-client-2.7.2.jar /html_docs/ebricks_2018/mail_to_dormant_account/config
:x

#make jar file(2)
vi sh_make_jar.sh
/usr/lib/jvm/zulu8.54.0.21-ca-jdk8.0.292-linux_x64/bin/jar cvfm MailToDormantAccount.jar manifest.txt kr/co/ebricks/dormant/common/*.class kr/co/ebricks/dormant/connector/*.class kr/co/ebricks/dormant/controller/*.class kr/co/ebricks/dormant/mailsender/*.class
chmod +x sh_make_jar.sh
./sh_make_jar.sh

#run file
#google -> java manifest log4j2
#https://stackoverflow.com/questions/28574905/specifying-log4j2-configuration-file-when-using-executable-jar
vi sh_run_jar.sh
java -Dlog4j.configurationFile=/html_docs/ebricks_2018/mail_to_dormant_account/config/log4j2.xml -jar MailToDormantAccount.jar
chmod +x sh_run_jar.sh
chmod 777 -R /html_docs/ebricks_2018/mail_to_dormant_account

#create tables
mysql -h localhost -u root -p

create database dbebricks_new;
grant all privileges on dbebricks_new.* TO 'ebricks'@'localhost' identified by 'tkghlvudfhs!!';
flush privileges;
exit;
mysql -h localhost -u ebricks -p
show databases;
use dbebricks_new;

create table if not exists EBLOG_ADMIN_LOGIN (
  EAL_IDX                int(11)        NOT NULL   auto_increment  primary key
  , EAL_USERACOUNT varchar(50)   NOT NULL    
  , EAL_LOGINDATE   datetime      NOT NULL   
  , EAL_AGENT         varchar(300)  NOT NULL   
  , EAL_LOGINYN      enum('Y','N')  NOT NULL   default 'N'
  , EAL_IP               varchar(15)    NOT NULL 
  , key   EAL_USERACOUNT (EAL_USERACOUNT)
)
CHARACTER SET 'utf8' 
, COMMENT  '관리자 로그인 정보'
;


create table if not exists EBLOG_LOGIN (
  EL_IDX                 int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY
  , EL_USERACOUNT varchar(50) NOT NULL
  , EL_LOGINDATE   datetime NOT NULL
  , EL_AGENT          varchar(300) NOT NULL
  , EL_LOGINYN      enum('Y','N') NOT NULL DEFAULT 'N'
  , EL_IP                 varchar(15) NOT NULL
  , KEY EL_USERACOUNT (EL_USERACOUNT)
)
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT  '관리자 로그인 정보'
;



create table if not exists  EBLOG_LOGIN_GB (
  EL_IDX                 int(11) NOT NULL AUTO_INCREMENT 
  , EL_USERACOUNT varchar(50) NOT NULL
  , EL_LOGINDATE   datetime NOT NULL
  , EL_AGENT          varchar(300) NOT NULL
  , EL_LOGINYN       enum('Y','N') NOT NULL DEFAULT 'N'
  , EL_IP                  varchar(15) NOT NULL
  , PRIMARY KEY (EL_IDX) USING BTREE
  , KEY EL_USERACOUNT (EL_USERACOUNT) USING BTREE
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT  '다국어 로그인 정보'
;


create table if not exists EB_ADMIN_MEMBER (
  EAM_IDX                   int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY
  , EAM_USERACCOUNT varchar(50) NOT NULL
  , EAM_PWD                varchar(200) NOT NULL
  , EAM_USERNAME       varchar(100) NOT NULL
  , EAM_CREDATE          datetime NOT NULL
  , EAM_EMAIL              varchar(50) NOT NULL
  , EAM_HP                  varchar(50) NOT NULL
  , EAM_UPDDATE         varchar(100) 
  , EAM_REALYN           enum('Y','N') NOT NULL DEFAULT 'Y'
  , EAM_APPROVAL_YN  enum('Y','N') NOT NULL DEFAULT 'N'
  , EAM_IP                    varchar(15) NOT NULL
) 
CHARACTER SET 'utf8' 
, COMMENT  '관리자'
;


create table if not exists EB_ADMIN_MEMBER_20201125 (
  EAM_IDX                   int(11) NOT NULL AUTO_INCREMENT
  , EAM_USERACCOUNT varchar(50) NOT NULL
  , EAM_PWD               varchar(200) NOT NULL
  , EAM_USERNAME     varchar(100) NOT NULL
  , EAM_CREDATE        datetime NOT NULL
  , EAM_EMAIL           varchar(50) NOT NULL
  , EAM_HP                varchar(50) NOT NULL
  , EAM_UPDDATE      varchar(100) 
  , EAM_REALYN          enum('Y','N') NOT NULL DEFAULT 'Y'
  , EAM_APPROVAL_YN enum('Y','N') NOT NULL DEFAULT 'N'
  , EAM_IP                  varchar(15) NOT NULL
  , PRIMARY KEY (EAM_IDX) USING BTREE
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT  '관리자'
;


create table if not exists  EB_ADMIN_MEMBER_20201125_1454 (
  EAM_IDX                   int(11) NOT NULL AUTO_INCREMENT
  , EAM_USERACCOUNT varchar(50) NOT NULL
  , EAM_PWD                varchar(200) NOT NULL
  , EAM_USERNAME      varchar(100) NOT NULL
  , EAM_CREDATE         datetime NOT NULL
  , EAM_EMAIL             varchar(50) NOT NULL
  , EAM_HP                  varchar(50) NOT NULL
  , EAM_UPDDATE        varchar(100) 
  , EAM_REALYN           enum('Y','N') NOT NULL DEFAULT 'Y'
  , EAM_APPROVAL_YN enum('Y','N') NOT NULL DEFAULT 'N'
  , EAM_IP                  varchar(15) NOT NULL
  , PRIMARY KEY (EAM_IDX) USING BTREE
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT  '관리자'
;


create table if not exists EB_BANNER (
  EB_SEQ            int(11)            NOT NULL     auto_increment primary key
  , EB_TITLE         varchar(50)       COMMENT '제목'
  , EB_TYPE          enum('M','S')    default 'M'   COMMENT '타입 M메인'  
  , EB_LANGTYPE   enum('K','E')      default 'K'   COMMENT 'K한국어 E영어'
  , EB_FILEPATH     varchar(50)      COMMENT '경로'
  , EB_FILENAME    varchar(50)      COMMENT '파일명'
  , EB_OPTION       varchar(50)      COMMENT '옵션'
  , EB_FILENAME_M varchar(50)     COMMENT '모바일용'
  , EB_URL            varchar(200)     COMMENT 'URL'
  , EB_TARGET       enum('I','O')      default 'I'  COMMENT '외부여부'
  , EB_ORDERBY     int(11)            default 0   COMMENT '순번'
  , EB_CREDATE     datetime         NOT NULL  COMMENT '등록일'
  , EB_UPDDATE     datetime           
  , EB_REAL_YN      enum('Y','N')    NOT NULL   default 'Y'  COMMENT '삭제여부'
  , EB_DISPLAY_YN  enum('Y','N')     NOT NULL   default 'Y' COMMENT '전시여부'
  , EB_IP          varchar(15)   
)
CHARACTER SET 'utf8' 
, COMMENT  '배너'
;

insert into EB_BANNER 
(EB_TITLE, EB_CREDATE, EB_REAL_YN, EB_DISPLAY_YN)
value('배너6', curdate(), 'Y', 'Y');


create table if not exists EB_BOOK (
  BOOKSEQ int(10) unsigned NOT NULL AUTO_INCREMENT
  , MEMSEQ int(10) NOT NULL DEFAULT '0'
  , BRANDNAME varchar(50) DEFAULT ''
  , SKILL varchar(50) DEFAULT ''
  , SERIES varchar(50) DEFAULT ''
  , SERIES_YN enum('Y','N') DEFAULT 'Y'
  , READERS varchar(50) DEFAULT ''
  , TEXTBOOK varchar(50) DEFAULT ''
  , LEVEL_F varchar(50) DEFAULT ''
  , LEVEL varchar(50) DEFAULT ''
  , BOOKMAP_LEVEL varchar(50) DEFAULT ''
  , BOOKMAP_LEVEL_SUB varchar(50) DEFAULT ''
  , CEFR varchar(50) DEFAULT ''
  , BOOKPREVIEW varchar(50) DEFAULT ''
  , BOOKPRELISTENING varchar(50) DEFAULT ''
  , EBOOKVIEW varchar(50) DEFAULT '' COMMENT '이북'
  , EBOOKVIEW_CNT int(11) DEFAULT '0'
  , BOOKPATH varchar(50) DEFAULT ''
  , SERIESNO varchar(50) DEFAULT ''
  , BRICKSCODE varchar(50) DEFAULT ''
  , BARCODE varchar(50) DEFAULT ''
  , SALE varchar(50) DEFAULT ''
  , DATE varchar(10) DEFAULT ''
  , FIRSTDATE varchar(10) DEFAULT ''
  , PAGE int(10) DEFAULT '0'
  , BOOKSIZE varchar(50) DEFAULT ''
  , VOLUMNCODE varchar(50) DEFAULT ''
  , ADMINTEXT text
  , ADMIN_REVIEW text
  , WRITER varchar(255) DEFAULT ''
  , IMGFILE1 varchar(100) DEFAULT NULL
  , IMGFILE2 varchar(100) DEFAULT NULL
  , IMGFILE3 varchar(100) DEFAULT NULL
  , NEWSFILE varchar(100) DEFAULT NULL
  , REAL_YN char(1) DEFAULT 'Y'
  , COUNT int(10) DEFAULT '0'
  , UPDDATE datetime DEFAULT NULL
  , CREDATE datetime DEFAULT NULL
  , IP varchar(15) DEFAULT NULL
  , PRIMARY KEY (BOOKSEQ)
  , KEY SKILL_SERIES (SKILL,SERIES)
  , KEY READERS_TEXTBOOK_LEVEL_F_LEVEL_BOOKMAP_LEVEL (READERS,TEXTBOOK,LEVEL_F,LEVEL,BOOKMAP_LEVEL)
  , KEY BRANDNAME (BRANDNAME)
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT DYNAMIC
;


create table if not exists  EB_BOOKCONTENT_ENG (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(300) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(400) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(200) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL,
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  SEARCH_LEVEL varchar(50) DEFAULT NULL COMMENT '검색용레벨',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARACTER SET 'utf8' 
, COMMENT '다국어 자료'
;


create table if not exists  EB_BOOKCONTENT_ENG_BAK (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(50) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(50) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL,
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARACTER SET 'utf8' 
, COMMENT '다국어 자료'
;


create table if not exists EB_BOOKCONTENT_ENG_copy_0712 (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(200) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(200) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL,
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT '다국어 자료'
;


create table if not exists  EB_BOOKCONTENT_ENG_copy_replce (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(50) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(50) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL,
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST_OLDTEST mediumtext,
  BOOKLIST mediumtext,
  BOOKCONTENTS_OLDTEST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARACTER SET 'utf8' 
, ROW_FORMAT COMPACT
, COMMENT '다국어 자료'
;


create table if not exists EB_BOOKCONTENT_ENG_copy_test (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(200) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(400) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(200) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL,
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARSET utf8 
, ROW_FORMAT COMPACT 
, COMMENT '다국어 자료'
;


create table if not exists EB_BOOKCONTENT_KOR (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(300) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(400) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(200) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  SEARCH_LEVEL varchar(50) DEFAULT NULL COMMENT '검색용레벨',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext,
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARSET utf8 
, COMMENT='한국 자료'
;


create table if not exists EB_BOOKCONTENT_KOR_BAK (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(50) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(50) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARSET utf8 
, COMMENT '한국 자료'
;


create table if not exists EB_BOOKCONTENT_KOR_copy_내용원복소스 (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(50) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(50) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext,
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARSET utf8 
, ROW_FORMAT COMPACT 
, COMMENT '한국 자료'
;


create table if not exists EB_BOOKCONTENT_KOR_copy_Test (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(200) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(400) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(200) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST mediumtext,
  BOOKCONTENTS mediumtext,
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARSET utf8 
, ROW_FORMAT COMPACT 
, COMMENT='한국 자료'
;


create table if not exists EB_BOOKCONTENT_KOR_copy_replce (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL DEFAULT '0',
  BOOKNAME varchar(50) DEFAULT NULL COMMENT '도서명',
  SEARCH varchar(50) DEFAULT NULL COMMENT '유사검색어',
  BOOKTITLE varchar(50) DEFAULT NULL COMMENT '짧은서명(부제)',
  RECOMMEND_YN enum('Y','N') NOT NULL COMMENT '추천도서',
  DISPLAY_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  BOOKINTRO text COMMENT '도서소개',
  BOOKFORM varchar(500) DEFAULT NULL COMMENT '구성',
  BOOKFILENAME varchar(100) DEFAULT NULL COMMENT '구성파일명',
  BOOKFEATURE varchar(500) DEFAULT NULL COMMENT '특징',
  BOOKLIST_OLD mediumtext COMMENT '목차',
  BOOKLIST_OLDTEST mediumtext,
  BOOKLIST mediumtext,
  BOOKCONTENTS_OLDTEST mediumtext,
  BOOKCONTENTS mediumtext COMMENT '상세',
  BOOKCONTENTS_FILENAME mediumtext,
  CREDATE datetime NOT NULL,
  UPDDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_IDX)
) 
CHARSET utf8 
, ROW_FORMAT COMPACT 
, COMMENT '한국 자료'
;



create table if not exists EB_BOOKETC (
  NOTSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(10) unsigned DEFAULT '0',
  ADMSEQ int(10) unsigned DEFAULT '0',
  SUBTITLE_KOR varchar(255) DEFAULT NULL,
  SUBTITLE_ENG varchar(255) DEFAULT NULL,
  BARCODE varchar(30) DEFAULT '',
  BRICKSCODE varchar(30) DEFAULT '',
  PRICE int(10) DEFAULT '0',
  DISRATE varchar(30) DEFAULT '0',
  DIS_PRICE int(11) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MAIN_YN_KOR enum('Y','N') DEFAULT 'N',
  MAIN_YN_ENG enum('Y','N') DEFAULT 'N',
  PRIMARY KEY (NOTSEQ),
  KEY BOOKSEQ_BARCODE_BRICKSCODE (BOOKSEQ,BARCODE,BRICKSCODE)
)  
CHARSET utf8 
, ROW_FORMAT DYNAMIC
;


create table if not exists EB_BOOKETC_BAK (
  NOTSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(10) unsigned DEFAULT '0',
  ADMSEQ int(10) unsigned DEFAULT '0',
  SUBTITLE_KOR varchar(255) DEFAULT NULL,
  SUBTITLE_ENG varchar(255) DEFAULT NULL,
  BARCODE varchar(30) DEFAULT '',
  BRICKSCODE varchar(30) DEFAULT '',
  PRICE int(10) DEFAULT '0',
  DISRATE varchar(30) DEFAULT '0',
  DIS_PRICE int(11) DEFAULT '0',
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MAIN_YN_KOR enum('Y','N') DEFAULT 'N',
  MAIN_YN_ENG enum('Y','N') DEFAULT 'N',
  PRIMARY KEY (NOTSEQ)
) 
CHARSET utf8
;


create table if not exists EB_BOOKETC_copy (
  NOTSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(10) unsigned DEFAULT '0',
  ADMSEQ int(10) unsigned DEFAULT '0',
  SUBTITLE_KOR varchar(255) DEFAULT NULL,
  SUBTITLE_ENG varchar(255) DEFAULT NULL,
  BARCODE varchar(30) DEFAULT '',
  BRICKSCODE varchar(30) DEFAULT '',
  PRICE int(10) DEFAULT '0',
  DISRATE varchar(30) DEFAULT '0',
  DIS_PRICE int(11) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MAIN_YN_KOR enum('Y','N') DEFAULT 'N',
  MAIN_YN_ENG enum('Y','N') DEFAULT 'N',
  PRIMARY KEY (NOTSEQ)
) 
CHARSET utf8 
, ROW_FORMAT DYNAMIC
;


create table if not exists EB_BOOKSPOT (
  SPOTSEQ int(10) NOT NULL AUTO_INCREMENT,
  NAME varchar(30) DEFAULT NULL,
  EMAIL varchar(30) DEFAULT NULL,
  SUBJECT varchar(250) DEFAULT NULL,
  CONTENTS mediumtext,
  HTML_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  CNT int(10) DEFAULT '0',
  FILENAME varchar(50) DEFAULT NULL,
  CHECK_YN char(1) DEFAULT 'N',
  VIEW_YN char(1) DEFAULT 'Y',
  PRIMARY KEY (SPOTSEQ)
)  
CHARSET utf8
;


create table if not exists EB_BOOKVIEW (
  SEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  ADMSEQ int(10) unsigned DEFAULT '0',
  BOOKSEQ int(10) unsigned DEFAULT '0',
  FILENAME1 varchar(50) DEFAULT NULL,
  FILENAME2 varchar(50) DEFAULT NULL,
  FILENAME3 varchar(50) DEFAULT NULL,
  FILENAME4 varchar(50) DEFAULT NULL,
  FILENAME5 varchar(50) DEFAULT NULL,
  FILENAME6 varchar(50) DEFAULT NULL,
  FILENAME7 varchar(50) DEFAULT NULL,
  FILENAME8 varchar(50) DEFAULT NULL,
  FILENAME9 varchar(50) DEFAULT NULL,
  FILENAME10 varchar(50) DEFAULT NULL,
  FILENAME11 varchar(50) DEFAULT NULL,
  FILENAME12 varchar(50) DEFAULT NULL,
  FILENAME13 varchar(50) DEFAULT NULL,
  FILENAME14 varchar(50) DEFAULT NULL,
  FILENAME15 varchar(50) DEFAULT NULL,
  USE_YN char(1) DEFAULT 'N',
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (SEQ)
) 
CHARSET utf8
;


drop view if exists EB_BOOK_ALL;
CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_BOOK_ALL AS select A.BOOKSEQ AS BOOKSEQ,A.MEMSEQ AS MEMSEQ,A.BRANDNAME AS BRANDNAME,A.SKILL AS SKILL,A.SERIES AS SERIES,A.READERS AS READERS,A.TEXTBOOK AS TEXTBOOK,A.LEVEL AS LEVEL,A.BOOKMAP_LEVEL AS BOOKMAP_LEVEL,A.BOOKMAP_LEVEL_SUB AS BOOKMAP_LEVEL_SUB,A.BOOKPREVIEW AS BOOKPREVIEW,A.BOOKPRELISTENING AS BOOKPRELISTENING,A.BOOKPATH AS BOOKPATH,A.SERIESNO AS SERIESNO,A.BRICKSCODE AS BRICKSCODE,A.BARCODE AS BARCODE,A.SALE AS SALE,A.DATE AS DATE,A.FIRSTDATE AS FIRSTDATE,A.PAGE AS PAGE,A.BOOKSIZE AS BOOKSIZE,A.VOLUMNCODE AS VOLUMNCODE,A.ADMINTEXT AS ADMINTEXT,A.WRITER AS WRITER,A.IMGFILE1 AS IMGFILE1,A.IMGFILE2 AS IMGFILE2,A.IMGFILE3 AS IMGFILE3,A.NEWSFILE AS NEWSFILE,A.REAL_YN AS REAL_YN,A.COUNT AS COUNT,A.UPDDATE AS UPDDATE,A.CREDATE AS CREDATE,A.IP AS IP,B.EB_IDX AS EB_IDX_KOR,B.BOOKSEQ AS BOOKSEQ_KOR,B.BOOKNAME AS BOOKNAME_KOR,B.SEARCH AS SEARCH_KOR,B.BOOKTITLE AS BOOKTITLE_KOR,B.RECOMMEND_YN AS RECOMMEND_YN_KOR,B.DISPLAY_YN AS DISPLAY_YN_KOR,B.BOOKINTRO AS BOOKINTRO_KOR,B.BOOKFORM AS BOOKFORM_KOR,B.BOOKFILENAME AS BOOKFILENAME_KOR,B.BOOKFEATURE AS BOOKFEATURE_KOR,B.BOOKLIST AS BOOKLIST_KOR,B.BOOKCONTENTS AS BOOKCONTENTS_KOR,B.CREDATE AS CREDATE_KOR,B.UPDDATE AS UPDDATE_KOR,B.IP AS IP_KOR,C.EB_IDX AS EB_IDX_ENG,C.BOOKSEQ AS BOOKSEQ_ENG,C.BOOKNAME AS BOOKNAME_ENG,C.SEARCH AS SEARCH_ENG,C.BOOKTITLE AS BOOKTITLE_ENG,C.RECOMMEND_YN AS RECOMMEND_YN_ENG,C.DISPLAY_YN AS DISPLAY_YN_ENG,C.BOOKINTRO AS BOOKINTRO_ENG,C.BOOKFORM AS BOOKFORM_ENG,C.BOOKFILENAME AS BOOKFILENAME_ENG,C.BOOKFEATURE AS BOOKFEATURE_ENG,C.BOOKLIST AS BOOKLIST_ENG,C.BOOKCONTENTS AS BOOKCONTENTS_ENG,C.CREDATE AS CREDATE_ENG,C.UPDDATE AS UPDDATE_ENG,C.IP AS IP_ENG from ((EB_BOOK A join EB_BOOKCONTENT_KOR B on((A.BOOKSEQ = B.BOOKSEQ))) join EB_BOOKCONTENT_ENG C on((A.BOOKSEQ = C.BOOKSEQ))) 
;



create table if not exists EB_BOOK_BAK (
  BOOKSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) NOT NULL DEFAULT '0',
  BRANDNAME varchar(50) DEFAULT '' COMMENT '브랜드',
  SKILL varchar(50) DEFAULT '' COMMENT 'SKILL',
  SERIES varchar(50) DEFAULT '' COMMENT '시리즈',
  READERS varchar(50) DEFAULT '' COMMENT 'READERS',
  TEXTBOOK varchar(50) DEFAULT '' COMMENT '미국교과서',
  LEVEL_F varchar(50) DEFAULT '' COMMENT '레벨1단계',
  LEVEL varchar(50) DEFAULT '' COMMENT '레벨2단계',
  BOOKMAP_LEVEL varchar(50) DEFAULT '' COMMENT '북맵레벨 1(신규)',
  BOOKMAP_LEVEL_SUB varchar(50) DEFAULT '' COMMENT '북맵레벨2(신규)',
  CEFR varchar(50) DEFAULT '' COMMENT 'CEFR',
  BOOKPREVIEW varchar(50) DEFAULT '' COMMENT '미리보기(신규)',
  BOOKPRELISTENING varchar(50) DEFAULT '' COMMENT '미리듣기(신규)',
  BOOKPATH varchar(50) DEFAULT '' COMMENT '경로(신규)',
  SERIESNO varchar(50) DEFAULT '' COMMENT '도서시리즈번호',
  BRICKSCODE varchar(50) DEFAULT '' COMMENT '사평코드',
  BARCODE varchar(50) DEFAULT '' COMMENT '바코드',
  SALE varchar(50) DEFAULT '' COMMENT '판매여부',
  DATE varchar(10) DEFAULT '' COMMENT '발행일',
  FIRSTDATE varchar(10) DEFAULT '' COMMENT '초판발행일',
  PAGE int(10) DEFAULT '0' COMMENT '페이지',
  BOOKSIZE varchar(50) DEFAULT '' COMMENT '책크기',
  VOLUMNCODE varchar(50) DEFAULT '' COMMENT '판표시',
  ADMINTEXT text COMMENT '담당자',
  WRITER varchar(255) DEFAULT '' COMMENT '입력형식',
  IMGFILE1 varchar(100) DEFAULT NULL COMMENT 'L도서이미지',
  IMGFILE2 varchar(100) DEFAULT NULL COMMENT 'M도서이미지',
  IMGFILE3 varchar(100) DEFAULT NULL COMMENT 'S도서이미지',
  NEWSFILE varchar(100) DEFAULT NULL COMMENT '보도자료',
  REAL_YN char(1) DEFAULT 'Y',
  COUNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (BOOKSEQ),
  KEY BOOKSEQ (BOOKSEQ)
)  
CHARSET utf8
;


CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_BOOK_ENG AS select A.BOOKSEQ AS BOOKSEQ,A.MEMSEQ AS MEMSEQ,A.BRANDNAME AS BRANDNAME,A.SKILL AS SKILL,A.ADMIN_REVIEW AS ADMIN_REVIEW,A.SERIES AS SERIES,A.SERIES_YN AS SERIES_YN,A.READERS AS READERS,A.TEXTBOOK AS TEXTBOOK,A.LEVEL_F AS LEVEL_F,A.LEVEL AS LEVEL,A.CEFR AS CEFR,A.BOOKMAP_LEVEL AS BOOKMAP_LEVEL,A.BOOKMAP_LEVEL_SUB AS BOOKMAP_LEVEL_SUB,B.BOOKNAME AS BOOKNAME,B.BOOKTITLE AS BOOKTITLE,B.SEARCH AS SEARCH,A.SERIESNO AS SERIESNO,A.BRICKSCODE AS BRICKSCODE,A.BARCODE AS BARCODE,A.SALE AS SALE,A.DATE AS DATE,A.FIRSTDATE AS FIRSTDATE,A.PAGE AS PAGE,A.BOOKSIZE AS BOOKSIZE,A.VOLUMNCODE AS VOLUMNCODE,B.DISPLAY_YN AS DISPLAY_YN,B.BOOKINTRO AS BOOKINTRO,B.BOOKFORM AS BOOKFORM,B.BOOKFILENAME AS BOOKFILENAME,B.BOOKFEATURE AS BOOKFEATURE,B.BOOKCONTENTS AS BOOKCONTENTS,B.BOOKCONTENTS_FILENAME AS BOOKCONTENTS_FILENAME,B.RECOMMEND_YN AS RECOMMEND_YN,B.BOOKLIST AS BOOKLIST,B.BOOKLIST_OLD AS BOOKLIST_OLD,A.WRITER AS WRITER,A.IMGFILE1 AS IMGFILE1,A.IMGFILE2 AS IMGFILE2,A.IMGFILE3 AS IMGFILE3,A.BOOKPREVIEW AS BOOKPREVIEW,A.EBOOKVIEW AS EBOOKVIEW,A.BOOKPRELISTENING AS BOOKPRELISTENING,A.REAL_YN AS REAL_YN,A.COUNT AS COUNT,A.UPDDATE AS UPDDATE,A.CREDATE AS CREDATE,A.IP AS IP,B.SEARCH_LEVEL AS SEARCH_LEVEL from (EB_BOOK A join EB_BOOKCONTENT_ENG B on((A.BOOKSEQ = B.BOOKSEQ)))
;


CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_BOOK_KOR AS select A.BOOKSEQ AS BOOKSEQ,A.MEMSEQ AS MEMSEQ,A.BRANDNAME AS BRANDNAME,A.SKILL AS SKILL,A.ADMIN_REVIEW AS ADMIN_REVIEW,A.SERIES AS SERIES,A.SERIES_YN AS SERIES_YN,A.READERS AS READERS,A.TEXTBOOK AS TEXTBOOK,A.LEVEL_F AS LEVEL_F,A.LEVEL AS LEVEL,A.CEFR AS CEFR,A.BOOKMAP_LEVEL AS BOOKMAP_LEVEL,A.BOOKMAP_LEVEL_SUB AS BOOKMAP_LEVEL_SUB,B.BOOKNAME AS BOOKNAME,B.BOOKTITLE AS BOOKTITLE,B.SEARCH AS SEARCH,A.SERIESNO AS SERIESNO,A.BRICKSCODE AS BRICKSCODE,A.BARCODE AS BARCODE,A.SALE AS SALE,A.DATE AS DATE,A.FIRSTDATE AS FIRSTDATE,A.PAGE AS PAGE,A.BOOKSIZE AS BOOKSIZE,A.VOLUMNCODE AS VOLUMNCODE,B.DISPLAY_YN AS DISPLAY_YN,B.BOOKINTRO AS BOOKINTRO,B.BOOKFORM AS BOOKFORM,B.BOOKFILENAME AS BOOKFILENAME,B.BOOKFEATURE AS BOOKFEATURE,B.BOOKCONTENTS AS BOOKCONTENTS,B.BOOKCONTENTS_FILENAME AS BOOKCONTENTS_FILENAME,B.RECOMMEND_YN AS RECOMMEND_YN,B.BOOKLIST AS BOOKLIST,B.BOOKLIST_OLD AS BOOKLIST_OLD,A.WRITER AS WRITER,A.IMGFILE1 AS IMGFILE1,A.IMGFILE2 AS IMGFILE2,A.IMGFILE3 AS IMGFILE3,A.BOOKPREVIEW AS BOOKPREVIEW,A.EBOOKVIEW AS EBOOKVIEW,A.EBOOKVIEW_CNT AS EBOOKVIEW_CNT,A.BOOKPRELISTENING AS BOOKPRELISTENING,A.REAL_YN AS REAL_YN,A.COUNT AS COUNT,A.UPDDATE AS UPDDATE,A.CREDATE AS CREDATE,A.IP AS IP,B.SEARCH_LEVEL AS SEARCH_LEVEL from (EB_BOOK A join EB_BOOKCONTENT_KOR B on((A.BOOKSEQ = B.BOOKSEQ))) WITH CASCADED CHECK OPTION
;


create table if not exists EB_BOOK_MAP (
  BM_SEQ int(11) NOT NULL AUTO_INCREMENT COMMENT '북맵코드',
  BM_GUBUN varchar(20) DEFAULT NULL COMMENT '구분',
  BM_TITLE varchar(500) DEFAULT NULL COMMENT '제목',
  BM_SUMMERY varchar(500) DEFAULT NULL COMMENT '요약',
  BM_START_COL tinyint(4) DEFAULT NULL COMMENT '시작위치',
  BM_END_COL tinyint(4) DEFAULT NULL COMMENT '종료위치',
  BM_ORDER tinyint(4) DEFAULT NULL COMMENT '정렬순서',
  BM_LIST_IMG varchar(500) DEFAULT NULL COMMENT '리스트이미지경로',
  BM_BOOKSEQ varchar(500) DEFAULT NULL COMMENT '도서코드',
  BM_NOTSEQ varchar(500) DEFAULT NULL COMMENT '부가도서코드',
  BM_USE_YN enum('Y','N') DEFAULT 'N' COMMENT '사용여부',
  PRIMARY KEY (BM_SEQ)
) 
CHARSET utf8
;


create table if not exists EB_BOOK_MAP_CD (
  BMC_TYPE enum('G','L') DEFAULT 'G' COMMENT ' 구분 G:스킬, L:레벨',
  BMC_GCD varchar(10) DEFAULT NULL COMMENT '그룹코드',
  BMC_CD varchar(10) DEFAULT NULL COMMENT '북맵 코드',
  BMC_DESC varchar(50) DEFAULT NULL COMMENT '코드 설명',
  BMC_ORDER int(11) DEFAULT NULL COMMENT '정렬순서'
) 
CHARSET utf8
;


create table if not exists EB_BOOK_WITH_WRITER (
  NOTSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  ADMSEQ int(10) unsigned NOT NULL DEFAULT '0',
  AUTHSEQ int(10) unsigned NOT NULL DEFAULT '0',
  BOOKSEQ int(10) unsigned NOT NULL DEFAULT '0',
  GUBUN char(1) NOT NULL,
  NAME varchar(30) NOT NULL,
  BOOKNAME varchar(30) DEFAULT '',
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (NOTSEQ)
) 
CHARSET utf8
;


create table if not exists EB_BOOK___ (
  BOOKSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) NOT NULL DEFAULT '0',
  BRANDNAME varchar(50) DEFAULT '',
  SKILL varchar(50) DEFAULT '',
  SERIES varchar(50) DEFAULT '',
  READERS varchar(50) DEFAULT '',
  TEXTBOOK varchar(50) DEFAULT '',
  LEVEL varchar(50) DEFAULT '',
  BOOKNAME varchar(50) DEFAULT '',
  BOOKTITLE varchar(50) DEFAULT '',
  SEARCH varchar(255) DEFAULT '',
  SERIESNO varchar(50) DEFAULT '',
  BRICKSCODE varchar(50) DEFAULT '',
  BARCODE varchar(50) DEFAULT '',
  SALE varchar(50) DEFAULT '',
  DATE varchar(10) DEFAULT '',
  FIRSTDATE varchar(10) DEFAULT '',
  PAGE int(10) DEFAULT '0',
  BOOKSIZE varchar(50) DEFAULT '',
  VOLUMNCODE varchar(50) DEFAULT '',
  DISPLAY_YN char(1) DEFAULT 'N',
  ADMINTEXT text,
  BOOKINTRO text,
  BOOKCONTENTS mediumtext,
  BOOKLIST mediumtext,
  WRITER varchar(255) DEFAULT '',
  IMGFILE1 varchar(100) DEFAULT NULL,
  IMGFILE2 varchar(100) DEFAULT NULL,
  IMGFILE3 varchar(100) DEFAULT NULL,
  NEWSFILE varchar(100) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  COUNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  RECOMMEND_YN char(1) DEFAULT 'N',
  ENGBOOKINTRO text,
  ENGBOOKCONTENTS mediumtext,
  ENGRECOMMEND_YN char(1) DEFAULT 'N',
  ENGDISPLAY_YN char(1) DEFAULT 'N',
  ENGBOOKNAME varchar(255) DEFAULT NULL,
  PRIMARY KEY (BOOKSEQ)
) 
CHARSET utf8 
;


create table if not exists EB_BOOK_copy0417_ori (
  BOOKSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) NOT NULL DEFAULT '0',
  BRANDNAME varchar(50) DEFAULT '' COMMENT '브랜드',
  SKILL varchar(50) DEFAULT '' COMMENT 'SKILL',
  SERIES varchar(50) DEFAULT '' COMMENT '시리즈',
  READERS varchar(50) DEFAULT '' COMMENT 'READERS',
  TEXTBOOK varchar(50) DEFAULT '' COMMENT '미국교과서',
  LEVEL_F varchar(50) DEFAULT '' COMMENT '레벨1단계',
  LEVEL varchar(50) DEFAULT '' COMMENT '레벨2단계',
  BOOKMAP_LEVEL varchar(50) DEFAULT '' COMMENT '북맵레벨 1(신규)',
  BOOKMAP_LEVEL_SUB varchar(50) DEFAULT '' COMMENT '북맵레벨2(신규)',
  CEFR varchar(50) DEFAULT '' COMMENT 'CEFR',
  BOOKPREVIEW varchar(50) DEFAULT '' COMMENT '미리보기(신규)',
  BOOKPRELISTENING varchar(50) DEFAULT '' COMMENT '미리듣기(신규)',
  BOOKPATH varchar(50) DEFAULT '' COMMENT '경로(신규)',
  SERIESNO varchar(50) DEFAULT '' COMMENT '도서시리즈번호',
  BRICKSCODE varchar(50) DEFAULT '' COMMENT '사평코드',
  BARCODE varchar(50) DEFAULT '' COMMENT '바코드',
  SALE varchar(50) DEFAULT '' COMMENT '판매여부',
  DATE varchar(10) DEFAULT '' COMMENT '발행일',
  FIRSTDATE varchar(10) DEFAULT '' COMMENT '초판발행일',
  PAGE int(10) DEFAULT '0' COMMENT '페이지',
  BOOKSIZE varchar(50) DEFAULT '' COMMENT '책크기',
  VOLUMNCODE varchar(50) DEFAULT '' COMMENT '판표시',
  ADMINTEXT text COMMENT '담당자',
  WRITER varchar(255) DEFAULT '' COMMENT '입력형식',
  IMGFILE1 varchar(100) DEFAULT NULL COMMENT 'L도서이미지',
  IMGFILE2 varchar(100) DEFAULT NULL COMMENT 'M도서이미지',
  IMGFILE3 varchar(100) DEFAULT NULL COMMENT 'S도서이미지',
  NEWSFILE varchar(100) DEFAULT NULL COMMENT '보도자료',
  REAL_YN char(1) DEFAULT 'Y',
  COUNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (BOOKSEQ),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARSET utf8 
, ROW_FORMAT COMPACT
;



create table if not exists EB_BOOK_copy_old (
  BOOKSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) NOT NULL DEFAULT '0',
  BRANDNAME varchar(50) DEFAULT '' COMMENT '브랜드',
  SKILL varchar(50) DEFAULT '' COMMENT 'SKILL',
  SERIES varchar(50) DEFAULT '' COMMENT '시리즈',
  READERS varchar(50) DEFAULT '' COMMENT 'READERS',
  TEXTBOOK varchar(50) DEFAULT '' COMMENT '미국교과서',
  LEVEL varchar(50) DEFAULT '' COMMENT '레벨2단계',
  BOOKMAP_LEVEL varchar(50) DEFAULT '' COMMENT '북맵레벨 1(신규)',
  BOOKMAP_LEVEL_SUB varchar(50) DEFAULT '' COMMENT '북맵레벨2(신규)',
  CEFR varchar(50) DEFAULT '' COMMENT 'CEFR',
  BOOKPREVIEW varchar(50) DEFAULT '' COMMENT '미리보기(신규)',
  BOOKPRELISTENING varchar(50) DEFAULT '' COMMENT '미리듣기(신규)',
  BOOKPATH varchar(50) DEFAULT '' COMMENT '경로(신규)',
  BOOKNAME varchar(50) DEFAULT '' COMMENT '도서명',
  BOOKTITLE varchar(50) DEFAULT '' COMMENT '부제',
  SEARCH varchar(255) DEFAULT '' COMMENT '유사검색어',
  SERIESNO varchar(50) DEFAULT '' COMMENT '도서시리즈번호',
  BRICKSCODE varchar(50) DEFAULT '' COMMENT '사평코드',
  BARCODE varchar(50) DEFAULT '' COMMENT '바코드',
  SALE varchar(50) DEFAULT '' COMMENT '판매여부',
  DATE varchar(10) DEFAULT '' COMMENT '발행일',
  FIRSTDATE varchar(10) DEFAULT '' COMMENT '초판발행일',
  PAGE int(10) DEFAULT '0' COMMENT '페이지',
  BOOKSIZE varchar(50) DEFAULT '' COMMENT '책크기',
  VOLUMNCODE varchar(50) DEFAULT '' COMMENT '판표시',
  DISPLAY_YN char(1) DEFAULT 'N' COMMENT '홈페이지 공개여부',
  ADMINTEXT text COMMENT '담당자',
  BOOKINTRO text COMMENT '간략소개',
  BOOKCONTENTS mediumtext COMMENT '상세소개',
  BOOKLIST mediumtext COMMENT '목차',
  WRITER varchar(255) DEFAULT '' COMMENT '입력형식',
  IMGFILE1 varchar(100) DEFAULT NULL COMMENT 'L도서이미지',
  IMGFILE2 varchar(100) DEFAULT NULL COMMENT 'M도서이미지',
  IMGFILE3 varchar(100) DEFAULT NULL COMMENT 'S도서이미지',
  NEWSFILE varchar(100) DEFAULT NULL COMMENT '보도자료',
  REAL_YN char(1) DEFAULT 'Y',
  COUNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  RECOMMEND_YN char(1) DEFAULT 'N',
  ENGBOOKINTRO text,
  ENGBOOKCONTENTS mediumtext,
  ENGRECOMMEND_YN char(1) DEFAULT 'N',
  ENGDISPLAY_YN char(1) DEFAULT 'N',
  ENGBOOKNAME varchar(255) DEFAULT NULL,
  PRIMARY KEY (BOOKSEQ),
  KEY BOOKSEQ (BOOKSEQ)
) 
CHARSET utf8
, ROW_FORMAT DYNAMIC
;


create table if not exists EB_BUYSTORE (
  EB_SEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  EB_ADMSEQ int(10) DEFAULT '0',
  EB_AREA varchar(50) DEFAULT NULL,
  EB_STORENAME varchar(50) DEFAULT NULL,
  EB_NAME varchar(50) DEFAULT NULL,
  EB_PHONE varchar(50) DEFAULT NULL,
  EB_FAX varchar(50) DEFAULT NULL,
  EB_ADDRESS varchar(255) DEFAULT NULL,
  EB_REAL_YN char(1) DEFAULT 'Y',
  EB_UPDDATE datetime DEFAULT NULL,
  EB_CREDATE datetime DEFAULT NULL,
  EB_IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EB_SEQ)
) 
 CHARSET utf8
, ROW_FORMAT DYNAMIC
;


create table if not exists EB_CODE (
  CODE int(6) unsigned zerofill NOT NULL DEFAULT '000000',
  CODE_OLD int(6) DEFAULT NULL,
  NAME varchar(255) DEFAULT NULL,
  SUB_NAME varchar(255) DEFAULT NULL,
  PARENTID int(6) unsigned zerofill NOT NULL DEFAULT '000000',
  DEPTH_3 char(1) DEFAULT '1',
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  ORDNO_ smallint(10) unsigned DEFAULT '0',
  ORDNO smallint(10) unsigned DEFAULT '0',
  PRIMARY KEY (CODE),
  KEY PARENTID (PARENTID)
)  
CHARSET utf8
;


create table if not exists EB_CODE_copy (
  CODE int(6) unsigned zerofill NOT NULL DEFAULT '000000',
  NAME varchar(255) DEFAULT NULL,
  PARENTID int(6) unsigned zerofill NOT NULL DEFAULT '000000',
  DEPTH_3 char(1) DEFAULT '1',
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  ORDNO_ smallint(10) unsigned DEFAULT '0',
  ORDNO smallint(10) unsigned DEFAULT '0',
  PRIMARY KEY (CODE)
)  
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_COUNTRY (
  EC_IDX int(11) NOT NULL AUTO_INCREMENT,
  EC_NAME varchar(50) DEFAULT NULL COMMENT '국가명',
  EC_NUM varchar(10) DEFAULT NULL COMMENT '국가전화코드',
  PRIMARY KEY (EC_IDX)
)  
CHARSET utf8 
,COMMENT '국가코드'
;


CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_DOWN_BOOKLIST_ENG AS select distinct A.BOOKNAME AS BOOKNAME,A.BOOKSEQ AS BOOKSEQ,B.NOTSEQ AS NOTSEQ from (EB_BOOKETC B join EB_BOOK_ENG A on((B.BOOKSEQ = A.BOOKSEQ))) where ((A.REAL_YN = 'Y') and (B.REAL_YN = 'Y') and (B.MAIN_YN_ENG = 'Y')) group by A.BOOKSEQ having (B.NOTSEQ = min(B.NOTSEQ));


CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_DOWN_BOOKLIST_KOR AS select distinct A.BOOKNAME AS BOOKNAME,A.BOOKSEQ AS BOOKSEQ,B.NOTSEQ AS NOTSEQ from (EB_BOOKETC B join EB_BOOK_KOR A on((B.BOOKSEQ = A.BOOKSEQ))) where ((A.REAL_YN = 'Y') and (B.REAL_YN = 'Y') and (B.MAIN_YN_KOR = 'Y')) group by A.BOOKSEQ having (B.NOTSEQ = min(B.NOTSEQ));


create table if not exists EB_EBOOK (
  EE_IDX int(11) NOT NULL AUTO_INCREMENT,
  EE_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EE_ID varchar(50) NOT NULL COMMENT '회원id',
  EE_OPEN_YN char(1) DEFAULT NULL COMMENT '오픈구분(A:전체 P:부분)',
  EE_OPEN_START date DEFAULT NULL COMMENT '오픈시작날짜',
  EE_OPEN_END date DEFAULT NULL COMMENT '오픈마감날짜',
  EE_ADMIN_ID varchar(50) DEFAULT NULL COMMENT '담당자ID',
  EE_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EE_UPDATE datetime DEFAULT NULL COMMENT '수정일',
  EE_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EE_IDX) USING BTREE
)
 CHARSET utf8 
,ROW_FORMAT DYNAMIC
, COMMENT '이북관리'
;


create table if not exists EB_EBOOK_ACCESSCODE (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  GUBUN char(50) DEFAULT NULL COMMENT '사용분류(A:학생 T:선생)',
  LANG char(50) DEFAULT NULL COMMENT '업로드분류(K:한국어 E:다국어 A:모두)',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  USE_UPD_DT datetime DEFAULT NULL COMMENT '미사용처리 수정날짜',
  USE_UPD_ADMIN varchar(50) DEFAULT NULL COMMENT '미사용처리 수정한 관리자',
  PRIMARY KEY (ACCSEQ),
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE,USE_YN) USING BTREE
) 
CHARSET utf8
;


create table if not exists EB_EBOOK_ACCESSCODE_201228 (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  GUBUN char(50) DEFAULT NULL COMMENT '사용분류(A:학생 T:선생)',
  LANG char(50) DEFAULT NULL COMMENT '업로드분류(K:한국어 E:다국어 A:모두)',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  USE_UPD_DT datetime DEFAULT NULL COMMENT '미사용처리 수정날짜',
  USE_UPD_ADMIN varchar(50) DEFAULT NULL COMMENT '미사용처리 수정한 관리자',
  PRIMARY KEY (ACCSEQ) USING BTREE,
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE,USE_YN) USING BTREE
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_EBOOK_ACCESSCODE_20200417 (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  PRIMARY KEY (ACCSEQ),
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_EBOOK_ACCESSCODE_20200728_tmp (
  ACCSEQ int(10) NOT NULL COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  USE_UPD_DT datetime DEFAULT NULL COMMENT '미사용처리 수정날짜',
  USE_UPD_ADMIN varchar(50) DEFAULT NULL COMMENT '미사용처리 수정한 관리자',
  PRIMARY KEY (ACCSEQ) USING BTREE,
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE,USE_YN) USING BTREE
) 
CHARSET utf8
;


create table if not exists EB_EBOOK_ACCESSCODE_LOG (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  GUBUN char(1) DEFAULT NULL COMMENT '사용분류(A:학생 T:선생)',
  LANG char(1) DEFAULT NULL COMMENT '업로드분류(K:한국어 E:다국어 A:모두)',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  CHK_YN enum('Y','N','O') NOT NULL DEFAULT 'N',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  PRIMARY KEY (ACCSEQ),
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_EBOOK_ACCESSCODE_copy (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  PRIMARY KEY (ACCSEQ),
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_EBOOK_ACCESSCODE_copy_190919 (
  ACCSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  ACCESSCODE varchar(200) DEFAULT NULL COMMENT '엑세스코드',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  USE_YN enum('Y','N') NOT NULL DEFAULT 'N',
  PRIMARY KEY (ACCSEQ),
  KEY BOOKSEQ_ACCESSCODE (BOOKSEQ,ACCESSCODE)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_EBOOK_GB (
  EE_IDX int(11) NOT NULL AUTO_INCREMENT,
  EE_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EE_ID varchar(50) NOT NULL COMMENT '회원id',
  EE_OPEN_YN char(1) DEFAULT NULL COMMENT '오픈구분(A:전체 P:부분)',
  EE_OPEN_START date DEFAULT NULL COMMENT '오픈시작날짜',
  EE_OPEN_END date DEFAULT NULL COMMENT '오픈마감날짜',
  EE_ADMIN_ID varchar(50) DEFAULT NULL COMMENT '담당자ID',
  EE_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EE_UPDATE datetime DEFAULT NULL COMMENT '수정일',
  EE_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EE_IDX) USING BTREE
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT '이북관리(다국어)' 
;


create table if not exists EB_EBOOK_SERIES (
  EES_IDX int(11) NOT NULL AUTO_INCREMENT,
  EES_EE_IDX int(11) NOT NULL COMMENT 'ebook idx',
  EES_ETMS_SKILL varchar(50) DEFAULT NULL COMMENT 'skill',
  EES_ETMS_IDX int(11) DEFAULT NULL COMMENT 'top menu series idx',
  EES_CREDATE datetime NOT NULL COMMENT '등록일',
  PRIMARY KEY (EES_IDX) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT '이북관리 시리즈'
;


create table if not exists EB_EBOOK_SERIES_GB (
  EES_IDX int(11) NOT NULL AUTO_INCREMENT,
  EES_EE_IDX int(11) NOT NULL COMMENT 'ebook idx',
  EES_ETMS_SKILL varchar(50) DEFAULT NULL COMMENT 'skill',
  EES_ETMS_IDX int(11) DEFAULT NULL COMMENT 'top menu series idx',
  EES_CREDATE datetime NOT NULL COMMENT '등록일',
  PRIMARY KEY (EES_IDX) USING BTREE
)
 CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT '이북관리 시리즈(다국어)' 
;


create table if not exists EB_EMAIL_CHKNUM (
  EEC_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EEC_EMAIL varchar(50) DEFAULT NULL,
  EEC_NAME varchar(50) DEFAULT NULL,
  EEC_CHKNUM varchar(6) DEFAULT NULL,
  EEC_CREDATE datetime DEFAULT NULL,
  EEC_IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EEC_SEQ)
) 
CHARSET utf8 
,COMMENT '이메일 인증번호 발송이력'
;


create table if not exists EB_EMAIL_CHKNUM_GB (
  EEC_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EEC_EMAIL varchar(50) DEFAULT NULL,
  EEC_F_NAME varchar(50) DEFAULT NULL,
  EEC_L_NAME varchar(50) DEFAULT NULL,
  EEC_CHKNUM varchar(6) DEFAULT NULL,
  EEC_CREDATE datetime DEFAULT NULL,
  EEC_IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (EEC_SEQ) USING BTREE
) 
CHARSET utf8 
,COMMENT '이메일 인증번호 발송이력-다국어'
;


create table if not exists EB_FAQ (
  EF_SEQ int(10) NOT NULL AUTO_INCREMENT,
  EF_ADMSEQ int(10) DEFAULT '0' COMMENT '관리자 IDX',
  EF_ADMNAME varchar(30) DEFAULT NULL COMMENT '관리자이름',
  EF_TYPE varchar(30) DEFAULT NULL COMMENT 'FAQ분류',
  EF_TITLE varchar(250) DEFAULT NULL COMMENT '제목',
  EF_CONTENTS mediumtext COMMENT '내용',
  EF_FILENAME varchar(50) DEFAULT NULL COMMENT '파일명',
  EF_FILEPATH varchar(50) DEFAULT NULL COMMENT '파일경로',
  EF_REAL_YN enum('Y','N') DEFAULT 'Y' COMMENT '삭제여부',
  EF_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EF_UPDDATE datetime DEFAULT NULL COMMENT '수정일',
  EF_CNT int(10) DEFAULT '0' COMMENT '조회수',
  EF_IP char(15) DEFAULT NULL,
  PRIMARY KEY (EF_SEQ)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists  EB_GLOBAL_QNA (
  EGQ_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EGQ_USERSEQ int(11) DEFAULT NULL,
  EGQ_NAME varchar(50) DEFAULT NULL,
  EGQ_TYPE varchar(50) DEFAULT NULL,
  EGQ_EMAIL varchar(50) DEFAULT NULL,
  EGQ_TITLE varchar(50) DEFAULT NULL,
  EGQ_CONTENTS mediumtext,
  EGQ_FILEPATH varchar(50) DEFAULT NULL,
  EGQ_FILENAME varchar(50) DEFAULT NULL,
  EGQ_REAL_YN varchar(50) DEFAULT NULL,
  EGQ_CREDATE varchar(50) DEFAULT NULL,
  EGQ_UPDDATE varchar(50) DEFAULT NULL,
  EGQ_IP varchar(50) DEFAULT NULL,
  PRIMARY KEY (EGQ_SEQ)
) 
CHARSET utf8 
,COMMENT '다국어 1:1'
;


create table if not exists EB_INICIS_LOG (
  EIL_SEQ int(11) NOT NULL AUTO_INCREMENT,
  tid varchar(100) DEFAULT NULL,
  mode_type enum('P','M') DEFAULT NULL,
  resultCode varchar(100) DEFAULT NULL,
  resultMsg varchar(100) DEFAULT NULL,
  TotPrice varchar(100) DEFAULT NULL,
  MOID varchar(100) DEFAULT NULL,
  payMethod varchar(100) DEFAULT NULL,
  applNum varchar(100) DEFAULT NULL,
  applDate varchar(100) DEFAULT NULL,
  applTime varchar(100) DEFAULT NULL,
  CARD_Num varchar(100) DEFAULT NULL,
  CARD_Quota varchar(100) DEFAULT NULL,
  CARD_Code varchar(100) DEFAULT NULL,
  ACCT_BankCode varchar(100) DEFAULT NULL,
  CSHR_ResultCode varchar(100) DEFAULT NULL,
  CSHR_Type varchar(100) DEFAULT NULL,
  ACCT_Name varchar(100) DEFAULT NULL,
  VACT_Num varchar(100) DEFAULT NULL,
  VACT_BankCode varchar(100) DEFAULT NULL,
  vactBankName varchar(100) DEFAULT NULL,
  VACT_Name varchar(100) DEFAULT NULL,
  VACT_InputName varchar(100) DEFAULT NULL,
  VACT_Date varchar(100) DEFAULT NULL,
  VACT_Time varchar(100) DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(100) DEFAULT NULL,
  PRIMARY KEY (EIL_SEQ)
) 
CHARSET utf8 
,COMMENT '원본데이터 로그'
;


create table if not exists EB_INI_CODE (
  EIC_SEQ int(11) NOT NULL AUTO_INCREMENT,
  CODE_TYPE enum('C','B') DEFAULT NULL COMMENT 'C 카드 B 은행',
  CODE varchar(5) DEFAULT NULL,
  CODENAME varchar(50) DEFAULT NULL,
  PRIMARY KEY (EIC_SEQ),
  KEY CODE (CODE)
) 
CHARSET utf8 
,COMMENT '이니시스코드'
;


create table if not exists EB_INOUT (
  EI_IDX int(11) NOT NULL AUTO_INCREMENT,
  EI_DATE date NOT NULL COMMENT '조건문 날짜',
  EL_DATA1 int(11) NOT NULL DEFAULT '0' COMMENT '전체 회원가입',
  EL_DATA2 int(11) NOT NULL DEFAULT '0' COMMENT '선생님 회원가입',
  EL_DATA3 int(11) NOT NULL DEFAULT '0' COMMENT '학생회원가입',
  EL_DATA4 int(11) NOT NULL DEFAULT '0' COMMENT '전체탈퇴',
  EL_DATA5 int(11) NOT NULL DEFAULT '0' COMMENT '선생님탈퇴',
  EL_DATA6 int(11) NOT NULL DEFAULT '0' COMMENT '학생탈퇴',
  EL_DATA7 int(11) NOT NULL DEFAULT '0' COMMENT '가입경로 사회평론',
  EL_DATA8 int(11) NOT NULL DEFAULT '0' COMMENT '가입경로 페이스북',
  EL_DATA9 int(11) DEFAULT '0' COMMENT '가입경로 네이버',
  EL_DATA10 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA11 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA12 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA13 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  PRIMARY KEY (EI_IDX)
) 
CHARSET utf8 
,COMMENT '통계'
;


create table if not exists EB_INOUT_GB (
  EI_IDX int(11) NOT NULL AUTO_INCREMENT,
  EI_DATE date NOT NULL COMMENT '조건문 날짜',
  EL_DATA1 int(11) NOT NULL DEFAULT '0' COMMENT '전체 회원가입',
  EL_DATA2 int(11) NOT NULL DEFAULT '0' COMMENT '선생님 회원가입',
  EL_DATA3 int(11) NOT NULL DEFAULT '0' COMMENT '학생회원가입',
  EL_DATA4 int(11) NOT NULL DEFAULT '0' COMMENT '전체탈퇴',
  EL_DATA5 int(11) NOT NULL DEFAULT '0' COMMENT '선생님탈퇴',
  EL_DATA6 int(11) NOT NULL DEFAULT '0' COMMENT '학생탈퇴',
  EL_DATA7 int(11) NOT NULL DEFAULT '0' COMMENT '가입경로 사회평론',
  EL_DATA8 int(11) NOT NULL DEFAULT '0' COMMENT '가입경로 페이스북',
  EL_DATA9 int(11) DEFAULT '0' COMMENT '가입경로 네이버',
  EL_DATA10 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA11 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA12 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  EL_DATA13 int(11) NOT NULL DEFAULT '0' COMMENT '미사용 예비',
  PRIMARY KEY (EI_IDX) USING BTREE
) 
CHARSET utf8 
,COMMENT '통계 - 다국어'
;


create table if not exists EB_LESSON (
  LP_SEQ int(11) NOT NULL AUTO_INCREMENT COMMENT '순번',
  LP_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LP_BOOKNAME varchar(300) DEFAULT NULL COMMENT '도서이름',
  LP_LESSON int(11) DEFAULT NULL COMMENT '레슨순번',
  LP_THEME varchar(300) DEFAULT NULL COMMENT '테마명',
  LP_UNIT varchar(300) DEFAULT NULL COMMENT '유닛명',
  LP_PAGE varchar(300) DEFAULT NULL COMMENT '해당페이지',
  LP_TEACHING_OBJ varchar(1000) DEFAULT NULL COMMENT '학습내용',
  LP_HOMEWORK varchar(300) DEFAULT NULL COMMENT '과제',
  LP_USE_YN enum('Y','N') DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (LP_SEQ)
) 
CHARSET utf8
;



create table if not exists EB_PDS (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  ADMSEQ int(10) DEFAULT '0' COMMENT '등록자 번호',
  ADMNAME varchar(50) DEFAULT NULL COMMENT '등록자 이름',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '등록자 IP',
  FILENAME varchar(200) DEFAULT NULL COMMENT '파일이름',
  FILEPATH varchar(200) DEFAULT NULL COMMENT '파일경로',
  FILEGUBUN varchar(50) DEFAULT NULL COMMENT '파일구분',
  FILESIZE bigint(20) DEFAULT NULL COMMENT '파일용량',
  FILEEXT varchar(10) DEFAULT NULL COMMENT '파일확장자',
  GUBUN enum('A','T') DEFAULT 'A' COMMENT '사용분류',
  LANG varchar(50) DEFAULT 'AK' COMMENT '업로드분류',
  ZIPGUBUN enum('N','S','A','E') DEFAULT 'N' COMMENT '압축구분 N-압축아님, S-학생용압축, A-전체압축,E-다국어',
  LESSONPLAN_CONTENTS mediumtext COMMENT 'lessonplan내용',
  VIEW_CNT int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (NOTSEQ),
  KEY BOOKSEQ_FILENAME_FILEGUBUN_LANG (BOOKSEQ,FILENAME,FILEGUBUN,LANG)
)  
CHARSET utf8 
;


create table if not exists EB_LESSON_PLAN (
  LPM_SEQ int(11) NOT NULL AUTO_INCREMENT COMMENT '순번',
  LPM_LEVEL varchar(50) DEFAULT NULL COMMENT '플랜등급',
  LPM_GUBUN varchar(50) DEFAULT NULL COMMENT '플랜구분',
  LPM_LESSON_YN enum('Y','N') DEFAULT 'Y' COMMENT '레슨사용여부',
  LPM_ORDER int(11) DEFAULT NULL COMMENT '정렬순서',
  PRIMARY KEY (LPM_SEQ)
) 
CHARSET utf8
;


create table if not exists EB_LESSON_PLAN_DATA (
  LPD_SEQ int(11) DEFAULT NULL COMMENT '줄번호',
  LPD_LPM_SEQ int(11) DEFAULT NULL COMMENT '플랜코드',
  LPD_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LPD_NAME varchar(300) DEFAULT NULL COMMENT '플랜이름',
  LPD_ST_MONTH int(11) DEFAULT NULL COMMENT '시작월',
  LPD_ED_MONTH int(11) DEFAULT NULL COMMENT '종료월'
)
CHARSET utf8 
;


create table if not exists EB_LESSON_PLAN_DATA_191206copy (
  LPD_SEQ int(11) DEFAULT NULL COMMENT '줄번호',
  LPD_LPM_SEQ int(11) DEFAULT NULL COMMENT '플랜코드',
  LPD_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LPD_NAME varchar(300) DEFAULT NULL COMMENT '플랜이름',
  LPD_ST_MONTH int(11) DEFAULT NULL COMMENT '시작월',
  LPD_ED_MONTH int(11) DEFAULT NULL COMMENT '종료월'
)
CHARSET utf8 
,ROW_FORMAT COMPACT
;


create table if not exists EB_LESSON_PLAN_DATA_20200416copy (
  LPD_SEQ int(11) DEFAULT NULL COMMENT '줄번호',
  LPD_LPM_SEQ int(11) DEFAULT NULL COMMENT '플랜코드',
  LPD_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LPD_NAME varchar(300) DEFAULT NULL COMMENT '플랜이름',
  LPD_ST_MONTH int(11) DEFAULT NULL COMMENT '시작월',
  LPD_ED_MONTH int(11) DEFAULT NULL COMMENT '종료월'
) 
CHARSET utf8 
,ROW_FORMAT COMPACT
;


create table if not exists EB_LESSON_PLAN_DATA_210427 (
  LPD_SEQ int(11) DEFAULT NULL COMMENT '줄번호',
  LPD_LPM_SEQ int(11) DEFAULT NULL COMMENT '플랜코드',
  LPD_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LPD_NAME varchar(300) DEFAULT NULL COMMENT '플랜이름',
  LPD_ST_MONTH int(11) DEFAULT NULL COMMENT '시작월',
  LPD_ED_MONTH int(11) DEFAULT NULL COMMENT '종료월'
) 
 CHARSET utf8 
,ROW_FORMAT COMPACT
;


create table if not exists EB_LESSON_PLAN_DATA_copy (
  LPD_SEQ int(11) DEFAULT NULL COMMENT '줄번호',
  LPD_LPM_SEQ int(11) DEFAULT NULL COMMENT '플랜코드',
  LPD_BOOKSEQ int(11) DEFAULT NULL COMMENT '도서코드',
  LPD_NAME varchar(300) DEFAULT NULL COMMENT '플랜이름',
  LPD_ST_MONTH int(11) DEFAULT NULL COMMENT '시작월',
  LPD_ED_MONTH int(11) DEFAULT NULL COMMENT '종료월'
) 
CHARSET utf8 
,ROW_FORMAT COMPACT 
;


create table if not exists EB_LESSON_PLAN_copy (
  LPM_SEQ int(11) NOT NULL AUTO_INCREMENT COMMENT '순번',
  LPM_LEVEL varchar(50) DEFAULT NULL COMMENT '플랜등급',
  LPM_GUBUN varchar(50) DEFAULT NULL COMMENT '플랜구분',
  LPM_LESSON_YN enum('Y','N') DEFAULT 'Y' COMMENT '레슨사용여부',
  LPM_ORDER int(11) DEFAULT NULL COMMENT '정렬순서',
  PRIMARY KEY (LPM_SEQ)
) 
CHARSET utf8
;


create table if not exists EB_LEVELTEST (
  EL_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EL_MEMSEQ int(11) DEFAULT NULL,
  EL_MEMID varchar(50) DEFAULT NULL,
  EL_TESTYPE varchar(50) DEFAULT NULL,
  EL_TESTYPE_TXT varchar(50) DEFAULT NULL,
  EL_SCOREL varchar(50) DEFAULT NULL,
  EL_SCORER varchar(50) DEFAULT NULL,
  EL_SCORET varchar(50) DEFAULT NULL,
  EL_CRTLEVEL varchar(50) DEFAULT NULL,
  EL_COMMENT text,
  EL_DATE varchar(50) DEFAULT NULL,
  EL_CREDATE datetime NOT NULL,
  EL_LV varchar(50) DEFAULT NULL,
  EL_TESTTYPE_LVL varchar(50) DEFAULT NULL,
  EL_TESTTYPE_LVR varchar(50) DEFAULT NULL,
  EL_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  EL_IP varchar(50) NOT NULL,
  PRIMARY KEY (EL_SEQ)
)
CHARSET utf8 
,COMMENT '레벨테스트'
;


create table if not exists EB_LEVELTEST_ENG (
  EL_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EL_MEMSEQ int(11) DEFAULT NULL,
  EL_MEMID varchar(50) DEFAULT NULL,
  EL_TESTYPE varchar(50) DEFAULT NULL,
  EL_TESTYPE_TXT varchar(50) DEFAULT NULL,
  EL_SCOREL varchar(50) DEFAULT NULL,
  EL_SCORER varchar(50) DEFAULT NULL,
  EL_SCORET varchar(50) DEFAULT NULL,
  EL_CRTLEVEL varchar(50) DEFAULT NULL,
  EL_COMMENT mediumtext,
  EL_DATE varchar(50) DEFAULT NULL,
  EL_CREDATE datetime NOT NULL,
  EL_LV varchar(50) DEFAULT NULL,
  EL_TESTTYPE_LVL varchar(50) DEFAULT NULL,
  EL_TESTTYPE_LVR varchar(50) DEFAULT NULL,
  EL_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  EL_IP varchar(50) NOT NULL,
  PRIMARY KEY (EL_SEQ) USING BTREE
)
CHARSET utf8 
,ROW_FORMAT COMPACT 
,COMMENT '레벨테스트'
;


create table if not exists EB_MEMBER (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_OLD varchar(50) NOT NULL,
  PASS varchar(50) NOT NULL,
  PASS_YN enum('Y','N') NOT NULL DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '이용약관동의',
  AGREE_2 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보수집및이용동의',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'N' COMMENT '성별 및 생년월일 수집과 이용 동의',
  SUB_AGREE_2 enum('Y','N') DEFAULT 'N' COMMENT '직장명과 직장전화번호 수집과 이용동의',
  EMAIL_YN enum('Y','N') DEFAULT 'N' COMMENT '이메일 마케팅 수신동의',
  SMS_YN enum('Y','N') DEFAULT 'N' COMMENT 'sms마케팅수신동의',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  CORPSORT varchar(20) DEFAULT NULL COMMENT '소속구분',
  CERT_YN enum('Y','N') DEFAULT 'N' COMMENT 'sms인증 여부',
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'N',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO text,
  REAL_YN char(1) DEFAULT 'N',
  EBOOKMASTER_YN enum('Y','N') DEFAULT 'N' COMMENT '이북 권한',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y',
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ),
  KEY ID_NAME_EMAIL (ID,NAME,EMAIL)
) 
CHARSET utf8 
, ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_ (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') DEFAULT NULL COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS varchar(50) DEFAULT NULL,
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y' COMMENT '음력양력',
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '이용약관',
  AGREE_2 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보수집및이용동의',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보 수집및이용동의',
  SMS_YN enum('Y','N') DEFAULT 'Y' COMMENT 'SMS수신여부',
  EMAIL_YN enum('Y','N') DEFAULT 'Y' COMMENT '이메일수신여부',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'N',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  MEMO text,
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_BAKLAST (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_old varchar(50) DEFAULT NULL,
  PASS varchar(50) DEFAULT NULL,
  PASS_YN enum('Y','N') DEFAULT 'Y',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y',
  AGREE_2 enum('Y','N') DEFAULT 'Y',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN char(1) DEFAULT 'Y',
  SMS_YN char(1) DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') DEFAULT 'Y',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO text,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_BAK_180621 (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) NOT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_old varchar(500) DEFAULT NULL,
  PASS varchar(50) DEFAULT NULL,
  PASS_YN enum('Y','N') DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y' COMMENT '음력양력',
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '이용약관',
  AGREE_2 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보수집및이용동의',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN enum('Y','N') DEFAULT 'Y' COMMENT '개인정보 수집및이용동의',
  SMS_YN enum('Y','N') DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'N' COMMENT '구자료',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  MEMO text,
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_CERTIFY (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  EB_MEM_ID varchar(50) DEFAULT NULL COMMENT '회원 아이디',
  EB_MOBILE varchar(15) DEFAULT NULL COMMENT '휴대폰번호',
  EB_CERT_KEY varchar(5) DEFAULT NULL COMMENT '인증번호',
  EB_CERT_DT datetime DEFAULT NULL COMMENT '인증요청시간',
  EB_CERTIFY enum('Y','N') DEFAULT 'N' COMMENT '인증여부',
  EB_CHK_DT datetime DEFAULT NULL COMMENT '인증확인날짜',
  EB_COER_YN enum('Y','N') DEFAULT 'N' COMMENT '강제인증여부',
  PRIMARY KEY (EB_IDX)
) 
CHARSET utf8 
,COMMENT 'sms인증' 
;


create table if not exists EB_MEMBER_CERT_CD (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  EB_MEM_NAME varchar(50) DEFAULT NULL COMMENT '회원명',
  EB_MOBILE varchar(15) DEFAULT NULL COMMENT '휴대폰번호',
  EB_CERT_KEY varchar(5) DEFAULT NULL COMMENT '인증번호',
  EB_CERT_DT datetime DEFAULT NULL COMMENT '인증요청시간',
  EB_CERTIFY enum('Y','N') DEFAULT 'N' COMMENT '인증여부',
  EB_CHK_DT datetime DEFAULT NULL COMMENT '인증확인날짜',
  EB_AGREE enum('Y','N') DEFAULT 'N' COMMENT '개인정보인증',
  EB_AGR_DT datetime DEFAULT NULL COMMENT '인증확인날짜',
  PRIMARY KEY (EB_IDX) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'cd sms인증'
;


create table if not exists EB_MEMBER_GB (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS varchar(50) NOT NULL,
  PASS_YN enum('Y','N') NOT NULL DEFAULT 'N',
  F_NAME varchar(100) NOT NULL,
  L_NAME varchar(100) NOT NULL,
  COUNTRY varchar(100) DEFAULT '' COMMENT '국가',
  JOIN_LANG varchar(100) DEFAULT '' COMMENT '가입위치',
  COMPANY varchar(100) DEFAULT NULL COMMENT '기관명',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '이용약관동의',
  AGREE_2 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보수집및이용동의',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'N' COMMENT '성별 및 생년월일 수집과 이용 동의',
  SUB_AGREE_2 enum('Y','N') DEFAULT 'N' COMMENT '직장명과 직장전화번호 수집과 이용동의',
  EMAIL_YN enum('Y','N') DEFAULT 'N' COMMENT '이메일 마케팅 수신동의',
  SMS_YN enum('Y','N') DEFAULT 'N' COMMENT 'sms마케팅수신동의',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  CORPSORT varchar(20) DEFAULT NULL COMMENT '소속구분',
  CERT_YN enum('Y','N') DEFAULT 'N' COMMENT 'sms인증 여부',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO mediumtext,
  REAL_YN char(1) DEFAULT 'N',
  EBOOKMASTER_YN enum('Y','N') DEFAULT 'N' COMMENT '이북 권한',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y',
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ) USING BTREE,
  KEY ID_NAME_EMAIL (ID,F_NAME,EMAIL) USING BTREE
)
CHARSET utf8
;


create table if not exists EB_MEMBER_OLDDATA (
  EMO_SEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) unsigned NOT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MEMO text,
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (EMO_SEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_OLDDATA_BAK (
  EMO_SEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) unsigned NOT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MEMO text,
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (EMO_SEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_OLDDATA__ (
  EMO_SEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  MEMSEQ int(10) unsigned NOT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  MEMO text,
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (EMO_SEQ)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_copy (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_OLD varchar(50) NOT NULL,
  PASS varchar(50) NOT NULL,
  PASS_YN enum('Y','N') NOT NULL DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y',
  AGREE_2 enum('Y','N') DEFAULT 'Y',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN enum('Y','N') DEFAULT 'Y',
  SMS_YN enum('Y','N') DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') DEFAULT 'N',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO text,
  REAL_YN char(1) DEFAULT 'Y',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y',
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_copy0913 (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_OLD varchar(50) NOT NULL,
  PASS varchar(50) NOT NULL,
  PASS_YN enum('Y','N') NOT NULL DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y',
  AGREE_2 enum('Y','N') DEFAULT 'Y',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN enum('Y','N') DEFAULT 'Y',
  SMS_YN enum('Y','N') DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'N',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO text,
  REAL_YN char(1) DEFAULT 'N',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y',
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_copy190905 (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) DEFAULT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL DEFAULT 'E' COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_OLD varchar(50) NOT NULL,
  PASS varchar(50) NOT NULL,
  PASS_YN enum('Y','N') NOT NULL DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y',
  AGREE_2 enum('Y','N') DEFAULT 'Y',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN enum('Y','N') DEFAULT 'Y',
  SMS_YN enum('Y','N') DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'N',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  MEMO text,
  REAL_YN char(1) DEFAULT 'N',
  EBOOKMASTER_YN enum('Y','N') DEFAULT 'N' COMMENT '이북 권한',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y',
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ),
  KEY ID_NAME_EMAIL (ID,NAME,EMAIL)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MEMBER_copy_last (
  MEMSEQ int(10) unsigned NOT NULL AUTO_INCREMENT,
  SNS_KEY varchar(50) NOT NULL,
  JOIN_TYPE enum('F','N','E') NOT NULL COMMENT 'F:페이스북,N:네이버,E:사회평론',
  ID varchar(50) NOT NULL,
  PASS_old varchar(500) DEFAULT NULL,
  PASS varchar(50) DEFAULT NULL,
  PASS_YN enum('Y','N') DEFAULT 'N',
  NAME varchar(50) NOT NULL,
  BIRTHDATE date DEFAULT NULL,
  LUNAR_YN enum('Y','N') DEFAULT 'Y' COMMENT '음력양력',
  GENDER char(1) DEFAULT NULL,
  ADDRESS1 varchar(100) DEFAULT '',
  ADDRESS2 varchar(100) DEFAULT '',
  ZIPCODE char(7) DEFAULT '',
  PHONE varchar(15) DEFAULT '',
  MOBILE varchar(15) DEFAULT '',
  EMAIL varchar(50) DEFAULT '',
  AGREE_1 enum('Y','N') DEFAULT 'Y' COMMENT '이용약관',
  AGREE_2 enum('Y','N') DEFAULT 'Y' COMMENT '개인정보수집및이용동의',
  SUB_AGREE_1 enum('Y','N') DEFAULT 'Y',
  EMAIL_YN enum('Y','N') DEFAULT 'Y' COMMENT '개인정보 수집및이용동의',
  SMS_YN enum('Y','N') DEFAULT 'Y',
  MEMBERCODE varchar(10) DEFAULT '',
  JPHONE varchar(15) DEFAULT '',
  COMPANY varchar(100) DEFAULT NULL,
  OLD_YN enum('Y','N') NOT NULL DEFAULT 'Y' COMMENT '구자료',
  COUNT int(10) DEFAULT '0',
  LASTDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  MEMO text,
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP varchar(15) DEFAULT NULL,
  JOBCODE varchar(10) DEFAULT '',
  JADDRESS1 varchar(100) DEFAULT '',
  JADDRESS2 varchar(100) DEFAULT '',
  JZIPCODE char(7) DEFAULT '',
  TEACHITEM varchar(100) DEFAULT '',
  TEACHTARGET varchar(100) DEFAULT '',
  INTEREST01 varchar(100) DEFAULT '',
  INTEREST02 varchar(100) DEFAULT '',
  INTEREST03 varchar(100) DEFAULT '',
  INTEREST04 varchar(100) DEFAULT '',
  INTEREST05 varchar(100) DEFAULT '',
  INTEREST06 varchar(100) DEFAULT '',
  INTEREST07 varchar(100) DEFAULT '',
  TEACHINTEREST01 varchar(100) DEFAULT '',
  TEACHINTEREST02 varchar(100) DEFAULT '',
  TEMPBIRTH varchar(10) DEFAULT NULL,
  ISDORO char(1) DEFAULT NULL,
  PRIMARY KEY (MEMSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_MYBOOK (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  EB_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EB_ID varchar(50) NOT NULL COMMENT '회원id',
  EB_BOOKSEQ int(11) DEFAULT NULL COMMENT '책코드',
  EB_CODE varchar(50) DEFAULT NULL COMMENT '책분류코드',
  EB_CREDATE datetime NOT NULL COMMENT '등록일',
  EB_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EB_IDX) USING BTREE,
  KEY EUA_ID_EUA_CODE (EB_ID,EB_CODE) USING BTREE
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'mybook'
;


create table if not exists EB_MYBOOK_ENG (
  EB_IDX int(11) NOT NULL AUTO_INCREMENT,
  EB_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EB_ID varchar(50) NOT NULL COMMENT '회원id',
  EB_BOOKSEQ int(11) DEFAULT NULL COMMENT '책코드',
  EB_CODE varchar(50) DEFAULT NULL COMMENT '책분류코드',
  EB_CREDATE datetime NOT NULL COMMENT '등록일',
  EB_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EB_IDX) USING BTREE,
  KEY EUA_ID_EUA_CODE (EB_ID,EB_CODE) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'mybook' 
;


create table if not exists EB_NOTICE (
  EN_SEQ            int(10) NOT NULL AUTO_INCREMENT PRIMARY KEY
  , EN_ADMSEQ    int(10) DEFAULT 0 COMMENT '관리자 IDX'
  , EN_ADMNAME varchar(30)  COMMENT '관리자이름'
  , EN_TITLE         varchar(250) NOT NULL COMMENT '제목'
  , EN_CONTENTS mediumtext COMMENT '내용'
  , EN_NOTI_YN   enum('Y','N') DEFAULT 'N' COMMENT '중요여부'
  , EN_FILENAME varchar(50)  COMMENT '파일명'
  , EN_FILEPATH  varchar(50)  COMMENT '파일경로'
  , EN_REAL_YN  enum('Y','N') DEFAULT 'Y' COMMENT '삭제여부'
  , EN_CREDATE  datetime  COMMENT '등록일'
  , EN_UPDDATE  datetime  COMMENT '삭제일'
  , EN_CNT        int(10) DEFAULT '0' COMMENT '조회수'
  , EN_IP           char(15)  COMMENT 'IP'
)
CHARSET utf8 
, ROW_FORMAT DYNAMIC
;



create table if not exists EB_NOTICE_copy (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT,
  ADMSEQ int(10) DEFAULT '0',
  DEPTH int(10) DEFAULT '0',
  THREAD int(10) DEFAULT '0',
  NAME varchar(30) DEFAULT NULL,
  EMAIL varchar(30) DEFAULT NULL,
  GUBUN varchar(30) DEFAULT NULL,
  SUBJECT varchar(250) DEFAULT NULL,
  CONTENTS mediumtext,
  HTML_YN char(1) DEFAULT 'Y',
  NOTI_YN char(1) DEFAULT 'N',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  CNT int(10) DEFAULT '0',
  FILENAME varchar(50) DEFAULT NULL,
  COMCNT int(10) DEFAULT '0',
  MEMSEQ int(10) DEFAULT '0',
  PRIMARY KEY (NOTSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_NOTICE_copy2 (
  EN_SEQ int(10) NOT NULL AUTO_INCREMENT,
  EN_ADMSEQ int(10) DEFAULT '0' COMMENT '관리자 IDX',
  EN_ADMNAME varchar(30) DEFAULT NULL COMMENT '관리자이름',
  EN_TITLE varchar(250) DEFAULT NULL COMMENT '제목',
  EN_CONTENTS mediumtext COMMENT '내용',
  EN_NOTI_YN enum('Y','N') DEFAULT 'N' COMMENT '중요여부',
  EN_FILENAME varchar(50) DEFAULT NULL COMMENT '파일명',
  EN_FILEPATH varchar(50) DEFAULT NULL COMMENT '파일경로',
  EN_REAL_YN enum('Y','N') DEFAULT 'Y' COMMENT '삭제여부',
  EN_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EN_UPDDATE datetime DEFAULT NULL COMMENT '삭제일',
  EN_CNT int(10) DEFAULT '0' COMMENT '조회수',
  EN_IP char(15) DEFAULT NULL COMMENT 'IP',
  PRIMARY KEY (EN_SEQ)
) 
CHARSET utf8
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_NOTICE_copy_180702 (
  EN_SEQ int(10) NOT NULL AUTO_INCREMENT,
  EN_ADMSEQ int(10) DEFAULT '0' COMMENT '관리자 IDX',
  EN_ADMNAME varchar(30) DEFAULT NULL COMMENT '관리자이름',
  EN_TITLE varchar(250) DEFAULT NULL COMMENT '제목',
  EN_CONTENTS mediumtext COMMENT '내용',
  EN_NOTI_YN enum('Y','N') DEFAULT 'N' COMMENT '중요여부',
  EN_GUBUN varchar(50) DEFAULT NULL,
  EN_FILENAME varchar(50) DEFAULT NULL COMMENT '파일명',
  EN_FILEPATH varchar(50) DEFAULT NULL COMMENT '파일경로',
  EN_REAL_YN enum('Y','N') DEFAULT 'Y' COMMENT '삭제여부',
  EN_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EN_UPDDATE datetime DEFAULT NULL COMMENT '삭제일',
  EN_CNT int(10) DEFAULT '0' COMMENT '조회수',
  EN_IP char(15) DEFAULT NULL COMMENT 'IP',
  PRIMARY KEY (EN_SEQ)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
;


create table if not exists EB_ORDER (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill NOT NULL DEFAULT '0000000000',
  MEMBER_YN char(1) DEFAULT 'Y' COMMENT '회원여부 Y기본',
  MEMSEQ int(10) DEFAULT 0 COMMENT '주문자 seq',
  MEMID varchar(30) DEFAULT '' COMMENT '주문자 아이디',
  MEMNAME varchar(30) NOT NULL COMMENT '주문자 이름',
  ORDZIP varchar(7) DEFAULT NULL COMMENT '주문자 우편번호',
  ORDADDR1 varchar(100) DEFAULT NULL COMMENT '주문자 주소1',
  ORDADDR2 varchar(100) DEFAULT NULL COMMENT '주문자 주소2',
  ORDPHONE varchar(14) DEFAULT NULL COMMENT '주문자 연락처',
  ORDEMAIL varchar(50) DEFAULT NULL COMMENT '주문자 메일',
  ORDHP varchar(14) DEFAULT NULL COMMENT '주문자 핸드폰',
  DELINAME varchar(30) NOT NULL COMMENT '수신자 이름',
  DELIZIP char(7) NOT NULL COMMENT '수신자 우편번호',
  DELIADDR1 varchar(100) NOT NULL COMMENT '수신자 주소1',
  DELIADDR2 varchar(100) NOT NULL COMMENT '수신자 주소2',
  DELIPHONE char(14) DEFAULT '' COMMENT '수신자 연락처',
  DELIHP char(14) DEFAULT '' COMMENT '수신자 핸드폰',
  TOTALPRICE int(10) DEFAULT 0 COMMENT '주문금액(배달비포함) 구자료 배달비 미포함',
  PRICE int(10) DEFAULT 0 COMMENT '결제금액',
  DELIVERYCHARGE enum('Y','N') DEFAULT 'Y' COMMENT '배달비여부',
  SETTMETHOD_3 char(1) DEFAULT NULL COMMENT '1무통장 2신용카드 3 게좌이체 그외 사용안함',
  CARDNAME varchar(30) DEFAULT NULL COMMENT '카드이름 ',
  BANKCODE varchar(30) DEFAULT NULL COMMENT '송금은행',
  SETTNAME varchar(30) DEFAULT NULL COMMENT '입금자',
  ORDER_DELSEQ int(11) DEFAULT NULL COMMENT '택배사',
  ORDER_DELIVERYNO varchar(30) DEFAULT NULL COMMENT '송장번호',
  TRANSACTION varchar(50) DEFAULT NULL COMMENT '[이니시스]거래번호(이니시스 TID)',
  RETURNCODE varchar(20) DEFAULT NULL COMMENT '[이니시스]리턴코드 구(00) 신(0000)정상',
  CARDANO varchar(50) DEFAULT NULL COMMENT '[이니시스]신용카드 승인번호',
  CARDPERIOD varchar(2) DEFAULT NULL COMMENT '할부기간',
  CARDDATE datetime DEFAULT NULL COMMENT '승인일자',
  DATEORDE datetime DEFAULT NULL COMMENT '주문일자',
  DATESETT datetime DEFAULT NULL COMMENT '[이니시스] 결제일자',
  SETTSTATE_3 char(2) DEFAULT '1' COMMENT '결제결과 -> ( 1:주문중 2:결제실패 3:결제완료 4:배송지시 5:배송완료 6:주문취소 7:반품 8:최종배송완료 9:배송 중 )',
  MEMO text COMMENT '배송시 전달 메세지',
  CREDATE datetime DEFAULT NULL COMMENT '결제일',
  IP varchar(15) DEFAULT NULL COMMENT '아이피',
  REAL_YN char(1) DEFAULT 'Y' COMMENT '삭제여부',
  UPDDATE datetime DEFAULT NULL COMMENT '수정일',
  RESULTMEMO varchar(255) DEFAULT NULL COMMENT '[이니시스] 결제 결과 메세지',
  RESULTCODE varchar(100) DEFAULT NULL COMMENT '결과에러코드 [이니시스] 결제 실패시 -> 실패코드 , 결제 성공시 -> RESULTMEMO',
  VACCT varchar(30) NOT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금할 계좌번호',
  DTINPUT varchar(30) DEFAULT NULL COMMENT '입금 기한일',
  NMINPUT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금자명',
  NMVACCT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 예금주',
  CS_CONTENTS text COMMENT '관리자용 메모',
  INI_CHK enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'N결제전 Y결제후',
  ENDDATE datetime DEFAULT NULL COMMENT '배송일시',
  OLD_YN enum('Y','N') DEFAULT 'N' COMMENT '리뉴얼전 데이터',
  ESCRO_CHK char(1) DEFAULT 'N' COMMENT '에스크로 미사용',
  ESCRO_ORDER char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_TYPE char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_ID varchar(50) DEFAULT NULL COMMENT '에스크로 미사용',
  RESERVE_VALUE int(10) DEFAULT 0 COMMENT '사용한 적립금액 미사용',
  DELIEMAIL varchar(60) DEFAULT NULL COMMENT '수령인 이메일 미사용',
  MILEAGE_YN char(1) DEFAULT 'N' COMMENT '포인트몰 사용유무 미사용',
  RESERVE_YN char(1) DEFAULT 'N' COMMENT '적립금 사용유무 미사용',
  GIFT_PRICE varchar(20) DEFAULT NULL COMMENT '현재 주문에 적용된 사은품 가격 미사용',
  COUSEQ int(10) DEFAULT 0 COMMENT '사용한 쿠폰 시퀀스 미사용',
  NEEDDATE date DEFAULT NULL COMMENT '미사용',
  MILEAGE int(10) DEFAULT 0 COMMENT '포인트몰에서 사용한 적립금 미사용',
  COUPON_YN char(1) DEFAULT 'N' COMMENT '쿠폰 사용유무 미사용',
  SAVEPOINT int(10) DEFAULT 0 COMMENT '적립될 최종 적립금 미사용',
  PRIMARY KEY (SEQUENCE),
  KEY ORDSEQ_MEMSEQ_MEMID (ORDSEQ,MEMSEQ,MEMID)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'ORD 주문자 DEL 수령자'
;


create table if not exists EB_ORDERGOOD (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill DEFAULT '0000000000' COMMENT '주문번호',
  BOOKSEQ int(10) DEFAULT 0 COMMENT '책번호',
  SUBBOOKSEQ int(4) DEFAULT 0 COMMENT '부가도서번호',
  BOOKNAME varchar(255) DEFAULT '' COMMENT '책이름',
  BARCODE varchar(100) DEFAULT NULL COMMENT '바코드',
  SALEPRICE int(10) DEFAULT 0 COMMENT '결제가(개당)',
  PRICE int(10) DEFAULT 0 COMMENT '정상가(개당)',
  DISRATE int(10) DEFAULT 0 COMMENT '할인율',
  QUANTITY int(10) DEFAULT 0 COMMENT '수량',
  ORDERSTATE_6 char(1) DEFAULT '1' COMMENT '1:주문중 2:결제실패 3:결제완료 4:배송지시 5:배송완료 6:주문취소 7:반품 8:최종배송완료 9:배송 중',
  ORDERDATE datetime DEFAULT NULL COMMENT '주문날짜시간',
  DELSEQ int(6) unsigned zerofill DEFAULT '000000' COMMENT '택배사코드',
  DELIVERYNO varchar(30) DEFAULT '' COMMENT '송장번호',
  DATESETT datetime DEFAULT NULL COMMENT '[이니시스] 결제일자',
  DELIVERDATE datetime DEFAULT NULL COMMENT '배송지시일',
  ENDDATE datetime DEFAULT NULL COMMENT '배송일시',
  CANCELDATE datetime DEFAULT NULL COMMENT '취소날짜',
  RETURNDATE datetime DEFAULT NULL COMMENT '반품날짜',
  RETURNMEMO text COMMENT '반품메모',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '아이피',
  REAL_YN char(1) DEFAULT 'Y' COMMENT '삭제여부',
  MEMO text COMMENT '미사용',
  BUYPRICE int(10) DEFAULT 0 COMMENT '미사용',
  LIMIT_YN char(1) DEFAULT 'N' COMMENT '미사용',
  PRIMARY KEY (SEQUENCE),
  KEY ORDSEQ_BOOKSEQ_SUBBOOKSEQ_BOOKNAME (ORDSEQ,BOOKSEQ,SUBBOOKSEQ,BOOKNAME)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_ORDERGOOD_TMP (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill DEFAULT '0000000000',
  BOOKSEQ int(10) DEFAULT '0',
  SUBBOOKSEQ int(4) DEFAULT '0',
  BOOKNAME varchar(255) DEFAULT '',
  SALEPRICE int(10) DEFAULT '0',
  PRICE int(10) DEFAULT '0',
  DISRATE int(10) DEFAULT '0',
  QUANTITY int(10) DEFAULT '0',
  CREDATE datetime DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  PRIMARY KEY (SEQUENCE)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_ORDERNUM (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill NOT NULL DEFAULT '0000000000',
  MEMSEQ int(10) DEFAULT '0',
  PRIMARY KEY (SEQUENCE),
  KEY ORDSEQ_MEMSEQ (ORDSEQ,MEMSEQ)
) 
 CHARSET utf8 
,ROW_FORMAT FIXED
;


create table if not exists EB_ORDER_CANCEL (
  EOC_IDX int(11) NOT NULL AUTO_INCREMENT,
  EOC_ORDSEQ varchar(50) NOT NULL COMMENT '주문번호',
  EOC_PRICE int(11) NOT NULL DEFAULT '0' COMMENT '가격',
  EOC_BANKNAME varchar(50) DEFAULT NULL COMMENT '은행명',
  EOC_BANKNUM varchar(50) DEFAULT NULL COMMENT '계좌번호',
  EOC_BANKUSERNAME varchar(50) DEFAULT NULL COMMENT '입금자',
  EOC_CONTENT varchar(50) DEFAULT NULL COMMENT '취소사유',
  EOC_CREDATE datetime NOT NULL COMMENT '등록일',
  EOC_UPDDATE datetime DEFAULT NULL,
  EOC_CHK_YN enum('Y','N') NOT NULL DEFAULT 'N',
  EOC_TYPE enum('H','T','Y') NOT NULL DEFAULT 'H' COMMENT 'H: 홈페이지 T:: 전화 Y : 날짜지남',
  EOC_IP varchar(50) NOT NULL COMMENT '아이피',
  PRIMARY KEY (EOC_IDX)
) 
CHARSET utf8 
,COMMENT '취소요청' 
;


create table if not exists EB_ORDER_copy (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill NOT NULL DEFAULT '0000000000',
  MEMBER_YN char(1) DEFAULT 'Y' COMMENT '회원여부 Y기본',
  MEMSEQ int(10) DEFAULT 0 COMMENT '주문자 seq',
  MEMID varchar(30) DEFAULT '' COMMENT '주문자 아이디',
  MEMNAME varchar(30) NOT NULL COMMENT '주문자 이름',
  ORDZIP varchar(7) DEFAULT NULL COMMENT '주문자 우편번호',
  ORDADDR1 varchar(100) DEFAULT NULL COMMENT '주문자 주소1',
  ORDADDR2 varchar(100) DEFAULT NULL COMMENT '주문자 주소2',
  ORDPHONE varchar(14) DEFAULT NULL COMMENT '주문자 연락처',
  ORDEMAIL varchar(50) DEFAULT NULL COMMENT '주문자 메일',
  ORDHP varchar(14) DEFAULT NULL COMMENT '주문자 핸드폰',
  DELINAME varchar(30) NOT NULL COMMENT '수신자 이름',
  DELIZIP char(7) NOT NULL COMMENT '수신자 우편번호',
  DELIADDR1 varchar(100) NOT NULL COMMENT '수신자 주소1',
  DELIADDR2 varchar(100) NOT NULL COMMENT '수신자 주소2',
  DELIPHONE char(14) DEFAULT '' COMMENT '수신자 연락처',
  DELIHP char(14) DEFAULT '' COMMENT '수신자 핸드폰',
  TOTALPRICE int(10) DEFAULT 0 COMMENT '주문금액(배달비포함) 구자료 배달비 미포함',
  PRICE int(10) DEFAULT 0 COMMENT '결제금액',
  DELIVERYCHARGE enum('Y','N') DEFAULT 'Y' COMMENT '배달비여부',
  SETTMETHOD_3 char(1) DEFAULT NULL COMMENT '1무통장 2신용카드 3 게좌이체 그외 사용안함',
  CARDNAME varchar(30) DEFAULT NULL COMMENT '카드이름 ',
  BANKCODE varchar(30) DEFAULT NULL COMMENT '송금은행',
  SETTNAME varchar(30) DEFAULT NULL COMMENT '입금자',
  ORDER_DELSEQ int(11) DEFAULT NULL COMMENT '택배사',
  ORDER_DELIVERYNO varchar(30) DEFAULT NULL COMMENT '송장번호',
  TRANSACTION varchar(50) DEFAULT NULL COMMENT '[이니시스]거래번호(이니시스 TID)',
  RETURNCODE varchar(20) DEFAULT NULL COMMENT '[이니시스]리턴코드 구(00) 신(0000)정상',
  CARDANO varchar(50) DEFAULT NULL COMMENT '[이니시스]신용카드 승인번호',
  CARDPERIOD varchar(2) DEFAULT NULL COMMENT '할부기간',
  CARDDATE datetime DEFAULT NULL COMMENT '승인일자',
  DATEORDE datetime DEFAULT NULL COMMENT '주문일자',
  DATESETT datetime DEFAULT NULL COMMENT '[이니시스] 결제일자',
  SETTSTATE_3 char(2) DEFAULT '1' COMMENT '결제결과 -> ( 1:주문중 2:결제실패 3:결제완료 4:배송지시 5:배송완료 6:주문취소 7:반품 8:최종배송완료 9:배송 중 )',
  MEMO text COMMENT '배송시 전달 메세지',
  CREDATE datetime DEFAULT NULL COMMENT '결제일',
  IP varchar(15) DEFAULT NULL COMMENT '아이피',
  REAL_YN char(1) DEFAULT 'Y' COMMENT '삭제여부',
  UPDDATE datetime DEFAULT NULL COMMENT '수정일',
  RESULTMEMO varchar(255) DEFAULT NULL COMMENT '[이니시스] 결제 결과 메세지',
  RESULTCODE varchar(100) DEFAULT NULL COMMENT '결과에러코드 [이니시스] 결제 실패시 -> 실패코드 , 결제 성공시 -> RESULTMEMO',
  VACCT varchar(30) NOT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금할 계좌번호',
  DTINPUT varchar(30) DEFAULT NULL COMMENT '입금 기한일',
  NMINPUT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금자명',
  NMVACCT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 예금주',
  CS_CONTENTS text COMMENT '관리자용 메모',
  INI_CHK enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'N결제전 Y결제후',
  ENDDATE datetime DEFAULT NULL COMMENT '배송일시',
  OLD_YN enum('Y','N') DEFAULT 'N' COMMENT '리뉴얼전 데이터',
  ESCRO_CHK char(1) DEFAULT 'N' COMMENT '에스크로 미사용',
  ESCRO_ORDER char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_TYPE char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_ID varchar(50) DEFAULT NULL COMMENT '에스크로 미사용',
  RESERVE_VALUE int(10) DEFAULT 0 COMMENT '사용한 적립금액 미사용',
  DELIEMAIL varchar(60) DEFAULT NULL COMMENT '수령인 이메일 미사용',
  MILEAGE_YN char(1) DEFAULT 'N' COMMENT '포인트몰 사용유무 미사용',
  RESERVE_YN char(1) DEFAULT 'N' COMMENT '적립금 사용유무 미사용',
  GIFT_PRICE varchar(20) DEFAULT NULL COMMENT '현재 주문에 적용된 사은품 가격 미사용',
  COUSEQ int(10) DEFAULT 0 COMMENT '사용한 쿠폰 시퀀스 미사용',
  NEEDDATE date DEFAULT NULL COMMENT '미사용',
  MILEAGE int(10) DEFAULT 0 COMMENT '포인트몰에서 사용한 적립금 미사용',
  COUPON_YN char(1) DEFAULT 'N' COMMENT '쿠폰 사용유무 미사용',
  SAVEPOINT int(10) DEFAULT 0 COMMENT '적립될 최종 적립금 미사용',
  PRIMARY KEY (SEQUENCE),
  KEY ORDSEQ_MEMSEQ_MEMID (ORDSEQ,MEMSEQ,MEMID)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'ORD 주문자\r\r\nDEL 수령자'
;


create table if not exists EB_ORDER_copytest (
  SEQUENCE int(10) unsigned NOT NULL AUTO_INCREMENT,
  ORDSEQ int(10) unsigned zerofill NOT NULL DEFAULT '0000000000',
  MEMBER_YN char(1) DEFAULT 'Y' COMMENT '회원여부 Y기본',
  MEMSEQ int(10) DEFAULT '0' COMMENT '주문자 seq',
  MEMID varchar(30) DEFAULT '' COMMENT '주문자 아이디',
  MEMNAME varchar(30) NOT NULL COMMENT '주문자 이름',
  ORDZIP varchar(7) DEFAULT NULL COMMENT '주문자 우편번호',
  ORDADDR1 varchar(100) DEFAULT NULL COMMENT '주문자 주소1',
  ORDADDR2 varchar(100) DEFAULT NULL COMMENT '주문자 주소2',
  ORDPHONE varchar(14) DEFAULT NULL COMMENT '주문자 연락처',
  ORDEMAIL varchar(50) DEFAULT NULL COMMENT '주문자 메일',
  ORDHP varchar(14) DEFAULT NULL COMMENT '주문자 핸드폰',
  DELINAME varchar(30) NOT NULL COMMENT '수신자 이름',
  DELIZIP char(7) NOT NULL COMMENT '수신자 우편번호',
  DELIADDR1 varchar(100) NOT NULL COMMENT '수신자 주소1',
  DELIADDR2 varchar(100) NOT NULL COMMENT '수신자 주소2',
  DELIPHONE char(14) DEFAULT '' COMMENT '수신자 연락처',
  DELIHP char(14) DEFAULT '' COMMENT '수신자 핸드폰',
  TOTALPRICE int(10) DEFAULT '0' COMMENT '주문금액(배달비포함) 구자료 배달비 미포함',
  PRICE int(10) DEFAULT '0' COMMENT '결제금액',
  DELIVERYCHARGE enum('Y','N') DEFAULT 'Y' COMMENT '배달비여부',
  SETTMETHOD_3 char(1) DEFAULT NULL COMMENT '1무통장 2신용카드 3 게좌이체 그외 사용안함',
  CARDNAME varchar(30) DEFAULT NULL COMMENT '카드이름 ',
  BANKCODE varchar(30) DEFAULT NULL COMMENT '송금은행',
  SETTNAME varchar(30) DEFAULT NULL COMMENT '입금자',
  ORDER_DELSEQ int(11) DEFAULT NULL COMMENT '택배사',
  ORDER_DELIVERYNO varchar(30) DEFAULT NULL COMMENT '송장번호',
  TRANSACTION varchar(50) DEFAULT NULL COMMENT '[이니시스]거래번호(이니시스 TID)',
  RETURNCODE varchar(20) DEFAULT NULL COMMENT '[이니시스]리턴코드 구(00) 신(0000)정상',
  CARDANO varchar(50) DEFAULT NULL COMMENT '[이니시스]신용카드 승인번호',
  CARDPERIOD varchar(2) DEFAULT NULL COMMENT '할부기간',
  CARDDATE datetime DEFAULT NULL COMMENT '승인일자',
  DATEORDE datetime DEFAULT NULL COMMENT '주문일자',
  DATESETT datetime DEFAULT NULL COMMENT '[이니시스] 결제일자',
  SETTSTATE_3 char(2) DEFAULT '1' COMMENT '결제결과 -> ( 1:주문중 2:결제실패 3:결제완료 4:배송지시 5:배송완료 6:주문취소 7:반품 8:최종배송완료 9:배송 중 )',
  MEMO text COMMENT '배송시 전달 메세지',
  CREDATE datetime DEFAULT NULL COMMENT '결제일',
  IP varchar(15) DEFAULT NULL COMMENT '아이피',
  REAL_YN char(1) DEFAULT 'Y' COMMENT '삭제여부',
  UPDDATE datetime DEFAULT NULL COMMENT '수정일',
  RESULTMEMO varchar(255) DEFAULT NULL COMMENT '[이니시스] 결제 결과 메세지',
  RESULTCODE varchar(100) DEFAULT NULL COMMENT '결과에러코드 [이니시스] 결제 실패시 -> 실패코드 , 결제 성공시 -> RESULTMEMO',
  VACCT varchar(30) NOT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금할 계좌번호',
  DTINPUT varchar(30) DEFAULT NULL COMMENT '입금 기한일',
  NMINPUT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 입금자명',
  NMVACCT varchar(30) DEFAULT NULL COMMENT '[이니시스] 무통장 결제시 -> 예금주',
  CS_CONTENTS text COMMENT '관리자용 메모',
  INI_CHK enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'N결제전 Y결제후',
  ENDDATE datetime DEFAULT NULL COMMENT '배송일시',
  OLD_YN enum('Y','N') DEFAULT 'N' COMMENT '리뉴얼전 데이터',
  ESCRO_CHK char(1) DEFAULT 'N' COMMENT '에스크로 미사용',
  ESCRO_ORDER char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_TYPE char(1) DEFAULT '0' COMMENT '에스크로 미사용',
  ESCRO_ID varchar(50) DEFAULT NULL COMMENT '에스크로 미사용',
  RESERVE_VALUE int(10) DEFAULT '0' COMMENT '사용한 적립금액 미사용',
  DELIEMAIL varchar(60) DEFAULT NULL COMMENT '수령인 이메일 미사용',
  MILEAGE_YN char(1) DEFAULT 'N' COMMENT '포인트몰 사용유무 미사용',
  RESERVE_YN char(1) DEFAULT 'N' COMMENT '적립금 사용유무 미사용',
  GIFT_PRICE varchar(20) DEFAULT NULL COMMENT '현재 주문에 적용된 사은품 가격 미사용',
  COUSEQ int(10) DEFAULT '0' COMMENT '사용한 쿠폰 시퀀스 미사용',
  NEEDDATE date DEFAULT NULL COMMENT '미사용',
  MILEAGE int(10) DEFAULT '0' COMMENT '포인트몰에서 사용한 적립금 미사용',
  COUPON_YN char(1) DEFAULT 'N' COMMENT '쿠폰 사용유무 미사용',
  SAVEPOINT int(10) DEFAULT '0' COMMENT '적립될 최종 적립금 미사용',
  PRIMARY KEY (SEQUENCE),
  KEY ORDSEQ_MEMSEQ_MEMID (ORDSEQ,MEMSEQ,MEMID)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT 'ORD 주문자\r\nDEL 수령자'
;



CREATE ALGORITHM=UNDEFINED DEFINER=ebricks@localhost SQL SECURITY DEFINER VIEW EB_LESSONPLAN_VIEW AS select a.NOTSEQ AS NOTSEQ,a.ADMSEQ AS ADMSEQ,a.ADMNAME AS ADMNAME,a.BOOKSEQ AS BOOKSEQ,a.CREDATE AS CREDATE,a.IP AS IP,a.FILENAME AS FILENAME,a.FILEPATH AS FILEPATH,a.FILEGUBUN AS FILEGUBUN,a.FILESIZE AS FILESIZE,a.FILEEXT AS FILEEXT,a.GUBUN AS GUBUN,a.LANG AS LANG,a.ZIPGUBUN AS ZIPGUBUN,a.LESSONPLAN_CONTENTS AS LESSONPLAN_CONTENTS,a.VIEW_CNT AS VIEW_CNT,b.SKILL AS SKILL,b.BRANDNAME AS BRANDNAME,b.SERIES AS SERIES,b.READERS AS READERS,b.LEVEL AS LEVEL,b.LEVEL_F AS LEVEL_F,b.BOOKNAME AS BOOKNAME,b.BOOKTITLE AS BOOKTITLE,b.SALE AS SALE,b.IMGFILE2 AS IMGFILE from (EB_PDS a left join EB_BOOK_KOR b on((a.BOOKSEQ = b.BOOKSEQ))) where (b.REAL_YN = 'Y') ;


create table if not exists EB_PDS_210413 (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  ADMSEQ int(10) DEFAULT '0' COMMENT '등록자 번호',
  ADMNAME varchar(50) DEFAULT NULL COMMENT '등록자 이름',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '등록자 IP',
  FILENAME varchar(200) DEFAULT NULL COMMENT '파일이름',
  FILEPATH varchar(200) DEFAULT NULL COMMENT '파일경로',
  FILEGUBUN varchar(50) DEFAULT NULL COMMENT '파일구분',
  FILESIZE bigint(20) DEFAULT NULL COMMENT '파일용량',
  FILEEXT varchar(10) DEFAULT NULL COMMENT '파일확장자',
  GUBUN enum('A','T') DEFAULT 'A' COMMENT '사용분류',
  LANG varchar(50) DEFAULT 'AK' COMMENT '업로드분류',
  ZIPGUBUN enum('N','S','A','E') DEFAULT 'N' COMMENT '압축구분 N-압축아님, S-학생용압축, A-전체압축,E-다국어',
  LESSONPLAN_CONTENTS mediumtext COMMENT 'lessonplan내용',
  VIEW_CNT int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (NOTSEQ) USING BTREE,
  KEY BOOKSEQ_FILENAME_FILEGUBUN_LANG (BOOKSEQ,FILENAME,FILEGUBUN,LANG) USING BTREE
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_PDS_copy (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  ADMSEQ int(10) DEFAULT '0' COMMENT '등록자 번호',
  ADMNAME varchar(50) DEFAULT NULL COMMENT '등록자 이름',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '등록자 IP',
  FILENAME varchar(200) DEFAULT NULL COMMENT '파일이름',
  FILEPATH varchar(200) DEFAULT NULL COMMENT '파일경로',
  FILEGUBUN varchar(50) DEFAULT NULL COMMENT '파일구분',
  FILESIZE int(11) DEFAULT NULL COMMENT '파일용량',
  FILEEXT varchar(10) DEFAULT NULL COMMENT '파일확장자',
  GUBUN enum('A','T') DEFAULT 'A' COMMENT '사용분류',
  LANG varchar(10) DEFAULT 'A' COMMENT '업로드분류',
  ZIPGUBUN enum('N','S','A') DEFAULT 'N' COMMENT '압축구분 N-압축아님, S-학생용압축, A-전체압축',
  LESSONPLAN_CONTENTS mediumtext COMMENT 'lessonplan내용',
  VIEW_CNT int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (NOTSEQ)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_PDS_copy190904 (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  ADMSEQ int(10) DEFAULT '0' COMMENT '등록자 번호',
  ADMNAME varchar(50) DEFAULT NULL COMMENT '등록자 이름',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '등록자 IP',
  FILENAME varchar(200) DEFAULT NULL COMMENT '파일이름',
  FILEPATH varchar(200) DEFAULT NULL COMMENT '파일경로',
  FILEGUBUN varchar(50) DEFAULT NULL COMMENT '파일구분',
  FILESIZE int(11) DEFAULT NULL COMMENT '파일용량',
  FILEEXT varchar(10) DEFAULT NULL COMMENT '파일확장자',
  GUBUN enum('A','T') DEFAULT 'A' COMMENT '사용분류',
  LANG varchar(50) DEFAULT 'A' COMMENT '업로드분류',
  ZIPGUBUN enum('N','S','A') DEFAULT 'N' COMMENT '압축구분 N-압축아님, S-학생용압축, A-전체압축',
  LESSONPLAN_CONTENTS mediumtext COMMENT 'lessonplan내용',
  VIEW_CNT int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (NOTSEQ),
  KEY BOOKSEQ_FILENAME_FILEGUBUN_LANG (BOOKSEQ,FILENAME,FILEGUBUN,LANG)
)
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_PDS_copy190905 (
  NOTSEQ int(10) NOT NULL AUTO_INCREMENT COMMENT '순번',
  ADMSEQ int(10) DEFAULT '0' COMMENT '등록자 번호',
  ADMNAME varchar(50) DEFAULT NULL COMMENT '등록자 이름',
  BOOKSEQ int(10) DEFAULT '0' COMMENT '도서번호',
  CREDATE datetime DEFAULT NULL COMMENT '등록일',
  IP char(15) DEFAULT NULL COMMENT '등록자 IP',
  FILENAME varchar(200) DEFAULT NULL COMMENT '파일이름',
  FILEPATH varchar(200) DEFAULT NULL COMMENT '파일경로',
  FILEGUBUN varchar(50) DEFAULT NULL COMMENT '파일구분',
  FILESIZE int(11) DEFAULT NULL COMMENT '파일용량',
  FILEEXT varchar(10) DEFAULT NULL COMMENT '파일확장자',
  GUBUN enum('A','T') DEFAULT 'A' COMMENT '사용분류',
  LANG varchar(50) DEFAULT 'AK' COMMENT '업로드분류',
  ZIPGUBUN enum('N','S','A','E') DEFAULT 'N' COMMENT '압축구분 N-압축아님, S-학생용압축, A-전체압축',
  LESSONPLAN_CONTENTS mediumtext COMMENT 'lessonplan내용',
  VIEW_CNT int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (NOTSEQ),
  KEY BOOKSEQ_FILENAME_FILEGUBUN_LANG (BOOKSEQ,FILENAME,FILEGUBUN,LANG)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_QNA (
  EQ_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EQ_TYPE enum('Q','T','G') NOT NULL DEFAULT 'Q' COMMENT 'Q: 1:1, T: TEACHER''S ROOM,G:다국어',
  EQ_ING enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'Y:답변완료,N:답변대기',
  EQ_QSEQ int(11) DEFAULT NULL COMMENT '질문자IDX',
  EQ_QNAME varchar(50) NOT NULL COMMENT '질문자이름',
  EQ_QTYPE varchar(50) NOT NULL COMMENT '질문유형',
  EQ_QEAMIL varchar(100) NOT NULL COMMENT '질문자메일',
  EQ_QHP varchar(100) DEFAULT NULL COMMENT '질문자연락처',
  EQ_QTITLE varchar(100) NOT NULL COMMENT '질문자제목',
  EQ_QCONTENTS mediumtext NOT NULL COMMENT '질문자내용',
  EQ_ZIPCODE varchar(50) DEFAULT NULL,
  EQ_ADDR1 varchar(50) DEFAULT NULL,
  EQ_ADDR2 varchar(50) DEFAULT NULL,
  EQ_QFILEPATH varchar(100) DEFAULT NULL COMMENT '질문자 파일경로',
  EQ_QFILENAME varchar(100) DEFAULT NULL COMMENT '질문자 파일명',
  EQ_QCREDATE datetime NOT NULL COMMENT '질문등록일',
  EQ_QIP varchar(15) NOT NULL COMMENT '질문자 IP',
  EQ_ASEQ int(11) NOT NULL COMMENT '답변자IDX',
  EQ_ANAME varchar(50) DEFAULT NULL COMMENT '답변자이름',
  EQ_ATITLE varchar(100) DEFAULT NULL COMMENT '답변자제목',
  EQ_ACONTENTS mediumtext COMMENT '답변자내용',
  EQ_AFILEPATH varchar(100) DEFAULT NULL COMMENT '답변자 파일경로',
  EQ_AFILENAME varchar(100) DEFAULT NULL COMMENT '답변자 파일명',
  EQ_ACREDATE datetime NOT NULL COMMENT '답변등록일',
  EQ_AUPDATE datetime DEFAULT NULL COMMENT '답변수정일',
  EQ_AIP varchar(15) NOT NULL COMMENT '답변자 IP',
  EQ_VIEW_CNT int(11) DEFAULT '0' COMMENT '조회수',
  EQ_VIEW_YN enum('Y','N') DEFAULT 'N',
  EQ_ADMIN_MEMO text COMMENT '관리자메모',
  EQ_ADMIN_MEMO_DATE datetime DEFAULT NULL,
  EQ_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  PRIMARY KEY (EQ_SEQ),
  KEY EQ_QSEQ_EQ_QNAME_EQ_QHP_EQ_QTITLE_EQ_ASEQ_EQ_ANAME (EQ_QSEQ,EQ_QNAME,EQ_QHP,EQ_QTITLE,EQ_ASEQ,EQ_ANAME)
) 
CHARSET utf8 
,COMMENT 'QNA게시판\r\n다국어코드 G는 Contact Us 내용임.'
;


create table if not exists EB_QNA_210222 (
  EQ_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EQ_TYPE enum('Q','T','G') NOT NULL DEFAULT 'Q' COMMENT 'Q: 1:1, T: TEACHER''S ROOM,G:다국어',
  EQ_ING enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'Y:답변완료,N:답변대기',
  EQ_QSEQ int(11) DEFAULT NULL COMMENT '질문자IDX',
  EQ_QNAME varchar(50) NOT NULL COMMENT '질문자이름',
  EQ_QTYPE varchar(50) NOT NULL COMMENT '질문유형',
  EQ_QEAMIL varchar(100) NOT NULL COMMENT '질문자메일',
  EQ_QHP varchar(100) DEFAULT NULL COMMENT '질문자연락처',
  EQ_QTITLE varchar(100) NOT NULL COMMENT '질문자제목',
  EQ_QCONTENTS mediumtext NOT NULL COMMENT '질문자내용',
  EQ_ZIPCODE varchar(50) DEFAULT NULL,
  EQ_ADDR1 varchar(50) DEFAULT NULL,
  EQ_ADDR2 varchar(50) DEFAULT NULL,
  EQ_QFILEPATH varchar(100) DEFAULT NULL COMMENT '질문자 파일경로',
  EQ_QFILENAME varchar(100) DEFAULT NULL COMMENT '질문자 파일명',
  EQ_QCREDATE datetime NOT NULL COMMENT '질문등록일',
  EQ_QIP varchar(15) NOT NULL COMMENT '질문자 IP',
  EQ_ASEQ int(11) NOT NULL COMMENT '답변자IDX',
  EQ_ANAME varchar(50) DEFAULT NULL COMMENT '답변자이름',
  EQ_ATITLE varchar(100) DEFAULT NULL COMMENT '답변자제목',
  EQ_ACONTENTS mediumtext COMMENT '답변자내용',
  EQ_AFILEPATH varchar(100) DEFAULT NULL COMMENT '답변자 파일경로',
  EQ_AFILENAME varchar(100) DEFAULT NULL COMMENT '답변자 파일명',
  EQ_ACREDATE datetime NOT NULL COMMENT '답변등록일',
  EQ_AUPDATE datetime DEFAULT NULL COMMENT '답변수정일',
  EQ_AIP varchar(15) NOT NULL COMMENT '답변자 IP',
  EQ_VIEW_CNT int(11) DEFAULT '0' COMMENT '조회수',
  EQ_VIEW_YN enum('Y','N') DEFAULT 'N',
  EQ_ADMIN_MEMO mediumtext COMMENT '관리자메모',
  EQ_ADMIN_MEMO_DATE datetime DEFAULT NULL,
  EQ_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  PRIMARY KEY (EQ_SEQ) USING BTREE,
  KEY EQ_QSEQ_EQ_QNAME_EQ_QHP_EQ_QTITLE_EQ_ASEQ_EQ_ANAME (EQ_QSEQ,EQ_QNAME,EQ_QHP,EQ_QTITLE,EQ_ASEQ,EQ_ANAME) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT COMPACT 
,COMMENT 'QNA게시판\r\n다국어코드 G는 Contact Us 내용임.'
;


create table if not exists EB_QNA_GB (
  EQ_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EQ_TYPE enum('Q','T','G') NOT NULL DEFAULT 'Q' COMMENT 'Q: 1:1, T: TEACHER''S ROOM',
  EQ_ING enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'Y:답변완료,N:답변대기',
  EQ_QSEQ int(11) DEFAULT NULL COMMENT '질문자IDX',
  EQ_QNAME varchar(50) NOT NULL COMMENT '질문자이름',
  EQ_QTYPE varchar(50) NOT NULL COMMENT '질문유형',
  EQ_QEAMIL varchar(100) NOT NULL COMMENT '질문자메일',
  EQ_QTITLE varchar(100) NOT NULL COMMENT '질문자제목',
  EQ_QCONTENTS mediumtext NOT NULL COMMENT '질문자내용',
  EQ_ZIPCODE varchar(50) DEFAULT NULL,
  EQ_ADDR1 varchar(50) DEFAULT NULL,
  EQ_ADDR2 varchar(50) DEFAULT NULL,
  EQ_QFILEPATH varchar(100) DEFAULT NULL COMMENT '질문자 파일경로',
  EQ_QFILENAME varchar(100) DEFAULT NULL COMMENT '질문자 파일명',
  EQ_QCREDATE datetime NOT NULL COMMENT '질문등록일',
  EQ_QIP varchar(15) NOT NULL COMMENT '질문자 IP',
  EQ_ASEQ int(11) NOT NULL COMMENT '답변자IDX',
  EQ_ANAME varchar(50) DEFAULT NULL COMMENT '답변자이름',
  EQ_ATITLE varchar(100) DEFAULT NULL COMMENT '답변자제목',
  EQ_ACONTENTS mediumtext COMMENT '답변자내용',
  EQ_AFILEPATH varchar(100) DEFAULT NULL COMMENT '답변자 파일경로',
  EQ_AFILENAME varchar(100) DEFAULT NULL COMMENT '답변자 파일명',
  EQ_ACREDATE datetime NOT NULL COMMENT '답변등록일',
  EQ_AUPDATE datetime DEFAULT NULL COMMENT '답변수정일',
  EQ_AIP varchar(15) NOT NULL COMMENT '답변자 IP',
  EQ_VIEW_CNT int(11) DEFAULT '0' COMMENT '조회수',
  EQ_VIEW_YN enum('Y','N') DEFAULT 'N',
  EQ_ADMIN_MEMO mediumtext COMMENT '관리자메모',
  EQ_ADMIN_MEMO_DATE datetime DEFAULT NULL,
  EQ_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  PRIMARY KEY (EQ_SEQ) USING BTREE,
  KEY EQ_QSEQ_EQ_QNAME_EQ_QHP_EQ_QTITLE_EQ_ASEQ_EQ_ANAME (EQ_QSEQ,EQ_QNAME,EQ_QTITLE,EQ_ASEQ,EQ_ANAME) USING BTREE
) 
CHARSET utf8 
,COMMENT '다국어 Q&A'
;


create table if not exists EB_QNA_GB_backup (
  EQ_SEQ int(11) NOT NULL AUTO_INCREMENT,
  EQ_TYPE enum('Q','T','G') NOT NULL DEFAULT 'Q' COMMENT 'Q: 1:1, T: TEACHER''S ROOM',
  EQ_ING enum('Y','N') NOT NULL DEFAULT 'N' COMMENT 'Y:답변완료,N:답변대기',
  EQ_QSEQ int(11) DEFAULT NULL COMMENT '질문자IDX',
  EQ_QNAME varchar(50) NOT NULL COMMENT '질문자이름',
  EQ_QTYPE varchar(50) NOT NULL COMMENT '질문유형',
  EQ_QEAMIL varchar(100) NOT NULL COMMENT '질문자메일',
  EQ_QTITLE varchar(100) NOT NULL COMMENT '질문자제목',
  EQ_QCONTENTS mediumtext NOT NULL COMMENT '질문자내용',
  EQ_ZIPCODE varchar(50) DEFAULT NULL,
  EQ_ADDR1 varchar(50) DEFAULT NULL,
  EQ_ADDR2 varchar(50) DEFAULT NULL,
  EQ_QFILEPATH varchar(100) DEFAULT NULL COMMENT '질문자 파일경로',
  EQ_QFILENAME varchar(100) DEFAULT NULL COMMENT '질문자 파일명',
  EQ_QCREDATE datetime NOT NULL COMMENT '질문등록일',
  EQ_QIP varchar(15) NOT NULL COMMENT '질문자 IP',
  EQ_ASEQ int(11) NOT NULL COMMENT '답변자IDX',
  EQ_ANAME varchar(50) DEFAULT NULL COMMENT '답변자이름',
  EQ_ATITLE varchar(100) DEFAULT NULL COMMENT '답변자제목',
  EQ_ACONTENTS mediumtext COMMENT '답변자내용',
  EQ_AFILEPATH varchar(100) DEFAULT NULL COMMENT '답변자 파일경로',
  EQ_AFILENAME varchar(100) DEFAULT NULL COMMENT '답변자 파일명',
  EQ_ACREDATE datetime NOT NULL COMMENT '답변등록일',
  EQ_AUPDATE datetime DEFAULT NULL COMMENT '답변수정일',
  EQ_AIP varchar(15) NOT NULL COMMENT '답변자 IP',
  EQ_VIEW_CNT int(11) DEFAULT '0' COMMENT '조회수',
  EQ_VIEW_YN enum('Y','N') DEFAULT 'N',
  EQ_ADMIN_MEMO mediumtext COMMENT '관리자메모',
  EQ_ADMIN_MEMO_DATE datetime DEFAULT NULL,
  EQ_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  PRIMARY KEY (EQ_SEQ) USING BTREE,
  KEY EQ_QSEQ_EQ_QNAME_EQ_QHP_EQ_QTITLE_EQ_ASEQ_EQ_ANAME (EQ_QSEQ,EQ_QNAME,EQ_QTITLE,EQ_ASEQ,EQ_ANAME) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT '다국어 Q&A'
;


create table if not exists EB_REVIEW (
  REVSEQ int(10) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(10) NOT NULL,
  BOOKNAME varchar(255) DEFAULT NULL,
  MEMSEQ int(10) NOT NULL,
  DEPTH int(10) DEFAULT '0',
  THREAD int(10) NOT NULL DEFAULT '0',
  ID varchar(30) DEFAULT NULL,
  NAME varchar(30) DEFAULT NULL,
  SUBJECT varchar(200) DEFAULT NULL,
  CONTENTS mediumtext,
  SCORE int(10) DEFAULT '5',
  HTML_YN char(1) DEFAULT 'N',
  APP_YN char(1) DEFAULT 'N',
  CNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  COMCNT int(10) DEFAULT '0',
  FILENAME varchar(255) DEFAULT NULL,
  PRIMARY KEY (REVSEQ)
)  
CHARSET utf8 
,ROW_FORMAT DYNAMIC
;


create table if not exists EB_REVIEW_ (
  REVSEQ int(10) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(10) NOT NULL,
  BOOKNAME varchar(255) DEFAULT NULL,
  MEMSEQ int(10) NOT NULL,
  DEPTH int(10) DEFAULT '0',
  THREAD int(10) NOT NULL DEFAULT '0',
  ID varchar(30) DEFAULT NULL,
  NAME varchar(30) DEFAULT NULL,
  SUBJECT varchar(200) DEFAULT NULL,
  CONTENTS mediumtext,
  SCORE int(10) DEFAULT '5',
  HTML_YN char(1) DEFAULT 'N',
  APP_YN char(1) DEFAULT 'N',
  CNT int(10) DEFAULT '0',
  UPDDATE datetime DEFAULT NULL,
  CREDATE datetime DEFAULT NULL,
  IP char(15) DEFAULT NULL,
  REAL_YN char(1) DEFAULT 'Y',
  COMCNT int(10) DEFAULT '0',
  FILENAME varchar(255) DEFAULT NULL,
  PRIMARY KEY (REVSEQ)
) 
CHARSET utf8 
,COMMENT '리뷰' 
;


create table if not exists  EB_SEMINAR (
  ES_SEQ int(11) NOT NULL AUTO_INCREMENT,
  ES_TYPE enum('C','Y') NOT NULL COMMENT 'C:컨텐츠 Y:유튜브',
  ES_ADMSEQ int(11) NOT NULL COMMENT '관리자SEQ',
  ES_ADMNAME varchar(50) NOT NULL COMMENT '관리자이름',
  ES_TITLE varchar(200) NOT NULL COMMENT '제목',
  ES_CONTENTS mediumtext COMMENT '내용',
  ES_LINK varchar(500) DEFAULT NULL COMMENT '동영상링크',
  ES_CREDATE datetime NOT NULL COMMENT '등록일',
  ES_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  ES_UPDDATE datetime DEFAULT NULL COMMENT '수정일',
  ES_CNT int(11) NOT NULL DEFAULT '0' COMMENT '조회수',
  ES_IP varchar(15) NOT NULL COMMENT '아이피',
  PRIMARY KEY (ES_SEQ)
) 
CHARSET utf8 
,COMMENT='세미나 종합' 
;


create table if not exists EB_SUBBOOK (
  ES_SEQ int(11) NOT NULL AUTO_INCREMENT,
  BOOKSEQ int(11) NOT NULL,
  SUB_BOOKSEQ int(11) NOT NULL,
  SUB_NOTSEQ int(11) DEFAULT NULL,
  CREDATE datetime NOT NULL,
  PRIMARY KEY (ES_SEQ),
  KEY BOOKSEQ_SUB_BOOKSEQ_SUB_NOTSEQ (BOOKSEQ,SUB_BOOKSEQ,SUB_NOTSEQ)
) CHARSET utf8
,COMMENT '병행도서'
;



create table if not exists EB_TEST (
  e_code varchar(50) DEFAULT NULL
) 
CHARSET utf8
;


create table if not exists  EB_TEST2 (
  i_code varchar(50) DEFAULT NULL
) 
CHARSET utf8
;


create table if not exists EB_TOP_MENU_BOOK (
  ETMB_IDX int(11) NOT NULL AUTO_INCREMENT,
  ETMB_ETMS_IDX int(11) NOT NULL DEFAULT '0' COMMENT 'ETMS코드',
  ETMB_SKILL varchar(50) NOT NULL COMMENT '스킬',
  ETMB_SERIESNAME varchar(50) NOT NULL COMMENT '시리즈명',
  ETMB_BOOKNAME varchar(100) NOT NULL COMMENT '책이름',
  ETMB_BOOKSEQ int(11) NOT NULL DEFAULT '0' COMMENT '책코드',
  ETMB_NOTSEQ int(11) NOT NULL DEFAULT '0' COMMENT '상세코드',
  ETMB_ORDNUM int(11) NOT NULL DEFAULT '0' COMMENT '순번',
  ETMB_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y',
  ETMB_CREDATE datetime NOT NULL,
  PRIMARY KEY (ETMB_IDX),
  KEY ETMB_SKILL_ETMB_BOOKSEQ_ETMB_NOTSEQ (ETMB_SKILL,ETMB_BOOKSEQ,ETMB_NOTSEQ)
)
CHARSET utf8 
,COMMENT '상단 도서관리'
;


create table if not exists EB_TOP_MENU_SERIES (
  ETMS_IDX int(11) NOT NULL AUTO_INCREMENT,
  ETMS_TYPE enum('Y','N') NOT NULL DEFAULT 'Y' COMMENT 'Y 시리즈 N 레벨',
  ETMS_SKILL varchar(50) NOT NULL COMMENT '카테고리',
  ETMS_TITLE varchar(50) NOT NULL COMMENT '시리즈명',
  ETMS_SERIESCODE varchar(50) DEFAULT NULL COMMENT '기존시리즈일 경우 코드',
  ETMS_ORDNUM int(11) NOT NULL DEFAULT '0' COMMENT '순번',
  ETMS_CREDATE datetime NOT NULL COMMENT '등록일자',
  ETMS_IP varchar(50) NOT NULL COMMENT '아이피',
  ETMS_REAL_YN enum('Y','N') NOT NULL DEFAULT 'Y' COMMENT '삭제여부',
  PRIMARY KEY (ETMS_IDX),
  KEY ETMS_SKILL_ETMS_SERIESCODE (ETMS_SKILL,ETMS_SERIESCODE)
) 
CHARSET utf8 
,COMMENT='시리즈정리' 
;


create table if not exists  EB_USER_ACCESSCODE (
  EUA_IDX int(11) NOT NULL AUTO_INCREMENT,
  EUA_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EUA_ID varchar(50) NOT NULL COMMENT '회원id',
  EUA_BOOKSEQ int(11) NOT NULL COMMENT '책코드',
  EUA_CODE varchar(50) NOT NULL COMMENT '인증코드',
  EUA_CREDATE datetime NOT NULL COMMENT '등록일',
  EUA_IP varchar(15) NOT NULL COMMENT '아이피',
  PRIMARY KEY (EUA_IDX),
  KEY EUA_ID_EUA_CODE (EUA_ID,EUA_CODE)
)
CHARSET utf8 
,COMMENT '인증코드' 
;


create table if not exists EB_USER_ACCESSCODE_GB (
  EUA_IDX int(11) NOT NULL AUTO_INCREMENT,
  EUA_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EUA_ID varchar(50) NOT NULL COMMENT '회원id',
  EUA_BOOKSEQ int(11) NOT NULL COMMENT '책코드',
  EUA_CODE varchar(50) NOT NULL COMMENT '인증코드',
  EUA_CREDATE datetime NOT NULL COMMENT '등록일',
  EUA_IP varchar(15) NOT NULL COMMENT '아이피',
  PRIMARY KEY (EUA_IDX) USING BTREE,
  KEY EUA_ID_EUA_CODE (EUA_ID,EUA_CODE) USING BTREE
) 
 CHARSET utf8 
,COMMENT='인증코드' 
;


create table if not exists EB_USER_ACCESSCODE_LOG (
  EUA_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EUA_ID varchar(50) NOT NULL COMMENT '회원id',
  EUA_BOOKSEQ int(11) NOT NULL COMMENT '책코드',
  EUA_CODE varchar(50) NOT NULL COMMENT '인증코드',
  EUA_CREDATE datetime NOT NULL COMMENT '등록일',
  EUA_IP varchar(15) NOT NULL COMMENT '아이피',
  EUA_MEMO text NOT NULL
) 
CHARSET utf8
, COMMENT '인증 로그'
;


create table if not exists EB_USER_ACCESSCODE_LOG_GB (
  EUA_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EUA_ID varchar(50) NOT NULL COMMENT '회원id',
  EUA_BOOKSEQ int(11) NOT NULL COMMENT '책코드',
  EUA_CODE varchar(50) NOT NULL COMMENT '인증코드',
  EUA_CREDATE datetime NOT NULL COMMENT '등록일',
  EUA_IP varchar(15) NOT NULL COMMENT '아이피',
  EUA_MEMO mediumtext NOT NULL
)
CHARSET utf8
, COMMENT '인증 로그'  
;


create table if not exists EB_USER_ACCESSCODE_copy (
  EUA_IDX int(11) NOT NULL AUTO_INCREMENT,
  EUA_MEMSEQ int(11) NOT NULL COMMENT '회원idx',
  EUA_ID varchar(50) NOT NULL COMMENT '회원id',
  EUA_BOOKSEQ int(11) NOT NULL COMMENT '책코드',
  EUA_CODE varchar(50) NOT NULL COMMENT '인증코드',
  EUA_CREDATE datetime NOT NULL COMMENT '등록일',
  EUA_IP varchar(15) NOT NULL COMMENT '아이피',
  PRIMARY KEY (EUA_IDX),
  KEY EUA_ID_EUA_CODE (EUA_ID,EUA_CODE)
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT='인증코드'
;


create table if not exists EB_VOD (
  EV_IDX int(11) NOT NULL AUTO_INCREMENT,
  EV_BOOKSEQ int(10) unsigned DEFAULT '0' COMMENT 'book seq',
  EV_CR_ADMIN varchar(50) DEFAULT NULL COMMENT '등록 관리자 seq',
  EV_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EV_UPDATE datetime DEFAULT NULL COMMENT '수정일',
  EV_UP_ADMIN varchar(50) DEFAULT NULL COMMENT '수정 관리자 seq',
  EV_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EV_IDX) USING BTREE,
  KEY BOOKSEQ (EV_BOOKSEQ) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT='VOD 관리'
;


create table if not exists EB_VOD_INFO (
  EVI_IDX int(11) NOT NULL AUTO_INCREMENT,
  EVI_EV_IDX int(10) unsigned DEFAULT '0' COMMENT 'VOD IDX',
  EVI_VOD_TYPE varchar(3) DEFAULT NULL COMMENT 'VOD 타입',
  EVI_VOD_NAME varchar(500) DEFAULT NULL COMMENT 'VOD 파일명',
  EVI_UNIT_TYPE varchar(50) DEFAULT NULL COMMENT 'UNIT 타입',
  EVI_UNIT int(11) DEFAULT NULL COMMENT 'UNIT 번호',
  EVI_NUM int(11) DEFAULT NULL COMMENT '등록번호',
  EVI_MEMO varchar(500) DEFAULT NULL COMMENT '메모',
  EVI_URL varchar(500) DEFAULT NULL COMMENT 's3 url',
  EVI_CR_ADMIN varchar(50) DEFAULT NULL COMMENT '등록 관리자 seq',
  EVI_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EVI_UPDATE datetime DEFAULT NULL COMMENT '수정일',
  EVI_UP_ADMIN varchar(50) DEFAULT NULL COMMENT '수정 관리자 seq',
  EVI_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EVI_IDX) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC 
,COMMENT='VOD 정보'
;


create table if not exists EB_VOD_INFO_DEL (
  EVI_IDX int(11) NOT NULL AUTO_INCREMENT,
  EVI_EV_IDX int(10) unsigned DEFAULT '0' COMMENT 'VOD IDX',
  EVI_VOD_TYPE varchar(3) DEFAULT NULL COMMENT 'VOD 타입',
  EVI_VOD_NAME varchar(500) DEFAULT NULL COMMENT 'VOD 파일명',
  EVI_UNIT_TYPE varchar(50) DEFAULT NULL COMMENT 'UNIT 타입',
  EVI_UNIT int(11) DEFAULT NULL COMMENT 'UNIT 번호',
  EVI_NUM int(11) DEFAULT NULL COMMENT '등록번호',
  EVI_MEMO varchar(500) DEFAULT NULL COMMENT '메모',
  EVI_URL varchar(500) DEFAULT NULL COMMENT 's3 url',
  EVI_CR_ADMIN varchar(50) DEFAULT NULL COMMENT '등록 관리자 seq',
  EVI_CREDATE datetime DEFAULT NULL COMMENT '등록일',
  EVI_UPDATE datetime DEFAULT NULL COMMENT '수정일',
  EVI_UP_ADMIN varchar(50) DEFAULT NULL COMMENT '수정 관리자 seq',
  EVI_REAL_YN char(1) DEFAULT 'Y' COMMENT '사용여부',
  PRIMARY KEY (EVI_IDX) USING BTREE
) 
CHARSET utf8 
,ROW_FORMAT DYNAMIC
, COMMENT 'VOD 정보' 
;


create table if not exists IG_log_url (
  idx int(11) NOT NULL AUTO_INCREMENT,
  url varchar(1000) DEFAULT NULL,
  data varchar(1000) DEFAULT NULL,
  credate datetime DEFAULT NULL,
  ip varchar(50) DEFAULT NULL,
  PRIMARY KEY (idx)
) 
CHARSET utf8
;


create table if not exists ci_sessions (
  id varchar(40) NOT NULL,
  ip_address varchar(45) NOT NULL,
  timestamp int(10) unsigned NOT NULL DEFAULT '0',
  data blob NOT NULL,
  KEY ci_sessions_timestamp (timestamp)
)
CHARSET utf8
;


create table if not exists ci_sessions_ (
  session_id varchar(40) NOT NULL DEFAULT '0',
  ip_address varchar(16) NOT NULL DEFAULT '0',
  user_agent varchar(120) NOT NULL,
  last_activity int(10) unsigned NOT NULL DEFAULT '0',
  user_data text NOT NULL,
  PRIMARY KEY (session_id),
  KEY last_activity_idx (last_activity)
)
CHARSET utf8
;


create table if not exists jschoi_test (
  num int(11) DEFAULT NULL,
  skill varchar(50) DEFAULT NULL,
  skllcode varchar(50) DEFAULT NULL,
  code varchar(50) DEFAULT NULL,
  title varchar(50) DEFAULT NULL
) 
CHARSET utf8 
,COMMENT '테스트'
;


create table if not exists test_ab (
  a varchar(50) DEFAULT NULL,
  b text
)
 CHARSET utf8
;
