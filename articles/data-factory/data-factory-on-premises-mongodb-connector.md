<properties 
    pageTitle="Déplacer les données de MongoDB à l’aide de Data Factory | Microsoft Azure" 
    description="Découvrez comment déplacer des données de base MongoDB Azure Data Factory." 
    services="data-factory" 
    documentationCenter="" 
    authors="linda33wj" 
    manager="jhubbard" 
    editor="monicar"/>

<tags 
    ms.service="data-factory" 
    ms.workload="data-services" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.date="08/04/2016" 
    ms.author="jingwang"/>

# <a name="move-data-from-mongodb-using-azure-data-factory"></a>Déplacement des données à l’aide d’Azure Data Factory de MongoDB

Cet article décrit comment vous pouvez utiliser l’activité de copie dans une usine de données Azure pour déplacer des données à partir d’une base de données de MongoDB sur site vers un autre magasin de données. Cet article s’appuie sur l’article [d’activités de déplacement des données](data-factory-data-movement-activities.md) , qui présente une vue d’ensemble de transfert de données avec l’activité de copie et les combinaisons de magasin de données de source/récepteur qui prend en charge de l’activité de copie.

Service de fabrique de données prend en charge la connexion à des sources de MongoDB sur site à l’aide de la passerelle de gestion des données. Voir [Passerelle de gestion des données](data-factory-data-management-gateway.md) des articles pour en savoir plus sur la passerelle de gestion des données et [déplacer les données du site vers le cloud](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions détaillées sur la configuration de la passerelle un pipeline de données pour déplacer les données. 

> [AZURE.NOTE] Vous devez utiliser la passerelle pour se connecter à MongoDB, même s’il est hébergé dans Azure IaaS VMs. Si vous essayez de vous connecter à une instance de MongoDB hébergé dans le nuage, vous pouvez également installer l’instance passerelle dans la VM IaaS.

Usine de données prend actuellement en charge uniquement déplacement des données à partir de MongoDB à d’autres magasins de données, mais ne pas pour le déplacement des données provenant d’autres banques de données de MongoDB.

## <a name="prerequisites"></a>Conditions préalables
Pour le service Factory de données Azure être en mesure de se connecter à votre base de données de MongoDB sur site, vous devez installer les composants suivants : 

- Passerelle de gestion des données 2.0 ou une version ultérieure sur le même ordinateur qui héberge la base de données ou sur un ordinateur distinct afin d’éviter la concurrence pour les ressources de la base de données. Passerelle de gestion des données est un logiciel qui connecte les sources de données sur site pour les services en nuage de façon sécurisée et gérée. Consultez l’article de [La passerelle de gestion des données](data-factory-data-management-gateway.md) pour plus d’informations sur la passerelle de gestion des données.
  
    Lorsque vous installez la passerelle, il installe automatiquement un pilote de MongoDB ODBC Microsoft utilisé pour se connecter à MongoDB. 

## <a name="copy-data-wizard"></a>Assistant de duplication des données
Pour créer un pipeline qui copie les données à partir d’une base de données de MongoDB dans toute les prise en charge de récepteurs banques de données le plus simple est d’utiliser l’Assistant de copie de données. Consultez [didacticiel : créer un tuyau à l’aide d’Assistant copie](data-factory-copy-data-wizard-tutorial.md) pour une procédure pas à pas rapides sur la création d’un tuyau à l’aide de l’Assistant de copie de données. 

L’exemple suivant fournit des exemples de définitions de JSON qui vous permet de créer un tuyau à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) [Azure portal](data-factory-copy-activity-tutorial-using-azure-portal.md) ou [PowerShell d’Azure](data-factory-copy-activity-tutorial-using-powershell.md). Ils illustrent le copier des données à partir de la base de données de MongoDB dans le stockage Blob Azure. Toutefois, les données peuvent être copiées à un des récepteurs indiquée [ici](data-factory-data-movement-activities.md#supported-data-stores) à l’aide de l’activité de copie dans Azure Data Factory.

## <a name="sample-copy-data-from-mongodb-to-azure-blob"></a>Exemple : Copier des données à partir de MongoDB dans Azure Blob
Cet exemple montre comment copier les données d’une base de données de MongoDB local à un stockage d’objets Blob Azure. Toutefois, les données peuvent être copiées **directement** à un des récepteurs indiquée [ici](data-factory-data-movement-activities.md#supported-data-stores) à l’aide de l’activité de copie dans Azure Data Factory.  
 
L’exemple possède les entités de fabrique de données suivantes :

1.  Un service lié du type [OnPremisesMongoDb](#linked-service-properties).
2.  Un service lié du type [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service-properties).
3.  Un [groupe de données](data-factory-create-datasets.md) d’entrée de type [MongoDbCollection](#dataset-type-properties).
4.  Une sortie [dataset](data-factory-create-datasets.md) de type [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties).
4.  Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [MongoDbSource](#copy-activity-type-properties) et [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties).

L’exemple copie les données à partir des résultats d’une requête de base de données de MongoDB dans un blob toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivantes les exemples. 

Dans un premier temps, le programme d’installation la passerelle de gestion des données selon les instructions de l’article de la [Passerelle de gestion des données](data-factory-data-management-gateway.md) . 

**Service de MongoDB lié**

    { 
        "name": "OnPremisesMongoDbLinkedService", 
        "properties": 
        { 
            "type": "OnPremisesMongoDb", 
            "typeProperties": 
            { 
                "authenticationType": "<Basic or Anonymous>", 
                "server": "< The IP address or host name of the MongoDB server >",  
                "port": "<The number of the TCP port that the MongoDB server uses to listen for client connections.>", 
                "username": "<username>", 
                "password": "<password>",
               "authSource": "< The database that you want to use to check your credentials for authentication. >",
                "databaseName": "<database name>",
                "gatewayName": "<mygateway>"
            } 
        }
    } 


**Service de stockage lié Azure**

    {
      "name": "AzureStorageLinkedService",
      "properties": {
        "type": "AzureStorage",
        "typeProperties": {
          "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
        }
      }
    }

**Jeu de données d’entrée MongoDB** Définition de « externe » : « true » informe le service Factory de données que la table est externe à l’usine de données et qu’il n’est pas produite par une activité sur le factory de données.
    
    {
         "name":  "MongoDbInputDataset",
        "properties": { 
            "type": "MongoDbCollection", 
            "linkedServiceName": "OnPremisesMongoDbLinkedService", 
            "typeProperties": { 
                "collectionName": "<Collection name>"   
            }, 
            "availability": {
                "frequency": "Hour",
                "interval": 1
            },
            "external": true
        } 
    }



**Dataset de sortie Blob Azure**

Les données sont écrites dans un blob de nouveau toutes les heures (fréquence : heures, l’intervalle : 1). Le chemin d’accès du dossier pour le blob est calculée de manière dynamique en fonction de l’heure de début de la tranche qui est en cours de traitement. Le chemin d’accès du dossier utilise l’année, mois, jour et parties d’heures de l’heure de début.

    {
        "name": "AzureBlobOutputDataSet",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "mycontainer/frommongodb/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
                "format": {
                    "type": "TextFormat",
                    "rowDelimiter": "\n",
                    "columnDelimiter": "\t"
                },
                "partitionedBy": [
                    {
                        "name": "Year",
                        "value": {
                            "type": "DateTime",
                            "date": "SliceStart",
                            "format": "yyyy"
                        }
                    },
                    {
                        "name": "Month",
                        "value": {
                            "type": "DateTime",
                            "date": "SliceStart",
                            "format": "%M"
                        }
                    },
                    {
                        "name": "Day",
                        "value": {
                            "type": "DateTime",
                            "date": "SliceStart",
                            "format": "%d"
                        }
                    },
                    {
                        "name": "Hour",
                        "value": {
                            "type": "DateTime",
                            "date": "SliceStart",
                            "format": "%H"
                        }
                    }
                ]
            },
            "availability": {
                "frequency": "Hour",
                "interval": 1
            }
        }
    }



**Activité de copie de pipeline**

Le pipeline contient une activité de copie qui est configuré pour utiliser l’entrée ci-dessus et la sortie des groupes de données et est planifiée pour s’exécuter toutes les heures. Dans la définition de JSON de pipeline, le type de **source** est défini sur **MongoDbSource** et type de **récepteurs** est définie sur **BlobSink**. La requête SQL spécifiée pour la propriété de **requête** sélectionne les données de la dernière heure à copier.
    
    {
        "name": "CopyMongoDBToBlob",
        "properties": {
            "description": "pipeline for copy activity",
            "activities": [
                {
                    "type": "Copy",
                    "typeProperties": {
                        "source": {
                            "type": "MongoDbSource",
                            "query": "$$Text.Format('select * from  MyTable where LastModifiedDate >= {{ts\'{0:yyyy-MM-dd HH:mm:ss}\'}} AND LastModifiedDate < {{ts\'{1:yyyy-MM-dd HH:mm:ss}\'}}', WindowStart, WindowEnd)"
                        },
                        "sink": {
                            "type": "BlobSink",
                            "writeBatchSize": 0,
                            "writeBatchTimeout": "00:00:00"
                        }
                    },
                    "inputs": [
                        {
                            "name": "MongoDbInputDataset"
                        }
                    ],
                    "outputs": [
                        {
                            "name": "AzureBlobOutputDataSet"
                        }
                    ],
                    "policy": {
                        "timeout": "01:00:00",
                        "concurrency": 1
                    },
                    "scheduler": {
                        "frequency": "Hour",
                        "interval": 1
                    },
                    "name": "MongoDBToAzureBlob"
                }
            ],
            "start": "2016-06-01T18:00:00Z",
            "end": "2016-06-01T19:00:00Z"
        }
    }


## <a name="linked-service-properties"></a>Propriétés du service lié

Le tableau suivant fournit la description pour les éléments JSON spécifiques au service **OnPremisesMongoDB** lié.

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- | 
| type de | La propriété type doit être définie : **OnPremisesMongoDb** | Oui |
| serveur | Nom hôte ou adresse IP du serveur MongoDB. | Oui | 
| port | Port TCP utilisé par le serveur de MongoDB à l’écoute des connexions client. | Les valeur par défaut facultative : 27017 |
| authenticationType | Base ou anonyme. | Oui | 
| nom d’utilisateur | Compte d’utilisateur pour accéder aux MongoDB. | Oui (si l’authentification de base est utilisée).|
| mot de passe | Mot de passe de l’utilisateur. |   Oui (si l’authentification de base est utilisée). | 
| authSource | Nom de la base de données MongoDB que vous souhaitez utiliser pour vérifier vos informations d’identification pour l’authentification. | Facultatif (si l’authentification de base est utilisée). par défaut : utilise le compte de l’administration et la base de données spécifiée à l’aide de la propriété databaseName. |  
| databaseName | Nom de la base de données MongoDB auquel vous souhaitez accéder. | Oui |
| nom de la passerelle | Nom de la passerelle qui accède à la banque de données. | Oui | 
| encryptedCredential | Informations d’identification chiffrées par passerelle. | Facultatif |


Pour plus d’informations sur la définition des informations d’identification pour une source de données sur site MongoDB, reportez-vous à la section [informations d’identification de la configuration et de sécurité](data-factory-move-data-between-onprem-and-cloud.md#set-credentials-and-security) .

## <a name="dataset-type-properties"></a>Propriétés du type de groupe de données

Pour obtenir une liste complète des sections et les propriétés disponibles pour la définition des groupes de données, consultez l’article de la [Création de groupes de données](data-factory-create-datasets.md) . Sections de structure, de disponibilité et de stratégie d’un groupe de données JSON sont similaires pour tous les types de groupe de données (SQL Azure, blob Azure, table Azure, etc..).

La section **typeProperties** est différente pour chaque type de groupe de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le groupe de données de type **MongoDbCollection** possède les propriétés suivantes :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| collectionName | Nom de la collection dans la base de données de MongoDB. | Oui | 

## <a name="copy-activity-type-properties"></a>Copier les propriétés de type d’activité

Pour obtenir une liste complète des sections et les propriétés disponibles pour la définition des activités, voir l’article [Créer des Pipelines](data-factory-create-pipelines.md) . Les propriétés telles que nom, description, d’entrée et de sortie des tables et la stratégie sont disponibles pour tous les types d’activités. 

D’autre part, les propriétés disponibles dans la section **typeProperties** de l’activité varient avec chaque type d’activité. Pour les activités de copie, qui varient selon les types de sources et les destinations.

Lorsque la source est de type **MongoDbSource** les propriétés suivantes sont disponibles dans la section de typeProperties :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| -------- | ----------- | -------------- | -------- |
| requête | La requête personnalisée permet de lire les données. | Chaîne de requête de SQL-92. Par exemple : sélectionnez * from MyTable. | Non (si **collectionName** du **groupe de données** est spécifié) | 

## <a name="schema-by-data-factory"></a>Schéma en usine de données
Service de fabrique de données Azure déduit le schéma à partir d’une collection de MongoDB dans la collection à l’aide des 100 derniers documents. Si ces 100 documents ne contiennent pas de schéma complet, certaines colonnes peuvent être ignorés lors de l’opération de copie. 

## <a name="type-mapping-for-mongodb"></a>Mappage de type de MongoDB

Comme mentionné dans l’article [d’activités de déplacement des données](data-factory-data-movement-activities.md) , activité de copie effectue les conversions de type automatique à partir de types de sources pour recevoir des types avec l’approche 2-étape suivante :

1. Convertir les types de source native type .NET
2. Convertir à partir du type .NET pour type de récepteurs natif

Lors du déplacement des données vers MongoDB les mappages suivants servent à partir des types de MongoDB types .NET.

| Type de MongoDB | Type de.NET Framework |
| ------------------- | ------------------- | 
| Binaire | Byte] |
| Valeur booléenne | Valeur booléenne |
| Date | DateTime |
| NumberDouble | Double |
| NumberInt | Int32 |
| NumberLong | Int64 |
| ObjectID | Chaîne |
| Chaîne | Chaîne |
| UUID | GUID | 
| Objet | Renormalisés en aplatir les colonnes avec « _ » comme séparateur imbriqué |

> [AZURE.NOTE]  
> Pour en savoir plus sur la prise en charge de tableaux à l’aide de tables virtuelles, reportez-vous à la section ci-dessous de [prise en charge pour les types complexes à l’aide de tables virtuelles](#support-for-complex-types-using-virtual-tables) . 

Actuellement, les types de données MongoDB suivants ne sont pas prises en charge : DBPointer, JavaScript, Max/Min de clé, une Expression régulière, symbole, Timestamp, non défini

## <a name="support-for-complex-types-using-virtual-tables"></a>Prise en charge pour les types complexes à l’aide de tables virtuelles
Azure Data Factory utilise un pilote ODBC intégré pour vous connecter à et de copier des données à partir de votre base de données MongoDB. Pour les types complexes, tels que des tableaux ou des objets avec des types différents dans les documents, le pilote normalise nouveau des données dans des tables virtuelles correspondants. En particulier, si une table contient des colonnes, le pilote génère les tables virtuelles suivantes :

-   Une **table de base**, qui contient les mêmes données que la table réelle, sauf pour les colonnes de type complexe. La table de base utilise le même nom que la table réelle qu’il représente.
-   Une **table virtuelle** pour chaque colonne de type complexe, qui développe les données imbriquées. Les tables virtuelles sont nommés en utilisant le nom de la table réelle, un séparateur « _ » et le nom de l’objet ou le tableau.

Tables virtuelles font référence aux données de la table réelle, l’activation du pilote accéder aux données dénormalisées. Sous Détails, reportez-vous à la section exemple. Vous pouvez accéder au contenu de tableaux de MongoDB en interrogeant et en joignant les tables virtuelles.

Vous pouvez utiliser l' [Assistant copie](data-factory-data-movement-activities.md#data-factory-copy-wizard) intuitivement afficher la liste des tables dans la base de données de MongoDB, y compris les tables virtuelles et afficher des données à l’intérieur. Vous pouvez également créer une requête dans l’Assistant de copie et valider pour voir le résultat.

### <a name="example"></a>Exemple

Par exemple, « ExampleTable » ci-dessous est une table de MongoDB qui a une colonne avec un tableau d’objets dans chaque cellule – factures et une colonne avec un tableau de types scalaires – contrôle d’accès. 

_ID | Nom du client | Factures | Niveau de service | Contrôle d’accès
--- | ------------- | -------- | ------------- | -------
1111 | ABC | [{invoice_id : « 123 », article : « grille-pain », le prix : remise « 456 », : « 0,2 »}, {invoice_id : « 124 », article : « four », le prix : « 1235 », la remise : « 0,2 »}] | Argent | [5,6]
2222 | XYZ | [{invoice_id : « 135 », article : « réfrigérateur », prix : remise « 12543, » : « 0,0 »}] | Gold | [1,2]

Le pilote génère plusieurs tables virtuelles pour représenter cette table unique. La première table virtuelle est la table de base nommée « ExampleTable », ci-dessous. La table de base contienne toutes les données de la table d’origine, mais les données dans les tableaux a été omises sont développées dans les tables virtuelles.

_ID | Nom du client | Niveau de service
--- | ------------- | -------------
1111 | ABC | Argent
2222 | XYZ | Gold

Les tableaux suivants présentent les tables virtuelles qui représentent les tableaux d’origine dans l’exemple. Ces tables contiennent les éléments suivants : 

- Une référence à la colonne de clé primaire d’origine correspondant à la ligne du tableau d’origine (via la colonne _id)
- Une indication de la position des données dans le tableau d’origine 
- Les données de développé pour chaque élément dans le tableau

Table « ExampleTable_Invoices » :

_ID | ExampleTable_Invoices_dim1_idx | invoice_id | élément | prix | Remise
--- | -------------- | ---------- | ---- | ----- | --------
1111 | 0 | 123 | grille-pain | 456 | 0,2
1111 | 1 | 124 | FOUR | 1235 | 0,2
2222 | 0 | 135 | RÉFRIGÉRATEUR | 12543 | 0.0

Table « ExampleTable_Ratings » :

_ID | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings
--- | ------------- | -------------
1111 | 0 | 5
1111 | 1 | 6
2222 | 0 | 1
2222 | 1 | 2

[AZURE.INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

[AZURE.INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>Performances et réglage  
Voir [copie activité performances & Tuning Guide](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés cet impact sur les performances de transfert de données (copie activité) dans Azure Data Factory et les différentes manières de l’optimiser.

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article de [déplacer des données entre des locaux et cloud](data-factory-move-data-between-onprem-and-cloud.md) pour obtenir des instructions étape par étape pour la création d’un pipeline de données qui déplace les données à partir d’une banque de données locale à une banque de données Azure. 

