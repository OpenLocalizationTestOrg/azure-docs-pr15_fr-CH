<properties
    pageTitle="Se connecter à la base de données de SQL à l’aide de Ruby | Microsoft Azure"
    description="Donnez un exemple de code Ruby que vous pouvez exécuter pour vous connecter à la base de données de SQL Azure."
    services="sql-database"
    documentationCenter=""
    authors="ajlam"
    manager="jhubbard"
    editor=""/>


<tags
    ms.service="sql-database"
    ms.workload="drivers"
    ms.tgt_pltfrm="na"
    ms.devlang="ruby"
    ms.topic="article"
    ms.date="10/03/2016"
    ms.author="andrela"/>


# <a name="connect-to-sql-database-by-using-ruby"></a>Se connecter à la base de données de SQL à l’aide de Ruby 

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)] 

Cette rubrique indique comment se connecter et d’interroger une base de données de SQL Azure à l’aide de Ruby. Vous pouvez exécuter cet exemple à partir de la plate-forme Windows, Ubuntu Linux ou Mac.

## <a name="step-1-configure-development-environment"></a>Étape 1 : Configurer l’environnement de développement

[Composants requis pour utiliser le pilote de Ruby TinyTDS pour SQL Server](https://msdn.microsoft.com/library/mt711041.aspx)

## <a name="step-2-create-a-sql-database"></a>Étape 2 : Créer une base de données SQL

Consultez la [page de mise en route](sql-database-get-started.md) pour savoir comment créer une base de données exemple.  Il est important que vous suivez le guide pour créer un **modèle de base de données AdventureWorks**. Les exemples ci-dessous fonctionnent uniquement avec le **schéma AdventureWorks**.

## <a name="step-3-get-connection-details"></a>Étape 3 : Obtenir les détails de la connexion

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## <a name="step-4-run-sample-code"></a>Étape 4 : Exécution des exemples de code

[Preuve de Concept de connexion à SQL à l’aide de Ruby](http://msdn.microsoft.com/library/mt715797.aspx)

## <a name="next-steps"></a>Étapes suivantes

* Consultez [vue d’ensemble du développement de base de données SQL](sql-database-develop-overview.md)
* Plus d’informations sur le [Pilote de Ruby de Microsoft pour SQL Server](https://msdn.microsoft.com/library/mt691981.aspx)

## <a name="additional-resources"></a>Ressources supplémentaires 

* [Modèles de design pour les Applications de base de données SQL Azure mutualisée SaaS pour](sql-database-design-patterns-multi-tenancy-saas-applications.md)
* Découvrez toutes les [fonctionnalités de base de données de SQL](https://azure.microsoft.com/services/sql-database/)
