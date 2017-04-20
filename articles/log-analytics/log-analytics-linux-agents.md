<properties
    pageTitle="Connecter les ordinateurs Linux pour journal Analytique | Microsoft Azure"
    description="À l’aide du journal Analytique, vous pouvez collecter et agissent sur les données générées à partir des ordinateurs de Linux."
    services="log-analytics"
    documentationCenter=""
    authors="bandersmsft"
    manager="jwhit"
    editor=""/>

<tags
    ms.service="log-analytics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/10/2016"
    ms.author="banders"/>

# <a name="connect-linux-computers-to-log-analytics"></a>Connecter les ordinateurs Linux pour journal Analytique

À l’aide du journal Analytique, vous pouvez collecter et agissent sur les données générées à partir des ordinateurs de Linux. Ajout de données collectées à partir de Linux pour OMS permet de gérer les systèmes Linux et les solutions de conteneur comme Docker, quel que soit l’endroit où se trouvent vos ordinateurs, n’importe quel emplacement. Par conséquent, ces sources de données peuvent résider dans votre centre de données sur site en tant que serveurs physiques, des ordinateurs virtuels dans un service hébergé sur le nuage comme Amazon Web Services (AWS) ou Microsoft Azure ou même sur l’ordinateur portable sur votre bureau. En outre, OMS collecte également des données à partir d’ordinateurs Windows de la même façon, afin qu’il prend en charge un hybride véritablement environnement informatique.

Vous pouvez afficher et gérer des données à partir de toutes ces sources avec Analytique de journal dans l’OMS avec un portail de gestion unique. Cela réduit la nécessité de surveiller à l’aide de différents systèmes, est facile à utiliser et vous pouvez exporter les données que vous souhaitez pour n’importe quel solution analytique d’entreprise ou le système que vous avez déjà.

Cet article est un guide qui vous permettra de collecter et de gérer les données de vos ordinateurs Linux à l’aide de l’Agent de l’OMS pour Linux en main. Pour plus de détails techniques, tels que la configuration du serveur proxy, d’informations sur les mesures CollectD et sources de données JSON personnalisés, vous trouverez ces informations à [l’Agent de l’OMS pour la présentation de Linux](https://github.com/Microsoft/OMS-Agent-for-Linux) et [Agent d’OMS pour Linux une documentation complète](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md) sur Github.


Actuellement, vous pouvez recueillir les types suivants de données à partir d’ordinateurs de Linux :

- Mesures de performances
- Événements du journal système
- Alertes de Nagios et de Zabbix
- Journaux, stock et mesures de performances du conteneur docker

## <a name="supported-linux-versions"></a>Versions de Linux prises en charge

Versions à la fois x86 et x64 sont officiellement pris en charge sur un grand nombre de distributions Linux. Toutefois, l’Agent de l’OMS pour Linux peut également exécuter sur les autres distributions ne pas dans la liste.

- Amazon Linux 2012.09 via 2015.09
- Linux centOS 5, 6 et 7
- Oracle Linux 5, 6 et 7
- Serveur Red Hat Enterprise Linux 5, 6 et 7
- Debian GNU/Linux 6, 7 et 8
- Ubuntu 12.04 LTS, 14.04 LTS, 15.04, 15.10
- SUSE Linux Enterprise Server 11 et 12

## <a name="oms-agent-for-linux"></a>Agent d’OMS pour Linux
L’Agent de Suite de gestion des opérations pour Linux comprend plusieurs packages. Le fichier de version contient les packages suivants, disponibles en exécutant l’offre groupée de shell avec `--extract`.

**Package** | **Version** | **Description**
----------- | ----------- | --------------
omsagent | 1.1.0 | L’Agent de Suite de gestion des opérations pour Linux
omsconfig | 1.1.1 | Agent de configuration de l’Agent de l’OMS
OMI | 1.0.8.3 | Infrastructure de gestion ouverte (OMI)--un serveur CIM léger
SCX | 1.6.2 | Fournisseurs de CIM OMI pour les métriques de performances de système d’exploitation
Apache-cimprov | 1.0.0 | Fournisseur de l’OMI de surveillance des performances de serveur HTTP Apache. Installé uniquement si le serveur HTTP Apache est détecté.
MySQL-cimprov | 1.0.0 | Fournisseur de l’OMI de surveillance des performances de serveur MySQL. Installé uniquement si le serveur de MySQL/MariaDB est détecté.
docker-cimprov | 0.1.0 | Fournisseur de docker de l’OMI. Installé uniquement si Docker est détecté.

### <a name="additional-installation-artifacts"></a>Artefacts d’installation supplémentaires
Après l’installation de l’agent de l’OMS pour les packages de Linux, les modifications de configuration de système supplémentaires suivantes sont appliquées. Ces artefacts sont supprimés lors de la désinstallation du package omsagent.
- Un utilisateur non privilégié nommé : `omsagent` est créé. Le compte est que le démon omsagent s’exécute en tant que
- Création d’un fichier « include » de sudoers à /etc/sudoers.d/omsagent autorisent omsagent pour redémarrer les processus syslog et omsagent. Si les directives sudo « include » ne sont pas pris en charge dans la version installée de sudo, ces entrées sont écrites à /etc/sudoers.
- La configuration du journal système est modifiée afin de transférer un sous-ensemble d’événements à l’agent. Pour plus d’informations, reportez-vous à la section **Configuration de la collecte de données** ci-dessous

### <a name="linux-data-collection-details"></a>Détails de collection de données Linux

Le tableau suivant indique les méthodes de collecte de données et d’autres détails sur la manière dont les données sont collectées.

| source | Agent direct | Agent SCOM | Stockage Azure | SCOM requis ? | Données de l’agent SCOM envoyées par groupe d’administration | fréquence de collection |
|---|---|---|---|---|---|---|
|Zabbix|![Oui](./media/log-analytics-linux-agents/oms-bullet-green.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|            ![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|1 minute|
|Nagios|![Oui](./media/log-analytics-linux-agents/oms-bullet-green.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|            ![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|à l’arrivée|
|journal système|![Oui](./media/log-analytics-linux-agents/oms-bullet-green.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|            ![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|à partir du stockage Azure : 10 minutes ; à partir de l’agent : à l’arrivée|
|Compteurs de performance de Linux|![Oui](./media/log-analytics-linux-agents/oms-bullet-green.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|            ![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|Comme prévu, minimale de 10 secondes|
|le suivi des modifications|![Oui](./media/log-analytics-linux-agents/oms-bullet-green.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|            ![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|![N°](./media/log-analytics-linux-agents/oms-bullet-red.png)|toutes les heures|



### <a name="package-requirements"></a>Packages requis
| **Package requis**  | **Description**   | **Version minimale**|
|--------------------- | --------------------- | -------------------|
|Glibc |    Bibliothèque GNU C   | 2.5-12|
|OpenSSL    | Bibliothèques d’OpenSSL | 0.9.8e ou 1.0|
|Curl | client web de roulage | 7.15.5
|Python-ctypes |bibliothèques de fonctions | n/a|
|PAM | Pluggable authentication Modules  |n/a |

>[AZURE.NOTE] Rsyslog ou syslog-ng sont requis pour recueillir les messages syslog. Le démon syslog par défaut dans la version 5 de Red Hat Enterprise Linux et Oracle Linux CentOS (sysklog) n’est pas prise en charge de la collecte d’événements syslog. Pour collecter des données de journal système à partir de cette version de ces distributions, le démon rsyslog doit être installé et configuré pour remplacer les sysklog.

## <a name="quick-install"></a>Guide d’installation rapide

Exécutez les commandes suivantes pour télécharger l’omsagent, valider le checksum, puis installer et intégré l’agent. Les commandes sont 64 bits. L’ID de l’espace de travail et de la clé primaire sont trouvent dans le portail de l’OMS sous **paramètres** dans l’onglet **Sources connectées** .

![Détails de l’espace de travail](./media/log-analytics-linux-agents/oms-direct-agent-primary-key.png)

```
wget https://github.com/Microsoft/OMS-Agent-for-Linux/releases/download/v1.1.0-28/omsagent-1.1.0-28.universal.x64.sh
sha256sum ./omsagent-1.1.0-28.universal.x64.sh
sudo sh ./omsagent-1.1.0-28.universal.x64.sh --upgrade -w <YOUR OMS WORKSPACE ID> -s <YOUR OMS WORKSPACE PRIMARY KEY>
```

Il existe plusieurs autres méthodes pour installer l’agent et le mettre à niveau. Vous pouvez en savoir plus sur les étapes [pour installer l’Agent de l’OMS pour Linux](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#steps-to-install-the-oms-agent-for-linux).

Vous pouvez également consulter la [procédure pas à pas vidéo Azure](https://www.youtube.com/watch?v=mF1wtHPEzT0).

## <a name="choose-your-linux-data-collection-method"></a>Choisissez votre méthode de collecte de données Linux

Manière dont vous choisissez les types de données que vous souhaitez collecter dépend de si vous souhaitez utiliser le portail de l’OMS, ou si vous souhaitez modifier divers fichiers de configuration directement sur vos clients Linux. Si vous choisissez d’utiliser le portail, la configuration est automatiquement envoyée à tous les clients Linux. Si vous avez besoin de différentes configurations pour différents clients Linux, vous devez modifier les fichiers du client individuellement, ou utilisez une solution autre que PowerShell DSC, Chef ou Marionnette.

Vous pouvez spécifier les événements du journal système et les compteurs de performance que vous souhaitez collecter à l’aide de fichiers de configuration sur les ordinateurs Linux. *Si vous avez choisi de configurer la collecte de données en modifiant les fichiers de configuration de l’agent, vous devez désactiver la configuration centralisée.*  Des instructions sont fournies ci-dessous pour configurer la collecte de données dans les fichiers de configuration de l’agent ainsi que pour désactiver la configuration centrale de tous les Agents de l’OMS pour Linux, ou des ordinateurs individuels.

### <a name="disable-oms-management-for-an-individual-linux-computer"></a>Désactiver la gestion de l’OMS pour un ordinateur Linux

Collecte centralisée des données pour les données de configuration est désactivée pour un ordinateur Linux avec le script OMS_MetaConfigHelper.py. Cela peut être utile si un sous-ensemble d’ordinateurs doit avoir une configuration spécifique.

Pour désactiver la configuration centralisée :

```
sudo /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable
```

Pour réactiver la configuration centralisée :

```
sudo /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py –enable
```

## <a name="linux-performance-counters"></a>Compteurs de performance de Linux

Les compteurs de performance de Linux sont similaires aux compteurs de performance Windows : les deux fonctionnent de la même façon. Vous pouvez utiliser les procédures suivantes pour ajouter et les configurer. Une fois qu’ils sont ajoutés à l’OMS, données sont collectées pour les toutes les 30 secondes.

### <a name="to-add-a-linux-performance-counter-in-oms"></a>Pour ajouter un compteur de performance Linux dans OMS

1. Pour configurer les Agents de l’OMS pour Linux via le portail de l’OMS, vous pouvez ajouter des compteurs de performance de Linux sur la page Paramètres, cliquez sur **données**.  
2. Dans la page **paramètres** , sous **données** , cliquez sur **les compteurs de performance Linux** et puis sélectionnez ou tapez le nom du compteur que vous souhaitez ajouter.  
    ![données](./media/log-analytics-linux-agents/oms-settings-data01.png)
3. Si vous ne connaissez pas le nom complet du compteur, vous pouvez commencer à entrer un nom partiel et une liste des compteurs disponibles s’affiche. Lorsque vous avez trouvé le compteur que vous souhaitez ajouter et cliquez sur le nom dans la liste, puis cliquez sur l’icône de signe plu pour ajouter le compteur.
4. Après avoir ajouté le compteur, il s’affiche dans la liste des compteurs de mise en surbrillance avec une barre de couleur.
5. Par défaut, l’option **appliquer sous configuration à Mes ordinateurs** est sélectionnée. Si vous souhaitez désactiver l’envoi de données de configuration, désactivez la sélection.
6. Lorsque vous avez terminé la modification des compteurs de performance, au bas de la page, cliquez sur **Enregistrer** pour valider vos modifications. Les modifications de configuration que vous avez apportées sont ensuite envoyées à tous les Agents de l’OMS pour Linux qui sont enregistrés avec l’OMS, généralement dans les 5 minutes.

### <a name="configure-linux-performance-counters-in-oms"></a>Configurer les compteurs de performance de Linux dans OMS

Pour les compteurs de performances de Windows, vous pouvez choisir une instance spécifique pour chaque compteur de performance. Toutefois, pour les compteurs de performance de Linux, toute instance d’un compteur que vous choisissez s’applique à tous les compteurs d’enfant du compteur parent. Le tableau suivant montre les instances courantes disponibles pour les compteurs de performance Linux et Windows.

| **Nom de l’instance** | **Signification** |
| --- | --- |
| \_Total | Total de toutes les instances |
| \* | Toutes les instances |
| (/ & #124 ; / var) | Correspond à des instances nommées : / ou/var |


De même, l’intervalle d’échantillon que vous choisissez pour un compteur parent s’applique à tous les compteurs de son enfant. En d’autres termes, tous les intervalles d’échantillonnage enfant compteurs et instances sont liés entre eux.

### <a name="add-and-configure-performance-metrics-with-linux"></a>Ajouter et configurer des mesures de performances avec Linux

Les mesures de performances pour collecter sont contrôlées par la configuration dans /etc/opt/microsoft/omsagent/conf/omsagent.conf. Voir [les mesures de performances disponibles](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#appendix-available-performance-metrics) pour les mesures et les classes disponibles pour l’Agent de l’OMS pour Linux.

Chaque objet, ou une catégorie, des mesures de performances pour collecter doit être défini dans le fichier de configuration sous la forme d’un seul `<source>` élément. La syntaxe suit le modèle suivant.

```
<source>
  type oms_omi  
  object_name "Processor"
  instance_regex ".*"
  counter_name_regex ".*"
  interval 30s
</source>

```

Les paramètres configurables de cet élément sont les suivants :

- **Objet\_nom de**: le nom de l’objet de la collection.
- **Instance\_regex**: une *expression régulière* définissant les instances à collecter. La valeur : `.*` spécifie toutes les instances. Pour collecter uniquement les métriques de processeur pour le \_instance Total, vous pouvez spécifier `_Total`. Pour collecter des données de processus pour seulement les instances crond ou sshd, vous pourriez spécifier : `(crond|sshd)`.
- **Compteur\_nom de\_regex**: une *expression régulière* définissant les compteurs (pour l’objet) à collecter. Pour collecter tous les compteurs pour l’objet, spécifiez : `.*`. Pour collecter uniquement les compteurs espace swap pour l’objet en mémoire, vous pouvez spécifier :`.+Swap.+`
- **Intervalle :**: la fréquence à laquelle les compteurs de l’objet sont collectées.

La configuration par défaut pour les métriques de performances est la suivante :

```
<source>
  type oms_omi
  object_name "Physical Disk"
  instance_regex ".*"
  counter_name_regex ".*"
  interval 5m
</source>

<source>
  type oms_omi
  object_name "Logical Disk"
  instance_regex ".*
  counter_name_regex ".*"
  interval 5m
</source>

<source>
  type oms_omi
  object_name "Processor"
  instance_regex ".*
  counter_name_regex ".*"
  interval 30s
</source>

<source>
  type oms_omi
  object_name "Memory"
  instance_regex ".*"
  counter_name_regex ".*"
  interval 30s
</source>

```

### <a name="enable-mysql-performance-counters-using-linux-commands"></a>Activer les compteurs de performance de MySQL à l’aide des commandes de Linux

Si le serveur de MariaDB ou de MySQL Server est détecté sur l’ordinateur lors de l’installation de l’offre omsagent, un fournisseur pour le serveur MySQL d’analyse des performances sont installé automatiquement. Ce fournisseur se connecte au serveur MySQL/MariaDB local pour exposer des statistiques de performances. Vous devez configurer les informations d’identification utilisateur de MySQL afin que le fournisseur puisse accéder au serveur MySQL.

Pour définir un compte d’utilisateur par défaut pour le serveur MySQL sur localhost, utilisez l’exemple de commande suivant.

>[AZURE.NOTE] Le fichier d’informations d’identification doit être lisible par le compte omsagent. Exécution de la commande mycimprovauth en tant qu’omsgent est recommandée.


```
sudo su omsagent -c '/opt/microsoft/mysql-cimprov/bin/mycimprovauth default 127.0.0.1 <username> <password>'

sudo service omiserverd restart
```


Vous pouvez également spécifier les informations d’identification de MySQL requises dans un fichier, en créant le fichier : /var/opt/microsoft/mysql-cimprov/auth/omsagent/mysql-auth. Pour plus d’informations sur la gestion des informations d’identification de MySQL pour la surveillance via le fichier mysql-auth, consultez [MySQL de gérer les informations d’identification analyse dans le fichier d’authentification](#manage-mysql-monitoring-credentials-in-the-authentication-file).

Pour plus d’informations sur les autorisations requises par l’utilisateur MySQL pour collecter des données de performances de serveur MySQL, voir [autorisations de base de données requis pour les compteurs de performance de MySQL](#database-permissions-required-for-mysql-performance-counters) .

### <a name="enable-apache-http-server-performance-counters-using-linux-commands"></a>Activer les compteurs de performance d’Apache HTTP Server à l’aide des commandes de Linux

Si Apache HTTP Server est détecté sur l’ordinateur lors de l’installation de l’offre omsagent, un fournisseur pour Apache HTTP Server d’analyse des performances sont automatiquement installé. Ce fournisseur repose sur un « module » qui doit être chargé dans le serveur HTTP Apache pour accéder aux données de performance d’Apache.

Vous pouvez charger le module avec la commande suivante :

```
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -c
```

Pour décharger le module d’analyse Apache, exécutez la commande suivante :

```
sudo /opt/microsoft/apache-cimprov/bin/apache_config.sh -u
```
### <a name="to-view-performance-data-with-log-analytics"></a>Pour afficher les données de performance Analytique du journal

1. Dans le portail de la Suite de gestion des opérations, cliquez sur la mosaïque de recherche des journaux.
2. Dans la barre de recherche, tapez `* (Type=Perf)` pour afficher tous les compteurs de performance.


Car l’OMS collecte également des données de compteur de performance Windows, vous devez étendue vers le bas la recherche à des données spécifiques à Linux. Ainsi, l’exemple suivant permet d’afficher les données de performances spécifiques à un serveur de Linux exemple nommé Chorizo21.

```
Type=Perf Computer=chorizo*
```

![exemple de serveur affiché dans les résultats de la recherche](./media/log-analytics-linux-agents/oms-perfsearch01.png)

Dans les résultats, vous pouvez cliquer sur les **mesures** pour afficher les données collectées pour les compteurs. Données en temps réel sont affichées en tant que graphiques de chaque compteur.

![mesures](./media/log-analytics-linux-agents/oms-perfmetrics01.png)


## <a name="syslog"></a>Journal système

Syslog est un protocole de journalisation d’événements similaire aux journaux des événements Windows : les deux fonctionnent de la même façon, lorsqu’il est affiché dans l’OMS.

### <a name="to-add-a-new-linux-syslog-facility-in-oms"></a>Pour ajouter une nouvelle fonctionnalité de syslog de Linux de l’OMS

1. Dans la page **paramètres** , sous **données** , cliquez sur **journal système** et à gauche de l’icône du signe plu, tapez le nom de la fonction syslog que vous souhaitez ajouter.
    ![Linux syslog](./media/log-analytics-linux-agents/oms-linuxsyslog01.png)
2.  Si vous ne connaissez pas le nom complet de l’installation, vous pouvez commencer à entrer un nom partiel et une liste des installations de syslog disponibles s’affiche. Lorsque vous avez trouvé la fonction syslog que vous souhaitez ajouter, cliquez sur le nom dans la liste et puis cliquez sur l’icône de signe plu pour ajouter la fonction syslog.
3.  Après avoir ajouté la facilité, il apparaît dans la liste de mise en surbrillance avec une barre de couleur. Ensuite, choisissez les niveaux de gravité (catégories d’informations sur les installations syslog) que vous souhaitez collecter.
4.  Au bas de la page, cliquez sur **Enregistrer** pour valider vos modifications. Les modifications de configuration que vous avez apportées sont ensuite envoyées à tous les Agents de l’OMS pour Linux qui sont enregistrés avec l’OMS, généralement dans les 5 minutes.


### <a name="configure-linux-syslog-facilities-in-linux"></a>Configurer des installations de syslog Linux sous Linux

Événements du journal système sont envoyés à partir du démon syslog, par exemple rsyslog ou syslog-ng, pour que l’agent est à l’écoute sur un port local. Par défaut, le port 25224. Lorsque l’agent est installé, une configuration de journal système par défaut est appliquée. Il se trouve à :


Rsyslog : /etc/rsyslog.d/rsyslog-oms.conf

Syslog-ng : /etc/syslog-ng/syslog-ng.conf


La configuration par défaut OMS agent syslog transfère les événements syslog à partir de toutes les facilités dont la gravité de l’avertissement ou supérieur.

>[AZURE.NOTE] Si vous modifiez la configuration du journal système, vous devez redémarrer le démon syslog pour que les modifications soient prises en compte.

La configuration de syslog par défaut pour l’Agent de l’OMS pour Linux pour OMS est la suivante :

#### <a name="rsyslog"></a>Rsyslog

```
kern.warning       @127.0.0.1:25224
user.warning       @127.0.0.1:25224
daemon.warning     @127.0.0.1:25224
auth.warning       @127.0.0.1:25224
syslog.warning     @127.0.0.1:25224
uucp.warning       @127.0.0.1:25224
authpriv.warning   @127.0.0.1:25224
ftp.warning        @127.0.0.1:25224
cron.warning       @127.0.0.1:25224
local0.warning     @127.0.0.1:25224
local1.warning     @127.0.0.1:25224
local2.warning     @127.0.0.1:25224
local3.warning     @127.0.0.1:25224
local4.warning     @127.0.0.1:25224
local5.warning     @127.0.0.1:25224
local6.warning     @127.0.0.1:25224
local7.warning     @127.0.0.1:25224
```

#### <a name="syslog-ng"></a>Syslog-ng

```
#OMS_facility = all
filter f_warning_oms { level(warning); };
destination warning_oms { tcp("127.0.0.1" port(25224)); };
log { source(src); filter(f_warning_oms); destination(warning_oms); };
```

### <a name="to-view-all-syslog-events-with-log-analytics"></a>Pour afficher tous les événements du journal système avec journal Analytique

1. Dans le portail de la Suite de gestion des opérations, cliquez sur la mosaïque de **Recherche des journaux** .
2. Dans le groupe de **Gestion du journal** , choisissez une recherche prédéfinie syslog, puis sélectionnez une pour l’exécuter.

Cet exemple montre tous les événements du journal système.

![Événements syslog illustrés à la recherche de journal](./media/log-analytics-linux-agents/oms-linux-syslog.png)

Maintenant, vous pouvez affiner les résultats de recherche.

## <a name="linux-alerts"></a>Alertes de Linux

Si vous utilisez Nagios ou Zabbix pour gérer vos ordinateurs Linux, OMS peut recevoir les alertes générées par ces outils. Toutefois, il n’existe actuellement aucune méthode pour configurer les données d’alerte entrantes via le portail de l’OMS. Au lieu de cela, vous devez modifier un fichier de configuration pour commencer l’envoi des alertes d’OMS.



### <a name="collect-alerts-from-nagios"></a>Collecter des alertes à partir de Nagios

Pour collecter les alertes d’un serveur Nagios, vous devez apporter les modifications de configuration suivantes.

1. Accorder à l’utilisateur **omsagent** l’accès en lecture au fichier journal Nagios (c'est-à-dire /var/log/nagios/nagios.log/var/log/nagios/nagios.log). En supposant que le fichier nagios.log est détenu par le groupe **nagios** , vous pouvez ajouter l' utilisateur **omsagent** pour le groupe de **nagios** .

    ```
    sudo usermod –a -G nagios omsagent
    ```

2. Modifiez le fichier omsagent.confconfiguration (/ etc/opt/microsoft/omsagent/conf/omsagent.conf). Vérifiez que les entrées suivantes sont présentes et pas commenté :

    ```
    <source>
    type tail
    #Update path to point to your nagios.log
    path /var/log/nagios/nagios.log
    format none
    tag oms.nagios
    </source>

    <filter oms.nagios>
    type filter_nagios_log
    </filter>
    ```

3. Redémarrez le démon omsagent :

    ```
    sudo service omsagent restart
    ```

### <a name="collect-alerts-from-zabbix"></a>Collecter des alertes à partir de Zabbix

Pour collecter les alertes d’un serveur Zabbix, vous pourrez de réaliser des étapes similaires à celles des Nagios ci-dessus, mais vous devez spécifier un utilisateur et un mot de passe en *texte clair*. Cela n’est pas idéal, mais est susceptibles de changer plus rapidement. Pour résoudre ce problème, nous vous recommandons de créer l’utilisateur et de lui accorder l’autorisation de surveiller uniquement.

Une section de l’exemple du fichier de configuration omsagent.conf (/ etc/opt/microsoft/omsagent/conf/omsagent.conf) pour Zabbix doit avoir l’aspect suivant :

```
<source>
  type zabbix_alerts
  run_interval 1m
  tag oms.zabbix
  zabbix_url http://localhost/zabbix/api_jsonrpc.php
  zabbix_username Admin
  zabbix_password zabbix
</source>

```

### <a name="view-alerts-in-log-analytics-search"></a>Afficher les alertes de recherche de journal Analytique

Après avoir configuré vos ordinateurs Linux pour envoyer des alertes d’OMS, quelques requêtes de recherche de journal simple vous permet d’afficher les alertes. L’exemple de requête suivant recherche renvoie toutes les alertes enregistrées qui ont été générés. Par exemple, en cas de problème quelconque dans votre infrastructure informatique, puis les résultats de la requête d’exemple suivante peuvent indiquer où le problème peut être l’origine. Et bien, vous pouvez facilement développer pour les alertes par le système source afin d’affiner votre enquête. L’avantage est que vous ne pas nécessairement accéder à différents systèmes de gestion à partir du début, autant que vos alertes sont envoyées à l’OMS, vous pouvez commencer.

```
Type=Alert
```

#### <a name="to-view-all-nagios-alerts-with-log-analytics"></a>Pour afficher toutes les alertes Nagios Analytique du journal
1. Dans le portail de la Suite de gestion des opérations, cliquez sur la mosaïque de **Recherche des journaux** .
2. Dans la barre de la requête, tapez la requête suivante de la recherche

    ```
    Type=Alert SourceSystem=Nagios
    ```
![Alertes Nagios illustrés à la recherche de journal](./media/log-analytics-linux-agents/oms-linux-nagios-alerts.png)

Une fois que vous consultez les résultats de recherche, vous pouvez descendre dans des détails supplémentaires, tels que *AlertState*.

### <a name="to-view-all-zabbix-alerts-with-log-analytics"></a>Pour afficher toutes les alertes Zabbix Analytique du journal
1. Dans le portail de la Suite de gestion des opérations, cliquez sur la mosaïque de **Recherche des journaux** .
2. Dans la barre de la requête, tapez la requête suivante de la recherche

    ```
    Type=Alert SourceSystem=Zabbix
    ```
![Alertes Zabbix illustrés à la recherche de journal](./media/log-analytics-linux-agents/oms-linux-zabbix-alerts.png)

Une fois que vous consultez les résultats de recherche, vous pouvez affiner les détails supplémentaires tels que *Nom_alerte*.


## <a name="compatibility-with-system-center-operations-manager"></a>Compatibilité avec System Center Operations Manager

L’Agent de l’OMS pour Linux partage les fichiers binaires de l’agent avec l’agent de System Center Operations Manager. L’installation de l’Agent de l’OMS pour Linux sur un système actuellement géré par Operations Manager met à niveau les packages OMI et SCX sur l’ordinateur vers une version plus récente. L’Agent de l’OMS pour Linux et System Center 2012 R2 sont compatibles. Toutefois, **System Center 2012 SP1 et les versions antérieures ne sont actuellement pas compatible ou pris en charge par l’Agent de l’OMS pour Linux.**

>[AZURE.NOTE] Si l’Agent de l’OMS pour Linux est installé sur un ordinateur qui n’est pas actuellement géré par Operations Manager et que vous décidez de gérer l’ordinateur avec Operations Manager, vous devez modifier la configuration de l’OMI avant de découvrir de l’ordinateur. **Cette étape n’est pas nécessaire si l’agent Operations Manager est installé avant l’Agent de l’OMS pour Linux.**

### <a name="to-enable-the-oms-agent-for-linux-to-communicate-with-operations-manager"></a>Pour activer l’Agent de l’OMS pour Linux communiquer avec Operations Manager

1. Modifier le fichier /etc/opt/omi/conf/omiserver.conf
2. Assurez-vous que la ligne qui commence par **httpsport =** définit le port 1270. Tels que`httpsport=1270`
3. Redémarrez le serveur de l’OMI :

    ```
    service omiserver restart or systemctl restart omiserver
    ```




## <a name="database-permissions-required-for-mysql-performance-counters"></a>Autorisations de base de données requises pour les compteurs de performance de MySQL

Pour accorder des autorisations à un utilisateur de surveillance MySQL, l’utilisateur doit avoir le privilège « GRANT option » ainsi que le privilège est accordé.

Afin que l’utilisateur MySQL renvoyer les données de performance de l’utilisateur devront accéder pour les requêtes suivantes :

```
SHOW GLOBAL STATUS;
SHOW GLOBAL VARIABLES:
```

En plus de ces requêtes MySQL utilisateur requiert l’accès SELECT sur les tables par défaut suivantes :

- INFORMATION_SCHEMA
- MySQL

Ces privilèges peuvent être accordées en exécutant les commandes suivantes de la subvention.

```
GRANT SELECT ON information_schema.* TO ‘monuser’@’localhost’;
GRANT SELECT ON mysql.* TO ‘monuser’@’localhost’;
```

## <a name="manage-mysql-monitoring-credentials-in-the-authentication-file"></a>Gérer la surveillance des informations d’identification dans le fichier d’authentification de MySQL

Les sections suivantes vous aident à gérer les informations d’identification de MySQL.

### <a name="configure-the-mysql-omi-provider"></a>Configurer le fournisseur de MySQL OMI

Le fournisseur de MySQL OMI nécessite que l’utilisateur MySQL préconfiguré et installé les bibliothèques clientes MySQL pour interroger les informations d’intégrité des performances de l’instance de MySQL.

### <a name="mysql-omi-authentication-file"></a>Fichier d’authentification MySQL OMI

MySQL OMI utilise un fichier d’authentification pour déterminer quelle adresse de liaison et le port de l’instance est à l’écoute de MySQL et quelles informations d’identification à utiliser pour collecter des mesures. Lors de l’installation de l’OMI MySQL fournisseur analyse les fichiers de configuration de le my.cnf de MySQL (emplacements par défaut) pour la liaison-adresse et le port et partiellement défini l’OMI MySQL fichier d’authentification.

Pour effectuer la surveillance d’une instance de serveur MySQL, ajouter un fichier d’authentification MySQL OMI prégénérée dans le répertoire approprié.

### <a name="authentication-file-format"></a>Format de fichier de l’authentification

Le fichier d’authentification MySQL OMI est un fichier texte qui contient des informations sur :

- Port
- Adresse de liaison
- Nom d’utilisateur de MySQL
- Mot de passe encodé en base 64

Le fichier d’authentification MySQL OMI accorde uniquement des droits en lecture/écriture à l’utilisateur de Linux qui l’a généré.

```
[Port]=[Bind-Address], [username], [Base64 encoded Password]
(Port)=(Bind-Address), (username), (Base64 encoded Password)
(Port)=(Bind-Address), (username), (Base64 encoded Password)
AutoUpdate=[true|false]
```

Un fichier de MySQL OMI d’authentification par défaut contient une instance par défaut et un numéro de port, selon les informations disponibles et analysé à partir du fichier de configuration de MySQL trouvé.

L’instance par défaut est un moyen de vous permettent de gérer plusieurs instances de MySQL sur un hôte Linux plus facile et est représentée par l’instance avec le port 0. Toutes les instances ajoutées héritera des propriétés définies à partir de l’instance par défaut. Par exemple, si l’instance de MySQL à l’écoute sur le port '3308' est ajouté, l’instance par défaut-adresse de liaison, nom d’utilisateur et mot de passe encodé en base 64 servira pour essayer de contrôler l’instance écoute sur 3308. Si l’instance sur 3308 est relié à une autre adresse et utilise la même paire nom d’utilisateur et le mot de passe MySQL uniquement à la respecification de l’adresse de liaison est nécessaire, et les autres propriétés seront héritées.

Exemples du fichier d’authentification l’aspect suivant :

Instance par défaut et instance avec port 3308 :

```
0=127.0.0.1, myuser, cnBwdA==3308=, ,AutoUpdate=true
```

Instance par défaut et instance avec port 3308 + autre Base 64 codé le mot de passe :

```
0=127.0.0.1, myuser, cnBwdA==3308=127.0.1.1, , AutoUpdate=true
```


| **Propriété** | **Description** |
| --- | --- |
| Port | Port représente le port actuel qu'est à l’écoute sur l’instance de MySQL.  Le port 0 implique que les propriétés suivant sont utilisées pour l’instance par défaut. |
| Adresse de liaison | l’adresse de liaison est en cours MySQL-adresse de liaison |
| nom d’utilisateur | Ce nom d’utilisateur de l’utilisateur de MySQL que vous souhaitez utiliser pour surveiller l’instance de serveur MySQL. |
| Mot de passe crypté en Base64 | C’est le mot de passe de l’utilisateur de surveillance MySQL codé en Base64. |
| Mise à jour automatique | Lorsque le fournisseur de OMI MySQL est mis à niveau le fournisseur de rechercher des modifications dans le fichier my.cnf et remplacer le fichier d’authentification OMI de MySQL. Définissez cet indicateur sur true ou false en fonction des mises à jour requises pour le fichier d’authentification MySQL OMI. |

#### <a name="authentication-file-location"></a>Emplacement des fichiers d’authentification

Le fichier d’authentification de OMI MySQL doit être situé à l’emplacement suivant et nommé « mysql-auth » :

/var/opt/Microsoft/MySQL-cimprov/AUTH/omsagent/MySQL-AUTH

Le fichier (et le répertoire d’authentification/omsagent) doivent être possédés par l’utilisateur omsagent.

## <a name="agent-logs"></a>Journaux de l’agent

Les journaux de l’Agent de l’OMS pour Linux est à :

/ var/opt/microsoft/omsagent/journal /

Les journaux de l’Agent de l’OMS pour Linux omsconfig (configuration de l’agent) programme est à :

/ var/opt/microsoft/omsconfig/journal /

Journaux des composants OMI et SCX (qui fournissent des données métriques de performances) est à :

/ var/opt/omi/log/et /var/opt/microsoft/scx/log

## <a name="troubleshooting-the-oms-agent-for-linux"></a>Résolution des problèmes liés à l’Agent de l’OMS pour Linux

Utilisez les informations suivantes pour diagnostiquer et résoudre les problèmes courants.

Si aucune des informations de résolution des problèmes dans cette section vous aide à vous, vous pouvez également utiliser les ressources suivantes pour aider à résoudre votre problème.

- Clients avec le Premier support peut se connecter à une demande de support via [Premier](https://premier.microsoft.com/)
- Clients disposant de contrats de support Azure peuvent se connecter à des cas de prise en charge dans le [portail Azure](https://manage.windowsazure.com/?getsupport=true)
- Un [Problème de GitHub](https://github.com/Microsoft/OMS-Agent-for-Linux/issues) de fichier
- Évaluations des idées et pour créer un bogue rapport [http://aka.ms/opinsightsfeedback](http://aka.ms/opinsightsfeedback)

### <a name="important-log-locations"></a>Emplacement des journaux important

Fichier | Chemin d’accès
---- | -----
Agent d’OMS pour le fichier journal de Linux | `/var/opt/microsoft/omsagent/log/omsagent.log `
Fichier journal de Configuration de l’Agent OMS | `/var/opt/microsoft/omsconfig/omsconfig.log`

### <a name="important-configuration-files"></a>Fichiers de configuration importants

Catergory | Emplacement du fichier
----- | -----
Journal système | `/etc/syslog-ng/syslog-ng.conf`ou `/etc/rsyslog.conf` ou`/etc/rsyslog.d/95-omsagent.conf`
Performances, Nagios, Zabbix, sortie de l’OMS et l’agent générales | `/etc/opt/microsoft/omsagent/conf/omsagent.conf`
Configurations supplémentaires | `/etc/opt/microsoft/omsagent/conf.d/*.conf`

>[AZURE.NOTE] Modification des fichiers de configuration pour les compteurs de performance et syslog sont remplacés si la Configuration du portail OMS est activée. Vous pouvez désactiver la configuration du portail de l’OMS (pour tous les nœuds) ou pour les nœuds unique en exécutant la commande suivante :

```
sudo su omsagent -c /opt/microsoft/omsconfig/Scripts/OMS_MetaConfigHelper.py --disable
```


### <a name="enable-debug-logging"></a>Activer la journalisation du débogage

Pour activer la journalisation du débogage, vous pouvez utiliser le plug-in de sortie OMS et documenté.

#### <a name="oms-output-plugin"></a>Plug-in de sortie OMS

FluentD permet du plug-in spécifier les niveaux de journalisation pour les niveaux du journal différentes pour les entrées et les sorties. Pour spécifier un niveau de journal différent pour la sortie de l’OMS, modifier la configuration de l’agent générales dans le `/etc/opt/microsoft/omsagent/conf/omsagent.conf` fichier.

Au bas du fichier de configuration, vous devez modifier le `log_level` propriété à partir de `info` à `debug`.

 ```
 <match oms.** docker.**>
  type out_oms
  log_level debug
  num_threads 5
  buffer_chunk_limit 5m
  buffer_type file
  buffer_path /var/opt/microsoft/omsagent/state/out_oms*.buffer
  buffer_queue_limit 10
  flush_interval 20s
  retry_limit 10
  retry_wait 30s
</match>
 ```

Enregistrement de débogage vous permet de voir les téléchargements en lot au Service OMS séparés par type, le nombre d’éléments de données et le temps nécessaire pour envoyer.

*Exemple de journal de débogage activée :*
```
Success sending oms.nagios x 1 in 0.14s
Success sending oms.omi x 4 in 0.52s
Success sending oms.syslog.authpriv.info x 1 in 0.91s
```

#### <a name="verbose-output"></a>Sortie détaillée
Au lieu d’utiliser le plug-in de sortie OMS, vous pouvez également copier les éléments de données directement à `stdout`, qui est visible dans l’Agent de l’OMS pour le fichier journal de Linux.

Dans le fichier de configuration de l’agent générales OMS à `/etc/opt/microsoft/omsagent/conf/omsagent.conf`, commenter les plug-in de sortie de l’OMS en ajoutant une `#` devant chaque ligne.

```
#<match oms.** docker.**>
#  type out_oms
#  log_level info
#  num_threads 5
#  buffer_chunk_limit 5m
#  buffer_type file
#  buffer_path /var/opt/microsoft/omsagent/state/out_oms*.buffer
#  buffer_queue_limit 10
#  flush_interval 20s
#  retry_limit 10
#  retry_wait 30s
#</match>
```

Ci-dessous le plug-in de sortie, supprimez le commentaire dans la section suivante en supprimant le `#` symbole au début de chaque ligne.

```
<match **>
  type stdout
</match>
```

### <a name="forwarded-syslog-messages-do-not-appear-in-the-log"></a>Les messages Syslog transférés n’apparaissent pas dans le journal

#### <a name="probable-causes"></a>Causes probables

- La configuration appliquée au serveur Linux ne permet pas de collection des installations envoyées et/ou des niveaux du journal
- Syslog n'est pas transférée correctement sur le serveur Linux
- Le nombre de messages transférés par seconde est trop grand pour la configuration de base de l’Agent de l’OMS pour Linux à gérer

#### <a name="resolutions"></a>Résolutions

- Vérifiez que la configuration du portail de l’OMS pour Syslog toutes les facilités et les niveaux du journal correct
  - **OMS Portal > Paramètres > données > Syslog**
-  Vérifiez que natif syslog messagerie démons (`rsyslog`, `syslog-ng`) sont en mesure de recevoir les messages transférés
- Vérifiez les paramètres du pare-feu sur le serveur Syslog pour vous assurer que les messages ne sont pas bloqués
-  Simuler un message Syslog pour OMS à l’aide de la `logger` de commande - par exemple :
  - `logger -p local0.err "This is my test message"`

### <a name="problems-connecting-to-oms-when-using-a-proxy"></a>Problèmes de connexion à OMS lors de l’utilisation d’un proxy

#### <a name="probable-causes"></a>Causes probables

- Le serveur proxy spécifié lors de l’installation et la configuration de l’agent sont incorrect
- Les points de terminaison de Service OMS ne sont pas whitelistested dans votre centre de données

#### <a name="resolutions"></a>Résolutions

- Réinstallez l’Agent de l’OMS pour Linux à l’aide de la commande suivante avec l’option `-v` activé. Cela permet une sortie détaillée de l’agent de se connecter via le serveur proxy pour le Service OMS.
  - `/opt/microsoft/omsagent/bin/omsadmin.sh -w <OMS Workspace ID> -s <OMS Workspace Key> -p <Proxy Conf> -v`
  - Consultez la documentation de la [configuration de l’agent pour une utilisation avec un serveur proxy HTTP](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#configuring-the-agent-for-use-with-an-http-proxy-server) proxy d’OMS
- Vérifiez que les points de terminaison de Service OMS suivants sont autorisés

Ressource de l’agent | Ports
---- | ----
& #42 ;. ODS.opinsights.Azure.com | Port 443
& #42 ;. OMS.opinsights.Azure.com | Port 443
ODS.systemcenteradvisor.com | Port 443
& #42;.blob.core.windows.net/ | Port 443

### <a name="a-403-error-is-displayed-when-onboarding"></a>Une erreur 403 s’affiche lors de l’intégration

#### <a name="probable-causes"></a>Causes probables

- La date et l’heure sont incorrectes sur un serveur Linux
- L’ID de l’espace de travail et de la clé espace de travail sont incorrects

#### <a name="resolution"></a>Résolution

- Vérifiez l’heure de votre serveur Linux avec le `date` commande. Si les données sont supérieures ou inférieure à 15 minutes à l’heure actuelle, intégration échoue. Pour résoudre ce problème, mettez à jour de la date et/ou le fuseau horaire de votre serveur Linux.
- La dernière version de l’Agent de l’OMS pour Linux vous avertit si une différence de temps entraîne l’échec de l’intégration
- RE-intégré à l’aide de l’ID de l’espace de travail correcte et la clé de l’espace de travail. Pour plus d’informations, consultez [intégration à l’aide de la ligne de commande](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line) .

### <a name="a-500-error-or-404-error-appears-in-the-log-file-after-onboarding"></a>Une erreur 500 ou erreur 404 s’affiche dans le fichier journal après intégration

Il s’agit d’un problème connu qui se produit lors du premier téléchargement de données de Linux dans un espace de travail de l’OMS. Cela n’affecte pas les données envoyées ou autres problèmes. Vous pouvez ignorer les erreurs lorsque initialement arrivant.

### <a name="nagios-data-does-not-appear-in-the-oms-portal"></a>Données de Nagios n’apparaissant pas dans le portail de l’OMS

#### <a name="probable-causes"></a>Causes probables
- L’utilisateur omsagent n’a pas les autorisations nécessaires pour lire le fichier journal Nagios
- Les sections de Nagios source et de filtre sont toujours mis en commentaire dans le fichier omsagent.conf

#### <a name="resolutions"></a>Résolutions

- Ajoutez l’utilisateur omsagent pour lire le fichier Nagios. Pour plus d’informations, reportez-vous à la section [alertes de Nagios](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#nagios-alerts) .
- Dans l’Agent de l’OMS pour le fichier de configuration général de Linux à `/etc/opt/microsoft/omsagent/conf/omsagent.conf`, assurez-vous qu' **à la fois** la source Nagios et filtre sections comportent des commentaires supprimés, similaires à l’exemple suivant.

```
<source>
  type tail
  path /var/log/nagios/nagios.log
  format none
  tag oms.nagios
</source>

<filter oms.nagios>
  type filter_nagios_log
</filter>
```


### <a name="linux-data-doesnt-appear-in-the-oms-portal"></a>Données de Linux ne s’affiche pas dans le portail de l’OMS

#### <a name="probable-causes"></a>Causes probables

- Échec de l’intégration au Service OMS
- Connexion au Service OMS est bloquée.
- L’Agent de l’OMS pour les données de Linux est sauvegardé

#### <a name="resolutions"></a>Résolutions

- Vérifiez qu’arrivant au Service OMS a réussi en vérifiant que la `/etc/opt/microsoft/omsagent/conf/omsadmin.conf` existe.
- RE-intégré à l’aide de la ligne de commande omsadmin.sh. Pour plus d’informations, consultez [intégration à l’aide de la ligne de commande](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line) .
- Si vous utilisez un proxy, utilisez le proxy dépannage ci-dessus
- Dans certains cas, lorsque l’Agent de l’OMS pour Linux ne peuvent pas communiquer avec le Service OMS, les données de l’Agent sont sauvegardé à la taille de mémoire tampon totale de 50 Mo. Redémarrez l’Agent de l’OMS pour Linux en exécutant l’ou l’autre le `service omsagent restart` ou `systemctl restart omsagent` commandes.
  >[AZURE.NOTE] Ce problème est résolu dans l’Agent version 1.1.0-28 ou version ultérieure.

### <a name="syslog-linux-performance-counter-configuration-is-not-applied-in-the-oms-portal"></a>Configuration de compteur de performance de Syslog Linux n’est pas appliquée dans le portail de l’OMS

#### <a name="probable-causes"></a>Causes probables

- L’agent de configuration de l’Agent de l’OMS pour Linux n’a pas récupéré la configuration la plus récente à partir du portail de l’OMS.
- Les valeurs modifiées dans le portail n’ont pas été appliquées.

#### <a name="resolutions"></a>Résolutions

`omsconfig`est l’agent de configuration de l’Agent de l’OMS pour Linux qui Récupère les modifications de configuration du portail OMS toutes les 5 minutes. Cette configuration est ensuite appliquée à l’Agent de l’OMS pour les fichiers de configuration Linux qui se trouvent à `/etc/opt/microsoft/omsagent/conf/omsagent.conf`.

- Dans certains cas, l’Agent de l’OMS pour l’agent de configuration Linux n’est peut-être pas en mesure de communiquer avec le service de configuration du portail résultant dans la configuration la plus récente ne s’applique ne pas.
- Vérifiez que le `omsconfig` l’agent est installé avec les éléments suivants :
  - `dpkg --list omsconfig`ou`rpm -qi omsconfig`
  - Le cas contraire, réinstallez la version la plus récente de l’Agent de l’OMS pour Linux

- Vérifiez que le `omsconfig` agent peut communiquer avec le service OMS
  - Exécuter le `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'` commande
    - La commande ci-dessus retourne la configuration de cet agent récupère à partir du portail, y compris les paramètres du journal système, des compteurs de performance Linux et journaux personnalisés
    - En cas d’échec de la commande ci-dessus, exécutez le `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py` commande. Cette commande oblige l’agent de l’omsconfig de communiquer avec le service OMS pour récupérer la configuration la plus récente.


### <a name="custom-linux-log-data-does-not-appear-in-the-oms-portal"></a>Les données de journal Linux personnalisé n’apparaissant pas dans le portail de l’OMS

#### <a name="probable-causes"></a>Causes probables

- Intégration au Service OMS a échoué
- Le paramètre **Appliquer la configuration suivante à mes serveurs Linux** n’a pas été sélectionné.
- omsconfig n’a pas chercher le dernier journal personnalisé à partir du portail
- Le `omsagent` utilisation ne parvient pas à accéder au journal personnalisé en raison d’un problème d’autorisations ou `omsagent` n’a été trouvé. Dans ce cas, vous verrez la sortie suivante :
  - `[DATETIME] [warn]: file not found. Continuing without tailing it.`
  - `[DATETIME] [error]: file not accessible by omsagent.`
- Il s’agit d’un problème connu avec la Condition de concurrence qui a été corrigé dans l’Agent de l’OMS pour Linux version 1.1.0-217

#### <a name="resolutions"></a>Résolutions
- Vérifiez que vous avez correctement onboarded, en déterminant si le `/etc/opt/microsoft/omsagent/conf/omsadmin.conf` fichier existe.
  - Si nécessaire, intégré à nouveau à l’aide de la ligne de commande omsadmin.sh. Pour plus d’informations, consultez [intégration à l’aide de la ligne de commande](https://github.com/Microsoft/OMS-Agent-for-Linux/blob/master/docs/OMS-Agent-for-Linux.md#onboarding-using-the-command-line) .
- Dans le portail de l’OMS, sous **paramètres** de l’onglet **données** , vérifiez que le paramètre **Appliquer la configuration suivante à mes serveurs Linux** est sélectionné.  
  ![appliquer la configuration](./media/log-analytics-linux-agents/customloglinuxenabled.png)

- Vérifiez que le `omsconfig` agent peut communiquer avec le service OMS
  - Exécuter le `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/GetDscConfiguration.py'` commande
  - La commande ci-dessus retourne la configuration de cet agent récupère à partir du portail, y compris les paramètres du journal système, des compteurs de performance Linux et journaux personnalisés
  - En cas d’échec de la commande ci-dessus, exécutez le `sudo su omsagent -c 'python /opt/microsoft/omsconfig/Scripts/PerformRequiredConfigurationChecks.py` commande. Cette commande oblige l’agent de l’omsconfig de communiquer avec le service OMS et de récupérer la configuration la plus récente.


Au lieu de l’Agent de l’OMS pour les utilisateurs de Linux en tant qu’un utilisateur privilégié `root`, l’Agent de l’OMS pour Linux fonctionne comme le `omsagent` utilisateur. Dans la plupart des cas, une autorisation explicite doit disposer à l’utilisateur pour lire certains fichiers.

Pour accorder l’autorisation de `omsagent` utilisateur, exécutez les commandes suivantes :

1. Ajouter le `omsagent` utilisateur à un groupe spécifique`sudo usermod -a -G <GROUPNAME> <USERNAME>`
2. Accorder l’accès en lecture universel le fichier requis avec`sudo chmod -R ugo+rw <FILE DIRECTORY>`

Il existe un problème connu avec la Condition de concurrence qui a été corrigé dans l’Agent de l’OMS pour Linux version 1.1.0-217. Après la mise à jour à l’agent plus récent, exécutez la commande suivante pour obtenir la dernière version du plug-in de la sortie :

```
sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.conf /etc/opt/microsoft/omsagent/conf/omsagent.conf
```

## <a name="known-limitations"></a>Limitations connues
Consultez les sections suivantes pour en savoir plus sur les limites actuelles de l’Agent de l’OMS pour Linux.

### <a name="azure-diagnostics"></a>Diagnostics de Windows Azure

Pour les machines virtuelles du Linux qui exécute dans Azure, des étapes supplémentaires peuvent être nécessaires pour permettre la collecte de données par les tests de diagnostic Azure et Operations Management Suite. **Version 2.2** de l’Extension de Diagnostics pour Linux est nécessaire pour assurer la compatibilité avec l’Agent de l’OMS pour Linux.

Pour plus d’informations sur l’installation et la configuration de l’Extension de Diagnostic pour Linux, voir [utilisation de la commande CLI d’Azure pour activer l’Extension de Diagnostic de Linux](../virtual-machines/virtual-machines-linux-classic-diagnostic-extension.md#use-the-azure-cli-command-to-enable-the-linux-diagnostic-extension).

**Mettez à niveau l’Extension Diagnostics de Linux 2.0 vers 2.2 CLI Azure ASM :**

```
azure vm extension set -u <vm_name> LinuxDiagnostic Microsoft.OSTCExtensions 2.0
azure vm extension set <vm_name> LinuxDiagnostic Microsoft.OSTCExtensions 2.2 --private-config-path PrivateConfig.json
```

**ARM**

```
azure vm extension set -u <resource-group> <vm-name> Microsoft.Insights.VMDiagnosticsSettings Microsoft.OSTCExtensions 2.0
azure vm extension set <resource-group> <vm-name> LinuxDiagnostic Microsoft.OSTCExtensions 2.2 --private-config-path PrivateConfig.json
```

Ces exemples de commande font référence à un fichier nommé PrivateConfig.json. Le format de ce fichier doit ressembler à l’exemple suivant.

```
    {
    "storageAccountName":"the storage account to receive data",
    "storageAccountKey":"the key of the account"
    }
```

### <a name="sysklog-is-not-supported"></a>Sysklog n’est pas pris en charge.

Rsyslog ou syslog-ng sont requis pour recueillir les messages syslog. Le démon syslog par défaut dans la version 5 de Red Hat Enterprise Linux et Oracle Linux CentOS (sysklog) n’est pas prise en charge de la collecte d’événements syslog. Pour collecter des données de journal système à partir de cette version de ces distributions, le démon rsyslog doit être installé et configuré pour remplacer les sysklog. Pour plus d’informations sur le remplacement de sysklog avec rsyslog, consultez [installer le RPM de rsyslog nouvellement créé](http://wiki.rsyslog.com/index.php/Rsyslog_on_CentOS_success_story#Install_the_newly_built_rsyslog_RPM).

## <a name="next-steps"></a>Étapes suivantes

- [Solutions d’Analytique de journal ajouter à partir de la galerie de Solutions](log-analytics-add-solutions.md) pour ajouter des fonctionnalités et de collecter des données.
- Familiarisez-vous avec les [recherches de journaux](log-analytics-log-searches.md) afficher des informations détaillées collectées par les solutions.
- Utilisez les [tableaux de bord](log-analytics-dashboards.md) pour enregistrer et afficher vos propres recherches personnalisées.
