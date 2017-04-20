<properties
   pageTitle="Configuration de MPIO sur un hôte Linux de StorSimple | Microsoft Azure"
   description="Configuration de MPIO sur StorSimple connecté à un hôte Linux CentOS 6.6 en cours d’exécution"
   services="storsimple"
   documentationCenter="NA"
   authors="alkohli"
   manager="carmonm"
   editor="tysonn" />
<tags
   ms.service="storsimple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="09/21/2016"
   ms.author="alkohli" />

# <a name="configure-mpio-on-a-storsimple-host-running-centos"></a>Configuration de MPIO sur un hôte StorSimple exécutant CentOS

Cet article explique les étapes nécessaires à la configuration d’e/s de chemins d’accès multiples (MPIO) sur un serveur hôte Centos 6.6. Le serveur hôte est connecté à votre périphérique Microsoft Azure StorSimple pour une haute disponibilité via les initiateurs iSCSI. Il décrit en détail la détection automatique des périphériques à chemins multiples et le paramétrage spécifique uniquement pour les volumes de StorSimple.

Cette procédure s’applique à tous les modèles de périphériques de série 8000 de StorSimple.

>[AZURE.NOTE] Cette procédure ne peut pas être utilisée pour un périphérique virtuel StorSimple. Pour plus d’informations, consultez Comment configurer des serveurs pour votre périphérique virtuel.

## <a name="about-multipathing"></a>À propos des chemins d’accès multiples

La fonction de chemins d’accès multiples vous permet de configurer plusieurs chemins d’e/s entre le serveur hôte et un périphérique de stockage. Ces chemins d’e/s sont des connexions SAN physiques incluant des câbles distincts, les commutateurs, les interfaces réseau et les contrôleurs. Chemins d’accès multiples regroupe les chemins d’accès e/s, pour configurer un nouveau périphérique qui est associé à tous les chemins d’accès agrégées.

L’objectif de chemins d’accès multiples est double :

- **Haute disponibilité**: il fournit un chemin de remplacement en cas d’échec de n’importe quel élément du chemin d’e/s (par exemple, un câble, commutateur, interface réseau ou contrôleur).

- **Équilibrage de la charge**: selon la configuration de votre périphérique de stockage, il peut améliorer les performances en détectant les charges sur les chemins d’e/s et dynamiquement rééquilibrer les charges.


### <a name="about-multipathing-components"></a>À propos des composants de chemins d’accès multiples

Chemins d’accès multiples sous Linux est constitué de composants de noyau et espace utilisateur que sous forme de tableau ci-dessous.

- **Noyau**: le composant principal est le *périphérique Mappeur* qui redirige les e/s et prend en charge le basculement pour les chemins d’accès et les groupes de chemin d’accès.

1. **Espace utilisateur**: il s’agit *multipath-tools* qui gèrent des périphériques à chemins multiples en commandant le module multipath-correspondance que faire. Les outils comprennent :

    - **Chemins multiples**: affiche et configure les périphériques multipathed.

    - **Multipathd**: démon qui exécute plusieurs chemins et surveille les chemins d’accès.

    - **Devmap-nom**: fournit un nom de périphérique significatif à udev pour devmaps.

    - **Kpartx**: mappe devmaps linéaire aux partitions de périphérique partitionnables pour cartes à chemins multiples.

    - **Multipath.conf**: fichier de configuration de démon à chemins multiples qui est utilisée pour remplacer la table de configuration intégré.

### <a name="about-the-multipathconf-configuration-file"></a>À propos du fichier de configuration de multipath.conf

Le fichier de configuration `/etc/multipath.conf` met un grand nombre de chemins d’accès multiples fonctionnalités configurables par l’utilisateur. Le `multipath` commande et le démon de noyau `multipathd` utiliser des informations figurant dans ce fichier. Le fichier est consulté uniquement lors de la configuration des périphériques à chemins multiples. Assurez-vous que toutes les modifications sont apportées avant d’exécuter le `multipath` commande. Si vous modifiez le fichier par la suite, vous devrez arrêter et démarrer multipathd à nouveau pour que les modifications soient prises en compte.

Le multipath.conf a cinq sections :

- **Niveaux par défaut du système** *(par défaut)*: vous pouvez remplacer les valeurs par défaut au niveau du système.

1. **Dispositifs de Blacklisted** *(liste de blocage)*: vous pouvez spécifier la liste des périphériques qui ne doivent pas être contrôlés par le périphérique-mappeur.

1. **Exceptions de la liste de blocage** *(blacklist_exceptions)*: vous pouvez identifier des périphériques spécifiques devant être traitée en tant que périphériques à chemins multiples, même si répertoriées dans la liste de blocage.

1. **Paramètres spécifiques de contrôleur de stockage** *(périphériques)*: vous pouvez spécifier des paramètres de configuration qui seront appliquées aux périphériques pour lesquels des informations de produit et de fournisseur.

1. **Paramètres de périphérique spécifiques** *(multipaths)*: vous pouvez utiliser cette section pour ajuster les paramètres de configuration pour des LUN individuelles.

## <a name="configure-multipathing-on-storsimple-connected-to-linux-host"></a>Configurer les chemins d’accès multiples sur StorSimple connecté à un hôte Linux

Un périphérique StorSimple connecté à un hôte Linux peut être configuré pour la haute disponibilité et l’équilibrage de charge. Par exemple, si l’hôte Linux possède deux interfaces connectées au SAN et que le périphérique a deux interfaces connectées au réseau SAN, tels que ces interfaces soient sur le même sous-réseau, puis il sera 4 chemins d’accès disponibles. Cependant, si chaque interface de données de l’interface de périphérique et de l’hôte est sur un autre sous-réseau IP (et non routables), puis seulement 2 chemins sera disponibles. Vous pouvez configurer les chemins d’accès multiples pour automatiquement détecter tous les chemins disponibles, choisissez un algorithme d’équilibrage de la charge pour ces chemins d’accès, appliquer les paramètres de configuration spécifiques pour les volumes StorSimple uniquement, puis activer et vérifiez les chemins d’accès multiples.

La procédure suivante décrit comment configurer les chemins d’accès multiples lorsqu’un périphérique StorSimple avec deux interfaces réseau est connecté à un hôte avec deux interfaces réseau.

## <a name="prerequisites"></a>Conditions préalables

Cette section détaille la configuration préalable pour les serveur CentOS et votre périphérique StorSimple.

### <a name="on-centos-host"></a>Sur l’hôte de CentOS



1. Assurez-vous que votre hôte CentOS a 2 interfaces réseau activé. Type :

    `ifconfig`

    L’exemple suivant montre la sortie, lorsque deux interfaces réseau (`eth0` et `eth1`) sont présents sur l’hôte.

        [root@centosSS ~]# ifconfig
        eth0  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:41  
        inet addr:10.126.162.65  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3341/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3341/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:36536 errors:0 dropped:0 overruns:0 frame:0
        TX packets:6312 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:13994127 (13.3 MiB)  TX bytes:645654 (630.5 KiB)

        eth1  Link encap:Ethernet  HWaddr 00:15:5D:A2:33:42  
        inet addr:10.126.162.66  Bcast:10.126.163.255  Mask:255.255.252.0
        inet6 addr: 2001:4898:4010:3012:215:5dff:fea2:3342/64 Scope:Global
        inet6 addr: fe80::215:5dff:fea2:3342/64 Scope:Link
        UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
        RX packets:25962 errors:0 dropped:0 overruns:0 frame:0
        TX packets:11 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:1000
        RX bytes:2597350 (2.4 MiB)  TX bytes:754 (754.0 b)

        loLink encap:Local Loopback  
        inet addr:127.0.0.1  Mask:255.0.0.0
        inet6 addr: ::1/128 Scope:Host
        UP LOOPBACK RUNNING  MTU:65536  Metric:1
        RX packets:12 errors:0 dropped:0 overruns:0 frame:0
        TX packets:12 errors:0 dropped:0 overruns:0 carrier:0
        collisions:0 txqueuelen:0
        RX bytes:720 (720.0 b)  TX bytes:720 (720.0 b)


1. Installer les *utilitaires de l’initiateur iSCSI* sur votre serveur CentOS. Procédez comme suit pour installer les *utilitaires de l’initiateur iSCSI*.

    1. Ouvrez une session en tant que `root` dans votre hôte CentOS.

    1. Installez les *utilitaires de l’initiateur iSCSI*. Type :

        `yum install iscsi-initiator-utils`


    1. Après que les *utilitaires de l’initiateur iSCSI* est installé, démarrez le service iSCSI. Type :

        `service iscsid start`

        Parfois, `iscsid` ne peut effectivement pas démarrer et le `--force` option peut s’avérer nécessaire.

    1. Pour vous assurer que votre initiateur iSCSI est activé lors du démarrage, utilisez la `chkconfig` commande pour activer le service.

        `chkconfig iscsi on`


    1. Pour vérifier qu’il a été correctement le programme d’installation, exécutez la commande :

        `chkconfig --list | grep iscsi`

        Vous trouverez ci-dessous un exemple de sortie.

            iscsi   0:off   1:off   2:on3:on4:on5:on6:off
            iscsid  0:off   1:off   2:on3:on4:on5:on6:off

        Dans l’exemple ci-dessus, vous pouvez voir que votre environnement iSCSI s’exécutera à l’heure de démarrage sur les niveaux d’exécution 2, 3, 4 et 5.


1. Installer un *périphérique-Mappeur-MPIO*. Type :

    `yum install device-mapper-multipath`

    Le programme d’installation démarre. Tapez **Y** pour continuer lorsque vous êtes invité à confirmer.



### <a name="on-storsimple-device"></a>Périphérique de StorSimple

Votre périphérique de StorSimple devez avoir :

- Un minimum de deux interfaces activées pour iSCSI. Pour vérifier que les deux interfaces sont compatibles iSCSI sur le périphérique StorSimple, effectuez les opérations suivantes dans le portail Azure classique de votre périphérique StorSimple :

    1. Connectez-vous au portail classique de votre périphérique StorSimple.

    1. Sélectionnez votre service de gestionnaire de StorSimple, cliquez sur **périphériques** et choisissez le périphérique StorSimple. Cliquez sur **configurer** et vérifiez les paramètres de l’interface réseau. Voici une capture d’écran avec deux interfaces réseau iSCSI. DONNÉES 2 et données 3, à la fois 10 GbE interfaces sont activées pour iSCSI.

        ![Configuration de MPIO StorsSimple données 2](./media/storsimple-configure-mpio-on-linux/IC761347.png)

        ![MPIO StorSimple données 3 Config](./media/storsimple-configure-mpio-on-linux/IC761348.png)

        Dans la page **configurer**

        1. Assurez-vous que les deux interfaces réseau sont compatibles avec iSCSI. Le champ **iSCSI activé** doit indiquer **Oui**.
        2. Vérifiez que les interfaces réseau ont la même vitesse, les deux doivent être de 1 Gigabit Ethernet ou Gigabit Ethernet de 10.
        3. Notez les adresses IPv4 des interfaces prenant en charge l’iSCSI et les enregistrer pour une utilisation ultérieure sur l’hôte.


- Les interfaces iSCSI sur le périphérique StorSimple doivent être accessibles à partir du serveur CentOS.

    Pour ce faire, vous devez fournir les adresses IP des interfaces réseau iSCSI StorSimple sur un serveur hôte. Les commandes utilisées et à la sortie correspondante avec données2 (10.126.162.25) et DATA3 (10.126.162.26) est présenté ci-dessous :

        [root@centosSS ~]# iscsiadm -m discovery -t sendtargets -p 10.126.162.25:3260
        10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target
        10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g44mt-target


### <a name="hardware-configuration"></a>Configuration matérielle

Nous vous recommandons les deux interfaces de réseau iSCSI sur les chemins d’accès séparés pour la redondance. La figure ci-dessous illustre la configuration matérielle recommandée pour une haute disponibilité et l’équilibrage de la charge de chemins multiples pour votre serveur de CentOS et un dispositif de StorSimple.

![Configuration matérielle de MPIO pour StorSimple sur l’hôte Linux](./media/storsimple-configure-mpio-on-linux/MPIOHardwareConfigurationStorSimpleToLinuxHost2M.png)

Comme illustré dans la figure précédente :

- Votre périphérique de StorSimple est dans une configuration active-passive avec deux contrôleurs.

- Deux commutateurs SAN sont connectés à vos contrôleurs de périphérique.

- Deux initiateurs iSCSI sont activées sur le périphérique StorSimple.

- Deux interfaces réseau sont activées sur votre hôte CentOS.

La configuration ci-dessus produira 4 chemins d’accès séparés entre votre appareil et de l’hôte si les interfaces de l’hôte et les données sont routables.

>[AZURE.IMPORTANT]
>
>- Nous vous recommandons de ne pas mélanger 1 Gigabit Ethernet et les interfaces de réseau 10 Gigabit Ethernet pour les chemins d’accès multiples. Lors de l’utilisation de deux interfaces réseau, les interfaces doivent être du même type.
>- Sur votre périphérique StorSimple, DATA0 et DATA1, données4 DATA5 sont des interfaces Gigabit Ethernet 1 tandis que DATA2 et DATA3 sont des interfaces de réseau 10 Gigabit Ethernet. |

## <a name="configuration-steps"></a>Étapes de configuration

Les étapes de configuration pour les chemins d’accès multiples d’impliquer la configuration les chemins disponibles pour la découverte automatique, qui spécifie l’algorithme d’équilibrage de charge à utiliser, l’activation des chemins d’accès multiples et enfin vérification de la configuration. Chacune de ces étapes est décrite en détail dans les sections suivantes.

### <a name="step-1-configure-multipathing-for-automatic-discovery"></a>Étape 1 : Configurez les chemins d’accès multiples pour la découverte automatique

Les périphériques pris en charge MPIO peuvent être automatiquement détectés et configurés.

1. Initialiser `/etc/multipath.conf` fichier. Type :

     `Copy mpathconf --enable`

    La commande ci-dessus crée un `sample/etc/multipath.conf` fichier.


1. Démarrez le service de chemins multiples. Type :

    ``Copy service multipathd start``

    Vous verrez la sortie suivante :

    `Starting multipathd daemon:`

1. Activer la détection automatique de multipaths. Type :

    `mpathconf --find_multipaths y`

    Cela modifiera la section Paramètres par défaut de votre `multipath.conf` comme indiqué ci-dessous :

        defaults {
        find_multipaths yes
        user_friendly_names yes
        path_grouping_policy multibus
        }

### <a name="step-2-configure-multipathing-for-storsimple-volumes"></a>Étape 2 : Configurer les chemins d’accès multiples pour les volumes de StorSimple

Par défaut, tous les périphériques sont en noirs dans le fichier multipath.conf et seront ignorés. Vous devez créer des exceptions de la liste de blocage pour autoriser des chemins d’accès multiples pour les volumes à partir de périphériques de StorSimple.

1. Modifier la `/etc/mulitpath.conf` fichier. Type :

    `vi /etc/multipath.conf`

1. Recherchez la section blacklist_exceptions du fichier multipath.conf. Votre périphérique StorSimple doit figurer sous la forme d’une exception de la liste de blocage dans cette section. Vous pouvez supprimer les lignes appropriées dans ce fichier pour le modifier comme indiqué ci-dessous (utilisez uniquement le modèle du périphérique que vous utilisez) :

        blacklist_exceptions {
            device {
                       vendor  "MSFT"
                       product "STORSIMPLE 8100*"
            }
            device {
                       vendor  "MSFT"
                       product "STORSIMPLE 8600*"
            }
           }

### <a name="step-3-configure-round-robin-multipathing"></a>Étape 3 : Configurer les chemins d’accès multiples alternée

Cet algorithme d’équilibrage de la charge utilise tous les multipaths disponibles au contrôleur actif de manière équilibrée, round robin.

1. Modifier la `/etc/multipath.conf` fichier. Type :

    `vi /etc/multipath.conf`

1. Sous le `defaults` section, définir le `path_grouping_policy` à `multibus`. Le `path_grouping_policy` Spécifie le chemin d’accès par défaut regroupant les stratégies à appliquer aux multipaths non spécifiées. La section de paramètres par défaut se présente comme indiqué ci-dessous.

        defaults {
                user_friendly_names yes
                path_grouping_policy multibus
        }



> [AZURE.NOTE]
> Les valeurs les plus courantes de `path_grouping_policy` incluent :

> - basculement sur incident = 1 chemin par groupe de priorité
> - multibus = tous les chemins d’accès valides dans le groupe de 1 priorité

### <a name="step-4-enable-multipathing"></a>Étape 4 : Activer Multipath

1. Redémarrez le `multipathd` démon. Type :

    `service multipathd restart`

1. Le résultat sera comme indiqué ci-dessous :

        [root@centosSS ~]# service multipathd start
        Starting multipathd daemon:  [OK]




### <a name="step-5-verify-multipathing"></a>Étape 5 : Vérifiez les chemins d’accès multiples

1. Assurez-vous d’abord qu’iSCSI est établie avec le périphérique StorSimple comme suit :


    1. Découvrez votre périphérique StorSimple. Type :

        `iscsiadm -m discovery -t sendtargets -p  <IP address of network interface on the device>:<iSCSI port on StorSimple device>`

        La sortie lorsque l’adresse IP DATA0 est 10.126.162.25 et le port 3260 est ouvert sur le périphérique StorSimple pour le trafic sortant iSCSI est comme indiqué ci-dessous :

            10.126.162.25:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target
            10.126.162.26:3260,1 iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target


        Copier le nom IQN de votre périphérique StorSimple, `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`, à partir du résultat précédent.



    1. Connecter le périphérique à l’aide de la cible IQN. Le StorSimple est le périphérique iSCSI cible ici. Type :

        `iscsiadm -m node --login -T <IQN of iSCSI target>`

        L’exemple suivant montre la sortie avec un nom qualifié de la cible de `iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target`. La sortie indique que vous êtes connecté à deux interfaces réseau iSCSI sur votre périphérique.

            Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
            Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] (multiple)
            Logging in to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
            Logging in to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] (multiple)
            Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
            Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.25,3260] successful.
            Login to [iface: eth0, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.
                Login to [iface: eth1, target: iqn.1991-05.com.microsoft:storsimple8100-shx0991003g00dv-target, portal: 10.126.162.26,3260] successful.


        Si vous voyez l’interface qu’un hôte et deux chemins d’accès, vous devez ensuite activer les interfaces sur hôte pour iSCSI. Vous pouvez suivre les [instructions détaillées dans la documentation de Linux](https://access.redhat.com/documentation/Red_Hat_Enterprise_Linux/5/html/Online_Storage_Reconfiguration_Guide/iscsioffloadmain.html).


    1. Un volume est exposé au serveur CentOS à partir du périphérique StorSimple. Pour plus d’informations, consultez [étape 6 : créer un volume](storsimple-deployment-walkthrough.md#step-6-create-a-volume) via le portail Azure classique sur le périphérique StorSimple.

    1. Vérifiez les chemins d’accès disponibles. Type :

        `multipath –l`

        L’exemple suivant montre la sortie de deux interfaces réseau sur un périphérique StorSimple connecté à une interface de réseau hôte unique avec deux chemins disponibles.

            mpathb (36486fd20cc081f8dcd3fccb992d45a68) dm-3 MSFT,STORSIMPLE 8100
            size=100G features='0' hwhandler='0' wp=rw
            `-+- policy='round-robin 0' prio=0 status=active
              |- 7:0:0:1 sdc 8:32 active undef running
              `- 6:0:0:1 sdd 8:48 active undef running

        L’exemple suivant montre la sortie de deux interfaces réseau sur un périphérique StorSimple connecté à deux interfaces de réseau hôte avec quatre chemins disponibles.

            mpathb (36486fd27a23feba1b096226f11420f6b) dm-2 MSFT,STORSIMPLE 8100
            size=100G features='0' hwhandler='0' wp=rw
            `-+- policy='round-robin 0' prio=0 status=active
              |- 17:0:0:0 sdb 8:16 active undef running
              |- 15:0:0:0 sdd 8:48 active undef running
              |- 14:0:0:0 sdc 8:32 active undef running
              `- 16:0:0:0 sde 8:64 active undef running

        Après avoir configuré les chemins d’accès, consultez les instructions spécifiques à votre système d’exploitation hôte (point 6.6 Centos) pour monter et formater ce volume.


## <a name="troubleshoot-multipathing"></a>Résoudre les chemins d’accès multiples

Cette section fournit des conseils utiles si vous rencontrez des problèmes lors de la configuration des chemins d’accès multiples.

Q. Je ne vois pas les modifications apportées à `multipath.conf` fichier de prise d’effet.

A. Si vous avez apporté des modifications à la `multipath.conf` fichier, vous devez redémarrer le service de chemins multiples. Tapez la commande suivante :

    service multipathd restart

Q. J’ai activé deux interfaces réseau sur le périphérique StorSimple et deux interfaces réseau sur l’hôte. Lorsque la liste de chemins d’accès disponibles, voir uniquement deux chemins d’accès. Prévu voir les quatre chemins disponibles.

A. Assurez-vous que les deux chemins sont sur le même sous-réseau et routable. Si les interfaces réseau sont sur des VLAN différents et non routable, vous verrez seulement deux chemins d’accès. Une vérifier cela consiste à s’assurer que vous pouvez atteindre les interfaces hôtes à partir d’une interface réseau sur le périphérique StorSimple. Vous devez [contacter le support technique de Microsoft](storsimple-contact-microsoft-support.md) en tant que cette vérification peut uniquement se fera via une session de prise en charge.

Q. Lorsque la liste de chemins d’accès disponibles, je ne vois pas de sortie.

A. En général, ne voient ne pas les chemins d’accès multiples suggère un problème avec le démon de chemins d’accès multiples, et il est probable que n’importe quel problème ici se trouve le `multipath.conf` fichier.

Il est également important de vérifier que vous pouvez réellement voir certains disques après la connexion à la cible, comme les annonces à chemins multiples ne répond pas peut également signifier que vous n’avez pas tous les disques.

- Pour relancer le bus SCSI, utilisez la commande suivante :

    `$ rescan-scsi-bus.sh `(partie du package de sg3_utils)

- Tapez les commandes suivantes :

    `$ dmesg | grep sd*`

- Ou

    `$ fdisk –l`

    Ces renvoie les détails des disques récemment ajoutés.

- Pour déterminer s’il s’agit d’un disque de StorSimple, utilisez les commandes suivantes :

    `cat /sys/block/<DISK>/device/model`

    Cela retourne une chaîne, ce qui va déterminer si il s’agit d’un disque de StorSimple.

A moins de cause probable, mais possible pourrait également être périmé iscsid pid. La commande suivante permet de déconnecter les sessions iSCSI :

    iscsiadm -m node --logout -p <Target_IP>

Répétez cette commande pour toutes les interfaces réseau connecté sur la cible iSCSI, ce qui est de votre périphérique StorSimple. Une fois que vous avez déconnecté à partir de toutes les sessions iSCSI, utilisez la cible iSCSI IQN pour rétablir la session iSCSI. Tapez la commande suivante :

    iscsiadm -m node --login -T <TARGET_IQN>


Q. Je ne sais pas si le périphérique est la liste d’autorisation.

A. Pour vérifier si votre périphérique est la liste d’autorisation, utilisez la commande interactive dépannage suivante :

    multipathd –k
    multipathd> show devices
    available block devices:
    ram0 devnode blacklisted, unmonitored
    ram1 devnode blacklisted, unmonitored
    ram2 devnode blacklisted, unmonitored
    ram3 devnode blacklisted, unmonitored
    ram4 devnode blacklisted, unmonitored
    ram5 devnode blacklisted, unmonitored
    ram6 devnode blacklisted, unmonitored
    ram7 devnode blacklisted, unmonitored
    ram8 devnode blacklisted, unmonitored
    ram9 devnode blacklisted, unmonitored
    ram10 devnode blacklisted, unmonitored
    ram11 devnode blacklisted, unmonitored
    ram12 devnode blacklisted, unmonitored
    ram13 devnode blacklisted, unmonitored
    ram14 devnode blacklisted, unmonitored
    ram15 devnode blacklisted, unmonitored
    loop0 devnode blacklisted, unmonitored
    loop1 devnode blacklisted, unmonitored
    loop2 devnode blacklisted, unmonitored
    loop3 devnode blacklisted, unmonitored
    loop4 devnode blacklisted, unmonitored
    loop5 devnode blacklisted, unmonitored
    loop6 devnode blacklisted, unmonitored
    loop7 devnode blacklisted, unmonitored
    sr0 devnode blacklisted, unmonitored
    sda devnode whitelisted, monitored
    dm-0 devnode blacklisted, unmonitored
    dm-1 devnode blacklisted, unmonitored
    dm-2 devnode blacklisted, unmonitored
    sdb devnode whitelisted, monitored
    sdc devnode whitelisted, monitored
    dm-3 devnode blacklisted, unmonitored


Pour plus d’informations, accédez à [utiliser la résolution des problèmes de commande interactive de chemins d’accès multiples](http://www.centos.org/docs/5/html/5.1/DM_Multipath/multipath_config_confirm.html).

## <a name="list-of-useful-commands"></a>Liste des commandes utiles

|Type de|Commande|Description|
|---|---|---|
|**iSCSI**|`service iscsid start`|Démarrer le service iSCSI|
|&nbsp;|`service iscsid stop`|Arrêtez le service iSCSI|
|&nbsp;|`service iscsid restart`|Redémarrez le service iSCSI|
|&nbsp;|`iscsiadm -m discovery -t sendtargets -p <TARGET_IP>`|Découvrir les cibles disponibles à l’adresse spécifiée|
|&nbsp;|`iscsiadm -m node --login -T <TARGET_IQN>`|Se connecter à la cible iSCSI|
|&nbsp;|`iscsiadm -m node --logout -p <Target_IP>`|Se déconnecter de la cible iSCSI|
|&nbsp;|`cat /etc/iscsi/initiatorname.iscsi`|Imprimer le nom de l’initiateur iSCSI|
|&nbsp;|`iscsiadm –m session –s <sessionid> -P 3`|Vérifier l’état de la session iSCSI et le volume découvert sur l’hôte|
|&nbsp;|`iscsi –m session`|Affiche toutes les sessions iSCSI entre l’hôte et le périphérique de StorSimple|
| | | |
|**Chemins d’accès multiples**|`service multipathd start`|Démarrez les processus multichemin|
|&nbsp;|`service multipathd stop`|Arrêter le démon à chemins multiples|
|&nbsp;|`service multipathd restart`|Redémarrez le démon à chemins multiples|
|&nbsp;|`chkconfig multipathd on` </br> OU </br> `mpathconf –with_chkconfig y`|Activer multipath démon démarrer au démarrage|
|&nbsp;|`multipathd –k`|Démarrez la console interactive pour le dépannage|
|&nbsp;|`multipath –l`|Liste des connexions à chemins multiples et les périphériques|
|&nbsp;|`mpathconf --enable`|Créer un exemple de fichier mulitpath.conf dans`/etc/mulitpath.conf`|
||||

## <a name="next-steps"></a>Étapes suivantes

Comme vous configurez MPIO sur un hôte Linux, vous devez également faire référence aux documents CentoS 6.6 suivants :

- [Configuration de MPIO sur CentOS](http://www.centos.org/docs/5/html/5.1/DM_Multipath/setup_procedure.html)
- [Guide de formation Linux](http://linux-training.be/files/books/LinuxAdm.pdf)
