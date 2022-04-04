# Guide-LAMP-Moodle
Prérequis : Debian net Install 10.4 + hyperviseur + Cmder + Notepad++ (avec plugin Compare), Apache 2.4.38, MariaDB 10.3, PHP7.3, Moodle 3.9 et Git

## 

Installation Debian en VM

-   Partitionnement classique, sans LVM ni chiffrement, une seule partition (/dev/sda1) hébergeant /, /home, /var, et tmp
    
-   Partition logique de swap (= au nombre de la quantité de RAM)
    
-   Système de fichier ext4 pour la partition primaire /dev/sda
    
-   Bureau GNOME, SSH, utilitaires systèmes usuels, retirer serveur d’impression
    
-   Installation des Guest addition coté client/serveur selon (VirtualBox, VMware, Hyper-V…)
    

## 

Configuration Debian

-   Installation des paquets : dnsutils, git, resolvconf, sudo, tree, terminator
    
-   Ne pas modifier le fichier Sudoers, mais utiliser le programme Visudo (il édite /etc/sudoers.tmp). Pour le créer avec # sudo visudo -c
    
-   Ajouter ligne en dessous de ‘User privilege specification’
    

## 

Configuration réseau

Ajouter dans les fichiers :

dns-search xxx.moodle.xxx.lan

dns-nameservers 192.168.xxx.xxx 192.168.xxx.xxx

192.168.xxx.xxx vega.moodle.xxx.lan moodle

\# systemctl restart networking

(redémarrer ssh ou redémarrer VM)

\# dig vega.moodle.xxx.lan

## 

Installation Apache

• # apt-get update

• # apt-get install apache2 apache2-doc (file:///usr/share/doc/apache2-doc/2.x/fr/index.html)

• # apt-get install curl

• # apachectl -M (affiche les modules actifs /etc/apache2/mods-enabled/\*.load)

• # apachectl -S (affiche les sites actifs /etc/apache2/site-enabled/index.conf)

(/var/www/index.html => location site web par défaut)

Modifier le contenu de /etc/apache2/sites-available/vega.conf

<VirtualHost \*:80>

ServerName vega.moodle.xxx.lan

Redirect permanent / https:// vega.moodle.xxx.lan/

</VirtualHost>

<VirtualHost \*:443>

DocumentRoot /var/www/moodle/

ServerName vega.moodle.xxx.lan

<Directory /var/www/moodle/>

Options -Indexes +FollowSymlinks +MultiViews

AllowOverride None

Require all granted

</Directory>

ErrorLog /var/log/apache2/moodle.error.log

CustomLog /var/log/apache2/access.log combined

SSLEngine On

SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire

SSLCertificateFile /etc/ssl/certs/moodle.crt

SSLCertificateKeyFile /etc/ssl/private/moodle.key

</VirtualHost>

Pour vérifier la synthaxe de la configuration d’Apache :

Créer dossier moodle /var/www/moodle (avec un index.html dedans pour le test) :

\# mkdir /var/www/moodle

\# echo « test vega » > /var/www/moodle/index.html

\# a2dissite 000-default

\# a2ensite moodle

\# systemctl restart apach2

\# apachectl -S

\# curl htttp://localhost ou non dns

\# openssl genrsa -out /etc/ssl/private/moodle.key 2048 (génére une clé privée)

\# openssl req -new -key /etc/ssl/private/moodle.key -out /etc/ssl/certs/moodle.csr

(demande de signature du certificat par la clé privée précédente)

#open ssl x509 -req -days 365 -in /ect/ssl/certs/vega.csr -signkey /etc/ssl/private/moodle/key -out /etc/ssl/certs/vega.crt

(auto-signature de la demande en générant un certificat auto-signé)

\# a2enmod ssl (activation du module SSL)

Rajouter dans /etc/apache2/sites-available/vega.conf un virtual host 443 et une redirection http -> https + configuration SSL

<VirtualHost \*:80>

ServerName vega.moodle.xxx.lan

Redirect permanent / https:// vega.moodle.xxx.lan/

</VirtualHost>

<VirtualHost \*:443>

DocumentRoot /var/www/moodle/

ServerName vega.moodle.xxx.lan

<Directory /var/www/moodle/>

Options -Indexes +FollowSymlinks +MultiViews

AllowOverride None

Require all granted

</Directory>

ErrorLog /var/log/apache2/moodle.error.log

CustomLog /var/log/apache2/access.log combined

SSLEngine On

SSLOptions +FakeBasicAuth +ExportCertData +StrictRequire

SSLCertificateFile /etc/ssl/certs/moodle.crt

SSLCertificateKeyFile /etc/ssl/private/moodle.key

</VirtualHost>

\# apachectl configtest (pour vérifier la syntaxe du fichier de conf)

\# systemctl restart apache2

\# apachectl -S

\# curl -I http://localhost ou nom dns

\# apt-get install mariadb-server mariadb-client

\# systemctl list-unit-files –type=service –state=enabled

\# systemctl enable service

Configuration MariaDB (port par défaut 3306)

\# mysql\_secure\_installation (défini un mdp pour root, retirer utilisateurs anonymes oui, désactiver connection root à distance, retirer database de test oui, recharger table et privilege oui)

\# mysql -e “select host, user, plugin from mysql.user” (on retrouve le plugin unix\_socket qu’il faut changer en native\_password)

\# mysql -u root

\# use mysql; (on se place sur la bdd « mysql »)

\# UPDATE user SET plugin=’mysql\_native\_password’ WHERE user=’root’;

#FLUSH PRIVILEGES

#QUIT

\# mysql -u root -p (moodledb si créee)

\# CREATE DATABASE IF NOT EXISTS moodledb CHARACTER SET utf8mb4 COLLATE utf8mb4\_unicode-ci;

\# CREATE USER xxxx IDENTIFIED BY “mettre\_mdp”;

\# GRANT ALL PRIVILEGES ON moodledb.\* TO xxxx;

\# FLUSH PRIVILEGES;

\# SHOW DATABASES;

\# SELECT host, user FROM mysql.user;

\# SHOW VARIABLES LIKE ‘character\_set\_database’;

\# SHOW VARIABLES LIKE ‘collation\_database’;

\# apt-get install php php-acpu php-bz2 php-curl php-fpm php-gd php-geoip php-gmp php-intl php-ldap php-mbstring php-mcrypt php-memcached php-msgpack php-mysql php-pear php-soap php-xml php-xmlrpc php-zip php5-json

\# dpkg –get-selections | grep -v deinstall | grep php

\# php -m (module actif)

/etc/php/7.3/mods-available/ (configurations de modules installés)

/etc/php/7.3/cli/conf.d (configuration globales de modules chargés actifs)

/etc/php/7.3/fpm/conf.d (configuration apache de modules chargés actifs)

\# phpenmod ‘non\_du\_module’ (pour activer module)

\# phpdismod ‘non\_du\_module’

\# nano /etc/php/7.3/mods-available/opcache.ini (pour configurer OPcache pour Moodle)

‘opcache.memory\_consumption=128

opcache.interned\_strings\_buffer=8

opcache.max\_accelerated\_files=32531 ou 16229 si problème)

opcache.revalidate\_freq=60

opcache.fast\_shutdown=1

opcache.enable\_cli=1’

\# php --ini -c /etc/php/7.3/fpm/php.ini | less

\# php -c /etc/php/7.3/fpm/php.ini -i | less

\# echo “<?php phpinfo(); ?>” > /var/www/moodle/phpinfo.php (pour créer et lancer la page vérification php en lancant https://localhost/phpinfo.php ou nom dns)

\# nano -c /etc/php/7.3/fpm/php.ini

Changer les variables et décommenter

memory\_limit = 256M

file\_uploads = On

date.timezone = ‘Europe/Paris’

opcache.enable = 1

\# a2enmod proxy\_fcgi setenvif

\# a2enconf php7.3-fpm

\# systemctl status php7.3-fpm

\# systemctl restart php7.3-fpm

\# systemctl restart apache2

Activer les 2 modules pdo\_mysql et mysqli

Remplir les info de la bdd dans /etc/php/7.3/fpm/php.ini

Installation Moodle avec Git (port 9418)

\# adduser --system moodle

\# mkdir /var/moodledata/

\# chown -R www-data:www-data /var/moodledata/

\# chmod 0777 /var/moodledata/

\# chown -R root:www-data /var/www/moodle/

\# chmod -R 0755 /var/www/moodle/

\# apt-get install git git-core

\# cd /var/www/

\# git clone git://git.moodle.org/moodle.git

\# cd moodle

\# git branch -a

\# git branch --track MOODLE\_39\_STABLE origin/MOODLE\_39\_STABLE

\# git checkout MOODLE\_39\_STABLE

Mettre à jour Moodle avec Git

cd /var/www/moodle

\# git pull

Binding Moodle => MariaDB

\# nano /var/www/moodle/config.php

<?php // Moodle configuration file

unset($CFG);

global $CFG;

$CFG = new stdClass();

$CFG->dbtype = 'mariadb';

$CFG->dblibrary = 'native';

$CFG->dbhost = 'localhost';

$CFG->dbname = 'moodlxxx';

$CFG->dbuser = 'moodlxxx';

$CFG->dbpass = 'moodleadminpw';

$CFG->prefix = 'mdl\_';

$CFG->dboptions = array (

'dbpersist' => 0,

'dbport' => '',

'dbsocket' => '',

'dbcollation' => 'utf8mb4\_unicode\_ci',

);

$CFG->wwwroot = 'https://vega.xxx.fr;

$CFG->dataroot = '/var/moodledata';

$CFG->admin = 'admin';

$CFG->directorypermissions = 0777;

require\_once(\_\_DIR\_\_ . '/lib/setup.php');

// There is no php closing tag in this file,

// it is intentional because it prevents trailing whitespace problems!

\# nano /etc/mysql/my.cnf (qui est un lien symbolique de /alternatives/my.cnf et /mariadb.cnf )

Rajouter en dessous

\[client\]

default-character-set = utf8mb4

\[mysqld\]

innodb\_file\_format = Barracuda

innodb\_file\_per\_table = ON

innodb\_large\_prefix = ON

character-set-client-handshake = FALSE

character-set-server = utf8mb4

collation-server = utf8mb4\_unicode\_ci

\[mysql\]

default-character-set = utf8mb4

\# systemctl restar mysqld

\# systemctl restat apache2

Aller sur https://localhost/install.php pour configurer Moodle client

Sauvegarde Moodle et MariaDB + Migration

Mode de maintenance : placez votre site Moodle actuel en mode de maintenance pour empêcher toute modification dans la base de données. Ne laissez pas les administrateurs se connecter durant la migration, car ils ne sont pas affectés par le mode de maintenance.

Commande utile pour connaitre le poids de dossiers

#du -sh /

#du -shx -d 1 /

• config.php à recopier du serveur ancien (adresse Internet, dossiers moodledata, bdd) ($CFG->wwwroot et $CFG->dataroot)

• php.ini

• Dossier moodledata à transferer :

\# rsync -av -e ssh /var/www/moodledata/ xxxx@192.168.xxx.xxx:/var/moodledata/

arguments :

\-a : Sauvegarde toute l’arborescence (dossier, sous-dossier, …)

\-v : Affiche toutes les informations (Verbose)

\-z : Compresse les fichiers (diminue le temps d’envoi mais prend sur le CPU et peux brider le

tranfère)

\-n : Liste ce que fait la commande et les éventuelles erreurs sans l’exécuter réellement.

\-e : Spécifie le shell à utiliser (dans notre cas, SSH)

• Dossier /home/letsencrypt à transferer avec la commande rsync ?

Dump de la BDD

• Fichier à récupérer sur la précedente VM :

/var/archives/xxx-ead-all-mysql-databases.20200821.sql.bz2

/var/archives/xxx-ead-mysql-moodlexxx.20200821.sql.bz2

#mysqlcheck --user root -p --databases

\# mysql -u root -p (moodledb si créee)

\# SHOW DATABASES;

\- # SELECT host, user FROM mysql.user;

\- # SHOW VARIABLES LIKE ‘character\_set\_database’;

\- # SHOW VARIABLES LIKE ‘collation\_database’;

\# systemctl list-unit-files –type=service –state=enabled

sudo systemctl status mssql-server

cd /var/opt/mssql/log

ls –lrt

\# head -n 5 data-dump.sql

\# mysqldump --allow-keywords --opt --verbose -u moxxxodlxxx -p moxxxodlxxx | ssh xxxx@192.168.xxx.xxx "mysql -u xxxx@localhost -p xxx "

\# sed -e 's/oldserver.com/newserver.com/g' oldmysqldump.sql > newmysqldump.sql
