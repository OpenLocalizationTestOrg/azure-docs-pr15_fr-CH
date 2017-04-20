<properties 
    pageTitle="Didacticiel : Intégration d’Azure Active Directory avec TimeOffManager | Microsoft Azure" 
    description="Apprenez à utiliser TimeOffManager avec Azure Active Directory pour activer l’ouverture de session unique, la mise en service automatique et bien plus encore !" 
    services="active-directory" 
    authors="jeevansd"  
    documentationCenter="na" 
    manager="femila"/>
<tags 
    ms.service="active-directory" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="identity" 
    ms.date="10/10/2016" 
    ms.author="jeedes" />

#<a name="tutorial-azure-active-directory-integration-with-timeoffmanager"></a>Didacticiel : Intégration d’Azure Active Directory avec TimeOffManager
  
L’objectif de ce didacticiel est d’afficher l’intégration d’Azure et TimeOffManager.  
Le scénario décrit dans ce didacticiel suppose que vous ayez les éléments suivants :

-   Un abonnement Azure valide
-   Un TimeOffManager de l’authentification unique activée abonnement
  
Après la fin de ce didacticiel, les utilisateurs AD Azure que vous avez affecté à TimeOffManager pourront ouverture de session unique dans l’application à votre site de la société TimeOffManager (service fournisseur initiée signe), ou à l’aide de l' [Introduction dans le panneau d’accès](active-directory-saas-access-panel-introduction.md).
  
Le scénario décrit dans ce didacticiel comprend les éléments suivants :

1.  L’activation de l’intégration de l’application pour TimeOffManager
2.  Configuration de l’authentification unique
3.  Configuration d’approvisionnement de l’utilisateur
4.  Affectation d’utilisateurs

![Scénario] (./media/active-directory-saas-timeoffmanager-tutorial/IC795909.png "Scénario")

##<a name="enabling-the-application-integration-for-timeoffmanager"></a>L’activation de l’intégration de l’application pour TimeOffManager
  
L’objectif de cette section doit décrire comment activer l’intégration de l’application pour TimeOffManager.

###<a name="to-enable-the-application-integration-for-timeoffmanager-perform-the-following-steps"></a>Pour activer l’intégration de l’application pour TimeOffManager, effectuez les opérations suivantes :

1.  Dans le portail Azure classique, dans le volet de navigation de gauche, cliquez sur **Active Directory**.

    ![Active Directory] (./media/active-directory-saas-timeoffmanager-tutorial/IC700993.png "Active Directory")

2.  Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.

3.  Pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Applications] (./media/active-directory-saas-timeoffmanager-tutorial/IC700994.png "Applications")

4.  Cliquez sur **Ajouter** au bas de la page.

    ![Ajouter application] (./media/active-directory-saas-timeoffmanager-tutorial/IC749321.png "Ajouter application")

5.  Dans la boîte de dialogue **que voulez-vous faire** , cliquez sur **Ajouter une application à partir de la galerie**.

    ![Ajouter une application à partir de gallerry] (./media/active-directory-saas-timeoffmanager-tutorial/IC749322.png "Ajouter une application à partir de gallerry")

6.  Dans la **zone Rechercher**, tapez **TimeOffManager**.

    ![Galerie des applications] (./media/active-directory-saas-timeoffmanager-tutorial/IC795910.png "Galerie des applications")

7.  Dans le volet de résultats, sélectionnez **TimeOffManager**, puis cliquez sur **Terminer** pour ajouter l’application.

    ![TimeOffManager] (./media/active-directory-saas-timeoffmanager-tutorial/IC795911.png "TimeOffManager")

##<a name="configuring-single-sign-on"></a>Configuration de l’authentification unique
  
L’objectif de cette section doit décrire comment permettre aux utilisateurs de s’authentifier auprès de TimeOffManager avec leur compte Azure annonce à l’aide de la fédération basée sur le protocole SAML.  
Dans le cadre de cette procédure, vous devez télécharger un certificat codé en base 64 à vos clients TimeOffManager.  
Si vous n’êtes pas familiarisé avec cette procédure, reportez-vous à la section [comment convertir un certificat binaire dans un fichier texte](http://youtu.be/PlgrzUZ-Y1o)

###<a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :

1.  Dans Azure portal classique, sur la page d’intégration **TimeOffManager** application, cliquez **sur Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer Single Sign On** .

    ![Configurer l’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795912.png "Configurer l’authentification unique")

2.  Dans la page **Comment voulez-vous que les utilisateurs à se connecter à TimeOffManager** , sélectionnez **Microsoft Azure AD Single Sign-On**, puis cliquez sur **suivant**.

    ![Configurer l’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795913.png "Configurer l’authentification unique")

3.  Dans la page **Configurer l’URL App** , dans la zone de texte **URL de réponse TimeOffManager** , tapez l’URL de votre AssertionConsumerService TimeOffManager (par exemple : "*exemple : https://www.timeoffmanager.com/cpanel/sso/consume.aspx?company\_id = IC34216*», puis cliquez sur **suivant**.

    ![Configurer des URL de l’application] (./media/active-directory-saas-timeoffmanager-tutorial/IC795914.png "Configurer des URL de l’application")

    Vous pouvez obtenir l’URL de la réponse à partir de la TimeOffManager d’authentification unique sur la définition de la page.

    ![Paramètres d’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795915.png "Paramètres d’authentification unique")

4.  Dans la page **configuration de l’authentification unique à TimeOffManager** , pour télécharger votre certificat, cliquez sur **Télécharger le certificat**, puis enregistrez le fichier de certificat sur votre ordinateur.

    ![Configurer l’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795916.png "Configurer l’authentification unique")

5.  Dans une fenêtre de navigateur web différent, ouvrez une session dans le site de votre entreprise TimeOffManager en tant qu’administrateur.

6.  Accédez à **compte \> compte des Options \> unique de paramètres d’authentification**.

    ![Paramètres d’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795917.png "Paramètres d’authentification unique")

7.  Dans la section **Paramètres d’authentification unique** , procédez comme suit :

    ![Paramètres d’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795918.png "Paramètres d’authentification unique")

    une barre d’outils.  Créer un fichier **codé en Base 64** de votre certificat téléchargé.  

        >[AZURE.TIP] Pour plus d’informations, consultez [comment convertir un certificat binaire dans un fichier texte](http://youtu.be/PlgrzUZ-Y1o)

    b.  Ouvrez votre certificat codé en base 64 dans le bloc-notes et copiez le contenu de ce dernier dans votre Presse-papiers puis collez l’intégralité du certificat dans la zone de texte de **Certificat X.509** .
    
    c.  Dans le portail Azure classique, sur la page de la boîte de dialogue **configuration de l’authentification unique à TimeOffManager** , copiez la valeur de **l’URL de l’émetteur** et puis la coller dans la zone de texte **Idp émetteur** .
    
    d.  Dans le portail Azure classique, sur la page de la boîte de dialogue **configuration de l’authentification unique à TimeOffManager** , copiez la valeur de **L’URL de connexion à distance** et puis la coller dans la zone de texte **URL de point de terminaison IdP** .
    
    e.  En tant que **SAML d’appliquer**, sélectionnez **non**.
    

    f.  En tant qu' **Utilisateurs de création automatique**, cliquez sur **Oui**.
    
    g.  Dans le portail Azure classique, sur la page de la boîte de dialogue **configuration de l’authentification unique à TimeOffManager** , copiez la valeur **d’URL de déconnexion à distance** et puis la coller dans la zone de texte **URL de déconnexion** .
    
    h.  Cliquez sur **Enregistrer les modifications**.

8.  Dans le portail Azure classique, sur la page de la boîte de dialogue **configuration de l’authentification unique à TimeOffManager** , sélectionnez la confirmation de la configuration d’ouverture de session unique et puis cliquez sur **Terminer**.

    ![Configurer l’authentification unique] (./media/active-directory-saas-timeoffmanager-tutorial/IC795919.png "Configurer l’authentification unique")

9.  Dans le menu du haut, cliquez sur **attributs** pour ouvrir la boîte de dialogue **Attributs de jeton SAML** .

    ![Attributs] (./media/active-directory-saas-timeoffmanager-tutorial/IC795920.png "Attributs")

10. Pour ajouter les mappages d’attributs requis, procédez comme suit :

    ![attributs de jeton SAML] (./media/active-directory-saas-timeoffmanager-tutorial/123.png "attributs de jeton SAML")

  	|Nom de l’attribut|Valeur de l’attribut|
  	|---|---|
  	|Messagerie|User.Mail|
  	|Prénom|User.GivenName|
  	|Nom|User.Surname|

    une barre d’outils.  Pour chaque ligne de données dans le tableau ci-dessus, cliquez sur **Ajouter un attribut de l’utilisateur**.

    b.  Dans la zone de texte **Nom de l’attribut** , tapez le nom d’attribut affiché pour cette ligne.

    c.  Dans la zone de texte de **Valeur de l’attribut** , sélectionnez la valeur d’attribut affichée pour cette ligne.

    d.  Cliquez sur **terminé**.

11. Cliquez sur **appliquer les modifications**.

##<a name="configuring-user-provisioning"></a>Configuration d’approvisionnement de l’utilisateur
  
Afin de permettre aux utilisateurs d’AD Azure pour vous connecter à TimeOffManager, il doivent être configurés à TimeOffManager.  
TimeOffManager prend en charge au moment de l’approvisionnement de l’utilisateur. Il n’y a aucun élément d’action pour vous.  
Les utilisateurs sont ajoutés automatiquement au cours de la première connexion à l’ouverture de session unique.

>[AZURE.NOTE] Vous pouvez utiliser n’importe quel autres TimeOffManager utilisateur compte outils de création ou API fournies par TimeOffManager à disposition DAS des comptes d’utilisateurs.

##<a name="assigning-users"></a>Affectation d’utilisateurs
  
Pour tester votre configuration, vous devez accorder les utilisateurs AD Azure que vous souhaitez autoriser à l’aide de l’accès de l’application lui en leur affectant.

###<a name="to-assign-users-to-timeoffmanager-perform-the-following-steps"></a>Pour affecter des utilisateurs à TimeOffManager, effectuez les opérations suivantes :

1.  Dans le portail Azure classique, créez un compte de test.

2.  Sur la page d’intégration **TimeOffManager** application, cliquez sur **affecter des utilisateurs**.

    ![Affecter des utilisateurs] (./media/active-directory-saas-timeoffmanager-tutorial/IC795922.png "Affecter des utilisateurs")

3.  Sélectionnez votre utilisateur de test et cliquez sur **attribuer**, puis cliquez sur **Oui** pour confirmer votre affectation.

    ![Oui] (./media/active-directory-saas-timeoffmanager-tutorial/IC767830.png "Oui")
  
Si vous souhaitez tester vos paramètres d’ouverture de session unique, ouvrez le panneau d’accès. Pour plus d’informations sur le panneau d’accès, consultez [Introduction au panneau d’accès](active-directory-saas-access-panel-introduction.md).
