<properties
    pageTitle="Optimisez votre environnement avec la solution d’évaluation de SQL dans le journal Analytique | Microsoft Azure"
    description="Vous pouvez utiliser la solution d’évaluation de SQL afin d’évaluer les risques et la santé de vos environnements serveur à intervalles réguliers."
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
    ms.date="08/11/2016"
    ms.author="banders"/>

# <a name="optimize-your-environment-with-the-sql-assessment-solution-in-log-analytics"></a>Optimisez votre environnement avec la solution d’évaluation de SQL dans le journal Analytique


Vous pouvez utiliser la solution d’évaluation de SQL afin d’évaluer les risques et la santé de vos environnements serveur à intervalles réguliers. Cet article vous aidera à installer la solution de sorte que vous pouvez prendre des mesures correctives pour les problèmes potentiels.

Cette solution fournit une liste hiérarchisée des recommandations spécifiques pour votre infrastructure de serveur déployé. Les recommandations sont classées dans six domaines qui vous permettent de rapidement comprendre le risque et l’action corrective.

Les recommandations sont basées sur les connaissances et l’expérience acquise par les ingénieurs de Microsoft de milliers de visites de clients. Chaque recommandation fournit des recommandations sur la raison pour laquelle un problème peut avoir une importance pour vous et comment implémenter les modifications suggérées.

Vous pouvez choisir les domaines les plus importants pour votre organisation et suivre la progression vers un environnement libre et sain en cours d’exécution.

Une fois que vous avez ajouté à la solution et une évaluation est terminée, le résumé des informations pour les domaines sont affichées sur le tableau de bord **SQL évaluation** de l’infrastructure de votre environnement. Les sections suivantes décrivent comment utiliser les informations du tableau de bord **SQL évaluation** , dans laquelle vous pouvez afficher et puis entreprendre des actions recommandées pour votre infrastructure de serveur SQL.

![image de mosaïque d’évaluation de SQL](./media/log-analytics-sql-assessment/sql-assess-tile.png)

![image du tableau de bord d’évaluation de SQL](./media/log-analytics-sql-assessment/sql-assess-dash.png)

## <a name="installing-and-configuring-the-solution"></a>L’installation et la configuration de la solution
Évaluation de SQL fonctionne avec toutes les versions actuellement prises en charge de SQL Server pour les éditions Enterprise, Developer et Standard.

Utilisez les informations suivantes pour installer et configurer la solution.

- Agents doivent être installés sur les serveurs sur lesquels SQL Server installé.
- La solution d’évaluation de SQL requiert.NET Framework 4 est installé sur chaque ordinateur qui dispose d’un agent de l’OMS.
- Lors de l’utilisation de l’agent Operations Manager avec l’évaluation de SQL, vous devez utiliser un compte Operations Manager Run-As. Pour plus d’informations, consultez [opérations d’exécuter en tant que comptes du gestionnaire pour OMS](#operations-manager-run-as-accounts-for-oms) ci-dessous.

    >[AZURE.NOTE] L’agent MMA ne gère pas les comptes d’Operations Manager Run-As.

- Ajouter la solution d’évaluation de SQL à votre espace de travail de l’OMS à l’aide de la procédure décrite dans les [solutions d’Analytique de journal ajouter à partir de la galerie de Solutions](log-analytics-add-solutions.md). Il n’y a aucune configuration supplémentaire requise.

>[AZURE.NOTE] Après que vous avez ajouté la solution, le fichier AdvisorAssessment.exe est ajouté aux serveurs avec des agents. Données de configuration sont lues et ensuite envoyées au service OMS dans le nuage pour le traitement. La logique est appliquée pour les données reçues et le service en nuage enregistre les données.

## <a name="sql-assessment-data-collection-details"></a>Détails de collecte de données de SQL évaluation

Évaluation de SQL collecte les données WMI, les données du Registre, les données de performance et de SQL Server Gestion dynamique Afficher les résultats à l’aide d’agents que vous avez activés.

Le tableau suivant indique les méthodes de collecte de données pour les agents, Operations Manager (SCOM) est nécessaire et la façon dont souvent les données sont collectées par un agent.

| plate-forme | Agent direct | Agent SCOM | Stockage Azure | SCOM requis ? | Données de l’agent SCOM envoyées par groupe d’administration | fréquence de collection |
|---|---|---|---|---|---|---|
|Windows|![Oui](./media/log-analytics-sql-assessment/oms-bullet-green.png)|![Oui](./media/log-analytics-sql-assessment/oms-bullet-green.png)|![N°](./media/log-analytics-sql-assessment/oms-bullet-red.png)|    ![N°](./media/log-analytics-sql-assessment/oms-bullet-red.png)|![Oui](./media/log-analytics-sql-assessment/oms-bullet-green.png)|   7 derniers jours|

## <a name="operations-manager-run-as-accounts-for-oms"></a>Opérations exécuter en tant que comptes du gestionnaire pour OMS

Connectez-vous Analytique OMS utilise le groupe de gestion et de l’agent Operations Manager pour collecter et envoyer des données au service OMS. Builds OMS sur packs d’administration pour les charges de travail fournir des services à valeur ajoutée. Chaque charge de travail nécessite des privilèges spécifiques à la charge de travail pour exécuter les packs d’administration dans un contexte de sécurité différent, tel qu’un compte de domaine. Vous devez fournir des informations d’identification en configurant un compte Operations Manager exécuter en tant que.

Utilisez les informations suivantes pour définir le compte d’exécuter en tant que gestionnaire d’opérations pour l’évaluation de SQL.

### <a name="set-the-run-as-account-for-sql-assessment"></a>Définir le compte Exécuter en tant que pour l’évaluation de SQL

 Si vous utilisez déjà le pack d’administration de SQL Server, vous devez utiliser Exécuter en tant que compte.

#### <a name="to-configure-the-sql-run-as-account-in-the-operations-console"></a>Pour configurer le compte SQL exécuter en tant que dans la console d’opérations

>[AZURE.NOTE] Si vous utilisez l’agent direct OMS, plutôt que l’agent SCOM, le pack d’administration s’exécute toujours dans le contexte de sécurité du compte système Local. Ignorez les étapes 1 à 5 ci-dessous et exécutez le T-SQL ou un échantillon de Powershell, spécifiant le nom d’utilisateur NT AUTHORITY\SYSTEM.

1. Dans le Gestionnaire d’opérations, ouvrez la console d’opérations, puis cliquez sur **Administration**.

2. Sous **Configuration d’exécution**, cliquez sur **profils**et ouvrez **OMS SQL s’exécutent en tant que profil d’évaluation**.

3. Sur la page à **S’exécuter en tant que comptes** , cliquez sur **Ajouter**.

4. Permet de sélectionner un compte Windows exécuter en tant que qui contient les informations d’identification nécessaires pour SQL Server, ou cliquez sur **Nouveau** pour en créer un.
    >[AZURE.NOTE] Le type de compte Exécuter en tant que doit être Windows. Le compte Exécuter en tant que doit également faire partie du groupe Administrateurs Local sur tous les serveurs Windows hébergeant les Instances de SQL Server.

5. Cliquez sur **Enregistrer**.

6. Modifier et exécuter l’exemple de T-SQL suivant sur chaque Instance de SQL Server pour accorder les autorisations minimales requises pour exécuter en tant que compte pour effectuer l’évaluation de SQL. Toutefois, vous n’avez pas besoin pour ce faire, si un compte Exécuter en tant que fait déjà partie du rôle de serveur sysadmin sur les Instances de SQL Server.

```
---
    -- Replace <UserName> with the actual user name being used as Run As Account.
    USE master

    -- Create login for the user, comment this line if login is already created.
    CREATE LOGIN [<UserName>] FROM WINDOWS

    -- Grant permissions to user.
    GRANT VIEW SERVER STATE TO [<UserName>]
    GRANT VIEW ANY DEFINITION TO [<UserName>]
    GRANT VIEW ANY DATABASE TO [<UserName>]

    -- Add database user for all the databases on SQL Server Instance, this is required for connecting to individual databases.
    -- NOTE: This command must be run anytime new databases are added to SQL Server instances.
    EXEC sp_msforeachdb N'USE [?]; CREATE USER [<UserName>] FOR LOGIN [<UserName>];'

```
#### <a name="to-configure-the-sql-run-as-account-using-windows-powershell"></a>Pour configurer le compte SQL exécuter en tant qu’à l’aide de Windows PowerShell

Ouvrez une fenêtre PowerShell et exécutez le script suivant une fois que vous avez la mise à jour avec vos informations :

```

    import-module OperationsManager
    New-SCOMManagementGroupConnection "<your management group name>"
     
    $profile = Get-SCOMRunAsProfile -DisplayName "OMS SQL Assessment Run As Profile"
    $account = Get-SCOMrunAsAccount | Where-Object {$_.Name -eq "<your run as account name>"}
    Set-SCOMRunAsProfile -Action "Add" -Profile $Profile -Account $Account
```

## <a name="understanding-how-recommendations-are-prioritized"></a>Compréhension des priorités des recommandations

Une valeur de pondération qui identifie l’importance relative de la recommandation est attribuée à chaque recommandation formulée. Uniquement les dix recommandations plus importantes sont affichées.

### <a name="how-weights-are-calculated"></a>Mode de calcul des pondérations

Pondérations sont des valeurs de regroupement en fonction des trois facteurs clés :

- La *probabilité* qu’un problème identifié provoque des problèmes. Une plus forte probabilité équivaut à un score global supérieur de la recommandation.

- L' *impact* du problème sur votre organisation, si elle n’entraîne pas un problème. Un impact plus important équivaut à un score global supérieur de la recommandation.

- L' *effort* requis pour mettre en oeuvre la recommandation. Un effort supérieur équivaut à un plus petit score global de la recommandation.

La pondération de chaque recommandation est exprimée en pourcentage de la note totale pour chaque zone de focus. Par exemple, si une recommandation dans la zone de focus de sécurité et de conformité a un score de 5 %, mise en oeuvre de cette recommandation augmentera votre globale score à 5 % de sécurité et de conformité.

### <a name="focus-areas"></a>Domaines

**Sécurité et conformité** - ce sujet affiche des recommandations pour les menaces de sécurité potentielles et les violations, les stratégies d’entreprise et les exigences de conformité réglementaires, techniques et juridiques.

**Disponibilité et continuité d’activité** - ce sujet affiche des recommandations pour la disponibilité du service, de la souplesse de votre infrastructure et de protection de l’entreprise.

**Performances et évolutivité** - ce sujet affiche des recommandations pour aider votre organisation à infrastructure informatique augmentent, assurez-vous que votre environnement informatique répond aux exigences de performances actuelles et qu’il est capable de répondre à l’évolution des besoins de l’infrastructure.

**Mise à niveau, déploiement et Migration** - ce sujet présente des recommandations pour vous aider à mettre à jour, migrer et déployer SQL Server à votre infrastructure existante.

**Opérations et analyse** - ce sujet présente des recommandations pour aider à rationaliser vos opérations informatiques, mettre en œuvre la maintenance préventive et optimiser les performances.

**Modification et gestion de la Configuration** - ce sujet affiche des recommandations pour aider à protéger les opérations quotidiennes, de s’assurer que les modifications ne négatif affectent votre infrastructure, établir des procédures de contrôle des modifications et de suivre et d’auditer des configurations système.

### <a name="should-you-aim-to-score-100-in-every-focus-area"></a>Devraient viser à score de 100 % dans chaque zone de focus ?

Pas nécessairement. Les recommandations sont basées sur les connaissances et l’expérience acquise par les ingénieurs de Microsoft sur des milliers de visites de clients. Toutefois, aucune des infrastructures deux serveurs ne sont les mêmes, et des recommandations spécifiques peuvent être plus ou moins pertinentes pour vous. Par exemple, certaines recommandations de sécurité peuvent être moins pertinentes si vos ordinateurs virtuels ne sont pas exposés à Internet. Des recommandations de disponibilité peuvent être moins pertinentes pour les services qui fournissent une collection de données ad hoc de faible priorité et génération de rapports. Problèmes qui sont importants pour une entreprise peuvent être moins importantes d’une start-up. Vous souhaiterez peut-être identifier les domaines sont vos priorités et observez comment modifier de vos résultats au fil du temps.

Chaque recommandation inclut des instructions sur la raison pour laquelle il est important. Ce guide vous permet de déterminer si la mise en oeuvre de la recommandation est adaptée à votre cas, étant donné la nature de vos services informatiques et aux besoins de votre organisation.

## <a name="use-assessment-focus-area-recommendations"></a>Utiliser les recommandations de zone évaluation focus

Avant de pouvoir utiliser une solution d’évaluation de l’OMS, vous devez disposer de la solution est installée. Pour en savoir plus sur l’installation des solutions, reportez-vous à la section [Analytique de journal ajouter des solutions à partir de la galerie de Solutions](log-analytics-add-solutions.md). Après que l’avoir installé, vous pouvez afficher le résumé des recommandations à l’aide de la mosaïque de l’évaluation de SQL sur la page de vue d’ensemble de l’OMS.

Afficher les évaluations de la synthèse de conformité pour votre infrastructure et puis globaliser recommandations.

### <a name="to-view-recommendations-for-a-focus-area-and-take-corrective-action"></a>Pour afficher des recommandations pour une zone de focus et action corrective

1. Dans la page **vue d’ensemble** , cliquez sur la mosaïque de **l’Évaluation de SQL** .
2. Sur la page **d’Évaluation de SQL** , consultez les informations résumées de l’une des lames de zone du focus et puis cliquez sur une option pour afficher les recommandations de cette zone de focus.
3. Sur toutes les pages de la zone de focus, vous pouvez afficher les recommandations par ordre de priorité pour votre environnement. Cliquez sur une recommandation sous **Objets affectés** pour afficher plus d’informations sur la raison pour laquelle la recommandation a lieu.  
    ![image de recommandations de l’évaluation de SQL](./media/log-analytics-sql-assessment/sql-assess-focus.png)
4. Vous pouvez prendre des mesures correctives suggérées dans **Les Actions suggérées**. Lorsque l’élément a été adressé, les évaluations ultérieures enregistrera recommandé des actions entreprises et augmente votre score de conformité. Les éléments corrigés apparaissent comme **Des objets de passé**.

## <a name="ignore-recommendations"></a>Ignorer les recommandations

Si vous avez des recommandations que vous souhaitez ignorer, vous pouvez créer un fichier texte par OMS pour empêcher les recommandations d’apparaître dans les résultats de l’évaluation.

### <a name="to-identify-recommendations-that-you-will-ignore"></a>Pour identifier des recommandations qui vous ignore

1.  Vous connecter à votre espace de travail et ouvrir de recherche des journaux. Utilisez la requête suivante pour les recommandations de liste qui ont échoué pour les ordinateurs de votre environnement.

    ```
    Type=SQLAssessmentRecommendation RecommendationResult=Failed | select  Computer, RecommendationId, Recommendation | sort  Computer
    ```

    Voici une capture d’écran montrant la requête de recherche de journal : ![Échec de recommandations](./media/log-analytics-sql-assessment/sql-assess-failed-recommendations.png)

2.  Choisissez les recommandations que vous souhaitez ignorer. Vous allez utiliser les valeurs de RecommendationId dans la procédure suivante.


### <a name="to-create-and-use-an-ignorerecommendationstxt-text-file"></a>Pour créer et utiliser un fichier de texte IgnoreRecommendations.txt

1.  Créez un fichier nommé IgnoreRecommendations.txt.
2.  Collez ou tapez chaque RecommendationId pour chaque recommandation que vous souhaitez OMS pour ignorer sur une ligne distincte, puis enregistrez et fermez le fichier.
3.  Placez le fichier dans le dossier suivant sur chaque ordinateur où vous souhaitez OMS pour ignorer les recommandations.
    - Sur les ordinateurs avec la surveillance de l’Agent Microsoft (connecté directement ou par l’intermédiaire de Operations Manager) - *lecteur_système*: \Program Files\Microsoft Agent\Agent de surveillance
    - Sur le serveur d’administration Operations Manager - *lecteur_système*: \Program Files\Microsoft Manager\Server R2\Operations de System Center 2012

### <a name="to-verify-that-recommendations-are-ignored"></a>Pour vérifier que les recommandations sont ignorées.

1.  Après que la prochaine planifiée évaluation, par défaut tous les 7 jours, les recommandations spécifiées sont marquées ignorés et n’apparaissent pas sur le tableau de bord d’évaluation.
2.  Vous pouvez utiliser les requêtes de recherche de journal suivantes pour répertorier toutes les recommandations ignorées.

    ```
    Type=SQLAssessmentRecommendation RecommendationResult=Ignored | select  Computer, RecommendationId, Recommendation | sort  Computer
    ```
3.  Si vous décidez ultérieurement que vous souhaitez voir Recommandations ignorées, supprimez tous les fichiers IgnoreRecommendations.txt, ou vous pouvez supprimer les RecommendationIDs.

## <a name="sql-assessment-solution-faq"></a>Solution d’évaluation de SQL Forum aux questions

*La fréquence à laquelle une évaluation s’exécute-t-il ?*
- L’évaluation s’exécute tous les 7 jours.

*Existe-t-il un moyen pour configurer la fréquence à laquelle l’évaluation s’exécute ?*
- Pas pour l’instant.

*Si un autre serveur pour est découverte une fois que j’ai ajouté à la solution d’évaluation de SQL, il examinera ?*
- Oui, une fois qu’il est découvert qu’il est évalué à partir de là, tous les 7 jours.

*Si un serveur est mis hors service, lorsqu’il disparaissent de l’évaluation ?*
- Si un serveur ne présente pas les données pour les 3 semaines, il est supprimé.

*Quel est le nom du processus qui effectue la collecte de données ?*
- AdvisorAssessment.exe

*Combien de temps faut-il pour collecter les données ?*
- La collection de données sur le serveur prend environ 1 heure. Il peut prendre plus de temps sur les serveurs qui possèdent un grand nombre de bases de données ou des instances SQL.

*Le type de données est recueilli ?*
- Les types de données suivantes sont collectées :
    - WMI
    - Registre
    - Compteurs de performance
    - Vues de gestion dynamique SQL (DMV).

*Y a-t-il un moyen de configurer les données collectées ?*
- Pas pour l’instant.

*Pourquoi dois-je configurer un compte Exécuter en tant que ?*
- Pour SQL Server, un petit nombre de requêtes SQL est exécuté. Dans l’ordre de leur exécution, une exécution en tant que compte disposant d’autorisations de VIEW SERVER STATE sur SQL doit être utilisé.  En outre, pour interroger WMI, les informations d’identification de l’administrateur local sont requises.

*Pourquoi afficher uniquement les recommandations du top 10 ?*
- Au lieu de vous donner une liste exhaustive écrasante de tâches, nous vous recommandons que vous concentrer tout d’abord sur les recommandations hiérarchisées. Une fois que vous y remédier, des recommandations supplémentaires seront disponibles. Si vous préférez afficher la liste détaillée, vous pouvez afficher toutes les recommandations à l’aide de la recherche du journal OMS.

*Existe-t-il un moyen d’ignorer une recommandation ?*
- Oui, consultez [recommandations relatives à ignorer](#ignore-recommendations) ci-dessus.



## <a name="next-steps"></a>Étapes suivantes

- [Journaux de recherche](log-analytics-log-searches.md) pour afficher les données d’évaluation de SQL détaillées et des recommandations.
