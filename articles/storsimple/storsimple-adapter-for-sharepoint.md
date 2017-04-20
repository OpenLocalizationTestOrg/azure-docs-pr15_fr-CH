<properties 
   pageTitle="Carte StorSimple pour SharePoint | Microsoft Azure"
   description="Décrit comment installer et configurer ou supprimer la carte de StorSimple pour SharePoint dans une batterie de serveurs SharePoint."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="07/11/2016"
   ms.author="v-sharos" />

# <a name="install-and-configure-the-storsimple-adapter-for-sharepoint"></a>Installer et configurer la carte StorSimple pour SharePoint

## <a name="overview"></a>Vue d’ensemble

La carte StorSimple pour SharePoint est un composant qui vous permet de fournir un stockage flexible Microsoft Azure StorSimple et protection des données pour les batteries de serveurs SharePoint. Vous pouvez utiliser la carte pour déplacer le contenu de l’objet binaire volumineux (BLOB) à partir de bases de données de contenu SQL Server pour le périphérique de stockage cloud Microsoft Azure StorSimple hybride.

La carte StorSimple pour SharePoint fonctionne comme un fournisseur de stockage d’objets BLOB distants (RBS) et utilise la fonctionnalité de stockage BLOB distant de SQL Server pour stocker des contenus non structurés de SharePoint (sous la forme d’objets BLOB) sur un serveur de fichiers qui est sauvegardé par un dispositif de StorSimple.

>[AZURE.NOTE] La carte StorSimple pour SharePoint prend en charge SharePoint Server 2010 à distance stockage BLOB (RBS). Il ne gère pas le stockage BLOB de SharePoint Server 2010 externe (EBS).

- Pour télécharger la carte StorSimple pour SharePoint, accédez à la [Carte de StorSimple pour SharePoint] [ 1] dans du Microsoft Download Center.

- Pour plus d’informations sur la planification, de RBS et RBS, limitations, atteindre le [choix entre utiliser la RBS dans SharePoint 2013] [ 2] ou [prévoir la RBS (SharePoint Server 2010)][3].

Le reste de cette vue d’ensemble décrit brièvement le rôle de la carte de StorSimple pour SharePoint et les limites de capacité et les performances de SharePoint que vous devez connaître avant d’installer et de configurer la carte. Après avoir examiné ces informations, passez à la [Carte de StorSimple pour l’installation de SharePoint](#storsimple-adapter-for-sharepoint-installation) pour configurer la carte.

### <a name="storsimple-adapter-for-sharepoint-benefits"></a>Carte de StorSimple des avantages de SharePoint

Dans un site SharePoint, le contenu est stocké en tant que données BLOB non structurées dans une ou plusieurs bases de données de contenu. Par défaut, ces bases de données sont hébergées sur des ordinateurs qui exécutent SQL Server et qui se trouvent dans la batterie de serveurs SharePoint. BLOB peut augmenter rapidement la taille, utilisent une grande capacité de stockage de locaux. Pour cette raison, vous pouvez souhaiter trouver des solutions de stockage moins coûteux et un autre. SQL Server fournit une technologie appelée à distance Blob stockage (RBS) qui vous permet de stocker le contenu de l’objet BLOB dans le système de fichiers, à l’extérieur de la base de données SQL Server. Avec RBS, BLOB peut résider dans le système de fichiers sur l’ordinateur qui exécute SQL Server, ou qu’elles puissent être stockées dans le système de fichiers sur un autre serveur.

RBS nécessite que vous utilisez un fournisseur, par exemple, la carte de StorSimple pour SharePoint, pour permettre la RBS dans SharePoint. La carte StorSimple pour SharePoint fonctionne avec RBS, ce qui vous permet de déplacer des objets BLOB vers un serveur sauvegardé par le système Microsoft Azure StorSimple. Microsoft Azure StorSimple stocke les données BLOB localement ou dans le nuage, basée sur l’utilisation. Les objets BLOB qui est très actif (généralement appelé niveau 1 ou de données à chaud) réside localement. Données les moins actives et des données archivées résident dans le nuage. Après avoir activé la RBS sur une base de données de contenu, tout nouveau contenu BLOB créé dans SharePoint est stocké sur le périphérique StorSimple et non dans la base de données de contenu.

La mise en oeuvre de Microsoft Azure StorSimple de RBS offre les avantages suivants :

- En déplaçant le contenu BLOB sur un serveur distinct, vous pouvez réduire la charge de requête sur SQL Server, ce qui peut améliorer les temps de réponse de SQL Server. 

- StorSimple Azure utilise la déduplication et la compression pour réduire la taille des données.

- StorSimple Azure offre une protection de données dans le formulaire local et le nuage instantanés. En outre, si vous placez la base de données sur le périphérique StorSimple, vous pouvez sauvegarder le contenu de la base de données et les objets BLOB entre eux de façon cohérence après sinistre. (Déplacement de la base de données de contenu sur le périphérique est uniquement pris en charge pour le périphérique de la gamme 8000 de StorSimple. Cette fonctionnalité n'est pas prise en charge de la série 5000 ou 7000.)

- StorSimple Azure inclut des fonctions de récupération d’urgence, y compris le basculement, de récupération de fichier et le volume (y compris le test de restauration) et une restauration rapide des données.

- Vous pouvez utiliser le logiciel de récupération de données, tels que Kroll Ontrack PowerControls, avec les snapshots StorSimple des données BLOB pour effectuer une restauration au niveau des éléments de contenu de SharePoint. (Ce logiciel de récupération de données est un achat séparé).

- La carte StorSimple pour SharePoint se branche sur le portail de l’Administration centrale de SharePoint, ce qui vous permet de gérer votre solution SharePoint entière à partir d’un emplacement central.

Déplacement du contenu de l’objet BLOB dans le système de fichiers peut fournir les autres économies de coûts et les avantages. Par exemple, RBS peut réduire le besoin de stockage de niveau 1 coûteux et, dans la mesure où elle réduit la base de données de contenu, RBS peut réduire le nombre de bases de données requis dans la batterie de serveurs SharePoint. Toutefois, d’autres facteurs, tels que les limites de taille de base de données et la quantité de contenu de non-RBS, peuvent également affecter les besoins en stockage. Pour plus d’informations sur les coûts et les avantages de l’utilisation de RBS, voir [planification de RBS (SharePoint Foundation 2010)] [ 4] et le [choix entre utiliser la RBS dans SharePoint 2013][5].

### <a name="capacity-and-performance-limits"></a>Limites de capacité et de performances

Avant d’envisager l’utilisation de RBS de votre solution SharePoint, vous devez être conscient des limites de capacité de SharePoint Server 2010 et SharePoint Server 2013 et la relation entre ces limites et les performances acceptables et testés de performances. Pour plus d’informations, consultez [Software frontières et limites de SharePoint 2013](https://technet.microsoft.com/library/cc262787.aspx).

Passez en revue les éléments suivants avant de configurer RBS :

- Assurez-vous que la taille totale du contenu (la taille d’une base de données de contenu) ainsi que la taille d’un BLOB externalisées associé ne dépasse pas la limite de taille RBS pris en charge par SharePoint. Cette limite est de 200 Go. 

    **Contenu de base de données de mesure et de la taille BLOB**

     1. Exécutez cette requête sur les WFE d’Administration centrale. Démarrer l’interface de gestion SharePoint, puis entrez la commande Windows PowerShell suivante pour obtenir la taille des bases de données de contenu :

        `Get-SPContentDatabase | Select-Object -ExpandProperty DiskSizeRequired`

         Cette étape Obtient la taille de la base de données de contenu sur le disque.

     2. Exécutez une des requêtes SQL suivantes dans SQL Management Studio sur la zone du serveur SQL sur chaque base de données de contenu et ajouter le résultat à l’étape obtenu numéro 1.

        Dans les bases de données de contenu SharePoint 2013, entrez :

        `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[DocStreams] WHERE [Content] IS NULL`

        Dans les bases de données de contenu SharePoint 2010, entrez :

        `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[AllDocs] WHERE [Content] IS NULL`

        Cette étape Obtient la taille des BLOBs qui ont été externalisé.

- Nous vous conseillons de stocker les contenus BLOB et base de données localement sur le périphérique StorSimple. Le périphérique StorSimple est un cluster à deux nœuds pour une haute disponibilité. Placer les bases de données et les objets BLOB sur le périphérique StorSimple fournit une haute disponibilité.

    Traditionnel SQL Server migration conseillées permet de déplacer la base de données de contenu sur le périphérique StorSimple. Déplacer la base de données uniquement après que tous les contenus BLOB à partir de la base de données a été déplacé vers le partage de fichier via RBS. Si vous choisissez de déplacer la base de données de contenu sur le périphérique StorSimple, nous vous recommandons de configurer le stockage de base de données de contenu sur le périphérique sous la forme d’un volume principal.

- Dans Microsoft Azure StorSimple, si vous utilisez des volumes de plusieurs niveaux, il n’y a aucun moyen de garantir que le contenu stocké localement sur le périphérique ne sera pas en cascade pour un stockage en nuage Microsoft Azure de StorSimple. Par conséquent, nous recommandons l’utilisation de la StorSimple localement épinglé volumes conjointement avec SharePoint RBS. Cela garantira que tous les contenus BLOB conservé localement sur le périphérique StorSimple et ne sont pas déplacée vers Microsoft Azure.

- Si vous ne stockez pas les bases de données de contenu sur le périphérique StorSimple, utilisez SQL Server haute disponibilité meilleures pratiques traditionnelles qui prennent en charge de la RBS. La mise en cluster de SQL Server prend en charge la RBS, SQL Server lors de la mise en miroir ne le fait pas. 

>[AZURE.WARNING] Si vous n’avez pas activé RBS, nous ne recommandons pas le déplacement de la base de données de contenu sur le périphérique StorSimple. Il s’agit d’une configuration non testée.
 
## <a name="storsimple-adapter-for-sharepoint-installation"></a>Carte de StorSimple pour l’installation de SharePoint

Avant d’installer la carte de StorSimple pour SharePoint, vous devez configurer le périphérique StorSimple et assurez-vous que la batterie de serveurs SharePoint et l’instanciation de SQL Server répondent à toutes les conditions préalables. Ce didacticiel décrit la configuration requise, ainsi que des procédures d’installation et de mise à niveau de la carte de StorSimple pour SharePoint. 

## <a name="configure-prerequisites"></a>Configuration des composants requis

Avant d’installer la carte de StorSimple pour SharePoint, assurez-vous que le périphérique de StorSimple, batterie de serveurs SharePoint et lors de l’instanciation de SQL Server les connaissances préalables suivantes.

### <a name="system-requirements"></a>Configuration système requise

La carte StorSimple pour SharePoint fonctionne avec les matériels et logiciels suivants :

- Prise en charge du système d’exploitation Windows Server 2008 R2 SP1, Windows Server 2012 et Windows Server 2012 R2 

- Versions de SharePoint prises en charge : SharePoint Server 2010 ou SharePoint Server 2013

- Versions de SQL Server prises en charge – Édition entreprise de SQL Server 2008, SQL Server 2008 R2, Enterprise Edition ou Enterprise Edition de SQL Server 2012

- Prise en charge des périphériques de StorSimple – série 8000 de StorSimple, série 7000 de StorSimple ou StorSimple 5000 series.

### <a name="storsimple-device-configuration-prerequisites"></a>Conditions requises configuration de périphérique StorSimple

Le StorSimple de périphérique est un périphérique de bloc et doit donc utiliser un serveur de fichiers sur lequel les données peuvent être hébergées. Nous vous conseillons d’utiliser un serveur distinct plutôt que sur un serveur existant à partir de la batterie de serveurs SharePoint. Ce serveur de fichiers doit être sur le même réseau local (LAN) que l’ordinateur SQL Server qui héberge les bases de données de contenu. 

>[AZURE.TIP] 
>
>- Si vous configurez votre batterie de serveurs SharePoint pour une haute disponibilité, vous devez déployer le serveur de fichiers pour une haute disponibilité également.
>
>- Si vous ne stockez pas la base de données de contenu sur le périphérique StorSimple, suivez les recommandations traditionnelle de haute disponibilité qui prennent en charge de la RBS. La mise en cluster de SQL Server prend en charge la RBS, SQL Server lors de la mise en miroir ne le fait pas. 

Vérifiez que votre périphérique de StorSimple est correctement configuré et que les volumes appropriés pour prendre en charge votre déploiement de SharePoint sont configurés et accessibles à partir de votre ordinateur SQL Server. Accédez à [déployer votre périphérique de StorSimple sur site](storsimple-deployment-walkthrough.md) si vous n’avez pas encore déployé et configuré votre périphérique StorSimple. Notez l’adresse IP du périphérique StorSimple ; vous en aurez besoin lors de la carte de StorSimple pour l’installation de SharePoint. 

En outre, assurez-vous que le volume à utiliser pour l’externalisation de BLOB remplit les conditions suivantes :

- Le volume doit être formaté avec une taille d’unité d’allocation 64 Ko.

- Votre site web frontal (WFE) et les serveurs d’applications doivent être en mesure d’accéder au volume via un chemin d’accès UNC Universal Naming Convention (). 

- La batterie de serveurs SharePoint doit être configurée pour écrire sur le volume.

>[AZURE.NOTE] Après l’installation et la configuration de la carte, tous les externalisation de BLOB doit passer par le périphérique StorSimple (le périphérique présente les volumes à SQL Server et gérer les différents niveaux de stockage). Vous ne pouvez pas utiliser d’autres cibles pour l’externalisation de BLOB.
 
Si vous prévoyez d’utiliser des StorSimple Gestionnaire de snapshots pour prendre des clichés des données BLOB et base de données, veillez à installer Gestionnaire de snapshots StorSimple sur le serveur de base de données de sorte qu’il peut utiliser le Service du rédacteur SQL pour implémenter Windows Volume Shadow Copy Service (VSS). 

>[AZURE.IMPORTANT] Gestionnaire de snapshots StorSimple ne prend pas en charge l’enregistreur VSS de SharePoint et ne peut pas prendre des snapshots cohérents avec les applications de données SharePoint. Dans un scénario de SharePoint, Gestionnaire de snapshots StorSimple fournit uniquement des sauvegardes cohérentes à l’incident. 
 
## <a name="sharepoint-farm-configuration-prerequisites"></a>Configuration préalable de la batterie de serveurs SharePoint

Assurez-vous que votre batterie de serveurs SharePoint est configuré correctement, comme suit :

- Vérifiez que votre batterie de serveurs SharePoint est dans un état sain et vérifiez les points suivants : 

- Tous SharePoint WFE et enregistrés dans la batterie de serveurs d’applications sont en cours d’exécution et peuvent être exécutée à partir du serveur sur lequel vous allez installer la carte de StorSimple pour SharePoint sur.

- Le service de minuteur SharePoint (SPTimerV3 ou SPTimerV4) est en cours d’exécution sur chaque serveur WFE et le serveur d’applications.

- Le service de minuteur SharePoint et le pool d’applications IIS sous lequel le site d’Administration centrale de SharePoint s’exécute disposent de privilèges administratifs. 

- Assurez-vous que Internet Explorer Enhanced contexte de sécurité (IE ESC) est désactivé. Suivez ces étapes pour désactiver renforcée d’Internet Explorer :

    1. Fermez toutes les instances d’Internet Explorer.

    2. Démarrez le Gestionnaire de serveur.

    3. Dans le volet gauche, cliquez sur **Le serveur Local**.

    4. Dans le volet de droit, en regard de **La Configuration de sécurité renforcée d’Internet Explorer**, cliquez **sur**.

    5. **Les administrateurs**, cliquez sur **désactiver**.

    6. Cliquez sur **OK**.

## <a name="remote-blob-storage-rbs-prerequisites"></a>Conditions de stockage des objets BLOB (RBS) à distance

Assurez-vous que vous utilisez une version prise en charge de SQL Server. Seules les versions suivantes sont prises en charge et en mesure d’utiliser la RBS :

- SQL Server 2008 Enterprise Edition

- SQL Server 2008 R2, Enterprise Edition

- SQL Server 2012 Enterprise Edition

BLOB peut être externalisé sur ces volumes que le périphérique StorSimple présente à SQL Server. Aucune autre cible pour l’externalisation de BLOB n’est pris en charge.

Lorsque vous avez terminé toutes les étapes de configuration du logiciel requis, accédez à [installation de la carte de StorSimple pour SharePoint](#install-the-storsimple-adapter-for-sharepoint).

## <a name="install-the-storsimple-adapter-for-sharepoint"></a>Installer la carte de StorSimple pour SharePoint

Utilisez les étapes suivantes pour installer la carte de StorSimple pour SharePoint. Si vous réinstallez le logiciel, reportez-vous à la section [de mise à niveau ou la réinstallation de la carte de StorSimple pour SharePoint](#upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint). Le temps nécessaire à l’installation varie selon le nombre total de bases de données SharePoint dans votre batterie de serveurs SharePoint.

[AZURE.INCLUDE [storsimple-install-sharepoint-adapter](../../includes/storsimple-install-sharepoint-adapter.md)]


## <a name="configure-rbs"></a>Configuration de RBS

Après avoir installé la carte StorSimple pour SharePoint, configurer la RBS tel que décrit dans la procédure suivante.

>[AZURE.TIP] La carte StorSimple pour SharePoint s’intègre dans la page Administration centrale de SharePoint, autorisant la RBS être activée ou désactivée sur chaque base de données contenu dans la batterie de serveurs SharePoint. Toutefois, l’activation ou la désactivation la RBS sur la base de données de contenu, une réinitialisation d’IIS, ce qui, en fonction de la configuration de votre batterie de serveurs, peut perturber momentanément la disponibilité de SharePoint web front-end (WFE). (Les facteurs tels que l’utilisation d’un équilibreur de charge du front-end, la charge du serveur en cours, etc., peuvent limiter ou supprimer cette perturbation.) Pour protéger les utilisateurs contre une interruption, nous vous recommandons d’activer ou de désactiver RBS que pendant une fenêtre de maintenance planifiée.

[AZURE.INCLUDE [storsimple-sharepoint-adapter-configure-rbs](../../includes/storsimple-sharepoint-adapter-configure-rbs.md)]
 

## <a name="configure-garbage-collection"></a>Configurer le garbage collection

Lorsque des objets sont supprimés à partir d’un site SharePoint, ils ne sont pas automatiquement supprimés à partir du volume de la banque RBS. Au lieu de cela, un asynchrone, programme de maintenance en arrière-plan supprime les objets BLOB orphelins à partir de l’emplacement du fichier. Les administrateurs système peuvent planifier ce processus à exécuter régulièrement, ou ils peuvent démarrer chaque fois que nécessaire.

Ce programme de maintenance (Microsoft.Data.SqlRemoteBlobs.Maintainer.exe) est automatiquement installé sur tous les serveurs d’applications et de serveurs de SharePoint WFE lorsque vous activez la RBS. Le programme est installé à l’emplacement suivant : *lecteur d’amorçage*: \Program Files\Microsoft stockage Blob distant de SQL 10.50\Maintainer\

Pour plus d’informations sur la configuration et l’utilisation du programme de maintenance, consultez [Mettre à jour la RBS dans SharePoint Server 2013][8].

>[AZURE.IMPORTANT] Le programme d’intendant RBS consomme beaucoup de ressources. Vous devez planifier son exécution uniquement pendant les périodes d’activité du voyant de la batterie de serveurs SharePoint.

### <a name="delete-orphaned-blobs-immediately"></a>Supprimer immédiatement les objets BLOB orphelins

Si vous avez besoin de supprimer des objets BLOB orphelins immédiatement, vous pouvez utiliser les instructions suivantes. Notez que ces instructions sont un exemple de comment cela peut être fait dans un environnement SharePoint 2013 avec les composants suivants :

- Le nom de la base de données de contenu est WSS_Content.
- Le nom de SQL Server est SHRPT13-SQL12\SHRPT13.
- Le nom de l’application web est de SharePoint – 80.

[AZURE.INCLUDE [storsimple-sharepoint-adapter-garbage-collection](../../includes/storsimple-sharepoint-adapter-garbage-collection.md)]


## <a name="upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint"></a>Mise à niveau ou la réinstallation de la carte de StorSimple pour SharePoint

Utilisez la procédure suivante pour mettre à niveau SharePoint server puis réinstaller StorSimple carte pour SharePoint ou pour simplement mettre à niveau ou la réinstallation de la carte dans une batterie de serveurs SharePoint existante. 

>[AZURE.IMPORTANT] Passez en revue les informations suivantes avant de tenter de mettre à niveau votre logiciel SharePoint et/ou la mise à niveau ou la réinstallation de la carte de StorSimple pour SharePoint :
>
>- Tous les fichiers qui ont été précédemment déplacées vers le stockage externe via RBS ne sera pas disponibles jusqu'à ce que la réinstallation est terminée et que la fonctionnalité RBS est activée à nouveau. Pour limiter l’impact sur les utilisateurs, effectuez une mise à niveau ou la réinstallation pendant une fenêtre de maintenance planifiée.
>
>- Le temps nécessaire à la mise à niveau/réinstallation peut varier en fonction du nombre total de bases de données SharePoint dans la batterie de serveurs SharePoint.
>
>- Une fois la mise à niveau/réinstallation terminée, vous devez activer RBS pour les bases de données de contenu. Pour plus d’informations, reportez-vous à la section [Configurer la RBS](#configure-rbs) .
>
>- Si vous configurez RBS pour une batterie de serveurs SharePoint qui a un très grand nombre de bases de données (plus de 200), la page **Administration centrale de SharePoint** peut avoir expiré. Qui est le cas, actualisez la page. Cela n’affecte pas le processus de configuration.

[AZURE.INCLUDE [storsimple-upgrade-sharepoint-adapter](../../includes/storsimple-upgrade-sharepoint-adapter.md)]
 
## <a name="storsimple-adapter-for-sharepoint-removal"></a>Carte de StorSimple pour la suppression de SharePoint

Les procédures suivantes décrivent comment ramener les objets BLOB pour les bases de données de contenu SQL Server et puis désinstallez la carte de StorSimple pour SharePoint. 

>[AZURE.IMPORTANT] Vous devez replacer les objets BLOB pour les bases de données de contenu avant de désinstaller le logiciel de la carte. 

### <a name="before-you-begin"></a>Avant de commencer 

Collectez les informations suivantes avant de commencer le processus de suppression de carte et déplacez les données vers les bases de données de contenu SQL Server :

- Les noms de toutes les bases de données pour lesquels la RBS est activée
- Le chemin d’accès UNC du magasin BLOB configuré

### <a name="move-the-blobs-back-to-the-content-databases"></a>Replacez les objets BLOB pour les bases de données de contenu

Avant de désinstaller la carte de StorSimple pour le logiciel SharePoint, vous devez migrer tous les objets BLOB qui ont été externalisé vers les bases de données de contenu SQL Server. Si vous essayez de désinstaller la carte StorSimple pour SharePoint avant de vous déplacez tous les objets BLOB pour les bases de données de contenu, vous verrez le message suivant.

![Message d’avertissement](./media/storsimple-adapter-for-sharepoint/sasp1.png)
 
####<a name="to-move-the-blobs-back-to-the-content-databases"></a>Pour déplacer les objets BLOB vers les bases de données de contenu 

1. Chacun des objets externalisées télécharger.

2. Ouvrez la page **Administration centrale de SharePoint** et accédez aux **Paramètres système**. 

3. **StorSimple d’Azure**, cliquez sur **Configurer l’adaptateur de StorSimple**.

4. Dans la page **Configurer l’adaptateur de StorSimple** , cliquez sur le bouton **désactiver** ci-dessous, chaque base de données de contenu que vous souhaitez supprimer du stockage BLOB externe. 

5. Supprimez les objets de SharePoint et les télécharger à nouveau.

Vous pouvez également utiliser Microsoft` RBS Migrate()` l’applet de commande PowerShell inclus avec SharePoint. Pour plus d’informations, reportez-vous à la section [migration contenu dans ou en dehors de la RBS](https://technet.microsoft.com/library/ff628255.aspx).

Une fois que vous déplacez les objets BLOB vers la base de données de contenu, passez à l’étape suivante : [désinstaller la carte](#uninstall-the-adapter).

### <a name="uninstall-the-adapter"></a>Désinstallez la carte

Une fois que vous déplacez les objets BLOB vers les bases de données de contenu SQL Server, utilisez une des options suivantes pour désinstaller la carte StorSimple pour SharePoint.

#### <a name="to-use-the-installation-program-to-uninstall-the-adapter"></a>Pour utiliser le programme d’installation pour désinstaller la carte 

1. Utiliser un compte avec des privilèges d’administrateur pour ouvrir une session sur le serveur web frontal (WFE).
2. Double-cliquez sur la carte de StorSimple pour le programme d’installation de SharePoint. L’Assistant Installation démarre.

    ![L’Assistant Installation](./media/storsimple-adapter-for-sharepoint/sasp2.png)

3. Cliquez sur **suivant**. La page suivante s’affiche.

    ![Supprimer page de l’Assistant d’installation](./media/storsimple-adapter-for-sharepoint/sasp3.png)

4. Cliquez sur **Supprimer** pour sélectionner le processus de suppression. La page suivante s’affiche.

    ![Page de confirmation l’Assistant d’installation](./media/storsimple-adapter-for-sharepoint/sasp4.png)

5. Cliquez sur **Supprimer** pour confirmer la suppression. La progression apparaît.

    ![Page Progression de l’Assistant d’installation](./media/storsimple-adapter-for-sharepoint/sasp5.png)

6. Lorsque la suppression est terminée, la page de fin s’affiche. Cliquez sur **Terminer** pour fermer l’Assistant d’installation.

#### <a name="to-use-the-control-panel-to-uninstall-the-adapter"></a>Pour utiliser le panneau de configuration pour désinstaller la carte 

1. Ouvrez le panneau de configuration, puis cliquez sur **programmes et fonctionnalités**.

2. Sélectionnez **L’adaptateur StorSimple pour SharePoint**, puis cliquez sur **désinstaller**. 

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur StorSimple](storsimple-overview.md).

<!--Reference links-->
[1]: https://www.microsoft.com/download/details.aspx?id=44073
[2]: https://technet.microsoft.com/library/ff628583(v=office.15).aspx
[3]: https://technet.microsoft.com/library/ff628583(v=office.14).aspx
[4]: https://technet.microsoft.com/library/ff628569(v=office.14).aspx
[5]: https://technet.microsoft.com/library/ff628583(v=office.15).aspx
[8]: https://technet.microsoft.com/en-us/library/ff943565.aspx
