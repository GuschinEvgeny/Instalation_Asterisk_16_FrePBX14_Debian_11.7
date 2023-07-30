# First of all you can install DEBIAN system. The instruction describes the work on 11.7

## Install Debian 11 required packages.
"apt -y install locales sngrep build-essential aptitude openssh-server apache2 mariadb-server mariadb-client bison doxygen flex php-pear curl sox libncurses5-dev libssl-dev libmariadb-dev mpg123 libxml2-dev libnewt-dev sqlite3 libsqlite3-dev pkg-config automake libtool-bin autoconf git subversion uuid uuid-dev libiksemel-dev tftpd postfix mailutils nano ntp libspandsp-dev libcurl4-openssl-dev libical-dev libneon27-dev libasound2-dev libogg-dev libvorbis-dev libicu-dev libsrtp*-dev unixodbc unixodbc-dev python-dev xinetd e2fsprogs dbus sudo xmlstarlet lame ffmpeg dirmngr linux-headers* gnupg2 "**
## Install PHP
**Install dependencies**
'apt -y install curl apt-transport-https ca-certificates'

**add source in source.list**
'curl https://packages.sury.org/php/apt.gpg | apt-key add -
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/deb.sury.org.list'
**install php 5.6**
'apt-get update && apt-get install php5.6 php5.6-curl php5.6-cli php5.6-mysql php5.6-mbstring php5.6-gd php5.6-xml php-pear -y'

## Instal NodeJS. Curretn version 20.* RECOMENDED  18.17.0 
'curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -'
'apt -y install nodejs'

## Instal DAHDI. Yu need this library if you use any digital streams, such as E1
'cd /usr/src'
'wget http://downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-current.tar.gz '
'tar -zxvf dahdi-linux-complete-current.tar.gz '
'cd dahdi-linux-complete-3.* '
'make all'
'make install'
'make instal-config'

## ODBC. 
**Install connector.**
'cd /usr/src'
'wget https://downloads.mariadb.com/Connectors/odbc/connector-odbc-2.0.19/\
mariadb-connector-odbc-2.0.19-ga-debian-x86_64.tar.gz '
'tar -zxvf mariadb-connector-odbc-2.0.19*.tar.gz '
'cp lib/libmaodbc.so /usr/lib/x86_64-linux-gnu/odbc/'

**Create /etc/odbcinst.ini**
'cat >> /etc/odbcinst.ini << EOF'
'[MySQL]
Description = ODBC for MariaDB
Driver = /usr/lib/x86_64-linux-gnu/odbc/libmaodbc.so
Setup = /usr/lib/x86_64-linux-gnu/odbc/libodbcmyS.so
FileUsage = 1
  
EOF'

**Create /etc/odbc.ini**
'cat >> /etc/odbc.ini << EOF'
'[MySQL-asteriskcdrdb]
Description = MariaDB connection to 'asteriskcdrdb' database
driver = MySQL
server = localhost
database = asteriskcdrdb
Port = 3306
Socket = /var/run/mysqld/mysqld.sock
option = 3
  
EOF'

## MongoDB. Only required if you plan to use XMPP.
'wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -'
'echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" \
| sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list'
'apt update && apt install -y mongodb-org'
'systemctl enable mongod'

## Install Asterisk
'cd /usr/src'
'wget http://downloads.asterisk.org/pub/telephony/asterisk/asterisk-16-current.tar.gz '
'tar zxvf asterisk-16-current.tar.gz'
'cd /usr/src/asterisk-16*/'
'make distclean'

**Install additional dependencies**
'./contrib/scripts/install_prereq install'
'contrib/scripts/get_mp3_source.sh'
'./configure --with-jansson-bundled --with-pjproject-bundled'

**Set compile options**
'make menuselect'
Go to Applications submenu and make sure __app_macro__ is checked. 
'make'
'make install'

## Create Asterisk User, compile, install and set preliminary ownership.
'useradd -m asterisk'
'chown asterisk. /var/run/asterisk'
'chown -R asterisk. /etc/asterisk'
'chown -R asterisk. /var/{lib,log,spool}/asterisk'
'chown -R asterisk. /usr/lib/asterisk'
'touch /etc/asterisk/{modules,cdr}.conf'

## FreePBX install
'cd /usr/src'
'wget http://mirror.freepbx.org/modules/packages/freepbx/freepbx-14.0-latest.tgz '
'cd /usr/src/freepbx'
'./start_asterisk start'
'./install -n'

**Minimal module install**
'fwconsole ma downloadinstall framework core voicemail sipsettings infoservices \ featurecodeadmin logfiles callrecording cdr dashboard music soundlang recordings conferences pm2'
'fwconsole chown'
'fwconsole reload'

**Optionally, install all modules (not recommended).  You may need to run the following commands twice.**
'fwconsole ma installall'
'fwconsole chown'
'fwconsole reload'

**Set Freepbx to start on boot**
'cat >> /etc/systemd/system/freepbx.service << EOF'
'[Unit]
Description=Freepbx
After=mariadb.service
 
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/sbin/fwconsole start -q
ExecStop=/usr/sbin/fwconsole stop -q
 
[Install]
WantedBy=multi-user.target

EOF'

'systemctl enable freepbx'

## Configure Apache
Add __AllowOverride All__ to web directory so that __.htaccess__ is active.
'cat >> /etc/apache2/conf-available/allowoverride.conf << EOF '
'<Directory /var/www/html>
    AllowOverride All
    </Directory>
EOF'
'a2enconf allowoverride'

**Change default apache user/group, disable index.html, enable rewrite module**
'sed -i 's/\(APACHE_RUN_USER=\)\(.*\)/\1asterisk/g' /etc/apache2/envvars'
'sed -i 's/\(APACHE_RUN_GROUP=\)\(.*\)/\1asterisk/g' /etc/apache2/envvars'
'chown asterisk. /run/lock/apache2'
'mv /var/www/html/index.html /var/www/html/index.html.disable'
'a2enmod rewrite'

'systemctl restart apache2'

## Reboot and access GUI
'reboot'

Post-INSTALL TASK
## TFTP
**If you plan to use hardware SIP phones you will probably want to enable the tftp server.
Create tftp configuration file.**
'nano /etc/xinetd.d/tftp'
'service tftp'
'service tftp
{
protocol        = udp
port            = 69
socket_type     = dgram
wait            = yes
user            = nobody
server          = /usr/sbin/in.tftpd
server_args     = /tftpboot
disable         = no
}'

**Make the directory and restart the daemon to start tftp.**
'mkdir /tftpboot'
'chmod 777 /tftpboot'
'systemctl restart xinetd'


Sources were used when writing the article 
https://powerpbx.org/content/asterisk-freepbx-install-guide-debian-v11-asterisk-v16-freepbx-v16-v1
https://jakondo.ru/ustanovka-freepbx-14-v-svyazke-s-asterisk-16-na-debian-9-stretch/