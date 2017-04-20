<properties 
   pageTitle="Suivre les messages B2B dans vos applications de logique dans le Service d’application Azure | Microsoft Azure" 
   description="Cette rubrique décrit le suivi du traitement de B2B" 
   services="logic-apps" 
   documentationCenter=".net,nodejs,java" 
   authors="rajram" 
   manager="erikre" 
   editor=""/>

<tags
   ms.service="logic-apps"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration" 
   ms.date="04/20/2016"
   ms.author="rajram"/>


# <a name="track-b2b-messages"></a>Suivre les messages B2B

[AZURE.INCLUDE [app-service-logic-version-message](../../includes/app-service-logic-version-message.md)]

## <a name="b2b-tracking-information"></a>Informations de suivi B2B
Communication de B2B implique entre les partenaires commerciaux de traitement du message. Les relations sont définies en tant que les accords entre deux partenaires commerciaux. Une fois la communication établie, il faut une façon de contrôler si la communication qui se passe comme prévu. 

Nous avons mis en place de suivi des messages pour les scénarios B2B suivants : AS2, EDIFACT et X12.

## <a name="as2"></a>AS2
Une fois que vous avez créé une instance d’une application d’API AS2, accédez à cette instance, puis sélectionnez le **suivi**. Ici, vous pouvez afficher et filtrer toutes les informations de suivi AS2 :  

![][1]  

## <a name="edifact"></a>EDIFACT
Une fois que vous avez créé une instance d’une application d’API EDIFACT, accédez à cette instance, puis sélectionnez le **suivi**. Ici, vous pouvez afficher et filtrer tous les EDIFACT que des informations de suivi. En outre, vous pouvez afficher le niveau de l’échange, d’un niveau de regroupement, et transaction définie au niveau données, dans une seule vue. 

Si les lots sont créés dans le cadre d’accords EDIFACT dans l’API de gestion des partenaires commerciaux d’application associé, la section Batching répertorie tous ces lots. Vous pouvez sélectionner un lot pour voir le message actif (le cas échéant), ainsi que les informations relatives à la fin :  

![][2]      

## <a name="x12"></a>X12
Une fois que vous avez créé une instance d’un X12 API App, accédez à cette instance et sélectionnez **suivi**. Ici, vous pouvez afficher et filtrer toutes les X12 des informations de suivi. En outre, vous pouvez afficher le niveau de l’échange, d’un niveau de regroupement, et transaction définie au niveau données, dans une seule vue.

Si les lots sont créés dans le cadre de X12 des accords dans l’API de gestion des partenaires commerciaux d’application associée, puis la section Batching répertorie tous ces lots. Vous pouvez sélectionner un lot pour voir le message actif (le cas échéant), ainsi que les informations concernant les lots terminés.

<!--Image references-->
[1]: ./media/app-service-logic-track-b2b-messages/AS2Tracking.png
[2]: ./media/app-service-logic-track-b2b-messages/EDIFACTTracking.png
