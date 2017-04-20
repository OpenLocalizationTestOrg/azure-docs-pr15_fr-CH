<properties
    pageTitle="Didacticiel : Intégration d’Azure Active Directory avec SAP Business ByDesign | Microsoft Azure"
    description="Découvrez comment configurer l’authentification unique entre Azure Active Directory et de SAP Business ByDesign."
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
    ms.date="09/09/2016"
    ms.author="jeedes"/>


# <a name="tutorial-azure-active-directory-integration-with-sap-business-bydesign"></a>Didacticiel : Intégration d’Azure Active Directory avec SAP Business ByDesign

Dans ce didacticiel, vous allez apprendre à intégrer SAP Business ByDesign avec Azure Active Directory (AD Azure).

Intégrant SAP Business ByDesign AD Azure vous offre les avantages suivants :

- Vous pouvez contrôler dans Azure AD qui a accès à SAP Business ByDesign
- Vous pouvez autoriser les utilisateurs à automatiquement obtenir signé sur ByDesign d’entreprise SAP (Single Sign-On) avec leur compte Azure AD
- Vous pouvez gérer vos comptes dans un emplacement central : le portail classique Azure

Si vous souhaitez connaître plus de détails sur l’intégration de l’application SaaS avec AD Azure, consultez [accès aux applications et single sign-on avec Azure Active Directory](active-directory-appssoaccess-whatis.md).

## <a name="prerequisites"></a>Conditions préalables

Pour configurer l’intégration de publicités Azure avec SAP Business ByDesign, vous avez besoin des éléments suivants :

- Un abonnement Azure AD
- Une connexion unique SAP Business ByDesign sur abonnement activé


> [AZURE.NOTE] Pour tester les étapes de ce didacticiel, nous ne recommandons pas l’utilisation d’un environnement de production.


Pour tester les étapes de ce didacticiel, vous devez suivre ces recommandations :

- Vous ne devez pas utiliser votre environnement de production, sauf si cela est nécessaire.
- Si vous n’avez pas un environnement d’essai Azure AD, vous pouvez obtenir un mois d’évaluation [ici](https://azure.microsoft.com/pricing/free-trial/).


## <a name="scenario-description"></a>Description du scénario
Dans ce didacticiel, vous testez Azure AD de l’authentification unique dans un environnement de test.

Le scénario décrit dans ce didacticiel se compose de deux blocs de construction principaux :

1. Ajout de SAP Business ByDesign à partir de la galerie
2. Configuration et test AD Azure authentification unique


## <a name="adding-sap-business-bydesign-from-the-gallery"></a>Ajout de SAP Business ByDesign à partir de la galerie
Pour configurer l’intégration de SAP Business ByDesign dans AD Azure, vous devez ajouter SAP Business ByDesign à partir de la galerie à la liste des applications gérées de SaaS.

**Pour ajouter des SAP Business ByDesign à partir de la galerie, procédez comme suit :**

1. Dans **Azure portal classique**, dans le volet de navigation de gauche, cliquez sur **Active Directory**.

    ![Active Directory][1]

2. Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.


3. Pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Applications][2]

4. Cliquez sur **Ajouter** au bas de la page.

    ![Applications][3]

5. Dans la boîte de dialogue **que voulez-vous faire** , cliquez sur **Ajouter une application à partir de la galerie**.

    ![Applications][4]

6. Dans la zone Rechercher, tapez **SAP Business ByDesign**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_01.png)

7. Dans le volet de résultats, sélectionnez **SAP Business ByDesign**, puis cliquez sur **Terminer** pour ajouter l’application.

    ![Active Directory](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_02.png)


##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Configuration et test AD Azure authentification unique
Dans cette section, vous configurez et testez de l’authentification unique de l’annonce Azure avec SAP Business ByDesign est basée sur un utilisateur de test appelé « Brian Simon ».

Pour de l’authentification unique fonctionner, AD Azure doit connaître l’utilisateur équivalent dans SAP Business ByDesign à un utilisateur dans AD Azure. En d’autres termes, une relation de liaison entre un utilisateur AD Azure et l’utilisateur connexe dans SAP Business ByDesign doit être établi.

Ce lien est établi en affectant la valeur du **nom d’utilisateur** dans AD Azure comme valeur du **nom d’utilisateur** dans SAP Business ByDesign.

Pour configurer et tester AD Azure single sign-on avec SAP Business ByDesign, vous devez effectuer les blocs de construction suivantes :

1. **[Configuration AD Azure Single Sign-On](#configuring-azure-ad-single-sign-on)** - pour autoriser les utilisateurs à utiliser cette fonctionnalité.
2. **[Création d’une annonce Azure tester l’utilisateur](#creating-an-azure-ad-test-user)** - tester AD Azure single sign-on avec Britta Simon.
3. **[Création d’un ByDesign d’entreprise SAP tester l’utilisateur](#creating-an-sap-business-bydesign-test-user)** - d’avoir un équivalent de Britta Simon dans SAP Business ByDesign qui est lié à la représentation sous forme de publicité Azure de lui.
4. **[Tester l’utilisateur affectant la publicité Azure](#assigning-the-azure-ad-test-user)** - pour activer Britta Simon utilisation de l’authentification unique de l’annonce Azure.
5. **[Test de l’authentification unique](#testing-single-sign-on)** - pour vérifier si la configuration fonctionne.

### <a name="configuring-azure-ad-single-sign-on"></a>Configuration de l’authentification unique de l’annonce Azure

Dans cette section, vous activez Azure AD de l’authentification unique dans le portail classique et configurez l’authentification unique dans votre application SAP Business ByDesign.

SAP Business ByDesign application attend les assertions SAML dans un format spécifique. Veuillez configurer les demandes suivantes pour cette application. Vous pouvez gérer les valeurs de ces attributs à partir de l’onglet **« Atrribute »** de l’application. La capture d’écran suivante montre un exemple de cela. 


**Pour configurer AD Azure SSO avec SAP Business ByDesign, effectuez les opérations suivantes :**


1. Dans le portail Azure classique, sur la page d’intégration **SAP Business ByDesign** application, dans le menu de la partie supérieure, cliquez sur **attributs**.

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_80.png) 


2. Dans la liste des attributs de jeton SAML attributs, sélectionnez l’attribut nameidentifier, puis cliquez sur **Modifier**.

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_84.png) 

3. Dans la boîte de dialogue Modifier l’attribut utilisateur, effectuez les opérations suivantes :

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_85.png) 

    une barre d’outils. Dans la liste valeur de l’attribut, sélectionnez le fonction **ExtractMailPrefix()**

    b. Dans la liste de publipostage, sélectionnez l’attribut d’utilisateur que vous souhaitez utiliser pour votre implémentation. 
    Par exemple, si vous souhaitez utiliser le n° d’employé comme identificateur d’utilisateur unique et que vous avez stocké la valeur de l’attribut dans le ExtensionAttribute2, puis sélectionnez **user.extensionattribute2**. 

    c. Cliquez sur **terminé**. 
    

4. Dans le portail classique, sur la page d’intégration **SAP Business ByDesign** application, cliquez **sur Configurer l’authentification unique** pour ouvrir la boîte de dialogue **Configuration de l’authentification unique** .
     
    ![Configurer l’authentification unique][6] 

5. Dans la page **Comment voulez-vous que les utilisateurs à se connecter à SAP Business ByDesign** , sélectionnez **Azure AD de l’authentification unique**, puis cliquez sur **suivant**.

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_03.png) 

6. Sur la page de la boîte de dialogue **Configurer les paramètres d’application** , effectuez les opérations suivantes :

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_04.png) 

    une barre d’outils. Dans la zone de texte **URL de connexion** , tapez l’URL utilisée par vos utilisateurs à l’ouverture de session sur votre application SAP Business ByDesign selon le modèle suivant :`https://<servername>.sapbydesign.com`
    
    b. Cliquez sur **suivant**
 
7. Dans la page **configuration de l’authentification unique à SAP Business ByDesign** , effectuez les opérations suivantes :

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_05.png)

    une barre d’outils. Cliquez sur **télécharger les métadonnées**et puis enregistrez le fichier sur votre ordinateur.

    b. Cliquez sur **suivant**.


8. Pour obtenir l’authentification unique configurée pour votre application, effectuez les opérations suivantes :

    une barre d’outils. Ouvrez une session sur votre portail SAP Business ByDesign avec des droits d’administrateur.

    b. Accédez à **l’Application et les tâches courantes de gestion utilisateur** , puis cliquez sur l’onglet **Fournisseur d’identité** .

    c. Cliquez sur le **Nouveau fournisseur d’identité** et sélectionnez le fichier XML de métadonnées que vous avez téléchargé à partir du portail classique Azure. En important les métadonnées, le système télécharge automatiquement le certificat de signature requis et d’un certificat de cryptage.

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_54.png)

    d. Pour inclure l' **URL du Service de consommateur Assertion** dans la demande SAML, sélectionnez **Inclure l’Assertion consommateur Service URL**.

    e. Cliquez sur **Activer l’ouverture de session unique**.

    f. Enregistrez vos modifications.

    g. Cliquez sur l’onglet **Mon système** .

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_52.png)

    h. Copiez l' **URL de l’authentification unique** et le coller dans la zone de texte **URL d’authentification AD Azure** .

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_53.png)

    i. Permet de spécifier si l’employé peut choisir manuellement entre l’ouverture de session avec un ID utilisateur et de mot de passe ou de l’authentification unique en sélectionnant la **Sélection de fournisseur d’identité manuelle**.

    j. Dans la section **Authentification unique URL** , spécifiez l’URL qui doit être utilisé par l’employé pour ouvrir une session sur le système. 
    Dans l’URL envoyée à une liste déroulante des employés, vous pouvez choisir entre les options suivantes :
    
    **URL de la non-SSO**
 
    Le système envoie uniquement l’URL du système normal à l’employé. L’employé ne peut pas ouvrir une session à l’aide de l’authentification unique et doit utiliser le mot de passe ou le certificat à la place.

    **URL DE L’AUTHENTIFICATION UNIQUE** 

    Le système envoie uniquement l’URL de l’authentification unique à l’employé. L’employé peut ouvrir une session à l’aide de l’authentification unique. Demande d’authentification est redirigé via la IdP.

    **Sélection automatique**
 
    Si l’authentification unique n’est pas actif, le système envoie l’URL système normal à l’employé. Si l’authentification est active, le système vérifie si l’employé a un mot de passe. Si un mot de passe est disponible, les URL de l’authentification unique et Non-SSO sont envoyés à l’employé. Toutefois, si l’employé n’a aucun mot de passe, seulement l’URL de l’authentification unique est envoyée à l’employé.

    k. Enregistrez vos modifications.

9. Dans le portail classique, sélectionnez la confirmation de la configuration d’ouverture de session unique, puis cliquez sur **suivant**.
    
    ![Azure AD unique Single Sign-On][10]

10. Sur la page de **confirmation d’ouverture de session unique** , cliquez sur **Terminer**.  
 
    ![Azure AD unique Single Sign-On][11]


### <a name="creating-an-azure-ad-test-user"></a>Création d’un utilisateur de test AD Azure
Dans cette section, vous créez un utilisateur test dans le portail classique appelé Britta Simon.


![Créer utilisateur d’AD Azure][20]

**Pour créer un utilisateur test dans Azure AD, effectuez les opérations suivantes :**

1. Dans **Azure portal classique**, dans le volet de navigation de gauche, cliquez sur **Active Directory**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_09.png) 

2. Dans la liste **répertoire** , sélectionnez le répertoire pour lequel vous souhaitez activer l’intégration d’annuaire.

3. Pour afficher la liste des utilisateurs, dans le menu de la partie supérieure, cliquez sur **utilisateurs**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_03.png) 

4. Pour ouvrir la boîte de dialogue **Ajouter un utilisateur** , dans la barre d’outils dans la partie inférieure, cliquez sur **Ajouter un utilisateur**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_04.png) 

5. Sur la page de dialogue **nous dire sur cet utilisateur** , effectuez les opérations suivantes :
    
    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_05.png) 

    une barre d’outils. En tant que Type d’utilisateur, sélectionnez nouvel utilisateur de votre organisation.

    b. Dans la **zone de texte**nom d’utilisateur, tapez **BrittaSimon**.

    c. Cliquez sur **suivant**.

6.  Sur la page de la boîte de dialogue **Profil d’utilisateur** , effectuez les opérations suivantes :
    
    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_06.png) 

    une barre d’outils. Dans la zone de texte **nom** , tapez **Brian**.  

    b. Dans la zone de texte **Nom** , type, **Simon**.

    c. Dans la zone de texte **Nom complet** , tapez **Brian Simon**.

    d. Dans la liste **rôle** , sélectionnez **utilisateur**.

    e. Cliquez sur **suivant**.

7. Sur la page de boîte de dialogue **obtenir le mot de passe temporaire** , cliquez sur **créer**.

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_07.png) 

8. Sur la page de boîte de dialogue **obtenir le mot de passe temporaire** , effectuez les opérations suivantes :

    ![Création d’un utilisateur de test AD Azure](./media/active-directory-saas-sapbusinessbydesign-tutorial/create_aaduser_08.png) 

    une barre d’outils. Notez que la valeur de **Nouveau mot de passe**.

    b. Cliquez sur **terminé**.   



### <a name="creating-an-sap-business-bydesign-test-user"></a>Création d’un utilisateur de test de SAP Business ByDesign

Dans cette section, vous créez un utilisateur appelé Britta Simon dans SAP Business ByDesign. Travaillez avec l’équipe de support technique de SAP Business ByDesign pour ajouter les utilisateurs de la plate-forme SAP Business ByDesign. 

> [AZURE.NOTE] Veuillez vous assurer que la valeur de NameID doit correspondre avec le champ de nom d’utilisateur de la plate-forme SAP Business ByDesign.

### <a name="assigning-the-azure-ad-test-user"></a>Affectation de l’utilisateur de test AD Azure

Dans cette section, vous activez Britta Simon à utiliser Azure SSO en autorisant son accès à SAP Business ByDesign.

![Affecter des utilisateurs][200] 

**Pour affecter des Britta Simon à SAP Business ByDesign, effectuez les opérations suivantes :**

1. Sur le portail classique, pour ouvrir la vue des applications, dans la vue du répertoire, cliquez sur **Applications** dans le menu supérieur.

    ![Affecter des utilisateurs][201] 

2. Dans la liste des applications, sélectionnez **SAP Business ByDesign**.

    ![Configurer l’authentification unique](./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_sapbusinessbydesign_50.png) 

3. Dans le menu du haut, cliquez sur **utilisateurs**.

    ![Affecter des utilisateurs][203]

4. Dans la liste utilisateurs, sélectionnez **Brian Simon**.

5. Dans la barre d’outils dans la partie inférieure, cliquez sur **attribuer**.

    ![Affecter des utilisateurs][205]


### <a name="testing-single-sign-on"></a>Test de l’authentification unique

Dans cette section, vous testez votre annonce Azure unique configuration de l’authentification à l’aide du panneau d’accès.

Lorsque vous cliquez sur la mosaïque de SAP Business ByDesign dans le panneau d’accès, vous devez obtenir automatiquement signé-sur à votre application de SAP Business ByDesign.


## <a name="additional-resources"></a>Ressources supplémentaires

* [Liste des didacticiels sur la façon d’intégrer les applications SaaS avec Azure Active Directory](active-directory-saas-tutorial-list.md)
* [Quel est l’accès de l’application et de l’authentification unique avec Azure Active Directory ?](active-directory-appssoaccess-whatis.md)


<!--Image references-->

[1]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-sapbusinessbydesign-tutorial/tutorial_general_205.png
