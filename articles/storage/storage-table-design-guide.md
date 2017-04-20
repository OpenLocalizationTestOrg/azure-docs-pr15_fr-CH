<properties
    pageTitle="Guide de conception de Table stockage Azure | Microsoft Azure"
    description="Conception évolutive et performante des Tables dans le stockage par Table Azure"
    services="storage"
    documentationCenter="na"
    authors="jasonnewyork" 
    manager="tadb"
    editor="tysonn"/>

<tags
    ms.service="storage"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="storage"
    ms.date="09/22/2016"
    ms.author="jahogg"/>

# <a name="azure-storage-table-design-guide-designing-scalable-and-performant-tables"></a>Guide de conception de Table de stockage Azure : Conception évolutive et performante Tables

## <a name="overview"></a>Vue d’ensemble

Conception évolutive et tables performant vous devez prendre en compte un certain nombre de facteurs de performances, d’évolutivité et de coût. Si vous avez déjà conçu des schémas pour les bases de données relationnelles, ces considérations est familiers, mais s’il existe quelques similitudes entre le modèle de stockage Azure Table service et modèles relationnels, il existe également plusieurs différences importantes. Ces différences conduisent généralement à des conceptions très différentes qui peut sembler incorrecte à une personne connaissant des bases de données relationnelles ou intuitif, mais qui faire une bonne idée si vous concevez pour un stockage de clé/valeur NoSQL tels que le service de la Table d’Azure. La plupart des différences de votre conception reflète le fait que le service de la Table est conçu pour prendre en charge des applications à l’échelle de nuage qui peuvent contenir des milliards d’entités (lignes dans la terminologie de base de données relationnelle), de données ou pour des groupes de données doit prendre en charge des volumes de transactions très élevé : par conséquent, vous devez penser différemment à propos de la façon dont vous stockez vos données et comprendre comment fonctionne le service de la Table. Un magasin de données NoSQL bien conçu peut permettre à votre solution à l’échelle plus loin (et à un coût moindre) à une solution qui utilise une base de données relationnelle. Ce guide vous aide à ces rubriques.  

## <a name="about-the-azure-table-service"></a>Sur le service de la Table d’Azure

Cette section présente certaines des fonctionnalités clés de service de la Table qui se rapportent spécifiquement à la conception des performances et d’évolutivité. Si vous ne connaissez pas le stockage Azure et le service de la Table, lisez d’abord [Introduction au stockage Azure de Microsoft](storage-introduction.md) et la [mise en route avec le stockage par Table Azure à l’aide de .NET](storage-dotnet-how-to-use-tables.md) avant de lire le reste de cet article. Bien que l’objectif de ce guide soit sur le service de la Table, il inclut une discussion des file d’attente d’Azure et Blob de services et comment vous pouvez les utiliser avec le service de la Table dans une solution.  

Quel est le service de la Table ? Comme prévu à partir du nom, le service de la Table utilise un format sous forme de tableau pour stocker les données. Dans la terminologie standard, chaque ligne de la table représente une entité, et les colonnes de stockent les différentes propriétés de cette entité. Chaque entité possède une paire de clés pour identifier de manière unique, et une colonne timestamp que le service de la Table utilise pour suivre lorsque l’entité a été mis à jour (cela se produit automatiquement et l’horodatage Impossible de remplacer manuellement avec une valeur arbitraire). Le service de la Table utilise l’horodatage de dernière modification (LMT) pour gérer l’accès concurrentiel optimiste.  

>[AZURE.NOTE] Les opérations d’API REST de service Table retournent également une valeur **ETag** qui dérive de l’horodatage de dernière modification (LMT). Dans ce document nous utiliserons les termes ETag et LMT indifféremment, car elles référencent les mêmes données sous-jacentes.  

L’exemple suivant illustre une table simple pour stocker les entités employé et service. La plupart des exemples illustrées plus loin dans ce guide sont basées sur cette conception simple.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Horodatage</th>
<th></th>
</tr>
<tr>
<td>Marketing</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Jun</td>
<td>CAO</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>Département</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Marketing</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>Ventes</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


Jusqu’ici, il est très similaire à une table dans une base de données relationnelle avec les principales différences étant les colonnes obligatoires et la capacité de stocker plusieurs types d’entités dans la même table. En outre, chacune des propriétés définies par l’utilisateur tel que **FirstName** ou **âge** a un type de données, tels qu’integer ou string, simplement comme une colonne dans une base de données relationnelle. Bien que, contrairement à une base de données relationnelle, la nature sans schéma de service de la Table signifie qu’une propriété pas nécessairement les mêmes données tapez sur chaque entité. Pour stocker des types de données complexes dans une seule propriété, vous devez utiliser un format sérialisé comme JSON ou XML. Pour plus d’informations sur les types de données des services tels que la prise en charge de table, les plages de date pris en charge, les règles et les contraintes de taille, d’attribution de noms voir la [compréhension du modèle de données de Service de Table](http://msdn.microsoft.com/library/azure/dd179338.aspx).

Comme vous le verrez, votre choix de **PartitionKey** et de **RowKey** est fondamentale pour la conception d’une bonne table. Chaque entité stockée dans une table doit avoir une combinaison unique de **PartitionKey** et **RowKey**. Comme avec les clés dans une table de base de données relationnelle, les valeurs **PartitionKey** et **RowKey** sont indexés pour créer un index ordonné en clusters qui permet la recherche rapide ; Toutefois, le service de la Table ne crée pas d’index secondaire afin que ce sont les deux seules propriétés indexées (certains des modèles décrits afficher ultérieurement comment vous pouvez contourner cette limitation apparente).  

Un tableau est constitué d’une ou plusieurs partitions, et comme vous le verrez, pour la plupart, les décisions de conception que vous apportez seront autour des choix d’un **PartitionKey** et un **RowKey** approprié pour optimiser votre solution. Une solution peut consister simplement une table qui contient toutes vos entités organisées en partitions, mais en général une solution possède plusieurs tables. Tables vous aident à organiser vos entités logiquement, de vous aider à gérer l’accès aux données à l’aide de listes de contrôle d’accès, et vous pouvez supprimer une table entière à l’aide d’une opération de stockage unique.  

### <a name="table-partitions"></a>Partitions de table  
Le nom de compte, le nom de la table et le **PartitionKey** associés identifient la partition du service de stockage dans lequel le service de la table stocke l’entité. En que partie le schéma d’adressage pour les entités, les partitions définissant une portée pour les transactions (voir les [Transactions de l’entité groupe](#entity-group-transactions) ci-dessous) et constituent la base de la manière dont le service de table met à l’échelle. Pour plus d’informations sur les partitions, consultez [cibles de performances et de l’évolutivité du stockage Azure](storage-scalability-targets.md).  

Dans le service de la Table, un nœud services ou plus complète des partitions et les échelles de service par l’équilibrage de charge dynamique des partitions sur les nœuds. Si un nœud est sous charge, le service de la table peut *Fractionner* la plage de partitions traitées par ce nœud sur différents nœuds ; Lorsque le trafic est prise, le service peut *Fusionner* les plages de la partition à partir de nœuds silencieux sur un seul nœud.  

Pour plus d’informations sur les détails internes du service Table et en particulier la façon dont le service gère les partitions, voir le livre [Microsoft Azure Storage : A hautement disponible Cloud Service de stockage grâce à la cohérence fort](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx).  

### <a name="entity-group-transactions"></a>Transactions de groupe de l’entité
Dans le service de la Table, Transactions de groupe de l’entité (EGTs) sont le mécanisme intégré uniquement pour effectuer des mises à jour atomiques entre plusieurs entités. EGTs sont également appelées *transactions par lot* dans la documentation. EGTs peut uniquement utiliser des entités stockées dans la même partition (partage la même clé de partition d’une table), à tout moment atomique comportement transactionnel entre plusieurs entités, que vous devez vous assurer que ces entités sont dans la même partition. Il s’agit souvent d’un motif de tenir plusieurs types d’entités dans la même table (et de la partition) et de ne pas utiliser plusieurs tables pour les différents types d’entité. Un seul EGT peut fonctionner sur au moins 100 entités.  Si vous envoyez plusieurs EGTs simultanées pour traitement, qu'il est important de vérifier que ces EGTs ne fonctionnent pas sur les entités qui sont communes à EGTs comme dans le cas contraire le traitement peut être retardée.

EGTs également introduire un compromis potentiel pour évaluer dans votre conception : l’utilisation de plusieurs partitions augmente l’évolutivité de votre application, car Azure a plus d’opportunités pour les requêtes sur les nœuds d’équilibrage de charge, mais cela peut limiter la capacité de votre application pour effectuer des transactions atomiques et cohérence renforcée de vos données. En outre, il existe les objectifs spécifiques de l’évolutivité au niveau d’une partition qui peut limiter le débit des transactions, vous pouvez vous attendre pour un seul nœud : pour plus d’informations sur les cibles de l’évolutivité pour les comptes de stockage Azure et le service de la table, consultez [l’évolutivité du stockage Azure et les objectifs de Performance](storage-scalability-targets.md). Les sections suivantes de ce guide traitent différents conception de stratégies pour vous aider à gérer les compromis de ce genre et discuter de la meilleure façon de choisir votre clé de partition basée sur les exigences spécifiques de votre application cliente.  

### <a name="capacity-considerations"></a>Considérations relatives à la capacité
Le tableau suivant présente certaines des valeurs de clé en tenir compte lorsque vous concevez une solution de service de Table :  

|Capacité totale d’un compte de stockage Azure|500 TO|
|------------------------------------------|------|
|Nombre de tables dans un compte de stockage Azure | Limité uniquement par la capacité du compte de stockage |
|Nombre de partitions d’une table | Limité uniquement par la capacité du compte de stockage |
|Nombre d’entités dans une partition | Limité uniquement par la capacité du compte de stockage|
|Taille d’une entité individuelle | Jusqu'à 1 Mo, avec un maximum de 255 propriétés (y compris les **PartitionKey**, **RowKey**et **Timestamp**) |
|Taille de la **PartitionKey** | Une chaîne jusqu'à 1 Ko |
| Taille de la **RowKey** | Une chaîne jusqu'à 1 Ko |
|Taille d’une Transaction d’entité groupe | Une transaction peut inclure plus de 100 entités et la charge utile doit être inférieure à 4 Mo. Un EGT peut uniquement mettre à jour une entité qu’une seule fois. |

Pour plus d’informations, consultez [Présentation du modèle de données de Service de Table](http://msdn.microsoft.com/library/azure/dd179338.aspx).  

### <a name="cost-considerations"></a>Considérations budgétaires  
Stockage de table est relativement peu coûteux, mais vous devez inclure des estimations de coût pour l’utilisation de la capacité et de la quantité de transactions dans le cadre de l’évaluation d’une solution qui utilise le service de la Table. Toutefois, dans de nombreux scénarios, stockant les données dénormalisées ou en double afin d’améliorer les performances ou l’évolutivité de votre solution est une approche valide à prendre. Pour plus d’informations sur la tarification, consultez [Tarification de stockage Azure](https://azure.microsoft.com/pricing/details/storage/).  

## <a name="guidelines-for-table-design"></a>Instructions pour la conception de la table  
Ces listes résument des instructions clés que vous gardez à l’esprit lorsque vous concevez vos tables et ce guide y remédier en plus en détail plus loin dans. Ces instructions sont très différentes des lignes directrices que vous suivez habituellement pour la conception de la base de données relationnelle.  

Conception de votre solution de service de Table pour être efficace *de lecture* :

-   ***Conception pour l’interrogation dans des applications comportant beaucoup de lecture.*** Lorsque vous créez vos tables, réfléchissez aux requêtes (en particulier le temps de latence plus sensibles) que vous allez exécuter avant de définir comment mettre à jour vos entités. Cela entraîne en général une solution efficace et performante.  
-   ***Spécifiez à la fois PartitionKey et RowKey dans vos requêtes.*** *Requêtes du point* de telles sont les requêtes de service de table plus efficaces.  
-   ***Envisagez de stocker des copies des entités.*** Stockage de table est bon marché, envisagez donc stocker la même entité plusieurs fois (avec des clés différentes) pour activer des requêtes plus efficaces.  
-   ***Envisagez la dénormalisation de vos données.*** Stockage de table est bon marché, envisagez donc dénormalisation de vos données. Par exemple, stocker les entités de synthèse afin que les requêtes pour les données d’agrégation uniquement d’accéder à une entité unique.  
-   ***Utilisez les valeurs de clé composées.*** Les clés seulement que vous avez sont **PartitionKey** et **RowKey**. Par exemple, utiliser des valeurs de clés composées pour activer les chemins d’accès de clés de substitution pour les entités.  
-   ***Utiliser la projection de requête.*** Vous pouvez réduire la quantité de données que vous transférez via le réseau à l’aide de requêtes pour sélectionner uniquement les champs dont vous avez besoin.  

Conception de votre solution de service de Table pour être efficace *écrire* :  

-   ***Ne créez pas de partitions à chaud.*** Choisissez les touches qui vous permettent de répartir vos demandes de plusieurs partitions à tout moment.  
-   ***Éviter les pointes de trafic.*** Lisser le trafic pendant une période raisonnable de temps et d’éviter des pics de trafic.
-   ***Ne créez pas nécessairement une table distincte pour chaque type d’entité.*** Lorsque vous avez besoin des transactions atomiques sur les types d’entité, vous pouvez stocker ces plusieurs types d’entités dans la même partition dans la même table.
-   ***Envisagez le débit maximal, que vous devez obtenir.*** Vous devez connaître les objectifs d’évolutivité pour le service de la Table et s’assurer que votre conception n’entraîne pas les dépasser.  

Lorsque vous lisez ce guide, vous verrez des exemples que tous ces principes de mettre en pratique.  

## <a name="design-for-querying"></a>Création de requête  
Les solutions de service de table peuvent être lue beaucoup, beaucoup d’écriture ou un mélange des deux. Cette section se concentre sur les points à garder à l’esprit lorsque vous concevez votre service de Table pour prendre en charge efficacement les opérations de lecture. En général, une conception que prend en charge opérations de lecture efficace est également efficace pour les opérations d’écriture. Toutefois, il existe des considérations supplémentaires à prendre en compte lors de la conception pour prendre en charge les opérations d’écriture, décrites dans la section suivante, [Création pour la modification des données](#design-for-data-modification).

Un bon point de départ pour la conception de votre solution de service de Table pour vous permettre de lire les données efficacement est de demander « quelles requêtes mon application devra exécuter pour extraire les données dont il a besoin de service de la Table ? »  

>[AZURE.NOTE] Avec le service de la Table, il est important obtenir la conception appropriée en amont car il est difficile et coûteux à modifier par la suite. Par exemple, dans une base de données relationnelle il est souvent possible de résoudre les problèmes de performances par le simple ajout d’index pour une base de données existante : il n’est pas une option avec le service de la Table.  

Cette section se concentre sur les problèmes clés que vous devez résoudre lors de la création des tables pour l’interrogation. Les rubriques de cette section sont les suivants :

- [L’impact de votre choix de PartitionKey et de RowKey sur les performances des requêtes](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
- [Choix d’un PartitionKey approprié](#choosing-an-appropriate-partitionkey)
- [Optimisation des requêtes pour le service de la Table](#optimizing-queries-for-the-table-service)
- [Tri des données dans le service de la Table](#sorting-data-in-the-table-service)

### <a name="how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance"></a>L’impact de votre choix de PartitionKey et de RowKey sur les performances des requêtes  

Les exemples suivants supposent que le service de la table stocke les entités employé avec la structure suivante (la plupart des exemples omettre la propriété **Timestamp** pour plus de clarté) :  

|*Nom de la colonne* |*Type de données*|
|--------------|-----------|
|**PartitionKey** (Nom du service)|Chaîne|
|**RowKey** (Id employé)|Chaîne|
|**Prénom**|Chaîne|
|**Nom**|Chaîne|
|**Durée de vie**|Nombre entier|
|**EmailAddress**|Chaîne|

La section précédente, [vue d’ensemble du service de Table d’Azure](#overview) décrit quelques-unes des fonctionnalités clés du service Table de Azure qui ont une influence directe création de requête. Ces doublons résultent dans les instructions générales suivantes pour concevoir des requêtes de service de Table. Notez que la syntaxe de filtre utilisée dans les exemples ci-dessous est le service de Table API REST, pour plus d’informations consultez les [Entités de la requête](http://msdn.microsoft.com/library/azure/dd179421.aspx).  

-   Une ***Requête de Point*** est la recherche plus efficace d’utiliser et est recommandée à utiliser pour les recherches de gros volumes ou de recherches nécessitant une latence plus faible. Une telle requête peut utiliser l’index pour localiser une entité individuelle de très efficacement en spécifiant des valeurs de **PartitionKey** et de **RowKey** . Par exemple : $filter = (PartitionKey eq 'Sales') et (RowKey eq '2')  
-   Deuxième meilleure est une ***Requête de plage*** qui utilise le **PartitionKey** et les filtres sur une plage de valeurs **RowKey** pour renvoyer plus d’une entité. La valeur **PartitionKey** identifie une partition spécifique, et les valeurs de **RowKey** identifient un sous-ensemble des entités dans cette partition. Par exemple : $filter = ' t PartitionKey eq 'Vente et de RowKey ge' et RowKey lt'  
-   Meilleur tiers est une ***Analyse de la Partition*** qui utilise le **PartitionKey** et les filtres sur une autre propriété ne correspondant pas à une clé et qui peut retourner plusieurs entités. La valeur **PartitionKey** identifie une partition spécifique, et les valeurs de propriété Sélectionnez pour un sous-ensemble d’entités de cette partition. Par exemple : $filter = PartitionKey eq 'Vente' et LastName eq 'Smith'  
-   Une ***Analyse de Table*** n’inclut pas le **PartitionKey** et se révèle très inefficace car il recherche toutes les partitions qui composent votre table à son tour pour toutes les entités correspondantes. Il effectue une analyse de table, indépendamment ou non votre filtre utilise le **RowKey**. Par exemple : $filter = LastName eq 'Jones'  
-   Les requêtes qui renvoient plusieurs entités les renvoient triés dans l’ordre **PartitionKey** et **RowKey** . Pour éviter d’avoir recours les entités dans le client, choisissez un **RowKey** qui définit l’ordre de tri courante.  

Notez que l’utilisation d’un «**ou**» pour spécifier un filtre basé sur les résultats des valeurs **RowKey** dans une analyse de la partition et n’est pas traité comme une requête de plage. Par conséquent, vous devez éviter les requêtes qui utilisent des filtres tels que : $filter = PartitionKey eq 'Sales' et (RowKey eq '121' ou RowKey eq '322')  

Pour obtenir des exemples de code côté client qui utilisent la bibliothèque de Client de stockage à exécuter des requêtes efficaces, voir :  

-   [L’exécution d’une requête de point à l’aide de la bibliothèque de Client de stockage](#executing-a-point-query-using-the-storage-client-library)
-   [Extraction de plusieurs entités à l’aide de LINQ](#retrieving-multiple-entities-using-linq)
-   [Projection de côté serveur](#server-side-projection)  

Pour obtenir des exemples de code côté client qui peut gérer plusieurs types d’entités dans la même table, voir :  

-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)  

### <a name="choosing-an-appropriate-partitionkey"></a>Choix d’un PartitionKey approprié  

Votre choix de **PartitionKey** devez peser le pour permet l’utilisation de EGTs (pour assurer la cohérence) par rapport à la nécessité de distribuer vos entités sur plusieurs partitions (pour garantir une solution évolutive).  

À une extrême, vous pouvez stocker toutes vos entités sur une seule partition, mais cela peut limiter l’évolutivité de votre solution et empêche le service de la table d’être en mesure d’équilibrer les demandes. À l’autre extrême, vous pouvez stocker une entité par partition, qui serait hautement évolutive et qui permet au service de la table pour les requêtes de l’équilibrage de charge, mais qui vous empêchent de l’utilisation de transactions de groupe d’entité.  

Une solution idéale, **PartitionKey** est un qui vous permet d’utiliser des requêtes efficaces et qui a des partitions suffisantes pour assurer que votre solution est évolutive. En règle générale, vous trouverez que vos entités aura une propriété appropriée qui distribue vos entités sur les partitions nécessaires.

>[AZURE.NOTE] Par exemple, dans un système qui stocke les informations relatives aux utilisateurs ou aux employés, nom d’utilisateur peut être un bon PartitionKey. Vous pouvez avoir plusieurs entités qui utilisent un nom d’utilisateur donné en tant que clé de partition. Chaque entité qui stocke les données relatives à un utilisateur est regroupée en une seule partition et donc ces entités sont accessibles via les transactions de l’entité groupe, tout en étant hautement évolutive.

Il existe des considérations supplémentaires de votre choix de **PartitionKey** qui se rapportent à vous sera comment insérer, mettre à jour et supprimer des entités : reportez-vous à la section [Création de modifications de données](#design-for-data-modification) ci-dessous.  

### <a name="optimizing-queries-for-the-table-service"></a>Optimisation des requêtes pour le service de la Table  

Le service de la Table indexe automatiquement les entités en utilisant les valeurs **PartitionKey** et **RowKey** dans un seul index ordonné en clusters, donc la raison pour laquelle vous pointez les requêtes sont le plus efficace d’utiliser. Cependant, il n’y a aucun index autre que celui de l’index ordonné en clusters sur les **PartitionKey** et les **RowKey**.

Nombreux modèles doivent répondre aux conditions requises pour activer la recherche d’entités en fonction de plusieurs critères. Par exemple, localisation des entités employee en fonction de l’e-mail, id employé, ou nom de famille. Les modèles suivants dans la section [Modèles de Design de Table](#table-design-patterns) résoudre ces types de spécification et décrivent les méthodes de travail autour du fait que le service de la Table ne fournit pas les index secondaires :  

-   [Modèle d’index secondaire Intra-partition](#intra-partition-secondary-index-pattern) - banque plusieurs copies de chaque entité à l’aide de **RowKey** de différentes valeurs (dans la même partition) pour activer les recherches rapides et efficaces et tri des commandes à l’aide de différentes valeurs de **RowKey** .  
-   [Modèle d’index secondaire de la partition entre](#inter-partition-secondary-index-pattern) - stocker plusieurs copies de chaque entité à l’aide de différentes valeurs de RowKey dans des partitions distinctes ou dans des tables distinctes pour activer rapide et recherches efficaces et ordres de tri secondaires à l’aide de différentes valeurs de **RowKey** .  
-   [Modèle des entités Index](#index-entities-pattern) - mettre à jour les entités des index pour permettre des recherches efficaces qui retournent les listes des entités.  

### <a name="sorting-data-in-the-table-service"></a>Tri des données dans le service de la Table  

Le service de la Table retourne des entités triées dans l’ordre croissant selon **PartitionKey** puis par **RowKey**. Ces clés sont des valeurs de chaîne et pour vous assurer que les valeurs numériques effectue correctement le tri, vous devez les convertir à une longueur fixe et les compléter avec des zéros. Par exemple, si la valeur de l’id employé que vous servir de la **RowKey** est une valeur entière, vous devez convertir employee id **123** à **00000123**.  

De nombreuses applications ont des exigences requises pour utiliser les données triées dans des ordres différents : tri, par exemple, les employés par nom ou par date de jonction. Les modèles suivants dans la section [Modèles de Design de Table](#table-design-patterns) expliquent comment d’autres ordres de tri pour vos entités :  

-   [Modèle d’index secondaire Intra-partition](#intra-partition-secondary-index-pattern) - stocker plusieurs copies de chaque entité à l’aide de différentes valeurs de RowKey (dans la même partition) pour activer rapide et recherches efficaces et ordres de tri secondaires à l’aide de différentes valeurs de RowKey.  
-   [Modèle d’index secondaire de la partition entre](#inter-partition-secondary-index-pattern) - banque plusieurs copies de chaque entité en utilisant différentes valeurs de RowKey dans des partitions distinctes dans des tables distinctes pour permettre des recherches rapides et efficaces et à d’autres ordres de tri à l’aide de différentes valeurs de RowKey.
-   [Modèle de journal tail](#log-tail-pattern) - entités d’extraire les *n* plus récemment ajoutés à une partition à l’aide d’une valeur **RowKey** qui trie en inverse de date et de temps de l’ordre.  

## <a name="design-for-data-modification"></a>Conception de modifications de données
Cette section se concentre sur les considérations de conception d’optimisation des insertions, des mises à jour et des suppressions. Dans certains cas, vous devez évaluer le compromis entre les conceptions qui optimisent pour interroger les conceptions optimiser pour la modification des données comme vous le feriez dans les conceptions de bases de données relationnelles (même si les techniques permettant de gérer les compromis de conception sont différents dans une base de données relationnelle). La section [Modèles de Design de Table](#table-design-patterns) décrit des modèles de conception détaillée pour le service de la Table et met en évidence certaines ces compromis. Dans la pratique, vous constaterez que de nombreuses conceptions optimisées pour l’interrogation des entités fonctionnent également bien pour la modification des entités.  

### <a name="optimizing-the-performance-of-insert-update-and-delete-operations"></a>Optimisation des performances d’insérer, mettre à jour et supprimer des opérations  

Pour mettre à jour ou de suppression d’une entité, vous devez être en mesure d’identifier en utilisant les valeurs **PartitionKey** et **RowKey** . À cet égard, votre choix de **PartitionKey** et de **RowKey** pour la modification des entités doit suivre des critères similaires à votre choix pour prendre en charge les requêtes de point, car vous souhaitez identifier des entités aussi efficacement que possible. Vous ne souhaitez pas utiliser une analyse de table ou de la partition inefficace pour localiser une entité afin de découvrir les valeurs **PartitionKey** et **RowKey** , que vous devez mettre à jour ou le supprimer.  

Les modèles suivants dans la section adresse de [Modèles de conception de Table](#table-design-patterns) l’optimisation de la performance ou votre insertion, mise à jour et supprimer des opérations :  

-   [Volume élevé supprimer modèle](#high-volume-delete-pattern) - activer la suppression d’un volume élevé d’entités en stockant toutes les entités pour suppression simultanée dans leur propre table distincte ; suppression de la table permet de supprimer les entités.  
-   [Modèle de données série](#data-series-pattern) - série complète des données de banque d’informations dans une entité unique pour réduire le nombre de demandes.  
-   [Modèle des entités large](#wide-entities-pattern) - plusieurs entités physiques permet de stocker des entités logiques avec plus de 252 propriétés.  
-   [Modèle des entités volumineuses](#large-entities-pattern) - stockage des objets blob utilisé pour stocker des valeurs de propriété de grande taille.  

### <a name="ensuring-consistency-in-your-stored-entities"></a>Assurer la cohérence de vos entités stockées  

L’autre facteur clé qui influencent vos choix de clés pour l’optimisation des modifications de données consiste à garantir la cohérence à l’aide de transactions atomiques. Vous ne pouvez utiliser qu’un EGT pour fonctionner sur des entités stockées dans la même partition.  

Gestion de la cohérence d’adresse les modèles suivants dans la section [Modèles de Design de Table](#table-design-patterns) :  

-   [Modèle d’index secondaire Intra-partition](#intra-partition-secondary-index-pattern) - banque plusieurs copies de chaque entité à l’aide de **RowKey** de différentes valeurs (dans la même partition) pour activer les recherches rapides et efficaces et tri des commandes à l’aide de différentes valeurs de **RowKey** .  
-   [Modèle d’index secondaire de la partition entre](#inter-partition-secondary-index-pattern) - stocker plusieurs copies de chaque entité à l’aide de différentes valeurs de RowKey dans des partitions distinctes ou dans des tables distinctes pour activer rapide et recherches efficaces et ordres de tri secondaires à l’aide de différentes valeurs de **RowKey** .  
-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern) - activer le comportement finalement cohérent entre les limites de la partition ou les limites du système de stockage à l’aide de files d’attente Azure.
-   [Modèle des entités Index](#index-entities-pattern) - mettre à jour les entités des index pour permettre des recherches efficaces qui retournent les listes des entités.  
-   [Modèle de dénormalisation](#denormalization-pattern) - combiner des données dans une seule entité à vous permettent de récupérer toutes les données dont vous avez besoin d’une requête de point unique.  
-   [Modèle de données série](#data-series-pattern) - série complète des données de banque d’informations dans une entité unique pour réduire le nombre de demandes.  

Pour plus d’informations sur les transactions de groupe d’entité, consultez la section [Entité groupe de Transactions](#entity-group-transactions).  

### <a name="ensuring-your-design-for-efficient-modifications-facilitates-efficient-queries"></a>Assurer votre conception des modifications efficaces facilite les requêtes efficaces  

Dans de nombreux cas, un modèle pour des résultats de requêtes efficaces dans des modifications efficaces, mais vous évaluez toujours si c’est le cas pour votre scénario spécifique. Certains des modèles dans la section [Modèles de Design de Table](#table-design-patterns) explicitement évaluent les compromis entre l’interrogation et la modification des entités, et vous devez toujours prendre en compte le nombre de chaque type d’opération.  

Les modèles suivants dans la section [Modèles de Design de Table](#table-design-patterns) adresse des compromis entre la conception de requêtes efficaces et de la conception pour une modification efficace des données :  

-   [Motif de clé composée](#compound-key-pattern) - utiliser des valeurs de composés **RowKey** pour activer un client de rechercher des données apparentées avec une requête de point unique.  
-   [Modèle de journal tail](#log-tail-pattern) - entités d’extraire les *n* plus récemment ajoutés à une partition à l’aide d’une valeur **RowKey** qui trie en inverse de date et de temps de l’ordre.  

## <a name="encrypting-table-data"></a>Le cryptage des données de la Table    
     
La bibliothèque cliente .NET Azure stockage prend en charge le cryptage des chaînes de propriétés d’entité pour l’insertion d’opérations et de remplacement. Les chaînes cryptées sont stockées sur le service en tant que propriétés binaires, et ils sont convertis en chaînes après le décryptage.    

Pour les tables, en plus de la stratégie de cryptage, les utilisateurs doivent spécifier les cryptage des propriétés. Pour ce faire, vous pouvez spécifier un attribut [EncryptProperty] (pour les entités POCO dérivées de la TableEntity) ou un programme de résolution de cryptage dans les options de requête. Un résolveur de cryptage est un délégué qui prend un clé de partition, la clé de ligne et le nom de la propriété et retourne une valeur booléenne qui indique si cette propriété doit être chiffrée. Pendant le chiffrement, la bibliothèque cliente utilise ces informations pour décider si une propriété doit être chiffrée pendant l’écriture sur le réseau. Le délégué offre également la possibilité de logique autour de la façon dont les propriétés sont cryptées. (Par exemple, si X, puis crypter de propriété A ; de crypter les propriétés A et B.) Notez qu’il n’est pas nécessaire de fournir cette information lors de la lecture ou de l’interrogation des entités.

Notez que la fusion n’est pas pris en charge. Dans la mesure où un sous-ensemble de propriétés est peut-être encryptée précédemment à l’aide d’une clé différente, simplement fusionner les nouvelles propriétés et les métadonnées de mise à jour entraîne des pertes de données. La fusion soit nécessite de passer des appels de service supplémentaire à lire de l’entité existante dans le service, ou à l’aide d’une nouvelle clé par propriété, qui ne conviennent pas pour des raisons de performances.     

Pour plus d’informations sur le cryptage des données de la table, consultez [cryptage côté Client et la chambre forte de clé Azure pour le stockage Azure Microsoft](storage-client-side-encryption.md).  

## <a name="modelling-relationships"></a>Modélisation des relations  

Création de modèles de domaine est une étape clée dans la conception de systèmes complexes. En général, vous utilisez le processus de modélisation pour identifier les entités et les relations entre eux comme un moyen de comprendre le domaine de l’entreprise et la conception de votre système. Cette section se concentre sur la manière dont vous pouvez traduire certains des types de relations courantes trouvées dans des modèles de domaine pour les conceptions pour le service de la Table. Le processus de mappage à partir d’un modèle de données logique pour un modèle NoSQL la base de données physique est très différent de celui utilisé lors de la conception d’une base de données relationnelle. Conception de bases de données relationnelles suppose généralement un processus de normalisation de données optimisé pour minimiser la redondance et une fonction de requête déclarative qui résume comment la mise en oeuvre de la procédure de la base de données fonctionne.  

### <a name="one-to-many-relationships"></a>Relations un-à-plusieurs  

Relations un-à-plusieurs entre des objets de domaine d’entreprise se produisent très fréquemment : par exemple, un service a de nombreux employés. Il existe plusieurs façons d’implémenter les relations un-à-plusieurs dans le service de tableau chaque avec les avantages et les inconvénients qui peuvent être pertinentes pour le scénario particulier.  

Prenons l’exemple d’une grande entreprise multinationale avec des dizaines de milliers de départements et entités employé où chaque département a de nombreux employés et chaque employé associé à un service spécifique. Une approche consiste à stocker les service distincts et entités employé tels que ceux-ci :  

![][1]

Cet exemple montre une relation un-à-plusieurs implicite entre les types en fonction de la valeur de **PartitionKey** . Chaque département peut avoir de nombreux employés.  

Cet exemple montre également une entité department et ses entités connexes des employés dans la même partition. Vous pouvez choisir d’utiliser les différentes partitions, les tables ou les comptes de stockage de même pour les différents types d’entité.  

Une autre approche consiste à dénormaliser de vos données et de stocker uniquement les entités employé avec les données du département dénormalisées comme indiqué dans l’exemple suivant. Dans ce scénario particulier, cette approche dénormalisée peut-être pas la meilleure si vous avez besoin pour être en mesure de modifier les détails d’un directeur de département, car, pour ce faire, vous devez mettre à jour tous les employés du département.  

![][2]

Pour plus d’informations, consultez le [modèle de la dénormalisation](#denormalization-pattern) plus loin dans ce guide.  

Le tableau suivant résume les avantages et les inconvénients de chacune des approches décrites ci-dessus pour le stockage des employés et des entités de service qui ont une relation un-à-plusieurs. Vous devez également envisager la fréquence à laquelle vous vous attendez à effectuer diverses opérations : il peut être acceptable d’avoir une conception qui inclut une opération coûteuse si cette opération ne se produit que rarement.  

<table>
<tr>
<th>Approche</th>
<th>Professionnels de l’informatique</th>
<th>Inconvénients</th>
</tr>
<tr>
<td>Types d’entités distinctes, même partition, même table</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service en une seule opération.</li>
<li>Vous pouvez utiliser un EGT pour maintenir la cohérence si vous avez besoin de modifier une entité department chaque fois que vous mise à jour/Insérer/supprimer une entité d’employé. Par exemple, si vous conservez un nombre d’employés par département pour chaque département.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer un employé et une entité department pour certaines activités du client.</li>
<li>Les opérations de stockage se produisent dans la même partition. Volumes de transactions élevé, cela peut entraîner une zone réactive.</li>
<li>Vous ne pouvez pas déplacer un employé à un nouveau service à l’aide d’un EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Types d’entités distincts, des partitions différentes tables ou comptes de stockage</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service ou une entité employee avec une seule opération.</li>
<li>Volumes de transactions élevé, cela peut aider à répartir la charge sur plusieurs partitions.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer un employé et une entité department pour certaines activités du client.</li>
<li>Vous ne pouvez pas utiliser EGTs pour maintenir la cohérence lorsque vous mise à jour/Insérer/supprimer un employé et mise à jour d’un service. Par exemple, la mise à jour d’un nombre d’employés dans une entité department.</li>
<li>Vous ne pouvez pas déplacer un employé à un nouveau service à l’aide d’un EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Dénormaliser en un type d’entité unique</td>
<td>
<ul>
<li>Vous pouvez récupérer toutes les informations dont vous avez besoin d’une seule demande.</li>
</ul>
</td>
<td>
<ul>
<li>Il peut être coûteux à maintenir la cohérence si vous devez mettre à jour les informations de département (Cela vous devrez mettre à jour tous les employés d’un service).</li>
</ul>
</td>
</tr>
</table>

Comment vous choisissez entre ces options et les avantages et les inconvénients sont plus importants, qui dépend de vos scénarios d’application spécifiques. Par exemple, la fréquence à laquelle modifier les entités service ; toutes les requêtes de vos employés ont-ils besoin d’informations supplémentaires départements ; Comment fermer vous sont dans les limites de l’évolutivité de vos partitions ou votre compte de stockage ?  

### <a name="one-to-one-relationships"></a>Relations un à un  

Modèles de domaine peuvent inclure les relations un à un entre les entités. Si vous avez besoin mettre en œuvre une relation un à un dans le service de la Table, vous devez également choisir comment lier les deux entités associées lorsque vous devez récupérer toutes les deux. Ce lien peut être implicite, basé sur une convention dans les valeurs de clé ou explicite en stockant un lien sous la forme de valeurs **PartitionKey** et **RowKey** dans chaque entité de son entité connexe. Pour une discussion sur si vous devez stocker les entités associées dans la même partition, consultez la section [un-à-plusieurs relations](#one-to-many-relationships).  

Notez qu’il existe également des considérations d’implémentation qui peuvent vous amener à mettre en œuvre les relations un à un dans le service de la Table :  

-   Gestion des entités volumineuses (pour plus d’informations, reportez-vous à la section [Grand modèle d’entités](#large-entities-pattern)).  
-   Mise en œuvre de contrôles d’accès (pour plus d’informations, voir [contrôle de l’accès avec des Signatures de l’accès partagé](#controlling-access-with-shared-access-signatures)).  

### <a name="join-in-the-client"></a>Joindre dans le client  

Bien qu’il existe des façons de modéliser des relations dans le service de la Table, vous que les deux raisons principales pour utiliser le service de la Table sont l’évolutivité et les performances ne devez pas oublier. Si vous trouvez que vous sont modélisation de nombreuses relations de compromettre les performances et l’évolutivité de votre solution, demandez-vous si elle est nécessaire générer toutes les relations de données dans votre conception de la table. Vous ne pourrez pas la conception de simplifier et d’améliorer l’évolutivité et les performances de votre solution si vous laissez votre application cliente pour effectuer des jointures nécessaires.  

Par exemple, si vous avez des tables de petite taille qui contiennent des données qui ne changent pas très souvent, vous pouvez récupérer ces données une fois et mettre en cache sur le client. Cela permet d’éviter les allers et retours répétées pour extraire les mêmes données. Dans les exemples que nous avons étudié dans ce guide, l’ensemble des départements dans une petite entreprise est susceptible de petites et peu rend un bon candidat pour l’application cliente peut télécharger en une seule fois les données et le cache comme l’aspect des données, modifier.  

### <a name="inheritance-relationships"></a>Relations d’héritage  

Si votre application cliente utilise un ensemble de classes qui font partie d’une relation d’héritage pour représenter des entités commerciales, vous pouvez facilement conserver ces entités dans le service de la Table. Par exemple, peuvent avoir l’ensemble suivant de classes définies dans votre application cliente où la **personne** est une classe abstraite.

![][3]

Vous pouvez conserver des instances de deux classes concrètes dans le service de la Table à l’aide d’une seule table de personne à l’aide d’entités dans ressemblant à ceci :  

![][4]

Pour plus d’informations sur l’utilisation de plusieurs types d’entités dans la même table dans le code client, consultez la section [utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types) plus loin dans ce guide. Il fournit des exemples de reconnaître le type d’entité dans le code client.  

## <a name="table-design-patterns"></a>Modèles de Design de table
Dans les sections précédentes, vous avez vu que certains détaillées des discussions à propos de la façon d’optimiser votre création de table pour les données d’entité lors de la récupération à l’aide de requêtes et d’insertion, de mise à jour et de suppression des données d’entité. Cette section décrit certains modèles appropriés pour une utilisation avec les solutions de service de Table. En outre, vous verrez comment vous pouvez pratiquement résoudre certains des problèmes et des compromis générées précédemment dans ce guide. Le diagramme suivant résume les relations entre les différents modèles :  

![][5]

Le mappage de modèle ci-dessus met en évidence des relations entre les modèles (bleus) et des anti-modèles (orange) décrits dans ce guide. Il existe bien sûr de nombreux autres modèles qui méritent votre attention. Par exemple, un des scénarios clés pour le Service de la Table est d’utiliser le [Modèle de vue matérialisée](https://msdn.microsoft.com/library/azure/dn589782.aspx) dans le modèle de [Répartition de responsabilité requête commandes (CQRS)](https://msdn.microsoft.com/library/azure/jj554200.aspx) .  

### <a name="intra-partition-secondary-index-pattern"></a>Modèle d’index secondaire intra-partition
Stocker plusieurs copies de chaque entité à l’aide de différentes valeurs de **RowKey** (dans la même partition) rapide et efficaces recherches et le tri secondaire des commandes à l’aide de différentes valeurs de **RowKey** . Mises à jour entre les copies peuvent être cohérente à l’aide du EGT.  

#### <a name="context-and-problem"></a>Contexte et problème
Le service de la Table indexe automatiquement les entités en utilisant les valeurs **PartitionKey** et **RowKey** . Cela permet à une application client de récupérer une entité efficacement à l’aide de ces valeurs. Par exemple, à l’aide de la structure du tableau ci-dessous, une application cliente peut utiliser une requête de point pour récupérer une entité d’employé individuel en utilisant le nom du service et l’id d’employé (les valeurs **PartitionKey** et **RowKey** ). Un client peut également récupérer des entités triées par id d’employé au sein de chaque département.

![][6]

Si vous voulez également être en mesure de trouver une entité employee en fonction de la valeur d’une autre propriété, telles que l’adresse de messagerie, vous devez utiliser une analyse de partition moins efficace pour trouver une correspondance. C’est parce que le service de la table ne fournit pas les index secondaires. En outre, il n’existe aucune option permettant de demander une liste d’employés triés dans un ordre différent de l’ordre de **RowKey** .  

#### <a name="solution"></a>Solution
Pour contourner l’absence d’index secondaires, vous pouvez stocker plusieurs copies de chaque entité avec chaque copie à l’aide d’une valeur différente de **RowKey** . Si vous stockez une entité avec les structures indiquées ci-dessous, vous pouvez récupérer des entités employee en fonction de l’id de courrier électronique adresse ou de l’employé. Les valeurs de préfixe de **RowKey**, « empid_ » et « email_ » vous permettent de vous interroger pour un employé ou une plage d’employés à l’aide d’une plage d’adresses e-mail ou des ID de l’employé.  

![][7]

Suivant deux critères (une recherche par code d’employé et un recherchant par adresse de courriel) spécifient toutes deux requêtes de point :  

-   $filter = (PartitionKey eq 'Sales') et (RowKey eq 'empid_000223')  
-   $filter = (PartitionKey eq 'Sales') et (RowKey eq'email_jonesj@contoso.com')  

Si vous recherchez une plage d’entités de l’employé, vous pouvez spécifier une plage triés dans l’ordre des id employé ou une plage triés dans l’ordre d’adresse e-mail en interrogeant des entités avec le préfixe approprié dans le **RowKey**.  

-   Pour rechercher tous les employés du département ventes avec un id d’employé dans l’utilisation de la plage 000100 à 000199 : $filter = (PartitionKey eq 'Sales') et (RowKey ge 'empid_000100') et (le RowKey 'empid_000199')  
-   Pour rechercher tous les employés du département ventes avec une adresse commençant par la lettre « a » utilisation : $filter = (PartitionKey eq 'Sales') et (RowKey ge 'email_a') et (RowKey lt 'email_b')  

 Notez que la syntaxe de filtre utilisée dans les exemples ci-dessus est du service Table API REST, pour plus d’informations consultez les [Entités de la requête](http://msdn.microsoft.com/library/azure/dd179421.aspx).  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Stockage de table est relativement bon marché à utiliser pour le coût de stockage des données en double ne doit pas être une préoccupation majeure. Toutefois, vous devez toujours évaluer le coût de la conception en fonction de vos besoins de stockage prévue et ajouter uniquement les entités en double pour les requêtes qui qu'exécute votre application client de prendre en charge.  
-   Dans la mesure où les entités les index secondaires sont stockées dans la même partition que les entités d’origine, vous devez vous assurer que vous ne dépassez pas les cibles de l’évolutivité d’une partition individuelle.  
-   Vous pouvez conserver vos entités en double cohérents entre eux à l’aide de EGTs pour mettre à jour les deux copies de l’entité de façon atomique. Cela implique que vous devez stocker toutes les copies d’une entité dans la même partition. Pour plus d’informations, consultez la section [Transactions de groupe à l’aide de l’entité](#entity-group-transactions).  
-   La valeur utilisée pour la **RowKey** doit être unique pour chaque entité. Envisagez d’utiliser des valeurs de clé composées.  
-   Remplissage des valeurs numériques dans les **RowKey** (par exemple, l’id employé 000223), permet de correctes de tri et de filtrage des limites inférieure et supérieure en fonction.  
-   Vous n’avez pas nécessairement besoin dupliquer toutes les propriétés de l’entité. Par exemple, si les requêtes qui recherche les entités à l’aide de l’e-mail d’adresses dans le **RowKey** devez jamais age de l’employé, ces entités peuvent avoir la structure suivante :

![][8]

-   Il est généralement préférable de stocker les données en double et vous assurer que vous pouvez récupérer toutes les données dont vous avez besoin d’une seule requête, que d’utiliser une requête pour localiser une entité et une autre pour rechercher les données requises.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque votre application cliente a besoin de récupérer des entités à l’aide d’une variété de clés différentes, lorsque votre client a besoin de récupérer des entités dans différents ordres de tri, et dans le cas où vous pouvez identifier chaque entité utilisant un certain nombre de valeurs uniques. Toutefois, vous devez être sûr de ne pas dépasser les limites de l’évolutivité de partition lorsque vous effectuez des recherches d’entité en utilisant les différentes valeurs de **RowKey** .  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Motif de la partition entre les index secondaires](#inter-partition-secondary-index-pattern)
-   [Motif de clé composée](#compound-key-pattern)
-   [Transactions de groupe de l’entité](#entity-group-transactions)
-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)

### <a name="inter-partition-secondary-index-pattern"></a>Motif de la partition entre les index secondaires
Stocker plusieurs copies de chaque entité à l’aide de différentes valeurs de **RowKey** dans des partitions distinctes ou dans des tables distinctes rapide et efficaces recherches et le tri secondaire des commandes à l’aide de différentes valeurs de **RowKey** .  

#### <a name="context-and-problem"></a>Contexte et problème
Le service de la Table indexe automatiquement les entités en utilisant les valeurs **PartitionKey** et **RowKey** . Cela permet à une application client de récupérer une entité efficacement à l’aide de ces valeurs. Par exemple, à l’aide de la structure du tableau ci-dessous, une application cliente peut utiliser une requête de point pour récupérer une entité d’employé individuel en utilisant le nom du service et l’id d’employé (les valeurs **PartitionKey** et **RowKey** ). Un client peut également récupérer des entités triées par id d’employé au sein de chaque département.  

![][9]

Si vous voulez également être en mesure de trouver une entité employee en fonction de la valeur d’une autre propriété, telles que l’adresse de messagerie, vous devez utiliser une analyse de partition moins efficace pour trouver une correspondance. C’est parce que le service de la table ne fournit pas les index secondaires. En outre, il n’existe aucune option permettant de demander une liste d’employés triés dans un ordre différent de l’ordre de **RowKey** .  

Vous comptez un volume élevé de transactions par rapport à ces entités et que vous souhaitez minimiser les risques du service Table la limitation de votre client.  

#### <a name="solution"></a>Solution  
Pour contourner l’absence d’index secondaires, vous pouvez stocker plusieurs copies de chaque entité avec chaque copie en utilisant différentes valeurs de **PartitionKey** et **RowKey** . Si vous stockez une entité avec les structures indiquées ci-dessous, vous pouvez récupérer des entités employee en fonction de l’id de courrier électronique adresse ou de l’employé. Les valeurs de préfixe de **PartitionKey**, « empid_ » et « email_ » permettent d’identifier l’index que vous souhaitez utiliser pour une requête.  

![][10]

Suivant deux critères (une recherche par code d’employé et un recherchant par adresse de courriel) spécifient toutes deux requêtes de point :  

-   $filter = (PartitionKey eq ' empid_Sales') et (RowKey eq '000223')
-   $filter = (PartitionKey eq ' email_Sales') et (RowKey eq'jonesj@contoso.com')  

Si vous recherchez une plage d’entités de l’employé, vous pouvez spécifier une plage triés dans l’ordre des id employé ou une plage triés dans l’ordre d’adresse e-mail en interrogeant des entités avec le préfixe approprié dans le **RowKey**.  

-   Pour rechercher tous les employés du département ventes avec un id d’employé dans la plage **000100** à **000199** triés dans Utilisation de commande id employé : $filter = (PartitionKey eq ' empid_Sales') et (RowKey ge '000100') et (le RowKey '000199')  
-   Pour rechercher tous les employés du département des ventes avec une adresse de messagerie qui commence par « a » triée dans Utilisation de commande d’adresse e-mail : $filter = (PartitionKey eq ' email_Sales') et (RowKey ge 'a') et (RowKey lt 'b')  

Notez que la syntaxe de filtre utilisée dans les exemples ci-dessus est du service Table API REST, pour plus d’informations consultez les [Entités de la requête](http://msdn.microsoft.com/library/azure/dd179421.aspx).  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  
Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Vous pouvez conserver vos entités en double finalement cohérents entre eux en utilisant le [modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern) pour gérer les entités de l’index primaire et secondaire.  
-   Stockage de table est relativement bon marché à utiliser pour le coût de stockage des données en double ne doit pas être une préoccupation majeure. Toutefois, vous devez toujours évaluer le coût de la conception en fonction de vos besoins de stockage prévue et ajouter uniquement les entités en double pour les requêtes qui qu'exécute votre application client de prendre en charge.  
-   La valeur utilisée pour la **RowKey** doit être unique pour chaque entité. Envisagez d’utiliser des valeurs de clé composées.  
-   Remplissage des valeurs numériques dans les **RowKey** (par exemple, l’id employé 000223), permet de correctes de tri et de filtrage des limites inférieure et supérieure en fonction.  
-   Vous n’avez pas nécessairement besoin dupliquer toutes les propriétés de l’entité. Par exemple, si les requêtes qui recherche les entités à l’aide de l’e-mail d’adresses dans le **RowKey** devez jamais age de l’employé, ces entités peuvent avoir la structure suivante :

    ![][11]

-   Il est généralement préférable de stocker les données en double et vous assurer que vous pouvez récupérer toutes les données dont vous avez besoin d’une seule requête que d’utiliser une requête pour localiser une entité à l’aide de l’index secondaire et un autre pour rechercher les données requises dans l’index primaire.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  
Utilisez ce modèle lorsque votre application cliente a besoin de récupérer des entités à l’aide d’une variété de clés différentes, lorsque votre client a besoin de récupérer des entités dans différents ordres de tri, et dans le cas où vous pouvez identifier chaque entité utilisant un certain nombre de valeurs uniques. Utilisez ce modèle lorsque vous souhaitez éviter de dépasser les limites de l’évolutivité de partition lorsque vous effectuez des recherches d’entité en utilisant les différentes valeurs de **RowKey** .  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés
Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern)  
-   [Modèle d’index secondaire intra-partition](#intra-partition-secondary-index-pattern)  
-   [Motif de clé composée](#compound-key-pattern)  
-   [Transactions de groupe de l’entité](#entity-group-transactions)  
-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)  

### <a name="eventually-consistent-transactions-pattern"></a>Modèle de transactions finalement cohérente  

Activer un comportement cohérent finalement sur des limites de partition ou les limites du système de stockage à l’aide de files d’attente Azure.  

#### <a name="context-and-problem"></a>Contexte et problème  

EGTs activer les transactions atomiques entre plusieurs entités qui partagent la même clé de partition. Pour des raisons d’évolutivité et de performances, vous pouvez décider de stocker des entités ayant des exigences de cohérence dans des partitions distinctes ou dans un système de stockage distinct : dans ce cas, vous ne pouvez pas utiliser EGTs pour préserver la cohérence. Par exemple, vous pouvez avoir un besoin de maintenir la cohérence éventuelle entre :  

-   Entités sont stockées dans deux partitions différentes dans la même table, dans des tables différentes, dans des comptes de stockage différents.  
-   Une entité stockées dans le service de la Table et un blob stockées dans le service d’objet Blob.  
-   Une entité est stockée dans la Table service et un fichier dans un système de fichiers.  
-   Un magasin d’entité dans le service de la Table indexée encore l’utilisation du service de recherche d’Azure.  

#### <a name="solution"></a>Solution  

À l’aide de files d’attente Azure, vous pouvez implémenter une solution qui assure la cohérence éventuelle entre deux ou plusieurs partitions ou les systèmes de stockage.
Pour illustrer cela, supposons que vous avez besoin pour pouvoir archiver les anciennes entités employé. Anciennes entités employé sont rarement interrogées et doivent être exclues de toutes les activités qui traitent des employés actuels. Pour implémenter cette exigence, vous stockez des employés actifs dans la table **en cours** et les anciens employés dans la table **d’archivage** . L’archivage d’un employé nécessite que vous supprimez l’entité à partir de la table **en cours** et ajouter l’entité à la table **d’archivage** , mais vous ne pouvez pas utiliser un EGT pour effectuer ces deux opérations. Pour éviter le risque de défaillance entraîne une entité s’affiche dans les deux ou aucun des deux tables, l’opération d’archivage doit être finalement cohérente. Le diagramme de séquence suivante présente les étapes de cette opération. Plus de détails est fourni pour les chemins d’accès de l’exception dans le texte qui suit.  

![][12]

Un client lance l’opération d’archivage en plaçant un message dans une file d’attente Azure, dans cet exemple pour archiver l’employé #456. Un rôle worker interroge la file d’attente pour les nouveaux messages ; Lorsqu’il en trouve une, il lit le message et laisse une copie cachée sur la file d’attente. Le rôle de travail suivant extrait une copie de l’entité à partir de la table **actuelle** , insère une copie de la table **d’Archive** et puis supprime l’original de la table **en cours** . Enfin, s’il n’y a aucuns erreurs dans les étapes précédentes, le rôle de travail supprime le message masqué à partir de la file d’attente.  

Dans cet exemple, étape 4 insère l’employé dans la table **d’archivage** . Il a pu ajouter l’employé à un objet blob dans le service d’objet Blob ou un fichier dans un système de fichiers.  

#### <a name="recovering-from-failures"></a>Récupération des défaillances  

Il est important que les opérations dans les étapes **4** et **5** doivent être *équipotentes* au cas où le rôle de collaborateur doit redémarrer l’opération d’archivage. Si vous utilisez le service de la Table, à l’étape **4** , vous devez utiliser une opération « Insérer ou remplacer » ; à l’étape **5** , vous devez utiliser un « supprimer si existe » opération dans la bibliothèque de client que vous utilisez. Si vous utilisez un autre système de stockage, vous devez utiliser une opération équipotente approprié.  

Si le rôle de travail ne termine jamais l' étape **6**, puis après un délai d’attente le message réapparaît dans la file d’attente est prêt pour le rôle de collaborateur essayer de retraiter. Le rôle de travail pouvez vérifier combien de fois un message sur la file d’attente a été lu et, si nécessaire, indicateur il s’agit d’un message « incohérent » pour l’enquête en l’envoyant à une file d’attente distincte. Pour plus d’informations sur la vérification et la lecture des messages de la file d’attente le nombre dequeue, consultez [Obtenir les Messages](https://msdn.microsoft.com/library/azure/dd179474.aspx).  

Certaines erreurs dans les services de Table et de la file d’attente sont des erreurs transitoires et votre application cliente doit inclure la logique de nouvelle tentative approprié pour les gérer.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Cette solution ne fournit pas d’isolation de la transaction. Par exemple, un client peut lire les tables **en cours** et **l’archivage** , lorsque le rôle de travail était entre les étapes **4** et **5**et voir une vue incohérente des données. Notez que les données seront cohérentes par la suite.  
-   Il se peut que vous devez vous assurer que les étapes 4 et 5 sont idempotents afin d’assurer la cohérence éventuelle.  
-   Vous pouvez adapter la solution à l’aide de plusieurs files d’attente et les instances de rôle de travail.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  
Utilisez ce modèle lorsque vous souhaitez garantir la cohérence éventuelle entre les entités qui existent dans des partitions ou des tables. Vous pouvez étendre ce modèle pour garantir la cohérence éventuelle pour les opérations sur le service de la Table et le service d’objet Blob et autres stockage Azure sources de données comme la base de données ou le système de fichiers.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  
Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  
-   [Transactions de groupe de l’entité](#entity-group-transactions)  
-   [Fusion ou remplacer](#merge-or-replace)  

>[AZURE.NOTE] Si l’isolement de la transaction est important pour votre solution, vous devez envisager de redéfinir vos tables pour vous permettre d’utiliser EGTs.  

### <a name="index-entities-pattern"></a>Modèle des entités index
Tenir à jour les entités des index pour permettre des recherches efficaces qui retournent les listes des entités.  

#### <a name="context-and-problem"></a>Contexte et problème  

Le service de la Table indexe automatiquement les entités en utilisant les valeurs **PartitionKey** et **RowKey** . Cela permet à une application client de récupérer une entité efficacement à l’aide d’une requête de point. Par exemple, à l’aide de la structure du tableau ci-dessous, une application cliente peut récupérer une entité d’employé individuel en utilisant le nom du service et l’id d’employé ( **PartitionKey** et **RowKey**).  

![][13]

Si vous voulez également être en mesure de récupérer une liste des entités employee en fonction de la valeur d’une autre propriété non unique, par exemple son nom, vous devez utiliser une analyse de partition moins efficace pour rechercher des correspondances, plutôt que de les rechercher directement à l’aide d’un index. C’est parce que le service de la table ne fournit pas les index secondaires.  

#### <a name="solution"></a>Solution  

Pour activer la recherche par nom de famille avec la structure d’entité indiquée ci-dessus, vous devez gérer des listes d’ID de l’employé. Si vous voulez extraire les entités employé avec un nom donné, tels que Jones, vous devez d’abord localiser la liste des ID d’employés pour les employés Jones, comme leur nom et ensuite récupérer ces entités employé. Il existe trois options principales pour stocker les listes d’ID de l’employé :  

-   Utilisez le stockage blob.  
-   Créer des entités de l’index dans la même partition que les entités employé.  
-   Créer des entités de l’index dans une table ou une partition séparée.  

<u>Option #1 : Stockage de blob d’utiliser</u>  

Pour la première option, vous créez un blob pour chaque nom unique et chaque banque de blob une liste de la **PartitionKey** (département) et **RowKey** (id employé) pour les employés qui portent ce nom. Lorsque vous ajoutez ou supprimez un employé, vous devez vous assurer que le contenu de l’objet blob pertinent est finalement cohérent avec les entités employé.  

<u>Option #2 :</u> Créer des entités de l’index dans la même partition  

Pour la deuxième option, utiliser des entités d’index qui stockent les données suivantes :  

![][14]

La propriété **EmployeeIDs** contient une liste des ID d’employés pour les employés, avec le nom stocké dans le **RowKey**.  

Les étapes suivantes décrivent la procédure à suivre lorsque vous ajoutez un nouvel employé si vous utilisez la seconde option. Dans cet exemple, nous ajoutons un employé avec l’Id 000152 et un nom de famille Jones dans le service des ventes :  
1.  Récupérer l’entité de l’index avec une valeur de **PartitionKey** « Ventes » et de la valeur de **RowKey** « Jones ». Enregistrer l’ETag de cette entité à utiliser à l’étape 2.  
2.  Créer une transaction de groupe d’entité (c'est-à-dire, une opération par lots) qui insère la nouvelle entité d’employé (**PartitionKey** valeur « Ventes » et la valeur de **RowKey** « 000152 ») et les mises à jour de l’entité de l’index (**PartitionKey** valeur « Ventes » et **RowKey** valeur « Jones ») en ajoutant le nouvel id d’employé à la liste dans le champ EmployeeIDs. Pour plus d’informations sur les transactions de groupe d’entité, consultez [Transactions de groupe de l’entité](#entity-group-transactions).  
3.  Si la transaction d’entité groupe échoue en raison d’une erreur d’accès concurrentiel optimiste (quelqu'un a simplement modifié l’entité index), vous devez recommencer à l’étape 1.  

Vous pouvez utiliser une approche similaire pour la suppression d’un employé si vous utilisez la seconde option. Modification du nom d’un employé est légèrement plus complexe, car vous devrez exécuter une transaction d’entité de groupe qui met à jour les trois entités : l’entité employee, l’entité d’index pour l’ancien nom et l’entité d’index pour le nouveau nom. Vous devez récupérer chaque entité avant d’apporter des modifications afin de récupérer les valeurs ETag que vous pouvez ensuite utiliser pour effectuer les mises à jour à l’aide de l’accès concurrentiel optimiste.  

Les étapes suivantes décrivent la procédure à suivre lorsque vous devez rechercher tous les employés dont le nom d’un département donné si vous utilisez la seconde option. Dans cet exemple, nous recherchons des tous les employés avec le nom de famille Jones dans le service des ventes :  

1.  Récupérer l’entité de l’index avec une valeur de **PartitionKey** « Ventes » et de la valeur de **RowKey** « Jones ».  
2.  Analyser la liste des ID dans le champ EmployeeIDs des employés.  
3.  Si vous avez besoin de plus d’informations sur chacun de ces employés (par exemple leurs adresses de messagerie), extraire chacune des entités employé à l’aide de la valeur de **PartitionKey** « Ventes » et **RowKey** les valeurs de la liste des employés que vous avez obtenu à l’étape 2.  

<u>Option #3 :</u> Créer des entités de l’index dans une table ou une partition séparée  

Pour la troisième option, utiliser des entités d’index qui stockent les données suivantes :  

![][15]

La propriété **EmployeeIDs** contient une liste des ID d’employés pour les employés, avec le nom stocké dans le **RowKey**.  

Avec la troisième option, vous ne pouvez pas utiliser EGTs pour préserver la cohérence car les entités index se trouvent dans une partition séparée des entités employé. Vous devez vous assurer que les entités d’index sont finalement cohérentes avec les entités employé.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  
-   Cette solution nécessite au moins deux requêtes pour récupérer des entités correspondantes : une pour interroger les entités d’index pour obtenir la liste des valeurs de **RowKey** , puis sur requêtes pour récupérer chaque entité dans la liste.  
-   Étant donné qu’une entité a une taille maximale de 1 Mo, option #2 et option #3 dans la solution supposent que la liste des ID d’employés pour n’importe quel nom donnée n’est jamais supérieure à 1 Mo. Si la liste des ID d’employés est susceptible d’être supérieure à la taille de 1 Mo, utilisez l’option #1 et stocker les données d’index dans le stockage blob.  
-   Si vous utilisez l’option #2 (à l’aide de EGTs pour gérer l’ajout et la suppression des employés et la modification du nom d’un employé) vous devez évaluer si le volume des transactions atteignent les limites de l’évolutivité dans une partition donnée. Si c’est le cas, vous devez envisager une solution finalement cohérente (option #1 ou #3) qui utilise des files d’attente pour gérer les demandes de mise à jour et vous permet de stocker vos entités d’index dans une partition séparée des entités employé.  
-   Option #2 dans cette solution suppose que vous souhaitez rechercher par nom de famille au sein d’un département : par exemple, vous souhaitez récupérer une liste d’employés avec un nom de famille Jones dans le service des ventes. Si vous souhaitez être en mesure de rechercher tous les employés avec un nom de famille Jones sur l’ensemble de l’organisation, utilisez l’option #1 ou #3.
-   Vous pouvez implémenter une solution basée sur la file d’attente qui assure la cohérence éventuelle (voir le [modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern) pour plus de détails).  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous souhaitez rechercher un ensemble d’entités qui partagent toutes une valeur de propriété commune, comme tous les employés dont le nom de famille Jones.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  
-   [Motif de clé composée](#compound-key-pattern)  
-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern)  
-   [Transactions de groupe de l’entité](#entity-group-transactions)  
-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)  

### <a name="denormalization-pattern"></a>Motif de la dénormalisation  

Combiner les données connexes dans une seule entité à vous permettent de récupérer toutes les données dont vous avez besoin d’une requête de point unique.  

#### <a name="context-and-problem"></a>Contexte et problème  

Dans une base de données relationnelle, vous généralement Normalisez les données pour supprimer des doublons qui en résulte dans les requêtes qui extraient des données de plusieurs tables. Si vous normalisez les données dans les tables Azure, vous devez vous plusieurs allers-retours à partir du client vers le serveur pour récupérer vos données connexes. Par exemple, avec la structure de la table ci-dessous vous devez deux allers-retours pour récupérer les détails d’un service : une pour extraire l’entité department contenant les id du responsable et puis d’une autre requête pour extraire des informations du responsable dans une entité employé.  

![][16]

#### <a name="solution"></a>Solution  

Au lieu de stocker les données dans les deux entités distinctes, dénormaliser les données et conserver une copie les détails du responsable de l’entité department. Par exemple :  

![][17]

Avec des entités de service stockées avec ces propriétés, vous pouvez désormais récupérer toutes les informations dont vous avez besoin à propos d’un service à l’aide d’une requête de point.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Il existe certains coût frais généraux associé au stockage des données deux fois. L’avantage de performances (résultant de moins de demandes au service de stockage) en général compense l’augmentation des coûts de stockage marginal (et que ce coût est partiellement compensé par une réduction du nombre de transactions que vous avez besoin d’extraire les détails d’un service).  
-   Vous devez maintenir la cohérence des deux entités qui stockent des informations sur les gestionnaires. Vous pouvez gérer le problème de la cohérence à l’aide de EGTs pour mettre à jour des entités multiples dans une seule transaction atomique : dans ce cas, l’entité department et l’entité d’employé du chef de service sont stockés dans la même partition.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle
Utilisez ce modèle lorsque vous devez fréquemment rechercher des informations connexes. Ce modèle permet de réduire le nombre de requêtes que votre client doit effectuer pour récupérer les données que requises.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés
Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  
-   [Motif de clé composée](#compound-key-pattern)  
-   [Transactions de groupe de l’entité](#entity-group-transactions)  
-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)

### <a name="compound-key-pattern"></a>Motif de clé composée  

COMPOSÉS **RowKey** valeurs permet d’activer un client de rechercher des données apparentées avec une requête de point unique.  

#### <a name="context-and-problem"></a>Contexte et problème  

Dans une base de données relationnelle, il est tout à fait naturel à utiliser des jointures dans les requêtes pour renvoyer les ensembles de données au client dans une seule requête. Par exemple, vous pouvez utiliser l’id d’employé pour rechercher une liste d’entités associées qui contiennent des performances et de consulter les données de cet employé.  

Supposons que vous stockez des entités employee dans le service de la Table à l’aide de la structure suivante :  

![][18]

Vous devez également stocker les données historiques relatives aux révisions et de performances pour chaque année de que l’employé a travaillé pour votre organisation et vous devez pouvoir accéder à ces informations par année. Une option consiste à créer une autre table qui stocke les entités avec la structure suivante :  

![][19]

Notez que, avec cette approche, vous pouvez décider de le dupliquer certaines informations (telles que le prénom et le nom) de la nouvelle entité pour vous permettre de récupérer vos données avec une requête unique. Toutefois, vous ne pouvez pas cohérence fort car vous ne pouvez pas utiliser un EGT mise à jour de manière atomique les deux entités.  

#### <a name="solution"></a>Solution
Stocker un nouveau type d’entité dans votre table d’origine à l’aide d’entités avec la structure suivante :  

![][20]

Notez que le **RowKey** est maintenant une clé composée, constituée de l’id d’employé et de l’année, les données de révision qui vous permet de récupérer les performances de l’employé et des données avec une requête unique pour une seule entité.  

L’exemple suivant décrit comment vous pouvez récupérer toutes les données de révision pour un employé particulier (comme 000123 employé du service des ventes) :  

$filter = (PartitionKey eq 'Sales') et (RowKey ge 'empid_000123') et (RowKey lt 'empid_000124') & $select = RowKey, évaluation du directeur, évaluation de l’homologue, commentaires  

#### <a name="issues-and-considerations"></a>Problèmes et considérations
Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Vous devez utiliser un caractère de séparation approprié qui facilite l’analyse de la valeur de **RowKey** : par exemple, **000123_2012**.  
-   Vous stockez également cette entité dans la même partition que les autres entités qui contiennent des données liées pour le même employé, ce qui signifie que vous pouvez utiliser EGTs pour préserver la cohérence renforcée.
-   Vous devez considérer la fréquence à laquelle vous interroge les données afin de déterminer si ce modèle est approprié.  Par exemple, si vous accéderez rarement les données de révision et les données d’employé principal souvent vous conservez les entités distinctes.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous devez stocker une ou plusieurs entités associées que vous interrogez fréquemment.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Transactions de groupe de l’entité](#entity-group-transactions)  
-   [Utilisation des types d’entités hétérogènes](#working-with-heterogeneous-entity-types)  
-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern)  

### <a name="log-tail-pattern"></a>Modèle de journal Queue  

Extraire les entités *n* plus récemment ajoutées à une partition à l’aide d’une valeur **RowKey** qui trie en inverse de date et de temps de l’ordre.  

#### <a name="context-and-problem"></a>Contexte et problème  

Une exigence courante est d’être en mesure d’extraire les entités plus récemment créées, par exemple les dix dernières dépenses soumises par un employé de revendications. Requêtes de table prend en charge une opération de requête **$top** pour renvoyer les premier entités *n* à partir d’un ensemble : aucune opération de requête équivalente pour renvoyer les derniers entités n dans un jeu.  

#### <a name="solution"></a>Solution  

Stocker les entités à l’aide d’un **RowKey** qui naturellement trie dans l’ordre inverse de date/heure à l’aide de l’entrée la plus récente est toujours la première condition dans la table.  

Par exemple, pour être en mesure de récupérer les déclarations de dépenses plus récentes dix soumises par un employé, vous pouvez utiliser une valeur de graduation inversée dérivée à partir de la date / l’heure en cours. L’exemple de code C# suivant montre comment créer une valeur de « inversé graduations » adapté pour une **RowKey** qui trie du plus récent au plus ancien :  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

Vous pouvez revenir à la valeur de temps de date à l’aide du code suivant :  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

La requête de table ressemble à ceci :  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Vous devez faire précéder la valeur de graduation inverse avec des zéros non significatifs afin de trie de la valeur de chaîne comme prévu.  
-   Vous devez être conscient des cibles de l’évolutivité au niveau d’une partition. Veillez à ne pas créer les partitions de zone réactive.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous avez besoin d’accéder aux entités dans l’ordre inverse de date/heure ou de lorsque vous devez accéder aux entités plus récemment ajoutées.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Ajouter / ajouter anti-modèle](#prepend-append-anti-pattern)  
-   [Lors de la récupération des entités](#retrieving-entities)  

### <a name="high-volume-delete-pattern"></a>Modèle de suppression de volume élevé  

Activer la suppression d’un volume élevé d’entités en stockant toutes les entités pour suppression simultanée dans leur propre table distincte ; suppression de la table permet de supprimer les entités.  

#### <a name="context-and-problem"></a>Contexte et problème  

De nombreuses applications supprimer les anciennes données qui n’a plus besoin d’être disponible pour une application cliente ou que l’application a archivé sur un autre support de stockage. Vous identifiez en général ces données par une date : par exemple, vous avez besoin de supprimer les enregistrements de toutes les demandes de connexion qui ont plus de 60 jours.  

Une conception possible est d’utiliser la date et l’heure de la demande d’ouverture de session dans le **RowKey**:  

![][21]

Cette approche évite les zones réactives de partition dans la mesure où l’application peut insérer et supprimer des entités de connexion pour chaque utilisateur dans une partition séparée. Toutefois, cette approche peut être coûteux et fastidieux si vous avez un grand nombre d’entités, car vous devez d’abord effectuer une analyse de table pour identifier toutes les entités à supprimer, et vous devez supprimer chaque entité ancienne. Notez que vous pouvez réduire le nombre d’allers-retours au serveur requis pour supprimer les anciennes entités par le traitement par lots de plusieurs demandes de suppression dans EGTs.  

#### <a name="solution"></a>Solution  

Utiliser une table distincte pour chaque jour de tentatives de connexion. Vous pouvez utiliser la conception de l’entité ci-dessus afin d’éviter les points d’accès lorsque vous insérez des entités, et la suppression des anciennes entités est simplement une question de la suppression d’une table tous les jours (une opération de stockage unique) au lieu de la recherche et la suppression des centaines et des milliers d’entités de connexion individuels de chaque jour.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Votre conception prend-elle en charge les autres méthodes que votre application utilisera les données telles que la recherche des entités spécifiques, en liaison avec d’autres données ou générer des informations agrégées ?  
-   Votre conception est éviter réactives lorsque vous insérez des nouvelles entités ?  
-   Prévoyez un délai d’attente si vous souhaitez réutiliser le même nom de table après sa suppression. Il est préférable de toujours utiliser des noms de table unique.  
-   Prévoyez une limitation lorsque vous utilisez d’abord une table alors que le service de la Table est capable d’assimiler les modèles d’accès et distribue les partitions sur les nœuds. Vous devez envisager la fréquence à laquelle vous avez besoin créer des tables.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous avez un volume élevé d’entités que vous devez supprimer en même temps.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Transactions de groupe de l’entité](#entity-group-transactions)
-   [Modification des entités](#modifying-entities)  

### <a name="data-series-pattern"></a>Modèle de série de données  

Série de données complète de magasin dans une entité unique pour réduire le nombre de demandes.  

#### <a name="context-and-problem"></a>Contexte et problème  

Un scénario courant est une application peut stocker une série de données dont il a besoin en général à récupérer en une seule fois. Par exemple, votre application peut enregistrer le nombre de messages MI chaque employé envoie toutes les heures et ensuite utiliser ces informations pour tracer le nombre de messages chaque utilisateur envoyée au cours des 24 heures précédentes. Un modèle peut être pour stocker les 24 entités pour chaque employé :  

![][22]

Grâce à cette conception, vous pouvez facilement localiser et mettre à jour l’entité mise à jour pour chaque employé chaque fois que l’application doit mettre à jour la valeur de nombre du message. Cependant, pour récupérer les informations pour tracer un graphique de l’activité des 24 heures précédentes, vous devez récupérer 24 entités.  

#### <a name="solution"></a>Solution  

Utilisez la conception suivante avec une propriété séparée pour stocker le nombre de messages pour chaque heure :  

![][23]

Grâce à cette conception, vous pouvez utiliser une opération de fusion pour mettre à jour le nombre de messages pour un employé sur une heure spécifique. Maintenant, vous pouvez récupérer toutes les informations dont vous avez besoin de tracer le graphique à l’aide d’une demande d’une entité unique.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  
-   Si votre série de données complète ne tient pas dans une entité unique (une entité peut avoir des propriétés jusqu'à 252), utiliser un autre magasin de données comme un objet blob.  
-   Si vous avez plusieurs clients mise à jour d’une entité simultanément, vous devrez utiliser l' **ETag** pour implémenter l’accès concurrentiel optimiste. Si vous avez de nombreux clients, vous pouvez rencontrer une contention élevée.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous devez mettre à jour et récupérer une série de données associée à une entité individuelle.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Modèle des entités volumineuses](#large-entities-pattern)  
-   [Fusion ou remplacer](#merge-or-replace)  
-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern) (si vous stockez la série de données dans un objet blob)  

### <a name="wide-entities-pattern"></a>Modèle des entités large  

Plusieurs entités physiques permet de stocker des entités logiques avec plus de 252 propriétés.  

#### <a name="context-and-problem"></a>Contexte et problème  

Une entité peut avoir des propriétés pas plus de 252 (à l’exclusion des propriétés système obligatoire) et ne peut pas stocker de plus de 1 Mo de données au total. Dans une base de données relationnelle, vous obtiendrez en général rond toutes les limites de la taille d’une ligne par ajout d’une nouvelle table et de mise en œuvre d’une relation 1-1 entre elles.  

#### <a name="solution"></a>Solution  

À l’aide du service de la Table, vous pouvez stocker plusieurs entités pour représenter un objet unique de grandes entreprises avec plus de 252 propriétés. Par exemple, si vous souhaitez stocker le nombre de messages instantanés envoyés par chaque employé au cours des 365 derniers jours, vous pouvez utiliser la conception suivante qui utilise deux entités avec des schémas différents :  

![][24]

Si vous souhaitez apporter une modification qui nécessite la mise à jour des deux entités pour les conserver synchronisés entre eux, vous pouvez utiliser un EGT. Dans le cas contraire, vous pouvez utiliser une seule opération de fusion pour mettre à jour le nombre de messages pour un jour spécifique. Pour récupérer toutes les données pour un employé individuel, vous devez récupérer les deux entités, vous pouvez faire avec deux demandes efficaces qui utilisent à la fois une **PartitionKey** et une valeur de **RowKey** .  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Extraction d’une entité logique complète implique au moins deux transactions de stockage : une pour récupérer chaque entité physique.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque nécessaire stocker les entités dont la taille ou le nombre de propriétés dépassent les limites d’une entité individuelle dans le service de la Table.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Transactions de groupe de l’entité](#entity-group-transactions)
-   [Fusion ou remplacer](#merge-or-replace)

### <a name="large-entities-pattern"></a>Modèle des entités volumineuses  

Utilisez le stockage blob pour stocker des valeurs de propriété de grande taille.  

#### <a name="context-and-problem"></a>Contexte et problème  

Une entité individuelle ne peut pas stocker plus de 1 Mo de données au total. Si une ou plusieurs de vos propriétés de stockent des valeurs qui provoquent la taille totale de votre entité de dépasser cette valeur, vous ne pouvez pas stocker l’entité entière dans le service de la Table.  

#### <a name="solution"></a>Solution  

Si votre entité dépasse 1 Mo la taille car une ou plusieurs propriétés contiennent une grande quantité de données, vous pouvez stocker des données dans le service d’objet Blob et ensuite stocker l’adresse de l’objet blob dans une propriété de l’entité. Par exemple, vous pouvez stocker la photo d’un employé dans le stockage blob et stocker un lien vers la photo dans la propriété de la **Photo** de votre entité employee :  

![][25]

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   Pour maintenir la cohérence éventuelle entre l’entité dans le service de la Table et les données dans le service d’objet Blob, utilisez le [modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern) pour maintenir vos entités.
-   Extraction d’une entité complète implique au moins deux transactions de stockage : une pour récupérer l’entité et l’autre pour extraire les données blob.  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Utilisez ce modèle lorsque vous avez besoin de stocker des entités dont la taille dépasse les limites d’une entité individuelle dans le service de la Table.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Modèle de transactions finalement cohérente](#eventually-consistent-transactions-pattern)  
-   [Modèle des entités large](#wide-entities-pattern)

<a name="prepend-append-anti-pattern"></a>
### <a name="prependappend-anti-pattern"></a>Ajouter/ajouter l’anti-modèle  

Augmenter l’évolutivité lorsque vous avez un volume élevé d’insertions en répartissant les insertions sur plusieurs partitions.  

#### <a name="context-and-problem"></a>Contexte et problème  

Ajoutant ou ajouter des entités à vos entités stockées généralement entraîne l’application Ajout de nouvelles entités à la première ou la dernière partition d’une séquence de partitions. Dans ce cas, toutes les insertions à tout moment ont lieu dans la même partition, création d’un point d’accès qui empêche le service de la table à partir de l’équilibrage de charge insertions sur plusieurs nœuds et probablement à l’origine de votre application sur les cibles de l’évolutivité de la partition. Par exemple, si vous possédez une application qui ouvre une session réseau et la partition de l’heure en cours devient un point d’accès, si le volume des transactions atteint la cible de l’évolutivité d’une partition individuelle peut entraîner l’accès aux ressources par les employés, puis une structure d’entité comme indiqué ci-dessous :  

![][26]

#### <a name="solution"></a>Solution  

La structure de l’autre entité suivante permet d’éviter une zone sensible sur n’importe quelle partition particulière comme les journaux des événements d’application :  

![][27]

Dans cet exemple, notez comment les **PartitionKey** et **RowKey** sont des clés composées. L' **PartitionKey** utilise des id du département et les employés pour distribuer l’enregistrement sur plusieurs partitions.  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment implémenter ce modèle :  

-   La structure de clés de remplacement qui évite de créer efficacement des partitions à chaud sur les insertions prend en charge les requêtes de que votre application cliente fait ?  
-   Votre volume escompté des transactions signifie que vous êtes susceptible d’atteindre les objectifs de l’évolutivité pour une partition individuelle et être limité par le service de stockage ?  

#### <a name="when-to-use-this-pattern"></a>Quand utiliser ce modèle  

Éviter l’anti-modèle prepend/Ajout lorsque le volume des transactions est susceptible d’entraîner la limitation par le service de stockage lorsque vous accédez à une partition à chaud.  

#### <a name="related-patterns-and-guidance"></a>Conseils et des modèles associés  

Les modèles et les conseils suivants peuvent également être intéressant lors de l’implémentation de ce modèle :  

-   [Motif de clé composée](#compound-key-pattern)  
-   [Modèle de journal Queue](#log-tail-pattern)  
-   [Modification des entités](#modifying-entities)  

### <a name="log-data-anti-pattern"></a>Modèle des données de journal  

En règle générale, vous devez utiliser le service d’objet Blob au lieu de service de la Table pour stocker des données de journal.  

#### <a name="context-and-problem"></a>Contexte et problème  

Cas d’usage commun pour les données du journal sont de récupérer une sélection d’entrées de journal pour une plage de date/heure spécifique : par exemple, vous souhaitez rechercher tous les messages d’erreur et critiques votre application connecté entre 15:04 et 15:06 à une date spécifique. Vous ne souhaitez pas utiliser la date et l’heure du message journal pour déterminer la partition que vous enregistrez le journal des entités : qui résulte d’une partition à chaud car à un moment donné, toutes les entités de journal partagent la même valeur de **PartitionKey** (voir la section [Prepend/ajouter l’anti-modèle](#prepend-append-anti-pattern)). Par exemple, le schéma de l’entité suivante pour un message de journal génère une partition à chaud car l’application écrit tous les messages du journal sur la partition pour la date et l’heure :  

![][28]

Dans cet exemple, la **RowKey** inclut la date et l’heure du message journal pour vous assurer que les messages du journal sont stockées triés dans l’ordre de date/heure et inclut un id de message, dans le cas où plusieurs messages de journal partagent les mêmes date et heure.  

Une autre approche consiste à utiliser un **PartitionKey** qui permet de s’assurer que l’application enregistre les messages au sein d’une plage de partitions. Par exemple, si la source du message de journal fournit un moyen de distribuer les messages sur plusieurs partitions, vous pouvez utiliser le schéma d’entité suivant :  

![][29]

Toutefois, le problème avec ce schéma est que pour récupérer tous les messages de journal pour un intervalle de temps spécifique vous devez recherche dans chaque partition de la table.

#### <a name="solution"></a>Solution  

La section précédente soulignait le problème consiste à utiliser le service de la Table pour stocker des entrées de journal et deux suggéré, non satisfaisante, conceptions. Une solution a conduit à une partition à chaud avec le risque de faibles performances, écrire des messages de journal ; l’autre solution provoqué médiocrité des performances en raison de la nécessité d’analyser chaque partition de la table pour récupérer les messages du journal pour un intervalle de temps spécifique. Stockage des objets BLOB offre une meilleure solution pour ce type de scénario et il s’agit de magasins de stockage Analytique Azure comment les données du journal, il collecte.  

Cette section décrit comment Analytique de stockage stocke les données du journal dans le stockage blob sous la forme d’une illustration de cette méthode de stockage de données qui en général d’une requête par plage.  

Analytique de stockage stocke les messages du journal dans un format délimité dans plusieurs BLOB. Le format délimité facilite pour une application cliente analyser les données dans le message de journal.  

Analytique de stockage utilise une convention de dénomination des BLOB qui vous permet de localiser le blob (ou BLOB) qui contiennent les messages du journal que vous recherchez. Par exemple, un blob appelé « queue/2014/07/31/1800/000001.log » contient des messages de journal qui sont liés au service de file d’attente pour les heures en commençant à 18:00 le 31 juillet 2014. Le « 000001 » indique qu’il s’agit du premier fichier journal pour cette période. Analytique de stockage enregistre également les horodatages les premier et le dernier des messages du journal stockés dans les métadonnées de l’objet blob dans le fichier. L’API pour permet de stockage blob vous recherchez des objets BLOB dans un conteneur selon un préfixe de nom : pour rechercher tous les objets BLOB qui contiennent des données de journal de file d’attente pour les heures en commençant à 18:00, vous pouvez utiliser le préfixe « file d’attente/2014/07/31/1800. »  

Mémoires tampons de stockage Analytique consigne les messages en interne et régulièrement mises à jour de l’objet blob approprié ou crée un nouveau avec le dernier lot d’entrées de journal. Cela permet de réduire le nombre d’écritures, qu'il doit exécuter pour le service d’objet blob.  

Si vous implémentez une solution similaire dans votre propre application, vous devez déterminer comment gérer le compromis entre la fiabilité (écriture de chaque entrée de journal pour le stockage des objets blob il se trouve) et de coût et de l’évolutivité (mise en mémoire tampon des mises à jour dans votre application et les écrire dans le stockage blob par lots).  

#### <a name="issues-and-considerations"></a>Problèmes et considérations  

Tenez compte des points suivants lorsque vous décidez comment stocker des données de journal :  

-   Si vous créez une structure de table afin d’éviter d’éventuelles partitions à chaud, vous pouvez trouver que vous ne peut pas accéder à vos données de journal efficacement.  
-   Pour traiter les données du journal, un client doit souvent charger plusieurs enregistrements.  
-   Bien que les données du journal sont souvent structurées, stockage blob peut être une meilleure solution.  

### <a name="implementation-considerations"></a>Considérations relatives à la mise en oeuvre  

Cette section décrit certaines des considérations à garder à l’esprit lorsque vous implémentez les modèles décrits dans les sections précédentes. La plupart de la présente section utilise des exemples écrits en C# qui utilisent la bibliothèque Client de stockage (version 4.3.0 au moment de la rédaction).  

### <a name="retrieving-entities"></a>Lors de la récupération des entités  

Comme expliqué dans la section [Création de requête](#design-for-querying), la requête la plus efficace est une requête de point. Toutefois, dans certains scénarios, vous devrez récupérer plusieurs entités. Cette section décrit certaines des approches courantes pour les entités à l’aide de la bibliothèque de Client de stockage de récupération.  

#### <a name="executing-a-point-query-using-the-storage-client-library"></a>L’exécution d’une requête de point à l’aide de la bibliothèque de Client de stockage  

Pour exécuter une requête de point, le plus simple consiste à utiliser l’opération de table **récupérer** comme indiqué dans le C# extrait de code suivant qui Récupère une entité avec un **PartitionKey** de valeur « Ventes » et un **RowKey** de valeur « 212 » :  

    TableOperation retrieveOperation =
        TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
    var retrieveResult = employeeTable.Execute(retrieveOperation);
    if (retrieveResult.Result != null)
    {
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
    }  

Notez la façon dont cet exemple s’attend à l’entité est extrait pour être de type **EmployeeEntity**.  

#### <a name="retrieving-multiple-entities-using-linq"></a>Extraction de plusieurs entités à l’aide de LINQ  

Vous pouvez extraire plusieurs entités par l’utilisation de LINQ avec la bibliothèque cliente de stockage et de spécifier une requête avec une clause **where** . Pour éviter une analyse de table, vous devez toujours inclure la valeur de **PartitionKey** dans la page où la clause, et si possible la valeur **RowKey** afin d’éviter de table et la partition. Le service de la table prend en charge un ensemble limité d’opérateurs de comparaison (supérieurs à, supérieurs ou égales à, inférieur à, inférieur ou égales, égales et n’est pas égales) à utiliser dans la page où clause. L’extrait de code C# suivante recherche tous les employés dont le nom commence par « B » (en supposant que la **RowKey** stocke le nom de famille), dans le département des ventes (en supposant que les banques de **PartitionKey** le nom du service) :  

    TableQuery<EmployeeEntity> employeeQuery =
            employeeTable.CreateQuery<EmployeeEntity>();
    var query = (from employee in employeeQuery
                where employee.PartitionKey == "Sales" &&
                employee.RowKey.CompareTo("B") >= 0 &&
                employee.RowKey.CompareTo("C") < 0
                select employee).AsTableQuery();
    var employees = query.Execute();  

Notez comment la requête spécifie un **RowKey** et un **PartitionKey** pour garantir des performances optimales.  

Suivant code exemple présente une fonctionnalité équivalente à l’aide de l’API fluent (pour plus d’informations sur l’API fluent voir en règle générale, les [Meilleures pratiques pour la conception d’une API Fluent](http://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx)) :  

    TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
     TableQuery.CombineFilters(
        TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales"),
      TableOperators.And,
      TableQuery.GenerateFilterCondition(
        "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
    ),
    TableOperators.And,
    TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
     )
    );
    var employees = employeeTable.ExecuteQuery(employeeQuery);  


>[AZURE.NOTE] L’exemple imbrique plusieurs méthodes **CombineFilters** pour inclure les trois conditions de filtre.  

#### <a name="retrieving-large-numbers-of-entities-from-a-query"></a>Récupérer un grand nombre d’entités à partir d’une requête  

Une requête optimale retourne une entité individuelle basée sur une valeur de **PartitionKey** et une valeur de **RowKey** . Toutefois, dans certains scénarios peuvent avoir une exigence pour de nombreuses entités de retour à partir de la même partition ou même à partir de plusieurs partitions.  

Vous devez toujours entièrement tester les performances de votre application dans ces scénarios.  

Une requête contre le service de la table peut renvoyer un maximum de 1 000 entités en une seule fois et peut exécuter un maximum de cinq secondes. Si le jeu de résultats contient des entités de plus de 1 000, si la requête ne s’est pas terminée dans les cinq secondes, ou si la requête dépasse la limite de partition, le service de la Table retourne un jeton de continuation pour permettre à l’application client demander l’ensemble suivant d’entités. Pour plus d’informations sur la continuation de jetons de travail, reportez-vous à la section [délai d’expiration de la requête et la Pagination](http://msdn.microsoft.com/library/azure/dd135718.aspx).  

Si vous utilisez la bibliothèque de Client de stockage, elle peut gérer automatiquement les jetons de continuation pour vous qu’il retourne des entités à partir du service de la Table. C# code ci-après à l’aide de la bibliothèque de Client de stockage automatiquement gère les jetons de continuation si le service de la table les retourne dans une réponse :  

    string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
    TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

    var employees = employeeTable.ExecuteQuery(employeeQuery);
    foreach (var emp in employees)
    {
        ...
    }  

Le code C# suivant gère explicitement les jetons de continuation :  

    string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
    TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

    TableContinuationToken continuationToken = null;

    do
    {
        var employees = employeeTable.ExecuteQuerySegmented(
            employeeQuery, continuationToken);
    foreach (var emp in employees)
    {
     ...
    }
    continuationToken = employees.ContinuationToken;
    } while (continuationToken != null);  

En utilisant des jetons de continuation explicitement, vous pouvez contrôler lorsque votre application récupère le segment de données suivant. Par exemple, si votre application cliente permet aux utilisateurs de parcourir les entités stockées dans une table, un utilisateur peut décider ne pas vers la page par le biais de toutes les entités récupérées par la requête, afin que votre application utilise uniquement un jeton de continuation pour récupérer le segment suivant lorsque l’utilisateur a terminé la pagination via toutes les entités dans le segment actuel. Cette approche présente plusieurs avantages :  

-   Il vous permet de limiter la quantité de données à récupérer à partir du service de la Table et que vous déplacez sur le réseau.  
-   Il vous permet d’effectuer des e/s asynchrones dans .NET.  
-   Il vous permet de sérialiser le jeton de continuation dans un stockage permanent pour vous pouvez de poursuivre dans le cas d’une panne de l’application.  

>[AZURE.NOTE] Un jeton de continuation renvoie généralement un segment contenant les entités de 1 000, bien qu’il soit moins. C’est également le cas si vous limitez le nombre d’entrées d’une requête renvoie à l’aide de **prendre** pour renvoyer les premier entités n correspondant à vos critères de recherche : le service de la table peut renvoyer un segment contenant moins de n entités avec un jeton de continuation pour vous permettent de récupérer les autres entités.  

Le code C# suivant montre comment modifier le nombre d’entités retournées à l’intérieur d’un segment :  

    employeeQuery.TakeCount = 50;  

#### <a name="server-side-projection"></a>Projection de côté serveur  

Une seule entité peut avoir jusqu'à 255 propriétés et être jusqu'à 1 Mo. Lorsque vous interrogez la table et récupérer des entités, vous n’avez pas besoin de toutes les propriétés et pouvez éviter de transférer des données inutilement (afin de réduire les coûts et les temps de latence). Vous pouvez utiliser la projection côté serveur pour transférer uniquement les propriétés dont vous avez besoin. L’exemple suivant est un extrait simplement la propriété **Email** (avec **PartitionKey**, **RowKey**, **horodatage**et **ETag**) à partir des entités sélectionnées par la requête.  

    string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
    List<string> columns = new List<string>() { "Email" };
    TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

    var entities = employeeTable.ExecuteQuery(employeeQuery);
    foreach (var e in entities)
    {
        Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
    }  

Notez comment la valeur de **RowKey** est disponible même s’il n’était pas inclus dans la liste de propriétés à récupérer.  

### <a name="modifying-entities"></a>Modification des entités  

La bibliothèque cliente de stockage vous permet de modifier vos entités stockées dans le service de la table par l’insertion, la suppression et la mise à jour des entités. Vous pouvez utiliser EGTs pour le traitement par lots de plusieurs opérations d’insertion, mise à jour et suppression afin de réduire le nombre d’allers-retours nécessaires et d’améliorer les performances de votre solution.  

Notez que les exceptions levées lorsque la bibliothèque Client de stockage s’exécute en général un EGT incluent l’index de l’entité qui a provoqué le lot échoue. Ceci est utile lorsque vous déboguez du code qui utilise EGTs.  

Vous devez également envisager comment votre conception affecte comment votre application cliente gère les opérations d’accès concurrentiel et de mise à jour.  

#### <a name="managing-concurrency"></a>Gérer l’accès concurrentiel  

Par défaut, le service de la table implémente optimiste de contrôles d’accès concurrentiel au niveau des entités individuelles pour **Insérer**, **Fusionner**et **Supprimer** des opérations, bien qu’il soit possible pour un client forcer le service de table pour contourner ces contrôles. Pour plus d’informations sur la façon dont le service de la table gère l’accès concurrentiel, consultez [Gestion de la simultanéité dans le stockage Azure de Microsoft](storage-concurrency.md).  

#### <a name="merge-or-replace"></a>Fusion ou remplacer  

La méthode **Replace** de la classe **TableOperation** remplace toujours l’entité complète dans le service de la Table. Si vous n’incluez pas une propriété de la demande lorsque cette propriété existe dans l’entité stockée, la requête supprime cette propriété de l’entité stockée. Sauf si vous souhaitez supprimer une propriété explicitement une entité stockée, vous devez inclure toutes les propriétés dans la demande.  

Vous pouvez utiliser la méthode **Merge** de la classe **TableOperation** pour réduire la quantité de données que vous envoyez au service de la Table lorsque vous souhaitez mettre à jour une entité. La méthode **Merge** remplace toutes les propriétés de l’entité stockée avec des valeurs de propriété de l’entité inclus dans la demande, mais laisse intactes toutes les propriétés de l’entité stockée ne sont pas incluses dans la demande. Cela est utile si vous avez entités volumineuses et mise à jour d’un petit nombre de propriétés dans une demande.  

>[AZURE.NOTE] Les méthodes de **Remplacer** et **Fusionner** échouent si l’entité n’existe pas. Comme alternative, vous pouvez utiliser les méthodes **InsertOrReplace** et **InsertOrMerge** qui créer une nouvelle entité si elle n’existe pas.  

### <a name="working-with-heterogeneous-entity-types"></a>Utilisation des types d’entités hétérogènes  

Le service de la Table est un magasin de table de *schéma sans* qui signifie qu’une seule table peut stocker des entités de plusieurs types, offrant une grande flexibilité dans la conception. L’exemple suivant illustre une table stockant les entités de service et de l’employé :  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Horodatage</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

Notez que chaque entité doit avoir toujours les valeurs **Timestamp** , **RowKey**et **PartitionKey**, mais peut avoir n’importe quel jeu de propriétés. En outre, il n’y a rien pour indiquer le type d’une entité, sauf si vous choisissez de stocker ces informations quelque part. Il existe deux options pour identifier le type d’entité :  

-   Faites précéder le nom du type d’entité à la **RowKey** (ou éventuellement du **PartitionKey**). Par exemple, **EMPLOYEE_000123** ou **DEPARTMENT_SALES** en tant que valeurs de **RowKey** .  
-   Utilisez une propriété distincte pour enregistrer le type d’entité, comme indiqué dans le tableau ci-dessous.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Horodatage</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Employé</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Employé</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Département</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Prénom</th>
<th>Nom</th>
<th>Durée de vie</th>
<th>Messagerie</th>
</tr>
<tr>
<td>Employé</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

La première option, en ajoutant le type d’entité à la **RowKey**, est utile s’il existe un risque que les deux entités de différents types peuvent avoir la même valeur de clé. Il regroupe également les entités du même type dans la partition.  

Les techniques présentées dans cette section sont rapportent spécifiquement à la discussion sur les [relations d’héritage](#inheritance-relationships) plus haut dans ce guide dans la section [relations de la modélisation](#modelling-relationships).  

>[AZURE.NOTE] Vous devez envisager incluant un numéro de version de la valeur de type d’entité pour activer les applications clientes de faire évoluer les objets POCO et fonctionner avec différentes versions.  

Le reste de cette section décrit certaines des fonctionnalités de la bibliothèque de Client de stockage qui facilitent l’utilisation de plusieurs types d’entités dans la même table.  

#### <a name="retrieving-heterogeneous-entity-types"></a>Récupération de types d’entités hétérogènes  

Si vous utilisez la bibliothèque de Client de stockage, vous avez trois options pour travailler avec plusieurs types d’entités.  

Si vous connaissez le type de l’entité stockée avec des valeurs de **RowKey** et des **PartitionKey** spécifiques, puis vous pouvez spécifier le type d’entité lorsque vous récupérez l’entité comme illustré dans les deux exemples précédents qui récupèrent les entités du type **EmployeeEntity**: [exécution d’une requête de point à l’aide de la bibliothèque de Client de stockage](#executing-a-point-query-using-the-storage-client-library) et [l’extraction de plusieurs entités utilisant des requêtes LINQ](#retrieving-multiple-entities-using-linq).  

La deuxième option consiste à utiliser le type **DynamicTableEntity** (un sac de propriété) au lieu d’un type d’entité concrète POCO (cette option peut également améliorer les performances, car il est inutile pour sérialiser et désérialiser l’entité types .NET). Le code C# suivant est potentiellement récupère plusieurs entités de types différents à partir de la table, mais retourne toutes les entités en tant qu’instances de **DynamicTableEntity** . Il utilise ensuite la propriété **EntityType** pour déterminer le type de chaque entité :  

    string filter =     TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("PartitionKey",
      QueryComparisons.Equal, "Sales"),
        TableOperators.And,
        TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition("RowKey",
                    QueryComparisons.GreaterThanOrEqual, "B"),
            TableOperators.And,
            TableQuery.GenerateFilterCondition("RowKey",
          QueryComparisons.LessThan, "F")
        )
    );
    TableQuery<DynamicTableEntity> entityQuery =
    new TableQuery<DynamicTableEntity>().Where(filter);
    var employees = employeeTable.ExecuteQuery(entityQuery);

    IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
    foreach (var e in entities)
    {
    EntityProperty entityTypeProperty;
    if (e.Properties.TryGetValue("EntityType", out entityTypeProperty))
    {
        if (entityTypeProperty.StringValue == "Employee")
        {
            // Use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
          }
     }
    }  

Notez que pour récupérer d’autres propriétés vous devez utiliser la méthode **TryGetValue** sur la propriété **Properties** de la classe **DynamicTableEntity** .  

Une troisième option consiste à combiner à l’aide du type **DynamicTableEntity** et une instance de **EntityResolver** . Cela vous permet de résoudre plusieurs types POCO dans la même requête. Dans cet exemple, le délégué **EntityResolver** est à l’aide de la propriété **EntityType** pour faire la distinction entre les deux types d’entité retourné par la requête. La méthode **Resolve** utilise le délégué de **résolution** pour résoudre des instances de **DynamicTableEntity** pour les instances de **TableEntity** .  

    EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
    {

        TableEntity resolvedEntity = null;
        if (props["EntityType"].StringValue == "Department")
        {
            resolvedEntity = new DepartmentEntity();
        }
        else if (props["EntityType"].StringValue == "Employee")
        {
            resolvedEntity = new EmployeeEntity();
        }
        else throw new ArgumentException("Unrecognized entity", "props");

        resolvedEntity.PartitionKey = pk;
        resolvedEntity.RowKey = rk;
        resolvedEntity.Timestamp = ts;
        resolvedEntity.ETag = etag;
        resolvedEntity.ReadEntity(props, null);
        return resolvedEntity;
    };

    string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
    TableQuery<DynamicTableEntity> entityQuery =
        new TableQuery<DynamicTableEntity>().Where(filter);

    var entities = employeeTable.ExecuteQuery(entityQuery, resolver);
    foreach (var e in entities)
    {
        if (e is DepartmentEntity)
        {
        ...
        }
        if (e is EmployeeEntity)
        {
        ...
        }
    }  

#### <a name="modifying-heterogeneous-entity-types"></a>Modification de types d’entités hétérogènes  

Vous n’avez pas besoin de connaître le type d’une entité à supprimer, et vous savez toujours le type d’une entité lorsque vous l’insérez. Toutefois, vous pouvez utiliser le type de **DynamicTableEntity** pour mettre à jour une entité sans connaître son type et sans l’aide d’une classe d’entité POCO. L’exemple de code suivant récupère une entité unique et il vérifie que la propriété **EmployeeCount** existe avant de mettre à jour.  

    TableResult result =
        employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
    DynamicTableEntity department = (DynamicTableEntity)result.Result;

    EntityProperty countProperty;

    if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
    {
        throw new
            InvalidOperationException("Invalid entity, EmployeeCount property not found.");
    }
    countProperty.Int32Value += 1;
    employeeTable.Execute(TableOperation.Merge(department));  

### <a name="controlling-access-with-shared-access-signatures"></a>Contrôle d’accès avec des Signatures de l’accès partagé  

Vous pouvez utiliser les jetons de Signature de l’accès partagé (SAS) pour permettre aux applications clientes de modifier (requête) directement sans avoir besoin de s’authentifier directement auprès du service de la table, les entités de table. Il existe trois principaux avantages à l’utilisation des associations de sécurité dans votre application :  

-   Vous n’avez pas besoin de distribuer votre clé de compte de stockage pour une plate-forme non sécurisée (par exemple, un périphérique mobile) afin de permettre à ce périphérique accéder et modifier des entités dans le service de la Table.  
-   Vous pouvez décharger certaines tâches de travail qui exécutent des rôles web et worker dans la gestion de vos entités pour les périphériques clients tels que les ordinateurs des utilisateurs finaux et les périphériques mobiles.  
-   Vous pouvez affecter une contrainte et temps limité le jeu d’autorisations à un client (par exemple, ce qui permet un accès en lecture seule aux ressources spécifiques).  

Pour plus d’informations sur l’utilisation de jetons d’associations de sécurité avec le service de la Table, consultez [L’aide partagés accès Signatures (SAS)](storage-dotnet-shared-access-signature-part-1.md).  

Toutefois, vous devez tout de même générer les jetons SAS qui accordent une application cliente pour les entités dans le service de la table : vous devez procéder dans un environnement disposant d’un accès sécurisé à vos clés de compte de stockage. En général, vous utilisez un rôle web ou travailleur pour générer les jetons d’associations de sécurité et de les livrer pour les applications clientes qui ont besoin d’accéder à vos entités. Dans la mesure où il existe toujours une surcharge impliqués dans la génération et la livraison des jetons SAS à des clients, vous devez envisager la meilleure méthode pour réduire cette charge, en particulier dans les scénarios à volume élevé.  

Il est possible de générer un jeton d’associations de sécurité qui accorde l’accès à un sous-ensemble des entités dans une table. Par défaut, vous créez un jeton SAS pour une table entière, mais il est également possible de spécifier que le jeton SAS accorder l’accès à une plage de valeurs de **PartitionKey** , ou à une plage de valeurs **PartitionKey** et **RowKey** . Vous pouvez choisir de générer des jetons SAS pour les utilisateurs individuels de votre système tel que chaque jeton d’utilisateur SAS ne lui permet d’accéder à leurs propres entités dans le service de la table.  

### <a name="asynchronous-and-parallel-operations"></a>Opérations asynchrones et parallèles  

Si vous effectuez la diffusion vos requêtes sur plusieurs partitions, vous pouvez améliorer la réactivité du débit et le client en utilisant des requêtes asynchrones ou parallèles.
Par exemple, vous pouvez avoir deux ou plusieurs instances de rôle de travail à accéder à vos tables en parallèle. Vous pourriez avoir des rôles de travailleur responsables des ensembles particuliers de partitions ou simplement avoir plusieurs instances de rôle Collaborateur, chacun étant en mesure d’accéder à toutes les partitions d’une table.  

Au sein d’une instance du client, vous pouvez améliorer le débit par l’exécution des opérations de stockage en mode asynchrone. La bibliothèque cliente de stockage facilite l’écriture des modifications et des requêtes asynchrones. Par exemple, vous pouvez démarrer avec la méthode synchrone qui Récupère toutes les entités dans une partition, comme illustré dans le code C# suivant :  

    private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
    {
        string filter = TableQuery.GenerateFilterCondition(
            "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
            new TableQuery<EmployeeEntity>().Where(filter);

        TableContinuationToken continuationToken = null;

        do
        {
            var employees = employeeTable.ExecuteQuerySegmented(
                employeeQuery, continuationToken);
            foreach (var emp in employees)
        {
        ...
        }
            continuationToken = employees.ContinuationToken;
        } while (continuationToken != null);
    }  

Vous pouvez facilement modifier ce code afin que la requête s’exécute de manière asynchrone, comme suit :  

    private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
    {
        string filter = TableQuery.GenerateFilterCondition(
            "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
            new TableQuery<EmployeeEntity>().Where(filter);
        TableContinuationToken continuationToken = null;

        do
        {
            var employees = await employeeTable.ExecuteQuerySegmentedAsync(
                employeeQuery, continuationToken);
            foreach (var emp in employees)
            {
             ...
            }
            continuationToken = employees.ContinuationToken;
            } while (continuationToken != null);
    }  

Dans cet exemple asynchrone, vous pouvez voir les modifications suivantes à partir de la version synchrone :  

-   Maintenant, la signature de méthode inclut le modificateur **asynchrone** et retourne une instance de **tâche** .  
-   Au lieu d’appeler la méthode **ExecuteSegmented** pour récupérer les résultats, la méthode appelle la méthode **ExecuteSegmentedAsync** et utilise le modificateur **await** pour récupérer les résultats de façon asynchrone.  

L’application cliente peut appeler cette méthode plusieurs fois (avec des valeurs différentes pour le paramètre de **service** ), et chaque requête sera exécutée sur un thread séparé.  

Notez qu’il n’existe aucune version asynchrone de la méthode **Execute** dans la classe **TableQuery** , car l’interface **IEnumerable** ne prend pas en charge les énumérations asynchrones.  

Vous pouvez également insérer, mettre à jour et supprimer des entités de manière asynchrone. L’exemple C# suivant illustre une méthode simple et synchrone pour insérer ou remplacer une entité d’employé :  

    private static void SimpleEmployeeUpsert(CloudTable employeeTable,
        EmployeeEntity employee)
    {
        TableResult result = employeeTable
            .Execute(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
    }  

Vous pouvez facilement modifier ce code afin que la mise à jour s’exécute de façon asynchrone comme suit :  

    private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable,
        EmployeeEntity employee)
    {
        TableResult result = await employeeTable
            .ExecuteAsync(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
    }  

Dans cet exemple asynchrone, vous pouvez voir les modifications suivantes à partir de la version synchrone :  

-   Maintenant, la signature de méthode inclut le modificateur **asynchrone** et retourne une instance de **tâche** .  
-   Au lieu d’appeler la méthode **Execute** pour mettre à jour l’entité, la méthode appelle la méthode **ExecuteAsync** et utilise le modificateur **await** pour récupérer les résultats de façon asynchrone.  

L’application cliente peut appeler plusieurs méthodes asynchrones telles que celle-ci, et chaque appel de la méthode s’exécute sur un thread séparé.  


### <a name="credits"></a>Crédits
Nous aimerions remercier les membres de l’équipe Azure pour leurs contributions suivants : Dominic Betts, Jason Hogg, Jean Ghanem, Jai Haridas, Jeff Irwin, Vamshidhar Kommineni, Vinay Shah et Serdar Ozler ainsi que Tom Hollander de DX de Microsoft. 

Nous aimerions également remercier le suivant Microsoft MVP pour leurs précieux commentaires au cours des cycles de révision : Igor Papirov et Edward Bakker.



[1]: ./media/storage-table-design-guide/storage-table-design-IMAGE01.png
[2]: ./media/storage-table-design-guide/storage-table-design-IMAGE02.png
[3]: ./media/storage-table-design-guide/storage-table-design-IMAGE03.png
[4]: ./media/storage-table-design-guide/storage-table-design-IMAGE04.png
[5]: ./media/storage-table-design-guide/storage-table-design-IMAGE05.png
[6]: ./media/storage-table-design-guide/storage-table-design-IMAGE06.png
[7]: ./media/storage-table-design-guide/storage-table-design-IMAGE07.png
[8]: ./media/storage-table-design-guide/storage-table-design-IMAGE08.png
[9]: ./media/storage-table-design-guide/storage-table-design-IMAGE09.png
[10]: ./media/storage-table-design-guide/storage-table-design-IMAGE10.png
[11]: ./media/storage-table-design-guide/storage-table-design-IMAGE11.png
[12]: ./media/storage-table-design-guide/storage-table-design-IMAGE12.png
[13]: ./media/storage-table-design-guide/storage-table-design-IMAGE13.png
[14]: ./media/storage-table-design-guide/storage-table-design-IMAGE14.png
[15]: ./media/storage-table-design-guide/storage-table-design-IMAGE15.png
[16]: ./media/storage-table-design-guide/storage-table-design-IMAGE16.png
[17]: ./media/storage-table-design-guide/storage-table-design-IMAGE17.png
[18]: ./media/storage-table-design-guide/storage-table-design-IMAGE18.png
[19]: ./media/storage-table-design-guide/storage-table-design-IMAGE19.png
[20]: ./media/storage-table-design-guide/storage-table-design-IMAGE20.png
[21]: ./media/storage-table-design-guide/storage-table-design-IMAGE21.png
[22]: ./media/storage-table-design-guide/storage-table-design-IMAGE22.png
[23]: ./media/storage-table-design-guide/storage-table-design-IMAGE23.png
[24]: ./media/storage-table-design-guide/storage-table-design-IMAGE24.png
[25]: ./media/storage-table-design-guide/storage-table-design-IMAGE25.png
[26]: ./media/storage-table-design-guide/storage-table-design-IMAGE26.png
[27]: ./media/storage-table-design-guide/storage-table-design-IMAGE27.png
[28]: ./media/storage-table-design-guide/storage-table-design-IMAGE28.png
[29]: ./media/storage-table-design-guide/storage-table-design-IMAGE29.png
 
