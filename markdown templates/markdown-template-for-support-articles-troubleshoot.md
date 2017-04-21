<properties
   pageTitle="suivi de 42 caractères | Microsoft Azure égale à 60 caractères. Ce titre s’affiche dans les moteurs de recherche qui seront coupées vous 60 caractères. Utilisez des mots clés, mais ne pas gaspiller de l’espace"
   description="Affichées dans les moteurs de recherche sous le titre. Vous avez plus de place ici, que vous utilisez plus de mots clés et une explication plus détaillée que le titre"
   services="service-name"
   documentationCenter="dev-center-name"
   authors="GitHub-alias-of-only-one-author"
   manager="manager-alias"
   editor=""
   tags="top-support-issue"/>

<tags
   ms.service="required"
   ms.devlang="may be required"
   ms.topic="article"
   ms.tgt_pltfrm="may be required"
   ms.workload="required"
   ms.date="mm/dd/yyyy"
   ms.author="Your MSFT alias or your full email address;semicolon separates two or more"/>

# <a name="title-maximum-120-characters-target-the-primary-keyword"></a>Titre (Maximum 120 caractères, cible, le mot clé primary)

_Utiliser des mots clés secondaires de 2-3 dans la description._

_Vous avez le choix entre les exclusions suivantes en fonction de votre scénario. Si votre article est indépendant du modèle de déploiement, ignorer._

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]modèle de déploiement classique.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

[AZURE.INCLUDE [learn-about-deployment-models](../../learn-about-deployment-models-both-include.md)]

[Paragraphe ouverture]
- _Décrivez brièvement les problèmes spécifiques que cet article vous aidera à résoudre les problèmes, et la racine commune cause(s)._
- _Le premier paragraphe est un bon emplacement pour utiliser des mots-clés différents de ceux figurant dans le titre, mais assurez-vous de ne pas rendre très longue. Les phrases doivent également de flux et être faciles à comprendre._
- _La liste des exceptions (facultatives) - les scénarios pertinents qui ne sont pas traitées dans cet article. Par exemple, « scénarios de Linux/systèmes d’exploitation ne sont pas abordées dans cet article »._

Ces erreurs {} | {} Problèmes comme {une raison très générale}.

_Voici un exemple de l’ouverture d’un paragraphe._

_Lorsque vous essayez de vous connecter à la base de données de SQL Azure, les erreurs de connexion courantes, que vous rencontrez sont :_
- _La connexion a échoué pour l’utilisateur. La modification de mot de passe a échoué._
- _Échec de la validation du mot de passe._
- _Impossible d’autoriser l’accès à l’abonnement spécifié._

_Ces erreurs se produisent parce que vous n’êtes pas autorisé à accéder à la source de données._

_S’il s’agit d’un article sur la rubrique facturation, inclure la note suivante (la note ci-dessous est légèrement différente de celui du bas de cet article) :_
> [AZURE.NOTE] Si vous avez besoin d’obtenir de l’aide à tout moment dans cet article, veuillez [contacter le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir votre problème résolu rapidement.

_S’il n’est pas un article de la facturation, inclure la référence suivante :_
[AZURE.INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="troubleshooting-guidance-optional"></a>Guide de dépannage (facultatif)
- _Utilisez cette section lorsque l’aide s’applique à tous les niveaux._
- _Ne passez pas dans les détails. Maintenir un niveau élevé comme un guide._

_Voici un exemple d’un guide de dépannage._

_En règle générale, tant que l’erreur n’indique pas « la taille de mémoire virtuelle demandée n’est pas pris en charge », vous pouvez toujours recommencer ultérieurement, comme suffisamment de ressources peut avoir été libérée dans le cluster pour s’adapter à votre demande. Si le problème est que la taille de mémoire virtuelle demandée n’est pas pris en charge, essayez une autre taille de machine virtuelle ; dans le cas contraire, la seule option est de supprimer la contrainte épingle._

## <a name="troubleshooting-steps"></a>Étapes de dépannage
_Énumérer les solutions dans l’ordre de la convivialité et la simplicité, ce qui signifie que le plus simple, la solution la plus efficace et plus utile doivent être placés en premier._

_Sélectionnez une des versions qui s’appliquent à votre situation._

| <em>Version 1 : Votre article est indépendant du modèle de déploiement</em> | <em>Version 2 : Étapes pour le Gestionnaire de ressources et classique sont en grande partie les mêmes</em> | <em>Version 3 : Étapes pour le Gestionnaire de ressources et classique sont essentiellement différentes. <br />Dans ce cas, utilisez la <a href="https://github.com/Azure/azure-content-pr/blob/master/contributor-guide/custom-markdown-extensions.md#simple-selectors">technique de sélecteurs simples dans Github</a>. <br />Remarque : Les articles de machine virtuelle pour ARM sont des exceptions et ne doivent pas utiliser le sélecteur de ARM/classique.</em> |
|:------------------------------------------------------|:-----------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <p><h3>[Problème 1] \| [Erreur 1]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h4>Solution 2</h4><em>(moins simple ou efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h3>[Problème 2] \| [Erreur 2]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h4>Solution 2</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /> | <p><h3>[Problème 1] \| [Erreur 1]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>Si vous utilisez le modèle de déploiement classique, [cela].<br />Si vous utilisez le modèle de déploiement Gestionnaire de ressources, [cela].</li><li>[Étape 3]</li></ol><p><h4>Solution 2</h4><em>(moins simple ou efficace)</em></p><ol><li>[Étape 1]</li><li>Si vous utilisez le modèle de déploiement classique, [cela].<br />Si vous utilisez le modèle de déploiement Gestionnaire de ressources, [cela].</li><li>[Étape 3]</li></ol><p><h3>[Problème 2] \| [Erreur 2]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>Si vous utilisez le modèle de déploiement classique, [cela].<br />Si vous utilisez le modèle de déploiement Gestionnaire de ressources, [cela].</li><li>[Étape 3]</li></ol><p><h4>Solution 2</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>Si vous utilisez le modèle de déploiement classique, [cela].<br />Si vous utilisez le modèle de déploiement Gestionnaire de ressources, [cela].</li><li>[Étape 3]</li></ol>  | <img src="media/markdown-template-for-support-articles-troubleshoot/rm-classic.png" alt="ARM-Classic"><p><h3>[Problème 1] \| [Erreur 1]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h4>Solution 2</h4><em>(moins simple ou efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h3>[Problème 2] \| [Erreur 2]</h3><h4>Cause</h4>[Détails de cause]</p><p><h4>Solution 1</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><p><h4>Solution 2</h4><em>(la plus simple et la plus efficace)</em></p><ol><li>[Étape 1]</li><li>[Étape 2]</li></ol><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /> |


## <a name="next-steps"></a>Étapes suivantes
_Inclure cette section s’il y a BETON-1 3, les étapes très pertinents que doivent effectuer les utilisateurs. Supprimer si aucune étape suivante. Ce n’est pas un emplacement dans une liste de liens. Si vous incluez des liens vers les étapes suivantes, veillez à inclure le texte pour expliquer pourquoi les étapes suivantes sont pertinentes / important._

_S’il s’agit d’un article sur la rubrique facturation, inclure la note suivante (la note ci-dessous est légèrement différente de celle au début de cet article) :_
> [AZURE.NOTE] Si vous en avez d’autres questions, veuillez [contacter le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir votre problème résolu rapidement.
