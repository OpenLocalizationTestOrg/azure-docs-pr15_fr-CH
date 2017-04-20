<properties
    pageTitle="Créer un tableau de bord personnalisé dans un journal Analytique | Microsoft Azure"
    description="Ce guide vous aide à comprendre comment des tableaux de bord journal Analytique permet d’afficher toutes vos recherches de journal enregistré, vous donnant ainsi une lentille unique pour afficher votre environnement."
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

# <a name="create-a-custom-dashboard-in-log-analytics"></a>Créer un tableau de bord personnalisé dans un journal Analytique

Ce guide vous aide à comprendre comment les tableaux de bord journal Analytique peuvent d’afficher toutes vos recherches de journal enregistré, vous donnant ainsi une lentille unique pour afficher votre environnement.

![Exemple de tableau de bord](./media/log-analytics-dashboards/oms-dashboards-example-dash.png)

Tous les tableaux de bord personnalisés que vous créez dans le portail de l’OMS est également disponibles dans l’application Mobile OMS. Consultez les pages suivantes pour plus d’informations sur les applications.

- [Application mobile d’OMS de Microsoft Store](http://www.windowsphone.com/store/app/operational-insights/4823b935-83ce-466c-82bb-bd0a3f58d865)
- [Application mobile d’OMS de Apple iTunes](https://itunes.apple.com/app/microsoft-operations-management/id1042424859?mt=8)

![tableau de bord mobile](./media/log-analytics-dashboards/oms-search-mobile.png)

## <a name="how-do-i-create-my-dashboard"></a>Comment pour créer mon tableau de bord ?

Pour commencer, accédez à la page de vue d’ensemble de l’OMS. Vous verrez la mosaïque de **Mon tableau de bord** sur la gauche. Cliquez sur Zoom avant dans votre tableau de bord.

![Vue d’ensemble](./media/log-analytics-dashboards/oms-dashboards-overview.png)


## <a name="adding-a-tile"></a>Ajout d’une mosaïque

Dans les tableaux de bord, mosaïques sont alimentés par vos recherches de journal enregistré. OMS est livré avec de nombreux préfabriqué journal enregistré des recherches, afin que vous puissiez immédiatement. Utilisez les étapes suivantes expliquent comment commencer.

Dans la vue de mon tableau de bord, il vous suffit de cliquez sur Personnaliser les **Personnaliser** pour entrer en mode.

![Représentation graphique](./media/log-analytics-dashboards/oms-dashboards-pictorial01.png)

 Le panneau qui s’affiche sur le côté droit de la page affiche toutes les recherches de journal enregistré de votre espace de travail. Pour visualiser une recherche de journal enregistré sous forme de mosaïque, placez sur une recherche enregistrée, puis cliquez sur le symbole **plus** .

![Ajouter des mosaïques 1](./media/log-analytics-dashboards/oms-dashboards-pictorial02.png)

Lorsque vous cliquez sur le symbole **plus** , une nouvelle vignette s’affiche dans la vue de mon tableau de bord.

![Ajouter des mosaïques 2](./media/log-analytics-dashboards/oms-dashboards-pictorial03.png)


## <a name="edit-a-tile"></a>Modifier une mosaïque

Dans la vue de mon tableau de bord, il vous suffit de cliquez sur Personnaliser les **Personnaliser** pour entrer en mode. Cliquez sur la mosaïque, que vous souhaitez modifier. Les modifications du panneau droit à modifier et donne une sélection d’options :

![Modifier la mosaïque](./media/log-analytics-dashboards/oms-dashboards-pictorial04.png)

![Modifier la mosaïque](./media/log-analytics-dashboards/oms-dashboards-pictorial05.png)

### <a name="tile-visualizations"></a>Visualisations de mosaïque#
Il existe trois types de visualisations de mosaïque pour choisir à partir de :

|type de graphique|ce qu’il fait|
|---|---|
|![Graphique à barres](./media/log-analytics-dashboards/oms-dashboards-bar-chart.png)|Affiche une chronologie des résultats de votre recherche journal enregistré sous la forme d’un graphique à barres, ou une liste de résultats par un champ en fonction de si votre recherche journal regroupe les résultats par un champ ou pas.
|![métrique](./media/log-analytics-dashboards/oms-dashboards-metric.png)|Affiche votre accès au journal recherche des résultats sous la forme d’un nombre dans une mosaïque. Mosaïques métriques vous permettent de définir un seuil qui met en surbrillance la mosaïque lorsque le seuil est atteint.|
|![ligne](./media/log-analytics-dashboards/oms-dashboards-line.png)|Affiche une chronologie votre journal enregistré résultat de résultats de recherche avec les valeurs sous la forme d’un graphique en courbes.|

### <a name="threshold"></a>Seuil
Vous pouvez créer un seuil sur une mosaïque à l’aide de la visualisation de métrique. Sélectionnez sur pour créer une valeur de seuil sur la mosaïque. Choisissez si vous souhaitez mettre en surbrillance la mosaïque lorsque la valeur est au-dessus ou en dessous du seuil choisi, puis définissez la valeur de seuil ci-dessous.

## <a name="organizing-the-dashboard"></a>Organiser le tableau de bord
Pour organiser votre tableau de bord, accédez à la vue de mon tableau de bord, cliquez sur **Personnaliser** pour entrer personnaliser le mode. Cliquez sur et faites glisser le carreau que vous souhaitez déplacer et la déplacer où vous voulez votre mosaïque à.

![Organiser votre tableau de bord](./media/log-analytics-dashboards/oms-dashboards-organize.png)

## <a name="remove-a-tile"></a>Supprimer une mosaïque
Pour retirer une tuile, accédez à la vue de mon tableau de bord, cliquez sur **Personnaliser** pour entrer personnaliser le mode. Sélectionnez le carreau que vous souhaitez supprimer, puis dans le volet droit, sélectionnez **Supprimer la mosaïque**.

![Supprimer une mosaïque](./media/log-analytics-dashboards/oms-dashboards-remove-tile.png)

## <a name="next-steps"></a>Étapes suivantes

- Créer des [alertes](log-analytics-alerts.md) dans le journal Analytique pour générer des notifications et de corriger les problèmes.
