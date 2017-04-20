<properties
   pageTitle="Intégration des Applications avec Azure Active Directory | Microsoft Azure"
   description="Pour plus d’informations sur comment ajouter, mettre à jour ou supprimer une application dans Azure Active Directory (AD Azure)."
   services="active-directory"
   documentationCenter=""
   authors="bryanla"
   manager="mbaldwin"
   editor="mbaldwin" />
<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="10/29/2016"
   ms.author="mbaldwin;bryanla" />

# <a name="integrating-applications-with-azure-active-directory"></a>Intégration des applications avec Azure Active Directory

[AZURE.INCLUDE [active-directory-devguide](../../includes/active-directory-devguide.md)]

Les développeurs d’entreprise et les fournisseurs de logiciels en tant que service (SaaS) peuvent développer des services en nuage commerciale ou des applications métier qui peut être intégrée avec Azure Active Directory (AD Azure) pour fournir une connexion sécurisée et l’autorisation de leurs services. Pour intégrer une application ou un service AD Azure, un développeur devez d’abord enregistrer les détails de leur application Azure publicité via le portail classique Azure.

Cet article vous explique comment ajouter, mettre à jour ou supprimer une application dans Azure AD. Vous allez découvrir les différents types d’applications qui peuvent être intégrées à AD Azure, comment configurer vos applications pour accéder aux autres ressources telles que les API de web et bien plus encore.

Pour en savoir plus sur les deux objets AD Azure qui représentent une application enregistrée et la relation entre eux, voir les [objets d’Application et les objets d’entité de Service](active-directory-application-objects.md); Pour en savoir plus sur les directives de personnalisation que vous devez utiliser lorsque vous développez des applications avec Azure Active Directory, consultez [Les instructions de personnalisation pour les applications intégrées](active-directory-branding-guidelines.md).

## <a name="adding-an-application"></a>Ajout d’une application

N’importe quelle application qui souhaite utiliser les fonctionnalités de publicité Azure doit d’abord être enregistrée dans un locataire AD Azure. Le processus d’enregistrement implique précisant la publicité Azure sur votre application, par exemple l’URL où il a trouvé, l’URL pour envoyer une réponse une fois un utilisateur est authentifié, l’URI qui identifie l’application et ainsi de suite.

Si vous créez une application web qui doit juste pour prendre en charge ouverture de session pour les utilisateurs dans Active Directory Azure, vous pouvez simplement suivre les instructions ci-dessous. Si votre application a besoin d’informations d’identification ou les autorisations pour accéder à un site web API, ou s’il doit autoriser les utilisateurs à partir d’autres locataires AD Azure pour y accéder, consultez la section [mise à jour d’une Application](#updating-an-application) pour continuer la configuration de votre application.

### <a name="to-register-a-new-application-in-the-azure-classic-portal"></a>Pour enregistrer une nouvelle application dans Azure portal classique

1. Connectez-vous à l' [Azure portal classique](https://manage.windowsazure.com).

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire souhaité.

1. Dans le menu supérieur, cliquez sur **Applications**. Si aucune application n’ont été ajoutées à votre répertoire, cette page affichera uniquement l’ajouter un lien de l’application. Cliquez sur le lien, ou vous pouvez également cliquer sur le bouton **Ajouter** dans la barre de commandes.

1. Sur la page que voulez-vous faire de page, cliquez sur le lien pour **Ajouter une application à mon entreprise se développe**.

1. Sur le nous sur votre page d’application, vous devez spécifier un nom pour votre application, ainsi que d’indiquer le type d’application que vous enregistrez avec AD Azure.  Vous pouvez choisir soit une [application WebClient](active-directory-dev-glossary.md#client-application) / application[Web ressources/API](active-directory-dev-glossary.md#resource-server) (peut également être utilisé en tant que les deux) ou une application [client natif](active-directory-dev-glossary.md#native-client) . Une fois que vous avez terminé, cliquez sur la flèche dans le coin en bas à droite de la page.

1. Sur la page de propriétés application, fournir l’ouverture de session d’URL et URI ID d’application si vous vous inscrivez une application web ou simplement rediriger URI pour une application client natif, puis cliquez sur la case à cocher dans le coin en bas à droite de la page.

1. Votre application a été ajoutée, et vous serez sur la page de démarrage rapide pour votre application. Selon que votre application est un site ou une application native, vous verrez différentes options sur la manière d’ajouter des fonctionnalités supplémentaires à votre application. Une fois que votre application a été ajoutée, vous pouvez commencer la mise à jour de votre application pour permettre aux utilisateurs de se connecter, web access API dans d’autres applications, ou configurer des applications (ce qui permettent à d’autres organisations à accéder à votre application).

>[AZURE.NOTE] Par défaut, l’enregistrement de la nouvelle application est configuré pour autoriser les utilisateurs à partir de votre répertoire pour vous connecter à votre application.

## <a name="updating-an-application"></a>Mise à jour d’une application

Une fois que votre application a été enregistrée avec l’annonce d’Azure, il est peut-être mise à jour pour fournir l’accès aux API de web, disponibles dans les autres organisations et bien plus encore. Cette section décrit les différentes méthodes dans lequel vous devrez peut-être configurer votre application plus. Tout d’abord, nous allons commencer avec une vue d’ensemble de l’infrastructure de consentement, qui est important à comprendre si vous générez des applications API des ressources qui va être consommées par les applications client générées par les développeurs de votre organisation ou d’une autre organisation.

Pour plus d’informations sur la façon dont l’authentification fonctionne dans AD Azure, consultez [Scénarios d’authentification pour Azure AD](active-directory-authentication-scenarios.md).

### <a name="overview-of-the-consent-framework"></a>Vue d’ensemble de l’infrastructure de consentement

Framework de consentement Azure annonce facilite le développement mutualisée web et les applications client natif qui doivent accéder à web API sécurisées par un locataire AD Azure, différent de celui dans lequel l’application cliente est enregistrée. Ces web API comprennent l’API graphique, Office 365 et autres services de Microsoft, en plus des API web de votre choix. La structure est basée sur un utilisateur ou un administrateur de donner leur consentement pour une application qui demande à être enregistrés dans leur répertoire, ce qui peut impliquer l’accès à des données de l’annuaire.

Par exemple, si une application cliente web doit appeler un web Office 365 application API/ressources pour lire les informations de calendrier de l’utilisateur, cet utilisateur devra à donner son consentement à l’application cliente. Une fois que le consentement est donné, l’application cliente sera en mesure d’appeler l’API de web Office 365 pour le compte de l’utilisateur et utiliser les informations du calendrier en fonction des besoins.

Le cadre de l’accord est construit sur OAuth 2.0 et ses flux différents, tels qu’accordent des informations d’identification client et octroi de code d’autorisation, à l’aide de clients publics ou confidentielles. À l’aide de OAuth 2.0, AD Azure rend possible de créer différents types d’applications clientes, comme sur un téléphone, Tablet PC, serveur ou une application web et d’accéder aux ressources requises.

Pour obtenir des informations plus détaillées sur le cadre de l’accord, consultez [2.0 OAuth dans Azure AD](https://msdn.microsoft.com/library/azure/dn645545.aspx), [Scénarios d’authentification de publicité Azure](active-directory-authentication-scenarios.md)et Office 365 rubrique [compréhension de l’authentification avec l’API d’Office 365](https://msdn.microsoft.com/office/office365/howto/common-app-authentication-tasks).

#### <a name="example-of-the-consent-experience"></a>Exemple de l’expérience de consentement

Les étapes suivantes vous montrera comment le consentement profiter de works pour le développeur de l’application et l’utilisateur.

1. Sur la page de configuration de votre application client web dans Azure portal classique, définissez les autorisations que votre application a besoin en utilisant les menus déroulants dans les autorisations pour un autre contrôle d’applications.

    ![Autorisations à d’autres applications](./media/active-directory-integrating-applications/permissions.png)

1. Considérez que les autorisations de votre application ont été mis à jour et l’application s’exécute, et un utilisateur est sur le point de l’utiliser pour la première fois. Si l’application n’a pas déjà acquis une actualisation ou un jeton, l’application doit atteindre le point de terminaison Azure annonce d’autorisation pour obtenir un code d’autorisation qui peut être utilisé pour acquérir un nouvel accès et actualiser jeton.

1. Si l’utilisateur n’est pas déjà authentifié, ils vous demandera à se connecter à Active Directory Azure.

    ![Utilisateur ou administrateur de se connecter à Active Directory Azure](./media/active-directory-integrating-applications/useradminsignin.png)

1. Une fois que l’utilisateur n’est connecté, AD Azure détermine si l’utilisateur a besoin d’afficher une page de consentement. Cette détermination dépend de si l’utilisateur (ou administrateur de leur organisation) a déjà accordé l’autorisation de l’application. Si l’autorisation n’a pas déjà été délivrée, AD Azure invitera l’utilisateur à autoriser et pour afficher les autorisations nécessaires, qu'il a besoin pour fonctionner. Le jeu d’autorisations qui s’affiche dans la boîte de dialogue de consentement est identique à ce qui a été sélectionné dans les autorisations pour un autre contrôle d’applications dans Azure portal classique.

    ![Expérience de consentement d’utilisateur](./media/active-directory-integrating-applications/userconsent.png)

1. Une fois le consentement de l’utilisateur, un code d’autorisation est retourné à votre application, ce qui peut être échangée à acquérir un jeton d’accès et d’actualiser le jeton. Pour plus d’informations sur ce flux, reportez-vous à la section [Application Web section API web](active-directory-authentication-scenarios.md#web-application-to-web-api) dans les [Scénarios d’authentification pour Azure AD](active-directory-authentication-scenarios.md).

### <a name="configuring-a-client-application-to-access-web-apis"></a>Configuration d’une application cliente pour accéder aux API de web

Commande pour une application cliente web/confidentielles être en mesure de participer à un flux d’octroi d’autorisation qui requiert l’authentification (et obtenir un jeton d’accès), il doit établir des informations d’identification sécurisées. La méthode d’authentification par défaut pris en charge par le portail Azure est l’ID de Client + la clé symétrique. Cette section décrit les étapes de configuration requises pour fournir des informations d’identification de votre client de la clé secrète.

En outre, avant un client pouvant accéder une API web exposés par une application de ressources (ie : Azure AD graphique API), le framework de consentement garantira le client obtient l’autorisation nécessaire, en fonction des autorisations demandées. Par défaut, toutes les applications peuvent choisir autorisations Azure Active Directory (Graph API) et les API de gestion de Service Azure, avec l’autorisation « ouverture de session activer et lecture du profil utilisateur » Azure AD déjà sélectionnée par défaut. Si votre application cliente est en cours d’inscription dans un locataire d’Azure AD Office 365, web API et autorisations pour SharePoint et Exchange Online seront également disponibles pour la sélection. Vous pouvez sélectionner [deux types d’autorisations](active-directory-dev-glossary.md#permissions) dans les menus déroulants en regard de l’API web de votre choix :

- Autorisations de l’application : Votre application cliente doit accéder au web API directement comme de lui-même (pas de contexte utilisateur). Ce type d’autorisation requiert le consentement de l’administrateur et il n’est également pas disponible pour les applications clientes natif.

- Autorisations déléguées : Votre application cliente doit accéder à l’API web que l’utilisateur connecté, mais avec un accès limité par l’autorisation sélectionnée. Ce type d’autorisation peut être accordé par un utilisateur, à moins que l’autorisation est configurée comme nécessitant une autorisation d’administrateur.

#### <a name="to-add-credentials-or-permissions-to-access-web-apis"></a>Pour ajouter des informations d’identification ou des autorisations d’accès aux API de web

1. Se connecter à la [Azure portal classique.](https://manage.windowsazure.com)

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire de votre choix.

1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration. Cliquez sur le lien **configurer** en haut de la page, ce qui vous amènera à la page de configuration de l’application.

1. Pour ajouter une clé secrète pour les informations d’identification de votre application web, faites défiler jusqu'à la section « Clés ».  

    - Premier clic « Sélectionnez durée » liste déroulante et sélectionnez 1 ou 2 ans. 
    - Ensuite, vous verrez une nouvelle ligne, avec les « Valide du » et « Expire » renseigné.
    - La colonne la plus à droite contient la valeur de la clé, après avoir enregistré les modifications de configuration (ci-dessous). N’oubliez pas de revenir à cette section Copier après avoir appuyé sur Enregistrer afin d’avoir à utiliser dans votre application cliente lors de l’authentification au moment de l’exécution.

2. Pour ajouter des autorisations pour accéder aux API de ressource à partir de votre client, faites défiler jusqu'à la section « Autorisations à d’autres applications ». 
    - Tout d’abord, cliquez sur le bouton « Ajouter l’application ».
    - Utilisez la liste déroulante « Show » pour sélectionner le type de ressources souhaitées pour le prélèvement.
    - La première colonne vous permet de sélectionner les applications dans votre répertoire de ressources disponibles qui exposent une API web. Cliquez sur la ressource qui que vous intéressez, puis cliquez sur la case à cocher dans le coin inférieur droit.
    - Une fois sélectionnée, vous verrez la ressource ajoutée à la liste des « Autorisations à d’autres applications ».
    - Les listes déroulantes « Application d’autorisations » et « Autorisations déléguées », sélectionnez les autorisations souhaitées pour votre application cliente.

1. Lorsque vous avez terminé, cliquez sur le bouton **Enregistrer** de la barre de commandes. Si vous avez spécifié une clé pour votre application, il sera également généré une fois que vous cliquez sur Enregistrer.

>[AZURE.NOTE] En cliquant sur le bouton Enregistrer d’également automatiquement définit des autorisations pour votre application dans votre annuaire en fonction des autorisations à d’autres applications que vous avez configuré.  Vous pouvez afficher ces autorisations de l’application en consultant l’onglet de propriétés application.

### <a name="configuring-a-resource-application-to-expose-web-apis"></a>Configuration d’une application de ressources pour exposer les API de web

Vous pouvez développer un site web API et le rendre disponible pour les applications clientes en exposant d’accès [étendues](active-directory-dev-glossary.md#scopes) et les [rôles](active-directory-dev-glossary.md#roles). Une API web correctement configuré est rendue disponible à l’instar des autre web Microsoft API, y compris les API de graphique et les API d’Office 365. Rôles et portées d’accès sont exposés via votre [manifeste d’application](active-directory-dev-glossary.md#application-manifest), qui est un fichier JSON qui représente la configuration de l’identité de votre application.  

La section suivante vous montrera comment exposer les étendues d’accès, en modifiant la ressource manifeste de l’application.

#### <a name="adding-access-scopes-to-your-resource-application"></a>Ajout d’étendues de l’accès à votre application de ressource

1. Connectez-vous à l' [Azure portal classique](https://manage.windowsazure.com).

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire de votre choix.

1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application de ressource que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration.

1. Cliquez sur le bouton **gérer un manifeste** dans la barre de commandes et sélectionnez **Télécharger le manifeste**.

1. Ouvrez le fichier manifeste d’application JSON et remplacez le nœud de « oauth2Permissions » avec l’extrait suivant de JSON. Cet extrait de code est un exemple de l’exposition d’une portée appelée « emprunt d’identité utilisateur », qui autorise le propriétaire d’une ressource à une application cliente un type de délégué l’accès à une ressource. Assurez-vous que vous modifiez le texte et les valeurs de votre propre application :

        "oauth2Permissions": [
        {
            "adminConsentDescription": "Allow the application full access to the Todo List service on behalf of the signed-in   user",
            "adminConsentDisplayName": "Have full access to the Todo List service",
            "id": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
            "isEnabled": true,
            "type": "User",
            "userConsentDescription": "Allow the application full access to the todo service on your behalf",
            "userConsentDisplayName": "Have full access to the todo service",
            "value": "user_impersonation"
            }
        ],

    La valeur id doit être un nouveau GUID généré que vous créez à l’aide d’un [outil de génération de GUID](https://msdn.microsoft.com/library/ms241442%28v=vs.80%29.aspx) ou par programme. Il représente un identificateur unique pour l’autorisation qui est exposée par l’API web. Une fois que votre client est correctement configuré pour demander l’accès à votre web API et les appels de l’API web, il présentera un jeton OAuth 2.0 JWT qui a la demande de la portée (scp) la valeur ci-dessus, qui dans ce cas est user_impersonation.

    >[AZURE.NOTE] Vous pouvez exposer des étendues supplémentaires ultérieurement si nécessaires. Considérez que votre site web API peut exposer plusieurs étendues associés à de nombreuses fonctions différentes. Maintenant vous pouvez contrôler l’accès à l’API web à l’aide de la déclaration de la portée (scp) dans le jeton OAuth 2.0 JWT reçu.

1. Enregistrez le fichier JSON mis à jour et télécharger en cliquant sur le bouton **gérer un manifeste** dans la barre de commandes, sélectionnant **Télécharger le manifeste**, la navigation dans votre fichier manifeste mis à jour, puis il. Une fois téléchargé, votre API web est maintenant configuré pour être utilisé par d’autres applications dans votre répertoire.

#### <a name="to-verify-the-web-api-is-exposed-to-other-applications-in-your-directory"></a>Pour vérifier le web API est exposée à d’autres applications dans votre répertoire

1. Dans le menu supérieur, cliquez sur **Applications**, sélectionnez l’application de client de votre choix que vous souhaitez configurer l’accès à l’API web, puis cliquez sur **configurer**.

1. Faites défiler les autorisations pour la section d’autres applications. Cliquez sur le menu déroulant de sélection d’une application et vous ne pourrez pas sélectionner le web API que vous avez exposée uniquement une autorisation pour. Dans la liste déroulante autorisations de délégué, sélectionnez la nouvelle autorisation.

![Autorisations de la liste TODO sont affichent.](./media/active-directory-integrating-applications/listpermissions.png)

#### <a name="more-on-the-application-manifest"></a>Plus d’informations sur le manifeste d’application
Le manifeste d’application sert en fait un mécanisme de mise à jour de l’entité de l’Application, qui définit tous les attributs de configuration d’identité d’une application Azure AD, y compris les étendues d’accès API que nous avons parlé. Pour plus d’informations sur l’entité de l’Application, reportez-vous à la [documentation d’entité Application API de Graph](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#application-entity). Dans, vous trouverez des informations de référence complètes sur les membres d’entité Application permet de spécifier des autorisations pour votre API :  

- le membre d’appRoles, qui est une collection d’entités [AppRole](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#approle-type) qui peut être utilisé pour définir les **Autorisations de l’Application** pour un site web API  
- le membre d’oauth2Permissions, qui est une collection d’entités [OAuth2Permission](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permission-type) qui peut être utilisé pour définir **Délégation des autorisations** pour un site web API

Pour plus d’informations sur l’application des concepts manifeste, en général, reportez-vous à [comprendre le manifeste d’application Azure Active Directory](active-directory-application-manifest.md).

### <a name="accessing-the-azure-ad-graph-and-office-365-apis"></a>Accéder aux API d’Office 365 et Azure AD graphique

Comme mentionné précédemment, en plus de l’exposition/accéder aux API dans vos propres applications de ressources, vous pouvez également mettre à jour votre application cliente pour accéder aux API exposées par les ressources de Microsoft.  L’API d’Azure AD graphique appelé « Azure Active Directory » dans la liste des autorisations à d’autres applications, est disponible par défaut pour toutes les applications qui sont enregistrés avec AD Azure. Si vous enregistrez votre application client dans un locataire Azure AD qui a été mis en service par Office 365, vous pouvez également accéder à toutes les autorisations exposées par les API pour les différentes ressources d’Office 365.

Pour une discussion complète sur les étendues d’accès exposés par :  

- Azure AD graphique API, veuillez consulter le [autorisations étendues | Graphique des concepts de l’API](https://msdn.microsoft.com/Library/Azure/Ad/Graph/howto/azure-ad-graph-api-permission-scopes) l’article.
- API d’Office 365, reportez-vous à l’article [d’authentification et autorisation à l’aide de cadre commun de consentement](https://msdn.microsoft.com/office/office365/howto/application-manifest) . Pour la plus grande discussion sur la création d’une application cliente qui s’intègre avec l’API d’Office 365, voir [configurer l’environnement de développement d’Office 365](https://msdn.microsoft.com/office/office365/HowTo/setup-development-environment) .

>[AZURE.NOTE] En raison d’une limitation actuelle, les applications client natif peuvent uniquement appeler l’API Azure AD graphique s’ils utilisent l’autorisation « Accéder à l’annuaire de votre organisation ».  Cette restriction ne s’applique pas pour les applications web.

### <a name="configuring-multi-tenant-applications"></a>Configuration d’applications de clients multiples

Lorsque vous ajoutez une application dans Azure AD, vous souhaiterez votre application à être accessibles qu’aux utilisateurs de votre organisation. Vous pouvez également votre application à être accessible par les utilisateurs d’organisations externes. Ces types de deux applications sont appelés mono-utilisateur et applications de plusieurs utilisateurs. Vous pouvez modifier la configuration d’une application mono-utilisateur pour rendre une application partagée, qui traite de cette section ci-dessous.

Il est important de noter les différences entre un client unique et les applications :  

- Une application mono-utilisateur est destinée à être utilisé dans une entreprise. Ils sont généralement un de métier (LoB) application écrit par un développeur d’entreprise. Une application mono-utilisateur ne doit être accessible par les utilisateurs dans un seul répertoire, et par conséquent, il doit uniquement être mis en service dans un répertoire.
- Une application partagée est conçue pour être utilisé dans de nombreuses organisations. Il s’agit d’une application de web software as a service (SaaS) généralement écrite par un éditeur de logiciels indépendant (ISV). Les applications mutualisées doivent être mis en service dans chaque répertoire dans lequel ils seront utilisés, ce qui requiert le consentement d’utilisateur ou un administrateur pour les enregistrer, pris en charge via la structure AD Azure du consentement. Notez que toutes les applications de client natif mutualisées par défaut lors de leur installation sur les périphériques du propriétaire de la ressource. Consultez la vue d’ensemble de la section de consentement de Framework ci-dessus pour plus de détails sur le cadre de l’accord.

#### <a name="enabling-external-users-to-grant-your-application-access-to-their-resources"></a>Permettre aux utilisateurs externes d’autoriser votre application à accéder à leurs ressources

Si vous écrivez une application que vous souhaitez rendre disponible à vos clients ou partenaires extérieurs à votre organisation, vous devez mettre à jour la définition d’application dans Azure portal classique.

>[AZURE.NOTE] Lors de l’activation de plusieurs utilisateurs, vous devez vous assurer que URI ID d’application de votre application appartient à un domaine vérifié. En outre, l’URL de renvoi doit commencer par https://. Pour plus d’informations, voir [les objets Application et Service Principal](active-directory-application-objects.md).

Pour activer l’accès à votre application pour les utilisateurs externes : 

1. Se connecter à la [Azure portal classique.](https://manage.windowsazure.com)

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire de votre choix.

1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche avec les options de configuration.

1. Développez la section **Configurer plusieurs applications** de la zone de démarrage rapide, puis cliquez sur le lien **configurer maintenant** dans la section Activer l’accès. La page de propriétés d’application s’affiche.

1. Cliquez sur le bouton **Oui** en regard de l’Application est partagée, puis cliquez sur le bouton **Enregistrer** de la barre de commandes.

Une fois que vous avez apporté les modifications ci-dessus, les utilisateurs et les administrateurs d’autres organisations pourront accorder votre application à accéder à son propre répertoire et d’autres données.

#### <a name="triggering-the-azure-ad-consent-framework-at-runtime"></a>Le framework de consentement Azure AD lors de l’exécution de déclenchement

Pour utiliser le cadre de l’accord, les applications clientes de mutualisée doivent demander l’autorisation à l’aide de OAuth 2.0. [Exemples de code](https://azure.microsoft.com/documentation/samples/?service=active-directory&term=multi-tenant) sont disponibles pour vous montrer comment une application web, application native ou démon du serveur/application demande des codes d’autorisation et les jetons d’accès pour appeler les API de web.

Votre application web peut également offrir une expérience d’abonnement pour les utilisateurs. Si vous offrez une expérience d’inscription, il est prévu que l’utilisateur clique sur une bouton d’inscription qu’allez rediriger le navigateur vers la OAuth2.0 AD Azure autoriser point de terminaison ou un point de terminaison userinfo OpenID se connecter. Ces points de terminaison permettent à l’application obtenir des informations sur le nouvel utilisateur en inspectant l’id_token. À la suite de la phase d’abonnement à l’utilisateur s’affiche avec une invite de consentement similaire à celui illustré ci-dessus dans la vue d’ensemble de la section de consentement de Framework.

Vous pouvez également les votre application web peut également offrir une expérience qui permet aux administrateurs de « Inscrivez-vous de ma société ». Cette expérience serait également rediriger l’utilisateur vers le AD OAuth Azure 2.0 autorisent le point de terminaison. Dans ce cas, vous transmettez une invite = paramètre admin_consent au point de terminaison autoriser pour forcer l’expérience administrateur consentement, où l’administrateur accorde l’autorisation pour le compte de leur organisation. Seul un utilisateur qui s’authentifie avec un compte qui appartient au rôle administrateur Global peut fournir le consentement ; d’autres utilisateurs recevront une erreur. Sur accord réussie, la réponse contient admin_consent = true. Lorsque le rachat d’un jeton d’accès, vous recevrez également un id_token qui fournit des informations sur l’organisation et l’administrateur ayant souscrit pour votre application.

### <a name="enabling-oauth-20-implicit-grant-for-single-page-applications"></a>L’activation de OAuth 2.0 implicite accorder pour des Applications à Page unique

Seule Page de l’Application (ZPS) sont généralement structurée avec un frontal de lourdes JavaScript qui s’exécute dans le navigateur, qui appelle les API web de l’application back end pour effectuer sa logique métier. Pour les ZPS hébergés dans Azure AD, vous utilisez l’octroi implicite de OAuth 2.0 pour authentifier l’utilisateur avec Active Directory Azure et obtenir un jeton que vous pouvez utiliser pour les appels sécurisés à partir du client JavaScript de l’application à son API de web back-end. Une fois que l’utilisateur a accordé l’autorisation, ce même protocole d’authentification peut servir à obtenir des jetons pour sécuriser les appels entre le client et les autre web ressources API configurés pour l’application. Pour en savoir plus sur l’autorisation implicite grant et vous aidera à définir si il est adapté à votre scénario d’application, reportez-vous à la section [Présentation du flux de subvention implicite OAuth2 dans Azure Active Directory ](active-directory-dev-understanding-oauth2-implicit-grant.md).

Par défaut, les autorisations implicites OAuth 2.0 sont désactivée pour les applications. Vous pouvez activer octroi implicite de OAuth 2.0 pour votre application en définissant le `oauth2AllowImplicitFlow`« ' valeur dans son [manifeste d’application](active-directory-application-manifest.md), qui est un fichier JSON qui représente la configuration de l’identité de votre application.

#### <a name="to-enable-oauth-20-implicit-grant"></a>Pour activer les autorisations implicites OAuth 2.0 

1. Se connecter à la [Azure portal classique.](https://manage.windowsazure.com)
1. Cliquez sur l’icône **d’Active Directory** dans le menu de gauche, puis cliquez sur le répertoire de votre choix.
1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration.
1. Cliquez sur le bouton **gérer un manifeste** dans la barre de commandes et sélectionnez **Télécharger le manifeste**.
Ouvrez le fichier manifeste d’application JSON et définissez la valeur « oauth2AllowImplicitFlow » sur « true ». Par défaut, il est « false ».

    `"oauth2AllowImplicitFlow": true,`

1. Enregistrez le fichier JSON mis à jour et télécharger en cliquant sur le bouton **gérer un manifeste** dans la barre de commandes, sélectionnant **Télécharger le manifeste**, la navigation dans votre fichier manifeste mis à jour, puis il. Une fois téléchargé, votre site web API est maintenant configuré pour utiliser l’octroi implicite de OAuth 2.0 pour authentifier les utilisateurs.


### <a name="legacy-experiences-for-granting-access"></a>Expériences hérités pour accorder l’accès

Cette section décrit l’expérience de consentement hérités avant le 12 mars 2014. Cette expérience est toujours pris en charge et est décrite ci-dessous. Avant les nouvelles fonctionnalités, vous pouvez uniquement accorder les autorisations suivantes :

- Les utilisateurs de leur organisation pour l’ouverture de session

- Ouverture de session des utilisateurs et de lire les données d’annuaire de leur organisation (comme l’application uniquement)

- Ouverture de session des utilisateurs et de lire et d’écrire des données d’annuaire de leur organisation (comme l’application uniquement)

Vous pouvez suivre les étapes [Développement d’Applications Web mutualisée avec AD Azure](https://msdn.microsoft.com/library/azure/dn151789.aspx) pour accorder l’accès pour les nouvelles applications enregistré dans Active Directory Azure. Il est important de noter que la nouvelle infrastructure de consentement permet à des applications beaucoup plus puissantes et permet également aux utilisateurs à donner son consentement à ces applications au lieu de simplement les administrateurs.

#### <a name="building-the-link-that-grants-access-for-external-users-legacy"></a>Création de la liaison qui autorise l’accès pour les utilisateurs externes (hérité)

Afin que les utilisateurs externes à l’inscription de votre application à l’aide de leurs comptes d’organisation, vous devez mettre à jour votre application pour afficher un bouton de qui établit un lien vers la page sur Azure AD qui leur permet d’accorder l’accès. Personnalisation des conseils pour ce bouton s’inscrire est expliquée dans la rubrique de [Instructions de personnalisation pour des Applications intégrées](active-directory-branding-guidelines.md) . Une fois que l’utilisateur accorde ou refuse l’accès, la page d’accès aux subvention AD Azure redirige le navigateur vers votre application avec une réponse. Pour plus d’informations sur les propriétés de l’application, consultez les [objets d’Application et principes de Service](active-directory-application-objects.md).

La page d’accès aux grant est créée par AD Azure, et vous trouverez un lien vers celui-ci dans la page de Configuration de votre application dans Azure portal classique. Pour accéder à la page de Configuration, cliquez sur le lien dans le menu supérieur de vos clients AD Azure **Applications** , cliquez sur l’application que vous souhaitez configurer, puis cliquez sur **configurer** dans le menu supérieur de la page de démarrage rapide.

Le lien de votre application se présente comme suit : `http://account.activedirectory.windowsazure.com/Consent.aspx?ClientID=058eb9b2-4f49-4850-9b78-469e3176e247&RequestedPermissions=DirectoryReaders&ConsentReturnURL=https%3A%2F%2Fadatum.com%2FExpenseReport.aspx%3FContextId%3D123456`. Le tableau suivant décrit les éléments de la liaison :

|Paramètre|Description|
|---|---|
|ClientId|Obligatoire. L’ID du Client que vous avez obtenu dans le cadre de l’ajout de votre application.|
|RequestedPermissions|Facultatif. Niveau d’accès que votre application effectue une demande, qui sera affiché à l’utilisateur d’accorder l’accès de votre application. Si non spécifié, le niveau d’accès demandé par défaut de l’authentification unique uniquement. Les autres options sont DirectoryReaders et DirectoryWriters. Pour plus d’informations sur ces niveaux d’accès, consultez niveaux d’accès aux applications.|
|ConsentReturnUrl|Facultatif. Réponse renvoyée à accéder à l’URL que vous souhaitez que la subvention. Cette valeur doit être codée en URL et doit être dans le même domaine que l’URL de la réponse configurée dans votre définition d’application. Si ne pas fourni, la réponse d’accès accorder affichera à votre URL de réponse configurée.|

Spécification d’un ConsentReturnUrl distinct à partir de l’URL de réponse permettra à votre application implémenter une logique distincte qui peut traiter la réponse sur une autre URL à partir de l’URL de la réponse (qui traite normalement les jetons SAML signe sur). Vous pouvez également spécifier des paramètres supplémentaires dans le ConsentReturnURL codé URL ; il recevra en tant que paramètres de chaîne de requête à votre application lors de la redirection.  Ce mécanisme peut être utilisé pour mettre à jour les infos supplémentaires ou pour lier la demande de votre application d’un droit d’accès à la réponse à partir d’AD Azure.

#### <a name="grant-access-user-experience-and-response-legacy"></a>Accorder l’accès d’utilisation et de la réponse (hérité)

Lorsqu’une application redirige vers le lien d’accès accorder, les images suivantes montrent que l’utilisateur sera confronté.

Si l’utilisateur n’est pas déjà connecté, il seront invité à le faire :

![Se connecter à un DAS](./media/active-directory-integrating-applications/signintoaad.png)

Une fois que l’utilisateur est authentifié, AD Azure redirige l’utilisateur vers la page d’accès aux grant :

![Autoriser l’accès](./media/active-directory-integrating-applications/grantaccess.png)

>[AZURE.NOTE] Seul l’administrateur de la société de l’organisation externe peut accorder l’accès à votre application. Si l’utilisateur n’est pas un administrateur de l’entreprise, ils aura la possibilité d’envoyer des messages à l’administrateur de la société pour demander que l’accès est accordé à cette application.

Une fois que le client a accordé l’accès de votre application en cliquant sur accorder l’accès, ou si l’accès est refusé en cliquant sur Annuler, AD Azure envoie une réponse à le ConsentReturnUrl ou à votre URL de réponse configurée. Cette réponse contient les paramètres suivants :

|Paramètre|Description|
|---|---|
|TenantId|ID unique de l’organisation dans Active Directory Azure qui a accordé l’accès de votre application. Ce paramètre n’est spécifié si le client l’accès accordé.|
|Consentement|La valeur est fixée à autorisé si l’application a été l’accès accordée ou refusée si la demande a été rejetée.|

Paramètres supplémentaires seront retournées à l’application si elles ont été définies en tant que partie de la ConsentReturnUrl codé URL. L’exemple suivant est une réponse à une demande d’accès de subvention qui indique l’application a été autorisée d’exemple, et il inclut un contexte qui a été fourni dans la demande d’accès accorder : `https://adatum.com/ExpenseReport.aspx?ContextID=123456&Consent=Granted&TenantId=f03dcba3-d693-47ad-9983-011650e64134`.

>[AZURE.NOTE] La réponse de grant access ne contiendra pas un jeton de sécurité de l’utilisateur ; l’application doit se connecter l’utilisateur séparément.

Voici une réponse à une demande d’octroi d’accès a été refusée d’exemple :`https://adatum.com/ExpenseReport.aspx?ContextID=123456&Consent=Denied`

#### <a name="rolling-app-keys-for-uninterrupted-access-to-the-graph-api-legacy"></a>Restauration des clés d’application d’accès ininterrompu à l’API de graphique (hérité)

Au cours de la durée de vie de votre application, vous devrez peut-être modifier les touches que vous utilisez lorsque vous appelez Azure AD pour acquérir un jeton d’accès pour appeler l’API graphique.  Modification des clés généralement tombe en deux catégories : rouleau d’urgence sur lequel votre clé a été compromise ou un rouleau sur lorsque votre clé actuelle est sur le point d’expirer. La procédure suivante doit être suivie pour fournir à votre application un accès ininterrompu pendant que vous actualisez vos clés (principalement pour le second cas).

1. Dans le portail Azure classique, cliquez sur clients de votre répertoire, cliquez sur **Applications** dans le menu supérieur, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration.

1. Cliquez sur **configurer** dans le menu du haut pour afficher la liste des propriétés de votre application, et vous verrez une liste de vos clés.

1. Sous les clés, cliquez sur la liste déroulante qui indique la **durée de sélectionner** choisir 1 ou 2 ans et puis cliquez sur **Enregistrer** dans la barre de commandes. Cette action génère une nouvelle clé de mot de passe pour votre application. Copiez cette nouvelle clé de mot de passe. À ce stade des clés existantes et nouvelles utilisable par votre application pour obtenir un jeton d’accès à partir d’AD Azure.

1. Revenez à votre application et mettre à jour la configuration pour commencer à utiliser la nouvelle clé de mot de passe. Reportez-vous [à l’aide de l’API graphique pour interroger Active Directory Azure](https://msdn.microsoft.com/library/azure/dn151791.aspx) pour obtenir un exemple où cette mise à jour doit se produire.

1. Vous devez maintenant déployer cette modification dans votre environnement de production – vérification tout d’abord sur le nœud d’un service, avant de le déployer sur le reste.

1. Une fois la mise à jour terminée dans votre déploiement de production, vous pouvez revenir au portail Azure classique et supprimer l’ancienne clé.

#### <a name="changing-app-properties-after-enabling-access-legacy"></a>Modification des propriétés de l’application après l’activation de l’accès (hérité)

Dès que vous activez l’accès des utilisateurs externes à votre application, vous pouvez toujours continuer à apporter des modifications aux propriétés de votre application dans Azure portal classique. Toutefois, les clients qui ont déjà accès à votre application avant application modifications ne verront pas ces modifications soient répercutées lors de l’affichage des informations sur votre application dans Azure portal classique. Une fois que l’application est rendue disponible pour les clients, vous devez être très prudent lorsque vous apportez certaines modifications. Par exemple, si vous mettez à jour l’URI ID App, vos clients existants qui a accordé l’accès avant cette modification ne pourront pas se connecter à votre application à l’aide de leurs comptes de la société ou de votre école.

Si vous apportez une modification aux RequestedPermissions de demander un niveau d’accès supérieur, les clients existants à l’aide de la fonctionnalité d’application qui tire désormais parti des nouveaux appels d’API à l’aide de cette augmentation niveau d’accès peut obtenir une réponse de l’API graphique de refus d’accès.  Votre application doit gérer ces cas et demandez au client pour accorder l’accès à votre application avec la demande d’un niveau d’accès.

## <a name="removing-an-application"></a>Suppression d’une application

Cette section explique comment supprimer une application à partir de votre client AD Azure.

### <a name="removing-an-application-authored-by-your-organization"></a>Suppression d’une application créée par votre organisation.
Ce sont les applications qui affichent sous le filtre « Applications possède de ma société » sur la page principale « Applications » pour vos clients AD Azure. En termes techniques, il s’agit d’applications que vous avez enregistré manuellement via le portail classique Azure, soit par programme via PowerShell ou l’API graphique. Plus spécifiquement, ils sont représentés par un objet Application et Service Principal de vos clients. Pour plus d’informations, consultez [les objets Application et Service Principal](active-directory-application-objects.md) .

#### <a name="to-remove-a-single-tenant-application-from-your-directory"></a>Pour supprimer une application mono-utilisateur à partir de votre répertoire

1. Se connecter à la [Azure portal classique.](https://manage.windowsazure.com)

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire souhaité.

1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration.

1. Cliquez sur le bouton **Supprimer** dans la barre de commandes.

1. Cliquez sur **Oui** dans le message de confirmation.

#### <a name="to-remove-a-multi-tenant-application-from-your-directory"></a>Pour supprimer une application partagée à partir de votre répertoire

1. Se connecter à la [Azure portal classique.](https://manage.windowsazure.com)

1. Cliquez sur l’icône d’Active Directory dans le menu de gauche, puis cliquez sur le répertoire souhaité.

1. Dans le menu supérieur, cliquez sur **Applications**, puis cliquez sur l’application que vous souhaitez configurer. La page de démarrage rapide s’affiche à l’ouverture de session unique et d’autres informations de configuration.

1. Dans l’Application est partagée, cliquez sur **non**. Il convertit votre application mono-utilisateur, mais l’application reste toujours dans une organisation qui a déjà consenti à celui-ci.

1. Cliquez sur le bouton **Supprimer** dans la barre de commandes.

1. Cliquez sur **Oui** dans le message de confirmation.

### <a name="removing-a-multi-tenant-application-authorized-by-another-organization"></a>Suppression d’une application mutualisée autorisée par une autre organisation
Il s’agit d’un sous-ensemble des applications qui affichent sous le filtre « Applications ma société utilise » sur la page principale « Applications » pour vos clients AD Azure, en particulier celles qui ne sont pas répertoriés dans la liste « Applications possède de ma société ». En termes techniques, il s’agit d’applications mutualisées enregistrées pendant le processus d’autorisation. Plus spécifiquement, ils sont représentés par un objet Principal de Service de vos clients. Pour plus d’informations, consultez [les objets Application et Service Principal](active-directory-application-objects.md) .

Pour supprimer l’accès à votre répertoire d’une application partagée (après avoir accordé consentement), l’administrateur de la société doit avoir un abonnement Azure pour supprimer l’accès via le portail classique Azure. Vous simplement accédez à la page de configuration de l’application et cliquez sur le bouton « Gérer les accès » dans la partie inférieure. Également, l’administrateur de la société peut utiliser les [applets de commande PowerShell de publicité Azure](http://go.microsoft.com/fwlink/?LinkId=294151) pour supprimer l’accès.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [Instructions de personnalisation pour les applications intégrées](active-directory-branding-guidelines.md) pour obtenir des conseils sur les conseils visual pour votre application.

- Pour plus d’informations sur la relation entre les objets d’Application et de Service Principal d’une application, consultez [les objets Application et Service Principal](active-directory-application-objects.md).

- Pour plus d’informations sur le rôle de la lecture de manifeste d’application, voir [comprendre le manifeste d’application Azure Active Directory](active-directory-application-manifest.md)

- Consultez le [Glossaire de développeur AD Azure](active-directory-dev-glossary.md) pour les définitions de certains des concepts développeur core Azure Active Directory (AD).

- Consultez le [guide du développeur de Active Directory](active-directory-developers-guide.md) pour une vue d’ensemble de tous les développeurs de contenu associé.
