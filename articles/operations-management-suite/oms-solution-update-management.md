<properties
    pageTitle="Mise à jour de la solution de gestion de l’OMS | Microsoft Azure"
    description="Cet article vise à vous aider à comprendre comment utiliser cette solution pour gérer les mises à jour pour les ordinateurs Windows et Linux."
    services="operations-management-suite"
    documentationCenter=""
    authors="MGoedtel"
    manager="jwhit"
    editor=""
    />
<tags
    ms.service="operations-management-suite"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="10/14/2016"
    ms.author="magoedte"/>

# <a name="update-management-solution-in-omsmediaoms-solution-update-managementupdate-management-solution-iconpng-update-management-solution-in-oms"></a>![Mise à jour de la Solution de gestion de l’OMS](./media/oms-solution-update-management/update-management-solution-icon.png) Mise à jour de la solution de gestion de l’OMS

La solution de gestion de la mise à jour de l’OMS vous permet de gérer les mises à jour pour les ordinateurs Windows et Linux.  Vous pouvez rapidement évaluer l’état des mises à jour disponibles sur tous les ordinateurs agents et lancer le processus d’installation des mises à jour requises pour les serveurs. 

## <a name="prerequisites"></a>Conditions préalables

-   Les agents Windows doivent être configurés pour communiquer avec un serveur Windows Server Update Services (WSUS) ou avoir accès à Microsoft Update.  

    >[AZURE.NOTE] L’agent Windows ne peuvent pas être géré simultanément par System Center Configuration Manager.  
  
-   Agents de Linux doivent avoir accès à un référentiel de mise à jour.  L’Agent de l’OMS pour Linux peut être téléchargé à partir de [GitHub](https://github.com/microsoft/oms-agent-for-linux). 

## <a name="configuration"></a>Configuration de

Procédez comme suit pour ajouter la solution de gestion de la mise à jour de votre espace de travail de l’OMS et ajouter des agents de Linux.  Les agents Windows sont ajoutés automatiquement sans aucune configuration supplémentaire.

1.  Ajouter la solution de gestion de la mise à jour de votre espace de travail de l’OMS à l’aide de la procédure décrite dans les [solutions de l’OMS d’ajouter](../log-analytics/log-analytics-add-solutions.md) à partir de la galerie de Solutions.  
2.  Dans le portail de l’OMS, sélectionnez **paramètres** , puis sur **Sources connectées**.  Notez l' **ID de l’espace de travail** et soit la **clé primaire** ou **secondaire**.
3.  Effectuez les étapes suivantes pour chaque ordinateur de Linux.

    une barre d’outils.  Installez la dernière version de l’Agent de l’OMS pour Linux en exécutant les commandes suivantes.  Remplacer <Workspace ID> avec l’ID de l’espace de travail et <Key> avec la clé primaire ou secondaire.

        cd ~
        wget https://github.com/Microsoft/OMS-Agent-for-Linux/releases/download/v1.2.0-75/omsagent-1.2.0-75.universal.x64.sh
        sudo bash omsagent-1.2.0-75.universal.x64.sh --upgrade -w <Workspace ID> -s <Key>

     b. Pour supprimer l’agent, exécutez la commande suivante.

        sudo bash omsagent-1.2.0-75.universal.x64.sh --purge

## <a name="management-packs"></a>Packs d’administration

Si votre groupe d’administration System Center Operations Manager est connecté à votre espace de travail de l’OMS, les packs d’administration suivant seront installés dans Operations Manager lorsque vous ajoutez cette solution. Il n’y a aucune configuration ou la maintenance de ces packs d’administration requis. 

-   Microsoft System Center Advisor mise à jour évaluation Intelligence Pack (Microsoft.IntelligencePacks.UpdateAssessment)
-   Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
-   Mettre à jour le déploiement MP

Pour plus d’informations sur la manière dont les packs d’administration solution sont mis à jour, voir [Se connecter Operations Manager pour l’Analytique de journal](../log-analytics/log-analytics-om-agents.md).

## <a name="data-collection"></a>Collecte de données

### <a name="supported-agents"></a>Prise en charge des agents

Le tableau suivant décrit les sources connectées sont pris en charge par cette solution.

Source connecté | Prise en charge | Description|
----------|----------|----------|
Agents Windows | Oui | La solution rassemble des informations sur les mises à jour du système à partir d’agents de Windows et lance l’installation de mises à jour requises.|
Agents de Linux | Oui | La solution rassemble des informations sur les mises à jour du système à partir d’agents de Linux.|
Groupe d’administration Operations Manager | Oui | La solution rassemble des informations sur les mises à jour du système à partir des agents dans un groupe d’administration connecté.<br>Une connexion directe à partir de l’agent Operations Manager pour l’Analytique de journal n’est pas nécessaire. Le référentiel de l’OMS, les données sont transférées à partir du groupe d’administration.|
Compte de stockage Azure | N° | Stockage Azure n’inclut pas d’informations sur les mises à jour du système.|  

### <a name="collection-frequency"></a>Fréquence de collection

Pour chaque ordinateur Windows géré, une analyse est effectuée deux fois par jour.  Une mise à jour est installée, ses informations sont mises à jour dans les 15 minutes.  

Pour chaque ordinateur géré de Linux, une analyse est effectuée toutes les 3 heures.  

## <a name="using-the-solution"></a>À l’aide de la solution

Lorsque vous ajoutez la solution de gestion de la mise à jour de votre espace de travail de l’OMS, la mosaïque de la **Gestion de la mise à jour** sera ajoutée à votre tableau de bord OMS. Cette mosaïque affiche un nombre et une représentation graphique du nombre d’ordinateurs dans votre environnement nécessitant actuellement des mises à jour du système.<br><br>
![Gestion de mise à jour récapitulative en mosaïque](media/oms-solution-update-management/update-management-summary-tile.png)  

## <a name="viewing-update-assessments"></a>Affichage des évaluations de mise à jour

Cliquez sur dans la fenêtre **Gestion de la mise à jour** pour ouvrir le tableau de bord de **Gestion de la mise à jour** . Le tableau de bord inclut les colonnes dans le tableau suivant. Chaque colonne répertorie jusqu'à dix éléments correspondant aux critères de cette colonne pour la portée spécifiée et la plage de temps. Vous pouvez exécuter une recherche de journal qui renvoie tous les enregistrements en cliquant sur **Afficher tout** en bas de la colonne ou en cliquant sur l’en-tête de colonne.

Colonne | Description|
----------|----------|
**Ordinateurs, mises à jour manquantes** ||
Critiques ou des mises à jour de sécurité | Listes trié de haut met à jour les dix ordinateurs manquants par le nombre de mises à jour, elles sont manquantes. Cliquez sur un nom d’ordinateur pour exécuter une recherche de journal retournant que toutes les mettre à jour les enregistrements de cet ordinateur.|
Critiques ou des mises à jour de sécurité antérieure à 30 jours| Identifie le nombre d’ordinateurs qui sont manquants critiques ou des mises à jour de sécurité regroupés par la longueur de temps depuis la publication de la mise à jour. Cliquez sur l’une des entrées à exécuter une recherche de journal de renvoyer les mises à jour manquantes et critiques.|
**Pas de mises à jour requises**||
Critiques ou des mises à jour de sécurité | Répertorie les classifications des mises à jour que les ordinateurs sont manquants triés par le nombre de mises à jour dans la catégorie des ordinateurs. Cliquez sur une catégorie pour exécuter une recherche de journal de retourner que tous les mettre à jour les enregistrements de cette classification.|
**Déploiement de la mise à jour**||
Déploiement de la mise à jour | Nombre de déploiements de mise à jour actuellement planifiée et la durée jusqu'à la prochaine exécution planifiée.  Cliquez sur la mosaïque pour afficher les planifications en cours d’exécution et mises à jour ou pour planifier un nouveau déploiement.|  
<br>  
![Mettre à jour le tableau de bord de gestion](./media/oms-solution-update-management/update-management-deployment-dashboard.png)<br>  
<br>
![Mise à jour de la vue d’ordinateur de tableau de bord de gestion](./media/oms-solution-update-management/update-management-assessment-computer-view.png)<br>  
<br>
![Mise à jour de la vue de Package de tableau de bord de gestion](./media/oms-solution-update-management/update-management-assessment-package-view.png)<br>  

## <a name="installing-updates"></a>L’installation de mises à jour

Une fois les mises à jour ont été évalués pour tous les ordinateurs Windows dans votre environnement, vous pouvez disposer d’une mises à jour installées par la création d’un *Déploiement de mise à jour*.  Un déploiement de mise à jour est une installation planifiée des mises à jour requises pour un ou plusieurs ordinateurs Windows.  Vous spécifiez la date et l’heure pour le déploiement, en plus d’un ordinateur ou un groupe d’ordinateurs qui doivent être inclus.  

Mises à jour sont installés par les procédures opérationnelles dans Azure Automation.  Vous ne pouvez pas visualiser ces procédures opérationnelles, et ils ne requièrent aucune configuration.  Lors de la création d’un déploiement de mise à jour, il crée une planification dans cette commence une procédure opérationnelle du maître de mise à jour à l’heure spécifiée pour les ordinateurs inclus.  Cette procédure opérationnelle maître démarre une procédure opérationnelle d’enfants sur chaque agent Windows qui exécute l’installation de mises à jour requises.  

### <a name="viewing-update-deployments"></a>Affichage des déploiements de mise à jour

Cliquez sur la mosaïque du **Déploiement de la mise à jour** pour afficher la liste des déploiements de mise à jour.  Ils sont groupés par état – **planifié**, **en cours d’exécution**et **terminées**.<br><br> ![Mettre à jour la Page planification de déploiements](./media/oms-solution-update-management/update-updatedeployment-schedule-page.png)<br>  

Les propriétés affichées pour chaque déploiement de mises à jour sont décrits dans le tableau suivant.

Propriété | Description|
----------|----------|
Nom | Nom du déploiement de la mise à jour.|
Planification | Type de planification.  *OneTime* est actuellement la seule valeur possible.|
Heure de début|Date et heure à laquelle le déploiement de la mise à jour est planifié pour démarrer.|
Durée | Nombre de minutes que le déploiement de la mise à jour est autorisé à s’exécuter.  Si les mises à jour ne sont pas installés sur cette durée, les mises à jour restantes doivent attendre jusqu'à ce que le déploiement de mise à jour suivante.|
Serveurs | Nombre d’ordinateurs concernés par le déploiement de la mise à jour.|
État | Statut actuel du déploiement de la mise à jour.<br><br>Les valeurs possibles sont :<br>-N’est pas démarré<br>-En cours d’exécution<br>-Terminé|  

Cliquez sur un déploiement de mise à jour pour afficher son écran de détail qui inclut les colonnes dans le tableau suivant.  Ces colonnes ne seront pas remplies si le déploiement de la mise à jour n’a pas encore démarré.<br>

Colonne | Description|
----------|----------|
**Résultats de l’ordinateur**||
S’est terminée correctement | Répertorie le nombre d’ordinateurs dans le déploiement de la mise à jour par l’état.  Cliquez sur un état à l’exécution d’une recherche de journal retournant que toutes les mettre à jour les enregistrements avec ce statut pour le déploiement de la mise à jour.|
État de l’Installation| Répertorie les ordinateurs impliqués dans le déploiement de la mise à jour et le pourcentage de mises à jour installées correctement. Cliquez sur l’une des entrées à exécuter une recherche de journal de renvoyer les mises à jour manquantes et critiques.|
**Mise à jour des résultats d’Instance**||
État de l’Installation instance | Répertorie les classifications des mises à jour que les ordinateurs sont manquants triés par le nombre de mises à jour dans la catégorie des ordinateurs. Cliquez sur un ordinateur pour exécuter une recherche de journal de retourner que tous les mettre à jour les enregistrements de cet ordinateur.|  
<br><br> ![Vue d’ensemble des résultats de déploiement de mise à jour](./media/oms-solution-update-management/update-la-updaterunresults-page.png)

### <a name="creating-an-update-deployment"></a>Création d’un déploiement de mise à jour

Créer un nouveau déploiement de mise à jour en cliquant sur le bouton **Ajouter** dans la partie supérieure de l’écran pour ouvrir la page de **Déploiement de mise à jour de nouveau** .  Vous devez fournir des valeurs pour les propriétés dans le tableau suivant.

Propriété | Description|
----------|----------|
Nom | Nom unique pour identifier le déploiement de la mise à jour.|
Fuseau horaire | Fuseau horaire à utiliser pour l’heure de début.|
Heure de début | Date et heure pour démarrer le déploiement de la mise à jour.|
Durée | Nombre de minutes que le déploiement de la mise à jour est autorisé à s’exécuter.  Si les mises à jour ne sont pas installés sur cette durée, les mises à jour restantes doivent attendre jusqu'à ce que le déploiement de mise à jour suivante.|
Ordinateurs | Noms d’ordinateurs ou de groupes d’ordinateurs à inclure dans le déploiement de la mise à jour.  Sélectionnez une ou plusieurs entrées dans la liste déroulante.|
<br><br> ![Nouvelle Page de déploiement de mise à jour](./media/oms-solution-update-management/update-newupdaterun-page.png)

### <a name="time-range"></a>Plage horaire

Par défaut, l’étendue des données analysées dans la solution de gestion de la mise à jour est de tous les groupes de gestion connectés générés dans les dernières 24 heures. 

Pour modifier la plage de temps des données, sélectionnez **la base de données** en haut du tableau de bord. Vous pouvez sélectionner les enregistrements créés ou mis à jour dans les 7 derniers jours, 1 jour ou 6 heures. Ou bien, vous pouvez sélectionner **personnalisée** et spécifier une plage de dates personnalisée.<br><br> ![Option d’intervalle de temps personnalisé](./media/oms-solution-update-management/update-la-time-range-scope-databasedon.png)  

## <a name="log-analytics-records"></a>Enregistrements de journal Analytique

La solution de gestion de la mise à jour crée les deux types d’enregistrements dans le référentiel de l’OMS.

### <a name="update-records"></a>Mettre à jour des enregistrements

Un enregistrement avec un type de **mise à jour** est créé pour chaque mise à jour est installé ou requis sur chaque ordinateur. Mise à jour enregistrements ont les propriétés dans le tableau suivant.

Propriété | Description|
----------|----------|
Type de | *Mise à jour*|
SourceSystem | La source qui a approuvé l’installation de la mise à jour.<br>Les valeurs possibles sont :<br>-Mise à jour de Microsoft<br>-Mise à jour de Windows<br>-SCCM<br>-Serveurs Linux (extraite des responsables du lot)|
Approuvé | Spécifie si la mise à jour a été approuvé pour l’installation.<br> Pour les serveurs Linux, cela est facultatif actuellement en tant que correction n’est pas géré par l’OMS.|
Classification pour Windows | Classification de la mise à jour.<br>Les valeurs possibles sont :<br>-Applications<br>-Mises à jour<br>-Mise à jour<br>-Feature Packs<br>-Mises à jour de sécurité<br>-Les service Packs<br>-Mises à jour<br>-Mises à jour|
Classification pour Linux | Cassification de la mise à jour.<br>Les valeurs possibles sont :<br>-Mises à jour<br>-Mises à jour de sécurité<br>-Autres mises à jour|
Ordinateur | Nom de l’ordinateur.|
InstallTimeAvailable | Spécifie si le temps d’installation est disponible à partir d’autres agents installés de la même mise à jour.|
InstallTimePredictionSeconds | Temps d’installation estimé en secondes en fonction des autres agents installés de la même mise à jour.|
KBID | ID de la base de connaissances qui décrit la mise à jour.|
ManagementGroupName | Nom du groupe de gestion pour les agents SCOM.  Pour les autres agents, il s’agit d’AOI -<workspace ID>.|
MSRCBulletinID | ID du bulletin de sécurité Microsoft qui décrit la mise à jour.|
MSRCSeverity | Gravité du bulletin de sécurité Microsoft.<br>Les valeurs possibles sont :<br>-Critique<br>-Important<br>-Modéré|
Facultatif | Spécifie si la mise à jour est facultative.|
Produit | Nom du produit qu'est la mise à jour.  Cliquez sur **affichage** pour ouvrir ce dernier dans un navigateur.|
PackageSeverity | La gravité de la vulnérabilité résolue dans cette mise à jour, comme indiqué par les fournisseurs de distro de Linux. | 
Date de publication | Date et heure de la mise à jour a été installée.|
RebootBehavior | Spécifie si la mise à jour force un redémarrage.<br>Les valeurs possibles sont :<br>-canrequestreboot<br>-neverreboots|
RevisionNumber | Numéro de révision de la mise à jour.|
SourceComputerId | GUID pour identifier de façon unique l’ordinateur.|
TimeGenerated | Date et heure auxquelles l’enregistrement a été mis à jour.|
Titre | Titre de la mise à jour.|
UpdateID | GUID pour identifier de manière unique la mise à jour.|
UpdateState | Spécifie si la mise à jour est installée sur cet ordinateur.<br>Les valeurs possibles sont :<br>-- La mise à jour est installé sur cet ordinateur.<br>-Si nécessaire - la mise à jour n’est pas installé et est requis sur l’ordinateur.|  

<br>
Lorsque vous effectuez une recherche de journal qui renvoie des enregistrements avec un type de **mise à jour** , vous pouvez sélectionner la vue de **mises à jour** qui affiche un ensemble de mosaïques résumant les mises à jour renvoyés par la recherche. Vous pouvez cliquer sur les entrées de l' **Missing et mises à jour** et **mises à jour obligatoires et facultatifs** est disposée en mosaïque pour l’étendue de la vue d’ensemble des mises à jour. Sélectionnez la vue **liste** ou **Table** pour renvoyer les enregistrements individuels.<br> 

![Affichage de mise à jour de recherche de journal avec la mise à jour du Type d’enregistrement](./media/oms-solution-update-management/update-la-view-updates.png)  

Dans la vue de la **Table** , vous pouvez cliquer sur le **KBID** pour tout enregistrement pour ouvrir un navigateur avec la base de connaissances. Cela vous permet de lire rapidement sur les détails de la mise à jour particulière.<br> 

![Vue de Table de recherche de journal avec des mises à jour du Type d’enregistrement de mosaïques](./media/oms-solution-update-management/update-la-view-table.png)

Dans la **liste** , vous cliquez sur le lien **Afficher** à côté de la KBID pour ouvrir la base de connaissances.<br>

![Vue de liste de recherche de journal avec des mises à jour du Type d’enregistrement de mosaïques](./media/oms-solution-update-management/update-la-view-list.png)


###<a name="updatesummary-records"></a>Enregistrements de UpdateSummary

Un enregistrement avec un type de **UpdateSummary** est créé pour chaque ordinateur agent. Cet enregistrement est mis à jour chaque fois que l’ordinateur est analysé pour les mises à jour. **UpdateSummary** enregistrements ont les propriétés dans le tableau suivant.

Propriété | Description|
----------|----------|
Type de | UpdateSummary|
SourceSystem | OpsManager |
Ordinateur | Nom de l’ordinateur.|
CriticalUpdatesMissing | Nombre de mises à jour critiques manquantes sur l’ordinateur.|
ManagementGroupName | Nom du groupe de gestion pour les agents SCOM. Pour les autres agents, il s’agit d’AOI -<workspace ID>.|
NETRuntimeVersion | Version du runtime .NET installée sur l’ordinateur.|
OldestMissingSecurityUpdateBucket | Compartiment pour classer le temps depuis la sécurité manquante plus ancienne mise à jour sur cet ordinateur a été publié.<br>Les valeurs possibles sont :<br>-Ancien<br>-180 jours<br>-150 jours<br>-120 jours<br>-90 jours<br>-60 jours.<br>-Passez de 30 jours<br>-Récentes|
OldestMissingSecurityUpdateInDays | Nombre de jours depuis la sécurité manquante plus ancienne mise à jour sur cet ordinateur a été publié.|
OsVersion | Version du système d’exploitation installé sur l’ordinateur.|
OtherUpdatesMissing | Nombre d’autres mises à jour manquantes sur l’ordinateur.|
SecurityUpdatesMissing | Nombre de mises à jour de sécurité manquantes sur l’ordinateur.|
SourceComputerId | GUID pour identifier de façon unique l’ordinateur.|
TimeGenerated | Date et heure auxquelles l’enregistrement a été mis à jour.|
TotalUpdatesMissing |Nombre total de mises à jour manquantes sur l’ordinateur.|
WindowsUpdateAgentVersion | Numéro de version de l’agent de mise à jour de Windows sur l’ordinateur.|
WindowsUpdateSetting | Paramètre de la façon dont l’ordinateur va installer les mises à jour importantes.<br>Les valeurs possibles sont :<br>-Désactivé<br>-Avertir avant l’installation<br>-Installation planifiée|
WSUSServer | Serveur d’URL de WSUS si l’ordinateur est configuré pour utiliser un.|  

## <a name="sample-log-searches"></a>Recherches de journaux d’exemple

Le tableau suivant fournit des exemples des recherches de journaux pour mettre à jour les enregistrements collectés par cette solution. 

Requête | Description|
----------|----------|
Tous les ordinateurs avec les mises à jour manquantes | Type = UpdateState de mise à jour = nécessaire facultatif = false & #124 ; Sélectionnez ordinateur, titre, KBID, Classification, UpdateSeverity, PublishedDate|
Mises à jour manquantes pour l’ordinateur « COMPUTER01.contoso.com » (Remplacez par le nom de votre ordinateur) | Type = UpdateState de mise à jour = nécessaire facultatif = false ordinateur = « COMPUTER01.contoso.com » & #124 ; Sélectionnez ordinateur, titre, KBID, produit, UpdateSeverity, PublishedDate|
Tous les ordinateurs sans les critiques ou des mises à jour de sécurité | Type = UpdateState de mise à jour = nécessaire facultatif = false (Classification = Classification ou de « Mises à jour de sécurité » = « Mises à jour critiques »)|
Mises à jour critiques ou de sécurité requis par les ordinateurs où les mises à jour sont appliquées manuellement | Type = UpdateState de mise à jour = nécessaire facultatif = false (Classification = Classification ou de « Mises à jour de sécurité » = « Mises à jour critiques ») ordinateur dans {Type = UpdateSummary WindowsUpdateSetting = manuel & #124 ; Ordinateur distinct} & #124 ; KBID distinct|
Événements d’erreur pour les ordinateurs qui ont manquant critiques ou de sécurité des mises à jour requises | Type = EventLevelName de l’événement = Erreur d’ordinateur dans {Type = mise à jour (Classification = Classification ou de « Mises à jour de sécurité » = « Mises à jour critiques ») UpdateState = nécessaire facultatif = false & #124 ; Ordinateur distinct}|
Tous les ordinateurs sans les correctifs cumulatifs | Type = mise à jour facultative = false Classification = « Correctifs cumulatifs » UpdateState = nécessaires & #124 ; Sélectionnez ordinateur, titre, KBID, Classification, UpdateSeverity, PublishedDate|
Mises à jour manquantes distinctes sur tous les ordinateurs | Type = UpdateState de mise à jour = nécessaire facultatif = false & #124 ; Titre distinct|
Appartenance d’ordinateur WSUS | Type = UpdateSummary & #124 ; count() mesure par WSUSServer|
Configuration de la mise à jour automatique | Type = UpdateSummary & #124 ; count() mesure par WindowsUpdateSetting|
Ordinateurs avec mise à jour automatique est désactivée | Type = UpdateSummary WindowsUpdateSetting = manuel|  
Liste de tous les ordinateurs Linux qui ont une mise à jour du package disponible | Type = mise à jour et OSType = Linux et UpdateState ! = « Pas nécessaire » & #124 ; count() mesure par ordinateur|
Liste de tous les ordinateurs Linux qui ont une mise à jour du package disponible qui corrige une vulnérabilité de sécurité ou critique | Type = mise à jour et OSType = Linux et UpdateState ! = « Pas nécessaire » et (Classification = Classification ou de « Mises à jour critiques » = « Mises à jour de sécurité ») & #124 ; count() mesure par ordinateur|
Liste de tous les packages de mise à jour disponible | Type = mise à jour et OSType = Linux et UpdateState ! = « Pas nécessaire »|
Liste de tous les packages qui ont une mise à jour qui résout une vulnérabilité de sécurité ou critique | Type = mise à jour et OSType = Linux et UpdateState ! = « Pas nécessaire » et (Classification = Classification ou de « Mises à jour critiques » = « Mises à jour de sécurité »)|
Liste de tous les ordinateurs « Ubuntu » toute mise à jour disponible | Type = mise à jour et OSType = Linux et OSName = Ubuntu & #124 ; count() mesure par ordinateur|

## <a name="next-steps"></a>Étapes suivantes

- Utiliser des recherches de journaux dans [Journal Analytique](../log-analytics/log-analytics-log-searches.md) permet d’afficher les données détaillées de mise à jour.

- [Créer vos propres tableaux de bord](../log-analytics/log-analytics-dashboards.md) affichant mise à jour de la conformité de vos ordinateurs gérés.

- [Créer alertes](../log-analytics/log-analytics-alerts.md) lorsque des mises à jour critiques sont détectées comme manquant à partir d’ordinateurs ou un ordinateur a des mises à jour automatiques est désactivés.  




