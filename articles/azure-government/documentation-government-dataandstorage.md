<properties
    pageTitle="Documentation de gouvernement Azure | Microsoft Azure"
    description="Cela fournit une comparaison des fonctionnalités et des conseils sur le développement d’applications pour Azure gouvernement"
    services="Azure-Government"
    cloud="gov" 
    documentationCenter=""
    authors="ryansoc"
    manager="zakramer"
    editor=""/>

<tags
    ms.service="multiple"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="azure-government"
    ms.date="09/30/2016"
    ms.author="ryansoc"/>


#  <a name="azure-government-data-and-storage"></a>Stockage et données de gouvernement azure

##  <a name="azure-storage"></a>Stockage Azure

Pour plus d’informations sur ce service et son utilisation, consultez la [documentation de publics de stockage Azure](https://azure.microsoft.com/documentation/services/storage/).

### <a name="variations"></a>Variations

Les URL pour les comptes de stockage Azure administrations sont différents :

Type de service|Public Azure|Gouvernement Azure
---|---|---
Stockage des objets BLOB|*. blob.core.windows.net|*. blob.core.usgovcloudapi.net
Stockage de la file d’attente|*. queue.core.windows.net|*. queue.core.usgovcloudapi.net
Stockage de table|*. table.core.windows.net| *. table.core.usgovcloudapi.net

>[AZURE.NOTE] Tous vos scripts et le code doit tenir compte des points de terminaison appropriés.  Voir [Configuration des chaînes de connexion de stockage Azure](../storage-configure-connection-string.md#creating-a-connection-string-to-the-explicit-storage-endpoint). 

Pour plus d’informations sur les API, consultez le <a href="https://msdn.microsoft.com/en-us/library/azure/mt616540.aspx">Constructeur de compte de stockage Cloud</a>.

Le suffixe de point de terminaison à utiliser dans ces surcharges est core.usgovcloudapi.net 

### <a name="considerations"></a>Considérations relatives à la

Les informations suivantes identifient la frontière Azure gouvernement pour stockage Azure :

| Données réglementé/contrôlé autorisé | Données réglementé/contrôlé non autorisé |
|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Les données entrées, stockées, traitées et au sein d’un système de stockage Azure produit peut contenir des données d’exportation contrôlée. Authentificateurs statiques, telles que les mots de passe et les broches de la carte à puce pour l’accès aux composants de la plateforme Azure. Clés privées des certificats utilisés pour gérer les composants de la plateforme Azure. Autres sécurité informations/secrets, tels que des certificats, des clés de chiffrement, des clés principales et les clés de stockage stockées dans des services Azure. | Les métadonnées de stockage Azure ne sont pas autorisée à contenir des données d’exportation contrôlée. Ces métadonnées incluent toutes les données de configuration saisies lors de la création et la gestion de votre produit de stockage.  N’entrez pas réglementés/contrôle des données dans les champs suivants : groupes de ressources, les noms de déploiement, les noms de ressources, les balises de ressources  

##  <a name="premium-storage"></a>Stockage de prime

Pour plus d’informations sur ce service et son utilisation, consultez [prime stockage : stockage hautes performances pour les charges de travail de Machine virtuelle Azure](../storage/storage-premium-storage.md).

###  <a name="variations"></a>Variations

Stockage de prime est généralement disponible dans le USGov Virginia. Cela inclut les ordinateurs virtuels de série DS. 

### <a name="considerations"></a>Considérations relatives à la

Les mêmes considérations de stockage données ci-dessus s’appliquent aux comptes de stockage de prime. 

##  <a name="sql-database"></a>Base de données SQL

Pour plus d’informations sur la configuration de visibilité des métadonnées et des meilleures pratiques de protection, voir le<a href="https://msdn.microsoft.com/en-us/library/bb510589.aspx"> Centre de sécurité Microsoft pour le moteur de base de données SQL</a> et de la [Documentation publique des base de données SQL Azure](https://azure.microsoft.com/documentation/services/sql-database/) .

### <a name="variations"></a>Variations

Base de données de SQL V12 sont généralement disponible dans Azure.

L’adresse des serveurs de SQL Azure dans Azure gouvernement est différente :

Type de service|Public Azure|Gouvernement Azure
---|---|---
Base de données SQL|*. database.windows.net|*. database.usgovcloudapi.net

### <a name="considerations"></a>Considérations relatives à la

Les informations suivantes identifient la frontière Azure gouvernement pour stockage Azure :

| Données réglementé/contrôlé autorisé | Données réglementé/contrôlé non autorisé |
|--------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Toutes les données stockées et traitées dans Microsoft Azure SQL peuvent contenir des données de gouvernement d’Azure réglementés. Vous devez utiliser les outils de base de données pour le transfert de données réglementé Azure gouvernement des données. | Les métadonnées SQL Azure ne sont pas autorisée à contenir des données d’exportation contrôlée. Ces métadonnées incluent toutes les données de configuration saisies lors de la création et la gestion de votre produit de stockage.  N’entrez pas réglementée/données dans les champs suivants : nom, nom d’abonnement, groupes de ressources, nom du serveur, connexion d’admin Server, noms de déploiement, les noms de ressources, les balises de ressources de la base de données

##  <a name="next-steps"></a>Étapes suivantes

Pour des informations supplémentaires et des mises à jour, vous abonner à la <a href="https://blogs.msdn.microsoft.com/azuregov/">Microsoft Azure gouvernement Blog.</a>
