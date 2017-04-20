<properties
    pageTitle="Enregistrement de dispositif automatique avec les périphériques Azure Active Directory for Windows Domain-Joined | Microsoft Azure"
    description="Les administrateurs informatiques la possibilité d’avoir leurs périphériques Windows à un domaine pour enregistrer automatiquement et silencieusement avec Azure Active Directory (AD Azure)."
    services="active-directory"
    documentationCenter=""
    authors="markusvi"
    manager="femila"
    editor=""/>

<tags
    ms.service="active-directory"
    ms.workload="identity"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/30/2016"
    ms.author="markvi"/>

# <a name="automatic-device-registration-with-azure-active-directory-for-windows-domain-joined-devices"></a>Enregistrement de dispositif automatique avec les périphériques à un domaine d’Azure Active Directory pour Windows

Comme un administrateur, vous pouvez choisir automatiquement et silencieusement enregistrer vos périphériques Windows à un domaine avec Azure Active Directory (AD Azure). Cela peut être utile si vous avez configuré des stratégies de périphérique en fonction des accès conditionnel à Office 365 applications ou gérés sur site par AD FS. Vous pouvez plus d’informations sur les scénarios d’inscription de périphérique en lisant la [Présentation de l’inscription de périphérique Azure pour Active Directory](active-directory-conditional-access-device-registration-overview.md).

>AZURE. Remarque pour dernières instructions sur comment faire pour configurer l’enregistrement de dispositifs automatiques voir, [comment configurer l’enregistrement automatique du domaine Windows joint périphériques avec Azure Active Directory](active-directory-conditional-access-automatic-device-registration-setup.md).

Enregistrement de dispositif automatique avec Azure Active Directory est disponible pour Windows 7 et Windows 8.1 les machines qui ont été liés à un domaine Active Directory. Ils sont généralement d’entreprise détenus des ordinateurs qui ont été fournis aux travailleurs de l’information.

Pour commencer l’enregistrement de votre domaine rejoint périphériques Windows Azure AD, suivez les informations requises ci-dessous. Une fois que vous implémentez les composants requis, configurer l’enregistrement de dispositifs automatiques pour votre domaine de périphériques Windows.

## <a name="prerequisites-for-automatic-device-registration-of-domain-joined-windows-devices-with-azure-active-directory"></a>Conditions requises pour l’inscription de dispositif automatique de domaine joint de périphériques Windows Azure Active Directory

<a name="deploy-ad-fs-and-connect-to-azure-active-directory-using-azure-active-directory-connect"></a>Déployer AD FS et se connecter à Azure Active Directory à l’aide d’Azure Active Directory se connecter
----------------------------------------------------------------------------------------------
1. Utilisez Azure Active Directory se connecter pour déployer Active Directory Federation Services (ADFS) avec Windows Server 2012 R2 et de définir une relation de fédération avec Azure Active Directory.
2. Configurer une Azure Active Directory confiance partie approbation demande règle supplémentaire.
3. Ouvrez la console de gestion AD FS et accédez à **AD FS**>**relations d’approbation > approbations des parties de confiance**. Avec le bouton droit sur l’objet de confiance de partie utilisatrice plate-forme d’identité Microsoft Office 365 et sélectionnez **Modifier les règles de revendication**
4. Sous l’onglet **Règles de transformation d’émission** , sélectionnez **Ajouter une règle**.
5. Sélectionnez **Envoyer des demandes à l’aide d’une règle personnalisée** à partir de la **règle de revendication** modèle zone de liste déroulante. Cliquez sur **suivant**.
6. Tapez *Auth méthode règle* dans le **nom de règle de revendication :** zone de texte.
7. Tapez la règle de revendication suivant dans la **règle de revendication :** zone de texte :

        c:[Type == "http://schemas.microsoft.com/claims/authnmethodsreferences"]
        => issue(claim = c);

8. Cliquez sur **OK** deux fois pour terminer la boîte de dialogue.

<a name="configure-an-additional-azure-active-directory-relying-party-trust-authentication-class-reference"></a>Configurer un Azure Active Directory supplémentaires s’appuyer l’approbation des parties référence de classe d’authentification
-----------------------------------------------------------------------------------------------------
Sur votre serveur de fédération, ouvrez une fenêtre de commande Windows PowerShell et tapez :


  `Set-AdfsRelyingPartyTrust -TargetName <RPObjectName> -AllowedAuthenticationClassReferences wiaormultiauthn`

Où <RPObjectName> est le nom d’objet partie utilisatrice de votre objet d’approbation partie utilisatrice Azure Active Directory. Cet objet est généralement appelé plate-forme d’identité Microsoft Office 365.

<a name="ad-fs-global-authentication-policy"></a>Stratégie globale de l’authentification AD FS
-----------------------------------------------------------------------------
Configurer la AD FS Global principale stratégie d’authentification pour permettre l’authentification intégrée de Windows pour l’Intranet (c’est la valeur par défaut).


<a name="internet-explorer-configuration"></a>Configuration d’Internet Explorer
------------------------------------------------------------------------------
Configurer les paramètres suivants dans Internet Explorer sur vos périphériques Windows pour la zone de sécurité intranet Local :

- Ne pas demander la sélection de certificat client lorsqu’il n’existe qu’un seul certificat : **Activer**
- Autoriser les scripts : **Activer**
- Ouverture de session automatique uniquement dans la zone Intranet : **activé**

Ce sont les paramètres par défaut pour la zone de sécurité intranet Local de Internet Explorer. Vous pouvez afficher ou gérer ces paramètres dans Internet Explorer, accédez à **Options Internet** > **sécurité** > intranet Local > Personnaliser le niveau. Vous pouvez également configurer ces paramètres à l’aide de la stratégie de groupe Active Directory.

<a name="network-connectivity"></a>Connectivité réseau
-------------------------------------------------------------
Domaine Windows, les périphériques doivent avoir la connectivité AD FS et un contrôleur de domaine Active Directory pour enregistrer automatiquement avec AD Azure. Cela signifie généralement que l’ordinateur doit être connecté au réseau d’entreprise. Il peut s’agir d’une connexion câblée, une connexion Wi-Fi, DirectAccess ou VPN.

## <a name="configure-azure-active-directory-device-registration-discovery"></a>Configurer la recherche de l’enregistrement de dispositif Azure Active Directory
Périphériques de Windows 7 et Windows 8.1 détectera le serveur périphérique d’enregistrement en combinant le nom de compte d’utilisateur avec un nom de serveur d’inscription de périphérique connu. Vous devez créer un enregistrement DNS CNAME qui pointe vers l’enregistrement A associé à votre Service d’inscription du périphérique Azure Active Directory. L’enregistrement CNAME doit utiliser préfixe bien connu **enterpriseregistration** suivi du suffixe UPN utilisé par les comptes d’utilisateurs dans votre organisation. Si votre organisation utilise plusieurs suffixes de noms UPN, plusieurs enregistrements CNAME doivent être créés dans le système DNS.

Par exemple, si vous utilisez deux suffixes UPN à votre organisation nommée @contoso.com et @region.contoso.com, vous allez créer les enregistrements DNS ci-après.

| Entrée                                     | Type de  | Adresse                            |
|-------------------------------------------|-------|------------------------------------|
| enterpriseregistration.contoso.com        | CNAME | enterpriseregistration.Windows.NET |
| enterpriseregistration.Region.contoso.com | CNAME | enterpriseregistration.Windows.NET |

##<a name="configure-automatic-device-registration-for-windows-7-and-windows-81-domain-joined-devices"></a>Configurer les périphériques de domaine joint automatique de périphérique d’enregistrement pour Windows 7 et Windows 8.1

Configurer l’inscription automatique de périphérique pour vos périphériques joint au domaine Windows 7 et Windows 8.1 en utilisant les liens ci-dessous. Assurez-vous que vous avez terminé la configuration requise ci-dessus avant de continuer.

* [Configurer les périphériques du domaine joint automatique de périphérique d’enregistrement pour Windows 8.1](active-directory-conditional-access-automatic-device-registration-windows-8-1.md)

* [Configurer les périphériques du domaine joint automatique de périphérique d’enregistrement pour Windows 7](active-directory-conditional-access-automatic-device-registration-windows7.md)

* [Enregistrement de dispositif automatique avec les périphériques Azure Active Directory for Windows 10 Domain-Joined](active-directory-azureadjoin-devices-group-policy.md)

<a name="additional-notes"></a>Remarques supplémentaires
--------------------------------------------------------------------

Enregistrement de dispositif avec AD Azure propose la plus vaste gamme de fonctionnalités de l’appareil. Avec l’enregistrement de dispositif de publicité Azure, vous pouvez enregistrer des appareils mobiles (BYOD) personnels et société appartenant, périphériques de domaine. Les périphériques peuvent être utilisés avec les services hébergés par exemple Office 365 et services gérés sur site avec AD FS.

Les sociétés qui utilisent des périphériques mobiles et traditionnels ou qui utilisent Office 365, AD Azure, ou autres services de Microsoft doivent être enregistrés les périphériques dans Azure AD à l’aide du service d’enregistrement de dispositif de publicité Azure. Si votre société n’utilise pas les périphériques mobiles et n’utilise pas tous les services Microsoft tels qu’Office 365, AD Azure, ou Microsoft Intune et plutôt hôtes uniquement sur site applications, vous pouvez choisir d’inscrire des périphériques dans Active Directory à l’aide d’ADFS.

Pour en savoir plus sur le déploiement de l’inscription de périphérique avec AD FS [ici](https://technet.microsoft.com/library/dn486831.aspx).

## <a name="additional-topics"></a>Rubriques supplémentaires

- [Vue d’ensemble de l’enregistrement de Active Directory périphérique Azure](active-directory-conditional-access-device-registration-overview.md)
- [Configurer l’inscription automatique pour appareils de joint au domaine Windows 7](active-directory-conditional-access-automatic-device-registration-windows7.md)
- [Configurer l’inscription automatique de périphérique pour les périphériques de joint au domaine Windows 8.1](active-directory-conditional-access-automatic-device-registration-windows-8-1.md)
- [Enregistrement de dispositif automatique avec les périphériques à un domaine d’Azure Active Directory pour Windows 10](active-directory-azureadjoin-devices-group-policy.md)
