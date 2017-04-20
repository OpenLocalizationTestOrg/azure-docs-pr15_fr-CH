<properties
pageTitle="Trello | Microsoft Azure"
description="Permet de créer des applications de logique avec le service d’application d’Azure. Trello vous donne une perspective sur tous vos projets, au travail et à domicile.  C’est un moyen simple, gratuit, souple et visual pour gérer vos projets et organiser quoi que ce soit.  Se connecter à Trello pour gérer vos cartes, les listes et les cartes"
services="logic-apps"   
documentationCenter=".net,nodejs,java"  
authors="msftman"   
manager="erikre"    
editor=""
tags="connectors" />

<tags
ms.service="logic-apps"
ms.devlang="multiple"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="integration"
ms.date="08/18/2016"
ms.author="deonhe"/>

# <a name="get-started-with-the-trello-connector"></a>Mise en route avec le connecteur de Trello

Trello vous donne une perspective sur tous vos projets, au travail et à domicile.  C’est un moyen simple, gratuit, souple et visual pour gérer vos projets et organiser quoi que ce soit.  Se connecter à Trello pour gérer vos cartes, les listes et les cartes.

>[AZURE.NOTE] Cette version de l’article s’applique à la version du schéma logique apps 2015-08-01-aperçu. 

Vous pouvez commencer par la création d’une application logique maintenant, voir [Création d’une application de logique](../app-service-logic/app-service-logic-create-a-logic-app.md).

## <a name="triggers-and-actions"></a>Déclencheurs et actions

Le connecteur de Trello peut être utilisé en tant qu’action ; Il a l’ou les déclencheurs. Tous les connecteurs prennent en charge les données au format JSON et XML. 

 Le connecteur de Trello a l’ou des actions et/ou les déclencheurs disponibles suivants :

### <a name="trello-actions"></a>Actions de Trello
Vous pouvez effectuer ces actions :

|Action|Description|
|--- | ---|
|[ListCards](connectors-create-api-trello.md#listcards)|Fiches de la carte|
|[GetCard](connectors-create-api-trello.md#getcard)|Obtenir la carte par id|
|[UpdateCard](connectors-create-api-trello.md#updatecard)|Mise à jour de carte|
|[DeleteCard](connectors-create-api-trello.md#deletecard)|Supprimer la carte|
|[CreateCard](connectors-create-api-trello.md#createcard)|Crée une nouvelle carte dans votre compte trello|
|[ListBoards](connectors-create-api-trello.md#listboards)|Liste des cartes|
|[GetBoard](connectors-create-api-trello.md#getboard)|Obtient la carte par Id|
|[ListLists](connectors-create-api-trello.md#listlists)|Carte de la liste listes de carte|
|[GetList](connectors-create-api-trello.md#getlist)|Obtient la liste par Id|
### <a name="trello-triggers"></a>Déclencheurs de Trello
Vous pouvez écouter ces événements :

|Déclencheur | Description|
|--- | ---|
|Lorsqu’il est ajoutée une nouvelle carte à un forum|Déclenche un flux lorsqu’une nouvelle carte est ajoutée à une carte|
|Lorsqu’une nouvelle carte est ajoutée à une liste|Déclenche un flux lorsqu’une nouvelle carte est ajoutée à une liste|


## <a name="create-a-connection-to-trello"></a>Créer une connexion à Trello
Pour créer des applications de logique avec Trello, vous devez d’abord créer une **connexion** puis fournir les détails pour les propriétés suivantes : 

|Propriété| Obligatoire|Description|
| ---|---|---|
|Jeton|Oui|Fournir des informations d’identification de Trello|
Après avoir créé la connexion, vous pouvez l’utiliser pour exécuter les actions et d’écouter pour les déclencheurs décrites dans cet article. 

>[AZURE.INCLUDE [Steps to create a connection to Trello](../../includes/connectors-create-api-trello.md)]

>[AZURE.TIP] Vous pouvez utiliser cette connexion dans les autres applications de logique.

## <a name="reference-for-trello"></a>Référence Trello
S’applique à la version : 1.0

## <a name="onnewcardinboard"></a>OnNewCardInBoard
Lorsqu’il est ajoutée une nouvelle carte à un forum : déclenche un flux lorsqu’une nouvelle carte est ajoutée à une carte 

```GET: /trigger/boards/{board_id}/cards``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|chemin d’accès|Aucun|Id unique de la carte pour extraire des cartes dans|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="onnewcardinlist"></a>OnNewCardInList
Lorsqu’une nouvelle carte est ajoutée à une liste : déclenche un flux lorsqu’une nouvelle carte est ajoutée à une liste 

```GET: /trigger/lists/{list_id}/cards``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|Id unique de la carte pour extraire des cartes dans|
|list_id|chaîne|Oui|chemin d’accès|Aucun|Id unique de la liste d’extraction de cartes|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="listcards"></a>ListCards
Liste des fiches de la carte : liste des fiches de la carte 

```GET: /boards/{board_id}/cards``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|chemin d’accès|Aucun|ID de la carte pour extraire toutes les cartes|
|actions|chaîne|aucune|requête|Aucun|Répertorier les actions à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|pièces jointes|valeur booléenne|aucune|requête|Aucun|Afficher les pièces jointes|
|attachment_fields|chaîne|aucune|requête|Aucun|Répertorier les champs pièce jointe à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|autocollants|valeur booléenne|aucune|requête|Aucun|Afficher les autocollants|
|membres|valeur booléenne|aucune|requête|Aucun|Afficher les membres|
|memeber_fields|chaîne|aucune|requête|Aucun|Liste les champs de membres à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|CheckItemStates|valeur booléenne|aucune|requête|Aucun|Renvoyer les États de la carte|
|Listes de vérification|chaîne|aucune|requête|Aucun|Afficher les listes de contrôle|
|limite de|nombre entier|aucune|requête|Aucun|Le nombre maximal de résultats à retourner, compris entre 1 et 1000|
|depuis le|chaîne|aucune|requête|Aucun|Extraction de toutes les cartes après cette date|
|avant|chaîne|aucune|requête|Aucun|Extraction de toutes les cartes avant cette date|
|filtre|chaîne|aucune|requête|Aucun|Filtrer la réponse|
|champs|chaîne|aucune|requête|Aucun|Liste les champs de la fiche à renvoyer. Spécifier « tous » ou une liste de valeurs valides|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="getcard"></a>GetCard
Obtenir la carte par id : obtenir la carte par id 

```GET: /cards/{card_id}``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|ID de la carte pour extraire des cartes dans|
|card_id|chaîne|Oui|chemin d’accès|Aucun|ID de la carte à extraire|
|actions|chaîne|aucune|requête|Aucun|Répertorier les actions à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|actions_entities|valeur booléenne|aucune|requête|Aucun|Entités de l’opération de retour|
|actions_display|valeur booléenne|aucune|requête|Aucun|Affiche de l’opération de retour|
|actions_limit|nombre entier|aucune|requête|Aucun|Nombre maximal d’actions à renvoyer|
|action_fields|chaîne|aucune|requête|Aucun|Liste des champs d’action à renvoyer pour chaque action. Spécifier « tous » ou une liste de valeurs valides|
|action_memberCreator_fields|chaîne|aucune|requête|Aucun|Liste de champs de créateur action membres à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|pièces jointes|valeur booléenne|aucune|requête|Aucun|Renvoyer les pièces jointes|
|attachement_fields|chaîne|aucune|requête|Aucun|Liste des champs de pièce jointe à renvoyer pour chaque pièce jointe. Spécifier « tous » ou une liste de valeurs valides|
|membres|valeur booléenne|aucune|requête|Aucun|Retourne les membres|
|member_fields|chaîne|aucune|requête|Aucun|Liste de champs de membres à renvoyer pour chaque membre. Spécifier « tous » ou une liste de valeurs valides|
|membersVoted|valeur booléenne|aucune|requête|Aucun|Retourne les membres ELECTEURS|
|memberVoted_fields|chaîne|aucune|requête|Aucun|Liste des champs de membre ELECTEURS à renvoyer pour chaque membre ELECTEURS. Spécifier « tous » ou une liste de valeurs valides|
|checkItemStates|valeur booléenne|aucune|requête|Aucun|États de retour carte|
|checkItemState_fields|chaîne|aucune|requête|Aucun|Liste de champs d’état de retour pour chaque état de la carte. Spécifier « tous » ou une liste de valeurs valides|
|listes de vérification|chaîne|aucune|requête|Aucun|Renvoyer les listes de contrôle|
|checklist_fields|chaîne|aucune|requête|Aucun|Liste des champs de liste de contrôle pour renvoyer la liste de chaque contrôle. Spécifier « tous » ou une liste de valeurs valides|
|carte mère|valeur booléenne|aucune|requête|Aucun|Retourner la carte auquel appartient la carte|
|board_fields|chaîne|aucune|requête|Aucun|Liste les champs de la carte à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|liste|valeur booléenne|aucune|requête|Aucun|Renvoyer la liste auquel appartient la carte|
|list_fields|chaîne|aucune|requête|Aucun|Liste les champs de la liste à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|autocollants|valeur booléenne|aucune|requête|Aucun|Renvoyer les autocollants|
|sticker_fields|chaîne|aucune|requête|Aucun|Répertorier les champs autocollant à renvoyer pour chaque étiquette. Spécifier « tous » ou une liste de valeurs valides|
|champs|chaîne|aucune|requête|Aucun|Liste les champs de la fiche à renvoyer. Spécifier « tous » ou une liste de valeurs valides|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="updatecard"></a>UpdateCard
Fiche de mise à jour : fiche de mise à jour 

```PUT: /cards/{card_id}``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|ID de la carte pour extraire des cartes à partir de|
|card_id|chaîne|Oui|chemin d’accès|Aucun|ID de la carte mise à jour|
|updateCard| |Oui|corps|Aucun|Valeurs de mise à jour de carte|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="deletecard"></a>DeleteCard
Supprimer la fiche : carte de suppression 

```DELETE: /cards/{card_id}``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|ID de la carte pour extraire des cartes à partir de|
|card_id|chaîne|Oui|chemin d’accès|Aucun|ID de la carte à supprimer|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="createcard"></a>CreateCard
Créer la carte : crée une nouvelle carte dans votre compte trello 

```POST: /cards``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|Id unique de la carte pour créer des cartes de|
|newCard| |Oui|corps|Aucun|Nouvelle carte à ajouter à la carte mère trello|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="listboards"></a>ListBoards
Liste des cartes mères : liste des cartes 

```GET: /member/me/boards``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|filtre|chaîne|aucune|requête|Aucun|Liste de filtres à appliquer aux résultats de la carte mère. Spécifier « tous » ou une liste de valeurs valides|
|champs|chaîne|aucune|requête|Aucun|Liste les champs de la carte à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|actions|chaîne|aucune|requête|Aucun|Répertorier les champs d’action à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|actions_entities|valeur booléenne|aucune|requête|Aucun|Entités de l’opération de retour|
|actions_limit|nombre entier|aucune|requête|Aucun|Nombre maximal d’actions à renvoyer|
|actions_format|chaîne|aucune|requête|Aucun|Spécifier le format des actions à renvoyer|
|actions_since|chaîne|aucune|requête|Aucun|Opérations de retour après la date spécifiée|
|action_fields|chaîne|aucune|requête|Aucun|Répertoriez les champs de l’action à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|appartenances|chaîne|aucune|requête|Aucun|Spécifiez les informations d’appartenance à retourner. Spécifier « tous » ou une liste de valeurs valides|
|organisation|valeur booléenne|aucune|requête|Aucun|Spécifier qu’il faut renvoyer des informations sur l’organisation|
|organization_fields|chaîne|aucune|requête|Aucun|Répertorier les champs d’entreprise à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|listes|chaîne|aucune|requête|Aucun|Spécifiez s’il faut retourner des listes qui appartiennent à la carte|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="getboard"></a>GetBoard
Obtient la carte par Id : Obtient la carte par Id 

```GET: /boards/{board_id}``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|chemin d’accès|Aucun|Id unique de la carte pour obtenir|
|actions|chaîne|aucune|requête|Aucun|Répertorier les actions à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|action_entities|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner des entités de l’action|
|actions_display|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner l’affichage des actions|
|actions_format|chaîne|aucune|requête|Aucun|Spécifier le format des actions à renvoyer|
|actions_since|chaîne|aucune|requête|Aucun|Retourner uniquement les actions après cette date|
|actions_limit|nombre entier|aucune|requête|Aucun|Nombre maximal d’actions à renvoyer|
|action_fields|chaîne|aucune|requête|Aucun|Les champs à renvoyer avec chaque champ de la liste. Spécifier « tous » ou une liste de valeurs valides|
|action_memeber|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner des membres de l’action|
|action_member_fields|chaîne|aucune|requête|Aucun|Répertorier les champs membres pour retourner avec chaque membre de l’action. Spécifier « tous » ou une liste de valeurs valides|
|action_memberCreator|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner le créateur de membre d’action|
|action_memberCreator_fields|chaîne|aucune|requête|Aucun|Répertoriez les champs de créateur action membres à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|cartes|chaîne|aucune|requête|Aucun|Spécifier les cartes pour retourner|
|card_fields|chaîne|aucune|requête|Aucun|Répertorier les champs à renvoyer pour chaque carte. Spécifier « tous » ou une liste de valeurs valides|
|card_attachments|valeur booléenne|Oui|requête|Aucun|Spécifiez s’il faut retourner les pièces jointes sur les cartes|
|card_attachment_fields|chaîne|aucune|requête|Aucun|Liste les champs de pièce jointe à renvoyer pour chaque pièce jointe. Spécifier « tous » ou une liste de valeurs valides|
|card_checklists|chaîne|aucune|requête|Aucun|Spécifier les listes de contrôle à retourner pour chaque carte.|
|card_stickers|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner les autocollants de la carte|
|boardStarts|chaîne|aucune|requête|Aucun|Spécifier les étoiles carte à renvoyer|
|étiquettes|chaîne|aucune|requête|Aucun|Spécifiez les étiquettes à renvoyer|
|label_fields|chaîne|aucune|requête|Aucun|Liste les champs de l’étiquette à renvoyer pour chaque étiquette. Spécifier « tous » ou une liste de valeurs valides|
|labels_limits|nombre entier|aucune|requête|Aucun|Nombre maximal d’étiquettes de retour|
|listes|chaîne|aucune|requête|Aucun|Spécifier les listes à renvoyer|
|list_fields|chaîne|aucune|requête|Aucun|Liste les champs de la liste à renvoyer pour chaque liste. Spécifier « tous » ou une liste de valeurs valides|
|appartenances|chaîne|aucune|requête|Aucun|Répertorier les appartenances à retourner. Spécifier « tous » ou une liste de valeurs valides|
|memberships_member|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner des membres de l’appartenance|
|memberships_member_fields|chaîne|aucune|requête|Aucun|Répertoriez les champs de membres à renvoyer pour chaque membre d’appartenance. Spécifier « tous » ou une liste de valeurs valides|
|membres|chaîne|aucune|requête|Aucun|Lister les membres à renvoyer|
|member_fields|chaîne|aucune|requête|Aucun|Liste les champs de membres à renvoyer pour chaque membre. Spécifier « tous » ou une liste de valeurs valides|
|membersInvited|chaîne|aucune|requête|Aucun|Spécifier les membres invités à retourner|
|membersInvited_fields|chaîne|aucune|requête|Aucun|Répertorier les champs à renvoyer pour chacun. Spécifier « tous » ou une liste de valeurs valides|
|listes de vérification|chaîne|aucune|requête|Aucun|Spécifier les listes de contrôle pour retourner|
|checklist_fields|chaîne|aucune|requête|Aucun|Liste les champs de la liste de contrôle pour renvoyer la liste de chaque contrôle. Spécifier « tous » ou une liste de valeurs valides|
|organisation|valeur booléenne|aucune|requête|Aucun|Indiquez si vous souhaitez renvoyer les informations de l’organisation|
|organization_fields|chaîne|aucune|requête|Aucun|Répertorier les champs d’entreprise à renvoyer pour chaque organisation. Spécifier « tous » ou une liste de valeurs valides|
|organization_memberships|chaîne|aucune|requête|Aucun|Les membres de l’organisation pour renvoyer la liste. Spécifier « tous » ou une liste de valeurs valides|
|myPerfs|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut retourner mon perfs|
|champs|chaîne|aucune|requête|Aucun|Répertorier les champs à renvoyer. Spécifier « tous » ou une liste de valeurs valides|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="listlists"></a>ListLists
Liste des listes de carte de la carte : liste des listes de carte de la carte 

```GET: /boards/{board_id}/lists``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|chemin d’accès|Aucun|Id unique de la carte pour extraire des listes|
|cartes|chaîne|aucune|requête|Aucun|Spécifier les cartes pour retourner|
|card_fields|chaîne|aucune|requête|Aucun|Répertoriez les champs de la fiche à retourner à partir de. Spécifier « tous » ou une liste de valeurs valides|
|filtre|chaîne|aucune|requête|Aucun|Spécifiez la propriété filtre pour les listes|
|champs|chaîne|aucune|requête|Aucun|Répertorier les champs à renvoyer. Spécifier « tous » ou une liste de valeurs valides|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="getlist"></a>GetList
Liste obtient par Id : liste obtient par Id 

```GET: /lists/{list_id}``` 

| Nom| Type de données|Obligatoire|Situé dans|Valeur par défaut|Description|
| ---|---|---|---|---|---|
|board_id|chaîne|Oui|requête|Aucun|Id unique de la carte pour extraire les listes à partir de|
|list_id|chaîne|Oui|chemin d’accès|Aucun|Id unique de la liste d’extraction|
|cartes|chaîne|aucune|requête|Aucun|Spécifier les cartes pour retourner|
|card_fields|chaîne|aucune|requête|Aucun|Liste les champs de la fiche à renvoyer pour chaque carte. Spécifier « tous » ou une liste de valeurs valides|
|carte mère|valeur booléenne|aucune|requête|Aucun|Spécifiez s’il faut renvoyer des informations de la carte|
|board_fields|chaîne|aucune|requête|Aucun|Liste les champs de la carte à renvoyer. Spécifier « tous » ou une liste de valeurs valides|
|champs|chaîne|aucune|requête|Aucun|Liste les champs de la liste à renvoyer. Spécifier « tous » ou une liste de valeurs valides|

#### <a name="response"></a>Réponse

|Nom|Description|
|---|---|
|200|Bien|
|400|Demande incorrecte|
|401|Non autorisé|
|403|Interdit|
|404|Non trouvé|
|500|Erreur de serveur interne. Une erreur inconnue s’est produite|
|par défaut|Échoué de l’opération|


## <a name="object-definitions"></a>Définitions d’objets 

### <a name="card"></a>Carte


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|ID|chaîne|N° |
|checkItemStates|tableau|N° |
|fermé|valeur booléenne|N° |
|dateLastActivity|chaîne|N° |
|DESC|chaîne|N° |
|idBoard|chaîne|N° |
|conjointe|chaîne|N° |
|idMembersVoted|tableau|N° |
|idShort|nombre entier|N° |
|idAttachmentCover|chaîne|N° |
|manualCoverAttachment|valeur booléenne|N° |
|idLabels|tableau|N° |
|nom|chaîne|N° |
|POS|nombre entier|N° |
|shortLink|chaîne|N° |
|badges|non défini|N° |
|échéance|chaîne|N° |
|Messagerie|chaîne|N° |
|shortUrl|chaîne|N° |
|abonné|valeur booléenne|N° |
|URL|chaîne|N° |



### <a name="badges"></a>Badges


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|Voix|nombre entier|N° |
|ViewingMemberVoted|valeur booléenne|N° |
|Abonné|valeur booléenne|N° |
|Fogbugz|chaîne|N° |
|CheckItems|nombre entier|N° |
|CheckItemsChecked|nombre entier|N° |
|Commentaires|nombre entier|N° |
|Pièces jointes|nombre entier|N° |
|Description|valeur booléenne|N° |
|Échéance|chaîne|N° |



### <a name="object"></a>Objet


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|



### <a name="createcard"></a>CreateCard


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|conjointe|chaîne|Oui |
|nom|chaîne|Oui |
|DESC|chaîne|N° |
|POS|chaîne|N° |
|idMembers|chaîne|N° |
|idLabels|chaîne|N° |
|urlSource|chaîne|N° |
|FichierGestion|chaîne|N° |
|idCardSource|chaîne|N° |
|keepFromSource|chaîne|N° |



### <a name="updatecard"></a>UpdateCard


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|nom|chaîne|N° |
|DESC|chaîne|N° |
|fermé|valeur booléenne|N° |
|idMembers|chaîne|N° |
|idAttachmentCover|chaîne|N° |
|conjointe|chaîne|N° |
|idBoard|chaîne|N° |
|POS|chaîne|N° |
|échéance|chaîne|N° |
|abonné|valeur booléenne|N° |



### <a name="board"></a>Carte mère


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|ID|chaîne|N° |
|fermé|valeur booléenne|N° |
|dateLastActivity|chaîne|N° |
|dateLastView|chaîne|N° |
|DESC|chaîne|N° |
|idOrganization|chaîne|N° |
|invitations|tableau|N° |
|invités|valeur booléenne|N° |
|labelNames|non défini|N° |
|appartenances|tableau|N° |
|nom|chaîne|N° |
|épinglés|valeur booléenne|N° |
|powerUps|tableau|N° |
|perfs|non défini|N° |
|shortLink|chaîne|N° |
|shortUrl|chaîne|N° |
|mieux|chaîne|N° |
|abonné|chaîne|N° |
|URL|chaîne|N° |



### <a name="label"></a>Étiquette


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|vert|chaîne|N° |
|jaune|chaîne|N° |
|orange|chaîne|N° |
|rouge|chaîne|N° |
|violet|chaîne|N° |
|bleu|chaîne|N° |
|ciel|chaîne|N° |
|citron vert|chaîne|N° |
|rose|chaîne|N° |
|noir|chaîne|N° |



### <a name="membership"></a>Appartenance


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|ID|chaîne|N° |
|idMember|chaîne|N° |
|memberType|chaîne|N° |
|non confirmée|valeur booléenne|N° |



### <a name="perfs"></a>Perfs


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|permissionLevel|chaîne|N° |
|vote|chaîne|N° |
|commentaires|chaîne|N° |
|invitations|chaîne|N° |
|selfJoin|valeur booléenne|N° |
|cardCovers|valeur booléenne|N° |
|calendarFeedEnabled|valeur booléenne|N° |
|arrière-plan|chaîne|N° |
|backgroundColor|chaîne|N° |
|backgroundImage|chaîne|N° |
|backgroundImageScaled|chaîne|N° |
|backgroundTile|valeur booléenne|N° |
|backgroundBrightness|chaîne|N° |
|canBePublic|valeur booléenne|N° |
|canBeOrg|valeur booléenne|N° |
|canBePrivate|valeur booléenne|N° |
|canInvite|valeur booléenne|N° |



### <a name="list"></a>Liste


| Nom de la propriété | Type de données | Obligatoire |
|---|---|---|
|ID|chaîne|N° |
|nom|chaîne|N° |
|fermé|valeur booléenne|N° |
|idBoard|chaîne|N° |
|POS|numéro de|N° |
|abonné|valeur booléenne|N° |
|cartes|tableau|N° |
|carte mère|non défini|N° |


## <a name="next-steps"></a>Étapes suivantes
[Créer une application de logique](../app-service-logic/app-service-logic-create-a-logic-app.md)