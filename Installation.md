# Installation de ZABBIX 7.4 sur Debian 12 Bookworm

## Introduction
***ZABBIX*** est une solution open source de supervision (monitoring) d’infrastructures informatiques, de réseaux, de serveurs et d’applications, offrant la collecte de métriques en temps réel et un système d'alerte avancé. Indispensable en entreprise, nous allons voir comment procéder à son installation complète de base. 

## Prérequis
L'installation s'appuie sur la stack **LAMP**, garante d'une infrastructure fiable, performante et 100 % open source :
- Un système d'exploitation basé sur Linux (**Debian 12**)
- Un serveur web **Apache2** en mode Event avec **PHP-FPM**
- Une base de données **MariaDB**
- Un langage de traitement **PHP**

Bien sûr, il faut prévoir une connexion à Internet pour le téléchargement de certains paquets.

## Préparation et approvisionnement de l'environnement
### 1. Installation du dépôt ZABBIX
 
```
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian12_all.deb
```
- Dans certains cas, il peut y avoir des répertoires de commandes système manquants. La commande ***export*** permet d'ajouter ces répertoires manquants.
```
export PATH=/usr/local/sbin:/usr/sbin:/sbin:$PATH
```
- Puis installez à partir du fichier installé précédemment et mettez à jour
```
sudo dpkg -i zabbix-release_latest_7.4+debian12_all.deb
sudo apt update
```

### 2. Installation des paquets

On peut passer à l'installation des paquets principaux de Zabbix Server
```
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

### 3. Configuration de la timezone 

La ***timezone*** est à configurer en fonction du fuseau horaire (**ici, c'est Europe/Paris**),
```
timedatectl set-timezone "Europe/Paris"

mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -p -u root mysql
```

### 4. Création de la base de données
Exécutez : 
```
sudo mysql_secure_installation
```

Répondez comme suit :

- Switch to unix_socket → N
- Change the root password → Y (choisissez un mot de passe fort)
- Remove anonymous users → Y
- Disallow root login remotely → Y
- Remove test database → Y
- Reload privilege tables → Y

Ensuite, connectez-vous à mysql : 
```
sudo mysql -uroot -p
```
Puis créez la base de données et l'utilisateur :
```
create database nom_de_la_bd character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix_admin'@'localhost' identified by '@SuperP4ssword';
grant all privileges on nom_de_la_bd.* to 'zabbix_admin'@'localhost';
set global log_bin_trust_function_creators = 1;
grant select on `mysql`.`time_zone_name` to 'zabbix_admin'@'localhost';
quit;
```

Importer le schéma initial et les données. Il faudra entrer le mot de passe pour pouvoir se connecter. 
```
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix_admin -p nom_de_la_bd
```
Désactiver la fonction ***log_bin_trust_function_creators*** après avoir importé le schéma de la base de données en entrant ***set global log_bin_trust_function_creators = 0;***
```
set global log_bin_trust_function_creators = 0;
quit;
```

### 5.  Configuration & Test
- Maintenant, il faut définir un mot de passe pour notre base de données de Zabbix Server dans le fichier **/etc/zabbix/zabbix_server.conf**.
```
DBPassword=@SuperP4ssword
```

- Redémarrez les services et vérifier les statuts sont sur ***active:(running)*** :
```
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

## Accès à ZABBIX 

  Maintenant, il faut se connecter à l'interface web. L'URL par défaut de l'interface utilisateur ZABBIX lors de l'utilisation du serveur web Apache est **http://adresse_ip_du_zabbix_server/zabbix**

Choisissez **Next stop** :
- Nom de la base de données : **nom_de_la_bd**
- Utilisateur base de données : **zabbix_admin**
- Mot de passe base de données : **@SuperP4ssword**
