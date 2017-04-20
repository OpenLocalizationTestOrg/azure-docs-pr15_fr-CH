<properties 
    pageTitle="À l’aide du Gestionnaire de récupération pour résoudre les problèmes de mappage partagé | Microsoft Azure" 
    description="La classe RecoveryManager permet de résoudre les problèmes avec les cartes partagé" 
    services="sql-database" 
    documentationCenter=""  
    manager="jhubbard"
    authors="ddove"/>

<tags 
    ms.service="sql-database" 
    ms.workload="sql-database" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="05/05/2016" 
    ms.author="ddove"/>

# <a name="using-the-recoverymanager-class-to-fix-shard-map-problems"></a>À l’aide de la classe RecoveryManager pour résoudre les problèmes de mappage partagé

La classe [RecoveryManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.aspx) permet à des applications ADO.Net de facilement détecter et corriger les incohérences entre le plan global partagé (GSM) et le mappage local partagé (LSM) dans un environnement de base de données sharded. 

Le GSM et LSM suivre le mappage de chaque base de données dans un environnement sharded. Parfois, un saut se produit entre le GSM et le LSM. Dans ce cas, utilisez la classe RecoveryManager pour détecter et réparer le saut.

La classe RecoveryManager est la partie de la [bibliothèque de client de base de données élastique](sql-database-elastic-database-client-library.md). 


![Mappage de partagé][1]


Pour les définitions des termes, reportez-vous au [Glossaire des outils élastique de la base de données](sql-database-elastic-scale-glossary.md). Pour comprendre comment la **ShardMapManager** est utilisée pour gérer des données dans une solution sharded, reportez-vous à la section [Éclater mapper la gestion](sql-database-elastic-scale-shard-map-management.md).


## <a name="why-use-the-recovery-manager"></a>Pourquoi utiliser le Gestionnaire de récupération ?

Dans un environnement de base de données sharded, il existe un locataire par base de données et nombreuses bases de données par serveur. Il peut être également de nombreux serveurs dans l’environnement. Chaque base de données est mappé dans le mappage partagé, afin que les appels peuvent être acheminés vers le serveur et la base de données. Bases de données sont traitées en fonction d’une **clé d’ont**et chaque partagé est assignée une **plage de valeurs de clé**. Par exemple, une clé ont peut représenter les noms des clients à partir de « D » à « f ». Le mappage de tous les milieu des fragments (également appelés des bases de données) et de leurs plages de mappage sont contenus dans le **plan de global partagé (GSM)**. Chaque base de données contient également un mappage des plages contenue dans le partagé — il s’agit du **local partagé mapper (LSM)**. Lorsqu’une application se connecte à un partagé, le mappage est mis en cache avec l’application pour une récupération rapide. Le gestionnaire LSM est utilisé pour valider les données en mémoire cache. 

Le GSM et LSM peuvent devenir désynchronisé pour les raisons suivantes :

1. La suppression d’un partagé dont la plage est supposée ne peut plus être utilisé, ou renommer un partagé. Si vous supprimez un partagé dans un **mappage d’éclater orphelins**. Similary, une base de données renommée peut entraîner un mappage partagé orphelins. En fonction de l’intention du modifier, le partagé peut doivent être supprimés, ou l’emplacement partagé doit être mis à jour. Pour restaurer une base de données supprimé, consultez [restauration d’une base de données à un point antérieur dans le temps, de restaurer une base de données supprimé ou de récupérer d’une panne du centre de données](sql-database-troubleshoot-backup-and-restore.md).
2. Un événement géo-basculement se produit. Pour continuer, une doit mettre à jour le nom du serveur et le nom de la base de données du Gestionnaire de table partagé dans l’application, puis mettre à jour les détails de mappage partagé pour toutes milieu des fragments dans un mappage partagé. En cas de basculement géo, cette logique de récupération doit être automatisé dans le flux de travail de basculement. Automatisation des actions de récupération permet une gestion frictionless géo-activé les bases de données et permet d’éviter les actions humaines manuelles.
3. Un partagé ou la base de données ShardMapManager est restauré à un plus haut point de temps.

Pour plus d’informations sur les outils de la base de données élastique de base de données SQL Azure, géo-réplication et restauration, consultez les éléments suivants : 

* [Vue d’ensemble : Cloud business continuité d’activité et de la base de données de récupération après sinistre de la base de données SQL](sql-database-business-continuity.md) 
* [Mise en route avec les outils de base de données élastique](sql-database-elastic-scale-get-started.md)  
* [Gestion de ShardMap](sql-database-elastic-scale-shard-map-management.md)

## <a name="retrieving-recoverymanager-from-a-shardmapmanager"></a>RecoveryManager lors de la récupération à partir d’un ShardMapManager 

La première étape consiste à créer une instance de RecoveryManager. La [méthode de GetRecoveryManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getrecoverymanager.aspx) retourne le Gestionnaire de récupération pour l’instance actuelle de la [ShardMapManager](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx) . Pour résoudre les incohérences dans la table partagé, vous devez d’abord récupérer le RecoveryManager du mappage partagé particulier. 

    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnnectionString,  
             ShardMapManagerLoadPolicy.Lazy);
             RecoveryManager rm = smm.GetRecoveryManager(); 

Dans cet exemple, le RecoveryManager est initialisé à partir de la ShardMapManager. Le ShardMapManager contenant un ShardMap est également déjà initialisé. 

Dans la mesure où ce code application manipule le mappage partagé, les informations d’identification utilisées dans la méthode de fabrique (dans l’exemple ci-dessus, smmConnectionString) doivent être les informations d’identification qui ont des autorisations en lecture-écriture sur la base de données GSM référencé par la chaîne de connexion. Ces informations d’identification sont généralement différentes des informations d’identification utilisées pour ouvrir les connexions de routage dépendant des données. Pour plus d’informations, reportez-vous à la section [en utilisant les informations d’identification dans le client de base de données élastique](sql-database-elastic-scale-manage-credentials.md).

## <a name="removing-a-shard-from-the-shardmap-after-a-shard-is-deleted"></a>Suppression d’un partagé à partir de la ShardMap après la suppression d’un partagé

La [méthode de DetachShard](https://msdn.microsoft.com/library/azure/dn842083.aspx) détache l’éclater donné à partir de la table partagé et supprime les mappages associés à l’éclater.  

* Le paramètre d’emplacement est l’emplacement partagé, notamment le nom du serveur et nom de base de données, du éclater qui est détaché. 
* Le paramètre shardMapName est le nom de mappage partagé. Cette opération n’est nécessaire lorsque plusieurs mappages d’éclater sont gérés par le même gestionnaire de table partagé. Facultatif. 

**Important**: utilisez cette technique seulement si vous êtes sûr que la plage de la mise à jour du mappage est vide. Les méthodes ci-dessus ne vérifient pas les données pour la plage qui est déplacée, il est préférable d’inclure des vérifications dans votre code.

Cet exemple supprime le mappage partagé milieu des fragments. 

    rm.DetachShard(s.Location, customerMap); 

Le mappage de l’emplacement partagé dans le GSM avant la suppression de l’éclater. Car le partagé a été supprimé, il est supposé c’était volontaire, et la plage de clés ont n’est plus en cours d’utilisation. Si ce n’est pas le cas, vous pouvez exécuter la restauration du point dans le temps. pour récupérer le partagé à partir d’un version antérieur point-à-temps. (Dans ce cas, consultez la section ci-dessous pour détecter les incohérences partagé.) Pour récupérer, consultez [restauration d’une base de données à un point antérieur dans le temps, de restaurer une base de données supprimé ou de récupérer d’une panne du centre de données](sql-database-troubleshoot-backup-and-restore.md).

Dans la mesure où il est supposé que la suppression de la base de données est intentionnelle, l’action de nettoyage d’administration final est pour supprimer l’entrée de l’éclater dans le Gestionnaire de table partagé. Cela empêche l’application d’écrire par inadvertance des informations pour une plage qui n’est pas attendue.

## <a name="to-detect-mapping-differences"></a>Pour détecter les différences de mise en correspondance 

La [méthode de DetectMappingDifferences](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.detectmappingdifferences.aspx) sélectionne et renvoie un des mappages partagé (globales ou locales) comme source de vérité et rapproche les mappages sur les deux cartes partagé (GSM et LSM).

    rm.DetectMappingDifferences(location, shardMapName);

* L' *emplacement* Spécifie le nom du serveur et le nom de la base de données. 
* Le paramètre *shardMapName* est le nom de mappage partagé. Cela est uniquement requis si plusieurs mappages d’éclater sont gérés par le même gestionnaire de table partagé. Facultatif. 

## <a name="to-resolve-mapping-differences"></a>Pour résoudre les différences de mise en correspondance

La [méthode de ResolveMappingDifferences](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.resolvemappingdifferences.aspx) sélectionne l’une des cartes partagé (globales ou locales) comme source de vérité et rapproche les mappages sur les deux cartes partagé (GSM et LSM).

    ResolveMappingDifferences (RecoveryToken, MappingDifferenceResolution.KeepShardMapping);
   
* Le paramètre *RecoveryToken* énumère les différences dans les mappages entre le GSM et le LSM pour l’éclater spécifique. 

* L' [énumération de MappingDifferenceResolution](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.mappingdifferenceresolution.aspx) est utilisé pour indiquer la méthode de résolution de la différence entre les mappages partagé. 
* **MappingDifferenceResolution.KeepShardMapping** est recommandé dans le cas où le LSM qui contient les correspondances exactes et le mappage dans le partagé doit donc être utilisé. C’est généralement le cas dans le cas d’un basculement sur incident : le partagé se trouve maintenant sur un nouveau serveur. Dans la mesure où le partagé doit tout d’abord être supprimé dans le GSM (à l’aide de la méthode RecoveryManager.DetachShard), un mappage n’existe plus sur le GSM. Par conséquent, le le gestionnaire LSM doit être utilisé pour rétablir le mappage partagé.

## <a name="attach-a-shard-to-the-shardmap-after-a-shard-is-restored"></a>Joindre un partagé à la ShardMap après la restauration d’un partagé 

La [méthode de AttachShard](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.attachshard.aspx) joint l’éclater donné dans le mappage d’éclater. Il détecte des incohérences de mappage partagé et met à jour les mappages pour faire correspondre le partagé au moment de la restauration d’éclater. Il est supposé que la base de données est également renommée pour refléter le nom de la base de données d’origine (avant lorsque le partagé a été restaurée), depuis le point de restauration par défaut, une nouvelle base de données auquel est ajouté l’horodatage du temps. 

    rm.AttachShard(location, shardMapName) 

* Le paramètre *d’emplacement* est le nom de serveur et de base de données, du éclater qui est attaché. 

* Le paramètre *shardMapName* est le nom de mappage partagé. Cette opération n’est nécessaire lorsque plusieurs mappages d’éclater sont gérés par le même gestionnaire de table partagé. Facultatif. 

Cet exemple ajoute un partagé au mappage partagé qui a été récemment restauré à partir d’un plus haut point de temps. Dans la mesure où le partagé (à savoir, le mappage de l’éclater dans le module LSM) a été restaurée, il est potentiellement incompatible avec l’écriture partagé dans le GSM. En dehors de cet exemple de code, le partagé a été restauré et renommé au nom de la base de données d’origine. Dans la mesure où elle a été restaurée, il est supposé être le mappage dans le gestionnaire LSM le mappage d’un niveau de confiance. 

    rm.AttachShard(s.Location, customerMap); 
    var gs = rm.DetectMappingDifferences(s.Location); 
      foreach (RecoveryToken g in gs) 
       { 
       rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping); 
       } 

## <a name="updating-shard-locations-after-a-geo-failover-restore-of-the-shards"></a>Mise à jour des emplacements de partagé après un basculement geo (restauration) au milieu des fragments

En cas de basculement géo, la base de données secondaire est en écriture accessible et devient la nouvelle base de données principale. Le nom du serveur et éventuellement la base de données (selon votre configuration), peut être différent de l’image principale d’origine. Par conséquent, les entrées de mappage pour l’éclater GSM et LSM doivent être fixées. De même, si la base de données est restaurée dans un emplacement ou un autre nom ou à un point antérieur dans le temps, ceci peut entraîner des incohérences dans les mappages d’éclater. Le Gestionnaire de mappage partagé gère la distribution de connexions ouvertes à la base de données correcte. Distribution est basée sur les données de la table d’éclater et la valeur de la clé ont qui est la cible de la demande de l’application. Après un basculement géo, ces informations doivent être mis à jour avec le nom précis, le nom de la base de données et le mappage partagé de la base de données récupérée. 

## <a name="best-practices"></a>Meilleures pratiques

Geo-basculement et la restauration sont des opérations généralement gérées par un administrateur de nuage de l’application intentionnellement en utilisant une des fonctionnalités de continuité d’activité de bases de données SQL Azure. Planification de la continuité d’activité nécessite des processus, procédures et mesures pour s’assurer que les activités de l’entreprise peuvent continuer sans interruption. Les méthodes disponibles comme partie de la classe RecoveryManager doit être utilisé dans ce flux de travail afin de garantir que le GSM et LSM sont mis à jour en fonction de l’action de récupération. Il y a 5 étapes de base pour correctement s’assurer que le GSM et LSM reflètent les informations précises après un événement de basculement. Le code de l’application d’exécution de ces étapes peut être intégré dans le flux de travail et des outils existants. 

1. Récupérer le RecoveryManager de la ShardMapManager. 
2. Détachez l’ancien partagé à partir de la carte d’éclater.
3. Joindre le nouveau partagé au mappage partagé, y compris le nouvel emplacement partagé.
4. Détecter des incohérences dans le mappage entre le GSM et LSM. 
5. Résoudre les différences entre le GSM et le LSM, approuver le LSM. 

Cet exemple effectue les opérations suivantes :
1. Supprime le mappage partagé qui reflètent les emplacements partagé avant l’événement de basculement milieu des fragments.
2. Attache le milieu des fragments au mappage partagé qui reflète le nouvel emplacement partagé (le paramètre « Configuration.SecondaryServer » est le nouveau nom de serveur, mais le même nom de base de données).
3. Récupère les jetons de récupération en détectant les différences de mise en correspondance entre le GSM et le LSM pour chaque partagé. 
4. Résout les incohérences en approuvant le mappage de chaque éclater le LSM. 

    milieu des fragments var = smm. GetShards() ;  foreach (partagé s dans le milieu des fragments) {si (s.Location.Server == Configuration.PrimaryServer) {ShardLocation slNew = nouvelle ShardLocation (Configuration.SecondaryServer, s.Location.Database) ; 
        
          rm.DetachShard(s.Location); 
        
          rm.AttachShard(slNew); 
        
          var gs = rm.DetectMappingDifferences(slNew); 
    
          foreach (RecoveryToken g in gs) 
            { 
               rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping); 
            } 
        } 
    } 



[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]


<!--Image references-->
[1]: ./media/sql-database-elastic-database-recovery-manager/recovery-manager.png
 
