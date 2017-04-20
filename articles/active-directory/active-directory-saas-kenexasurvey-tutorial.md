<properties
    pageTitle="Didacticiel : Intégration d’Azure Active Directory avec IBM Kenexa enquête entreprise | Microsoft Azure"
    description="Découvrez comment configurer l’authentification unique entre Azure Active Directory et IBM Kenexa enquête entreprise."
    services="active-directory"
    documentationCenter=""
    authors="jeevansd"
    manager="femila"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/20/2016"
    ms.author="jeedes"/>


# <a name="tutorial-azure-active-directory-integration-with-ibm-kenexa-survey-enterprise"></a>Didacticiel : Intégration d’Azure Active Directory avec IBM Kenexa enquête entreprise

Dans ce didacticiel, vous allez apprendre à intégrer IBM Kenexa enquête entreprise Azure Active Directory (AD Azure).

IBM Kenexa enquête entreprise intégrant des annonces Azure vous offre les avantages suivants :

- Vous pouvez contrôler dans Azure AD qui a accès à l’entreprise d’enquête IBM Kenexa
- Vous pouvez autoriser les utilisateurs à automatiquement obtenir signé sur IBM Kenexa enquête entreprise (Single Sign-On) avec leur compte Azure AD
- Vous pouvez gérer vos comptes dans un emplacement central : le portail classique Azure

Si vous souhaitez connaître plus de détails sur l’intégration de l’application SaaS avec AD Azure, consultez [accès aux applications et single sign-on avec Azure Active Directory](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Conditions préalables

Pour configurer l’intégration de publicités Azure avec IBM Kenexa enquête entreprise, vous devez les éléments suivants :

- Un abonnement Azure AD
- Une connexion unique IBM Kenexa étude entreprise sur abonnement activé


> [AZURE.NOTE] Pour tester les étapes de ce didacticiel, nous ne recommandons pas l’utilisation d’un environnement de production.


Pour tester les étapes de ce didacticiel, vous devez suivre ces recommandations :

- Vous ne devez pas utiliser votre environnement de production, sauf si cela est nécessaire.
- Si vous n’avez pas un environnement d’essai Azure AD, vous pouvez obtenir un mois d’évaluation [ici](https://azure.microsoft.com/pricing/free-trial/).


## <a name="scenario-description"></a>Description du scénario
Dans ce didacticiel, vous testez Azure AD de l’authentification unique dans un environnement de test. Le scénario décrit dans ce didacticiel se compose de deux blocs de construction principaux :

1. Ajout d’IBM Kenexa enquête entreprise à partir de la galerie
2. Configuration et test AD Azure authentification unique


## <a name="adding-ibm-kenexa-survey-enterprise-from-the-gallery"></a>Ajout d’IBM Kenexa enquête entreprise à partir de la galerie
Pour configurer l’intégration de IBM Kenexa enquête entreprise dans AD Azure, vous devez ajouter IBM Kenexa enquête entreprise à partir de la bibliothèque à la liste des applications gérées de SaaS.

**Pour ajouter des IBM Kenexa enquête entreprise à partir de la galerie, procédez comme suit :**

1. Dans **Azure portal classique**, dans le volet de navigation de gauche, cliquez sur **Active Directory**. 

    ![Active Directory][1]

2. Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.

3. Pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Applications][2]

4. Cliquez sur **Ajouter** au bas de la page.

    ![Applications][3]

5. Dans la boîte de dialogue **que voulez-vous faire** , cliquez sur **Ajouter une application à partir de la galerie**.

    ![Applications][4]

6. Dans la zone Rechercher, tapez **IBM Kenexa enquête entreprise**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_01.png)

7. Dans le volet résultats, sélectionnez **IBM Kenexa enquête entreprise**et puis cliquez sur **Terminer** pour ajouter l’application.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_02.png)


##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuration et test AD Azure authentification unique
Dans cette section, vous configurez et testez de l’authentification unique de l’annonce Azure avec IBM Kenexa enquête entreprise est basée sur un utilisateur de test appelé « Britta Simon ».

Pour de l’authentification unique fonctionner, AD Azure doit connaître l’utilisateur équivalent dans IBM Kenexa enquête entreprise à un utilisateur dans AD Azure. En d’autres termes, une relation de liaison entre un utilisateur AD Azure et l’utilisateur connexe dans IBM Kenexa étude entreprise doit être établi.

Ce lien est établi en affectant la valeur du **nom d’utilisateur** dans AD Azure comme valeur du **nom d’utilisateur** dans l’entreprise d’enquête IBM Kenexa.

Pour configurer et tester AD Azure single sign-on avec IBM Kenexa enquête entreprise, vous devez effectuer les blocs de construction suivantes :

1. **[Configuration AD Azure Single Sign-On](#configuring-azure-ad-single-sign-on)** - pour autoriser les utilisateurs à utiliser cette fonctionnalité.
2. **[Création d’une annonce Azure tester l’utilisateur](#creating-an-azure-ad-test-user)** - tester AD Azure single sign-on avec Britta Simon.
4. **[Création d’une entreprise d’étude de Kenexa IBM tester l’utilisateur](#creating-an-kenexasurvey-test-user)** - d’avoir un équivalent de Britta Simon dans IBM Kenexa enquête entreprise qui est lié à la représentation sous forme de publicité Azure de lui.
5. **[Tester l’utilisateur affectant la publicité Azure](#assigning-the-azure-ad-test-user)** - pour activer Britta Simon utilisation de l’authentification unique de l’annonce Azure.
6. **[Test de l’authentification unique](#testing-single-sign-on)** - pour vérifier si la configuration fonctionne.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuration de l’authentification unique de l’annonce Azure

Dans cette section, vous activez Azure AD de l’authentification unique dans le portail classique et configurez l’authentification unique dans votre application d’entreprise d’enquête IBM Kenexa.


**Pour configurer AD Azure SSO avec IBM Kenexa enquête entreprise, effectuez les opérations suivantes :**

1. Dans le portail classique, sur la page intégration d’application **IBM Kenexa enquête entreprise** , cliquez **sur Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configuration de l’authentification unique** .

    ![Configurer l’authentification unique][6] 

2. Dans la page **Comment voulez-vous que les utilisateurs à se connecter à l’entreprise d’enquête IBM Kenexa** , sélectionnez **Azure AD Single Sign-On**, puis cliquez sur **suivant**.
 
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_03.png)

3. Sur la page de la boîte de dialogue **Configurer les paramètres d’application** , effectuez les opérations suivantes :

    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_04.png)

    une barre d’outils. Dans la zone **identificateur** , tapez une URL en utilisant le modèle suivant :`https://surveys.kenexa.com/<company code>`

    b. Dans la zone de texte **URL de réponse** , tapez une URL en utilisant le modèle suivant :`https://surveys.kenexa.com/<company code>/tools/sso.asp`

    c. Cliquez sur **suivant**.

    > [AZURE.NOTE] Veuillez noter que ce ne sont pas les valeurs réelles. Vous devez mettre à jour ces valeurs avec l’identificateur et l’URL de réponse réelle. Contactez l’équipe d’assistance IBM Kenexa étude entreprise pour obtenir ces valeurs.

4. Dans la page **configuration de l’authentification unique à l’entreprise d’enquête IBM Kenexa** , cliquez sur **Télécharger le certificat** et puis enregistrez le fichier sur votre ordinateur :

    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_05.png) 

5. Pour obtenir l’authentification unique configurée pour votre application, contactez l’équipe d’assistance IBM Kenexa et de leur présenter les éléments suivants :

    • Le fichier du certificat téléchargé

    • L' **URL de l’émetteur**

    • L' **URL de l’authentification unique de SAML**

    • L' **URL du Service de déconnexion unique**

    > [AZURE.NOTE] Veuillez Notez que NameID valeur dans la réponse de la demande doit correspondre avec l’ID de l’authentification unique configurée dans le système de Kenexa. Veuillez travail avec Kenexa support team pour faire correspondre l’identificateur de l’utilisateur approprié dans votre organisation en tant qu’ID d’authentification unique. Par défaut AD Azure définira la NameIdentifier comme valeur de nom UPN. Vous pouvez le modifier à partir de l’onglet d’attribut comme illustré dans la capture d’écran ci-dessous. L’intégration fonctionne qu’au terme de la mise en correspondance correcte. 
    
    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_51.png)

6. Dans le portail classique, sélectionnez la confirmation de la configuration d’ouverture de session unique, puis cliquez sur **suivant**.

    ![Azure AD unique Single Sign-On][10]

7. Sur la page de **confirmation d’ouverture de session unique** , cliquez sur **Terminer**.  
  
    ![Azure AD unique Single Sign-On][11]

8. Dans le portail Azure classique, sur la page intégration d’application **IBM Kenexa enquête entreprise** , dans le menu de la partie supérieure, cliquez sur **attributs**.

    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_06.png)

9. Dans la boîte de dialogue **attributs de jeton SAML** , effectuez les opérations suivantes :

    une barre d’outils. Sélectionnez l’attribut de **NameIdentifier** , puis cliquez sur **Modifier** .

    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_07.png)
    
    b. Dans la liste **Valeur de l’attribut** , tapez la valeur de l’attribut d’ID de l’authentification unique est configuré dans le système de Kenexa.
    
    c. Cliquez sur **terminé**

### <a name="creating-an-azure-ad-test-user"></a>Création d’un utilisateur de test AD Azure
L’objectif de cette section est de créer un utilisateur test dans le portail classique appelé Britta Simon.

![Créer utilisateur d’AD Azure][20]

**Pour créer un utilisateur test dans Azure AD, effectuez les opérations suivantes :**

1. Dans **Azure portal classique**, dans le volet de navigation de gauche, cliquez sur **Active Directory**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_09.png) 

2. Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.

3. Pour afficher la liste des utilisateurs, dans le menu de la partie supérieure, cliquez sur **utilisateurs**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_03.png) 

4. Pour ouvrir la boîte de dialogue **Ajouter un utilisateur** , dans la barre d’outils dans la partie inférieure, cliquez sur **Ajouter un utilisateur**.
 
    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_04.png) 

5. Sur la page de dialogue **nous dire sur cet utilisateur** , effectuez les opérations suivantes :
 
    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_05.png) 

    une barre d’outils. En tant que Type d’utilisateur, sélectionnez nouvel utilisateur de votre organisation.

    b. Dans la **zone de texte**nom d’utilisateur, tapez **BrittaSimon**.

    c. Cliquez sur **suivant**.

6.  Sur la page de la boîte de dialogue **Profil d’utilisateur** , effectuez les opérations suivantes :

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_06.png) 

    une barre d’outils. Dans la zone de texte **nom** , tapez **Brian**.  

    b. Dans la zone de texte **Nom** , type, **Simon**.

    c. Dans la zone de texte **Nom complet** , tapez **Brian Simon**.

    d. Dans la liste **rôle** , sélectionnez **utilisateur**.

    e. Cliquez sur **suivant**.

7. Sur la page de boîte de dialogue **obtenir le mot de passe temporaire** , cliquez sur **créer**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_07.png) 

8. Sur la page de boîte de dialogue **obtenir le mot de passe temporaire** , effectuez les opérations suivantes :

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-kenexasurvey-tutorial/create_aaduser_08.png) 

    une barre d’outils. Notez que la valeur de **Nouveau mot de passe**.

    b. Cliquez sur **terminé**.   



### <a name="creating-an-ibm-kenexa-survey-enterprise-test-user"></a>Création d’un utilisateur de test IBM Kenexa enquête entreprise

Dans cette section, vous créez un utilisateur appelé Britta Simon dans IBM Kenexa enquête entreprise. Travaillez avec l’équipe d’assistance IBM Kenexa pour mapper l’ID de l’authentification unique pour tous les utilisateurs. Également cette valeur d’ID de l’authentification unique doit être mappée à la valeur NameIdentifier depuis Active Directory Azure. Vous pouvez modifier ce paramètre par défaut dans l’onglet attribut.


> [AZURE.NOTE] Si vous avez besoin créer un utilisateur manuellement, vous devez contacter l’équipe d’assistance IBM Kenexa enquête entreprise.


### <a name="assigning-the-azure-ad-test-user"></a>Affectation de l’utilisateur de test AD Azure

Dans cette section, vous activez Britta Simon à utiliser Azure SSO par lui accorder son accès à IBM Kenexa enquête entreprise.

![Affecter des utilisateurs][200] 

**Pour affecter des Britta Simon à IBM Kenexa enquête entreprise, effectuez les opérations suivantes :**

1. Sur le portail classique, pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Affecter des utilisateurs][201] 

2. Dans la liste des applications, sélectionnez **IBM Kenexa enquête entreprise**.

    ![Configurer l’authentification unique](./media/active-directory-saas-kenexasurvey-tutorial/tutorial_kenexasurvey_50.png) 

1. Dans le menu du haut, cliquez sur **utilisateurs**.

    ![Affecter des utilisateurs][203] 

1. Dans la liste utilisateurs, sélectionnez **Brian Simon**.

2. Dans la barre d’outils dans la partie inférieure, cliquez sur **attribuer**.
    
    ![Affecter des utilisateurs][205]



### <a name="testing-single-sign-on"></a>Test de l’authentification unique

Dans cette section, vous testez votre annonce Azure unique configuration de l’authentification à l’aide du panneau d’accès.

Lorsque vous cliquez sur la mosaïque IBM Kenexa enquête entreprise dans le panneau d’accès, vous devez obtenir automatiquement signé-sur à votre application d’entreprise d’enquête IBM Kenexa.


## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste des didacticiels sur la façon d’intégrer les applications SaaS avec Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Quel est l’accès de l’application et de l’authentification unique avec Azure Active Directory ?](active-directory-appssoaccess-whatis.md)



<!--Image references-->

[1]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-kenexasurvey-tutorial/tutorial_general_205.png
