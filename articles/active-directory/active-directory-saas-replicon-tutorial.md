<properties 
    pageTitle="Didacticiel : Intégration d’Azure Active Directory avec RÉPLICON | Microsoft Azure" 
    description="Découvrez comment utiliser RÉPLICON avec Azure Active Directory pour activer l’ouverture de session unique, la mise en service automatique et bien plus encore !" 
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
    ms.date="09/26/2016" 
    ms.author="jeedes" />

#<a name="tutorial-azure-active-directory-integration-with-replicon"></a>Didacticiel : Intégration d’Azure Active Directory avec RÉPLICON
  
L’objectif de ce didacticiel est d’afficher l’intégration d’Azure et RÉPLICON. Le scénario décrit dans ce didacticiel suppose que vous ayez les éléments suivants :

-   Un abonnement Azure valide
-   Un locataire RÉPLICON
  
À la fin de ce didacticiel, les utilisateurs AD Azure que vous avez affecté à RÉPLICON pourront à l’ouverture de session unique dans l’application au niveau du site de votre société RÉPLICON (service fournisseur initiée signe), ou à l’aide de l' [Introduction dans le panneau d’accès](active-directory-saas-access-panel-introduction.md).
  
Le scénario décrit dans ce didacticiel comprend les éléments suivants :

1.  L’activation de l’intégration de l’application pour RÉPLICON
2.  Configuration de l’authentification unique
3.  Configuration d’approvisionnement de l’utilisateur
4.  Affectation d’utilisateurs

![Scénario] (./media/active-directory-saas-replicon-tutorial/IC777798.png "Scénario")
##<a name="enabling-the-application-integration-for-replicon"></a>L’activation de l’intégration de l’application pour RÉPLICON
  
L’objectif de cette section doit décrire comment activer l’intégration de l’application de RÉPLICON.

###<a name="to-enable-the-application-integration-for-replicon-perform-the-following-steps"></a>Pour activer l’intégration de l’application pour RÉPLICON, effectuez les opérations suivantes :

1.  Dans le portail Azure classique, dans le volet de navigation de gauche, cliquez sur **Active Directory**.

    ![Active Directory] (./media/active-directory-saas-replicon-tutorial/IC700993.png "Active Directory")

2.  Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.

3.  Pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Applications] (./media/active-directory-saas-replicon-tutorial/IC700994.png "Applications")

4.  Cliquez sur **Ajouter** au bas de la page.

    ![Ajouter application] (./media/active-directory-saas-replicon-tutorial/IC749321.png "Ajouter application")

5.  Dans la boîte de dialogue **que voulez-vous faire** , cliquez sur **Ajouter une application à partir de la galerie**.

    ![Ajouter une application à partir de gallerry] (./media/active-directory-saas-replicon-tutorial/IC749322.png "Ajouter une application à partir de gallerry")

6.  Dans la **zone Rechercher**, tapez **RÉPLICON**.

    ![Galerie des applications] (./media/active-directory-saas-replicon-tutorial/IC777799.png "Galerie des applications")

7.  Dans le volet de résultats, sélectionnez **RÉPLICON**, puis cliquez sur **Terminer** pour ajouter l’application.

    ![RÉPLICON] (./media/active-directory-saas-replicon-tutorial/IC777800.png "RÉPLICON")
##<a name="configuring-single-sign-on"></a>Configuration de l’authentification unique
  
L’objectif de cette section doit décrire comment permettre aux utilisateurs de s’authentifier auprès de RÉPLICON avec leur compte Azure annonce à l’aide de la fédération basée sur le protocole SAML.

###<a name="to-configure-single-sign-on-perform-the-following-steps"></a>Pour configurer l’authentification unique, procédez comme suit :

1.  Dans Azure portal classique, sur la page intégration d’application **RÉPLICON** , cliquez **sur Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configurer Single Sign On** .

    ![De configurer l’authentification unique] (./media/active-directory-saas-replicon-tutorial/IC777801.png "De configurer l’authentification unique")

2.  Dans la page **Comment voulez-vous que les utilisateurs à se connecter à RÉPLICON** , sélectionnez **Microsoft Azure AD Single Sign-On**, puis cliquez sur **suivant**.

    ![De configurer l’authentification unique] (./media/active-directory-saas-replicon-tutorial/IC777802.png "De configurer l’authentification unique")

3.  Dans la page **Configurer l’URL App** , effectuez les opérations suivantes :

    ![Configurer des URL de l’application] (./media/active-directory-saas-replicon-tutorial/IC777803.png "Configurer des URL de l’application")

    1.  Dans la zone de texte **Connexion RÉPLICON d’adresse URL** , tapez l’URL de votre client RÉPLICON (par exemple : *https://na2.replicon.com/company/saml2/sp-sso/post*).
    2.  Dans la zone de texte **URL de réponse RÉPLICON** , tapez l’URL de votre RÉPLICON **AssertionConsumerService** (par exemple : *https://global.replicon.com/ ! / saml2/société/sso/post*).  

        >[AZURE.NOTE] Vous pouvez obtenir l’URL à partir des métadonnées RÉPLICON à :         **https://global.replicon.com/ ! /saml2/\<YourCompanyKey\>**.

    3.  Cliquez sur **suivant**

4.  Dans la page **configuration de l’authentification unique à RÉPLICON** , pour télécharger vos métadonnées, cliquez sur **télécharger les métadonnées**et puis enregistrez les métadonnées sur votre ordinateur.

    ![De configurer l’authentification unique] (./media/active-directory-saas-replicon-tutorial/IC777804.png "De configurer l’authentification unique")

5.  Dans une fenêtre de navigateur web différents, vous connecter à votre site d’entreprise RÉPLICON en tant qu’administrateur.

6.  Pour configurer SAML 2.0, effectuez les opérations suivantes :

    ![SAML d’activer l’authentification] (./media/active-directory-saas-replicon-tutorial/IC777805.png "SAML d’activer l’authentification")

    1.  Pour afficher la boîte de dialogue **EnableSAML authentification2** , ajoutez le code suivant à votre URL, après votre clé d’entreprise :  
        **/services/SecurityService1.svc/Help/test/EnableSAMLAuthentication2**  
        Vous trouverez ci-dessous le schéma d’URL complète :  
        **https://Na2.replicon.com/\<YourCompanyKey\>/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2**
    2.  Cliquez sur le **+** pour développer la section **v20Configuration** .
    3.  Cliquez sur le **+** pour développer la section **metaDataConfiguration** .
    4.  **Choisir un fichier**, sélectionnez le fichier XML de métadonnées du fournisseur identité, puis cliquez sur **Envoyer**.

7.  Sur le portail Azure classique, sélectionnez la confirmation de la configuration d’ouverture de session unique et puis cliquez sur **Terminer** pour fermer la boîte de dialogue **Configurer Single Sign On** .

    ![De configurer l’authentification unique] (./media/active-directory-saas-replicon-tutorial/IC778418.png "De configurer l’authentification unique")
##<a name="configuring-user-provisioning"></a>Configuration d’approvisionnement de l’utilisateur
  
Afin de permettre aux utilisateurs d’AD Azure pour vous connecter à RÉPLICON, ils doivent être mis en service dans RÉPLICON.  
Dans le cas de RÉPLICON, la mise en service est une tâche manuelle.

###<a name="to-configure-user-provisioning-perform-the-following-steps"></a>Pour configurer la mise en service de l’utilisateur, effectuez les opérations suivantes :

1.  Dans une fenêtre de navigateur web, ouvrez une session dans le site de votre société RÉPLICON en tant qu’administrateur.

2.  Accédez à **Administration \> les utilisateurs**.

    ![Utilisateurs] (./media/active-directory-saas-replicon-tutorial/IC777806.png "Utilisateurs")

3.  Cliquez sur **+ Ajouter un utilisateur**.

    ![Ajouter utilisateur] (./media/active-directory-saas-replicon-tutorial/IC777807.png "Ajouter utilisateur")

4.  Dans la section de **Profil utilisateur** , effectuez les opérations suivantes :

    ![Profil d’utilisateur] (./media/active-directory-saas-replicon-tutorial/IC777808.png "Profil d’utilisateur")

    1.  Dans la zone de texte **Nom de connexion** , tapez l’adresse de messagerie d’Azure AD de l’utilisateur AD Azure que vous souhaitez mettre en service.
    2.  Comme **Type d’authentification**, sélectionnez **authentification unique**.
    3.  Dans la zone de **service** , entrez le département de l’utilisateur.
    4.  En tant que **Type d’employé**, sélectionnez **administrateur**.
    5.  Cliquez sur **Enregistrer le profil de l’utilisateur**.

>[AZURE.NOTE]Vous pouvez utiliser n’importe quel autres RÉPLICON utilisateur compte outils de création ou API fournies par RÉPLICON aux comptes d’utilisateurs de disposition DAS.

##<a name="assigning-users"></a>Affectation d’utilisateurs
  
Pour tester votre configuration, vous devez accorder les utilisateurs AD Azure que vous souhaitez autoriser à l’aide de l’accès de l’application lui en leur affectant.

###<a name="to-assign-users-to-replicon-perform-the-following-steps"></a>Pour affecter des utilisateurs à RÉPLICON, effectuez les opérations suivantes :

1.  Dans le portail Azure classique, créez un compte de test.

2.  Sur la page intégration d’application **RÉPLICON **, cliquez sur **affecter des utilisateurs**.

    ![Affecter des utilisateurs] (./media/active-directory-saas-replicon-tutorial/IC777809.png "Affecter des utilisateurs")

3.  Sélectionnez votre utilisateur de test et cliquez sur **attribuer**, puis cliquez sur **Oui** pour confirmer votre affectation.

    ![Oui] (./media/active-directory-saas-replicon-tutorial/IC767830.png "Oui")
  
Si vous souhaitez tester vos paramètres d’ouverture de session unique, ouvrez le panneau d’accès. Pour plus d’informations sur le panneau d’accès, consultez [Introduction au panneau d’accès](active-directory-saas-access-panel-introduction.md).