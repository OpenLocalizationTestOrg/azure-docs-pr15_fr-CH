<properties
   pageTitle="Création et enregistrement du compte éditeur | Microsoft Azure"
   description="Instructions pour la création d’un compte Microsoft Developer afin que, lors de l’approbation, vous pouvez vendre plusieurs offrent des types sur Azure Marketplace."
   services="Azure Marketplace"
   documentationCenter=""
   authors="HannibalSII"
   manager="hascipio"
   editor=""/>

<tags
   ms.service="marketplace"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/18/2016"
   ms.author="hascipio"/>

# <a name="create-a-microsoft-developer-account"></a>Créer un compte Microsoft Developer
Cet article vous guide à travers la création de compte nécessaires et le processus d’inscription pour devenir un agréé Microsoft Developer à Azure Marketplace.

## <a name="1-create-a-microsoft-account"></a>1. Créez un compte Microsoft
Pour démarrer le processus de publication, vous devez créer un compte Microsoft. Ce compte sera utilisé pour enregistrer dans le **Centre de développement Microsoft** et le **Portail de publication Azure**. Vous devez avoir qu’un seul compte Microsoft pour vos offres d’Azure Marketplace. Il ne doit pas être spécifique aux services ou offres.

L’adresse qui constitue le nom d’utilisateur doit se trouver sur votre domaine et contrôlée par votre équipe informatique. Toutes les activités connexes publication doivent être effectuées par le biais de ce compte.

  >[AZURE.WARNING] Mots tels que **« Azure »** et **« Microsoft »** ne sont pas pris en charge pour l’enregistrement de compte Microsoft. Évitez d’utiliser ces mots pour terminer la création de compte et le processus d’inscription.

### <a name="instructions"></a>Instructions

1. Créer une liste de distribution (DL) ou d’un groupe de sécurité (SG) dans le domaine de votre société. À l’aide d’une liste de distribution permet à plusieurs personnes à recevoir des notifications par courrier électronique qui sont importantes pour la communication des informations de paiement. Elle garantit également que la propriété du compte Microsoft peut être transférée et qu’il n’est pas liée à une même personne.
Suivez les instructions ci-dessous.

    1. Ajoutez votre équipe arrivant à la liste de distribution.
    2. Vérifiez que DL/SG est une adresse de messagerie active et qu’il est en mesure de recevoir des e-mails, car le paiement, informations fiscales et création de rapports va être routés via ce compte.
    3. Nous recommandons d’utiliser quelque chose comme marketplace@partnercompany.com comme l’adresse e-mail de la liste de distribution/SG.

2. Ouvrez un nouveau Incognito de Chrome ou Internet Explorer InPrivate session de navigation pour vous assurer que vous n'êtes pas connecté à un compte existant.
3. Enregistrer la liste de distribution créé à l’étape 1 en tant que Microsoft compte en utilisant le lien [https://signup.live.com/signup.aspx](https://signup.live.com/signup.aspx). Suivez les instructions ci-dessous.

    1. Au cours de l’enregistrement de votre compte comme un compte Microsoft, vous devez fournir un numéro de téléphone valide pour le système pour vous envoyer un code de vérification de compte sous la forme d’un message texte ou appel automatique.
    2. Au cours de l’enregistrement de votre compte comme un compte Microsoft, vous devez fournir un id e-mail valide pour la réception d’un e-mail automatique pour la vérification du compte.

4. Vérifiez l’adresse de courrier électronique envoyé à la liste de distribution.
5. Vous êtes maintenant prêt à utiliser le nouveau compte de Microsoft dans Microsoft Developer Center.

## <a name="2-create-your-microsoft-developer-center-account"></a>2. Créez votre compte Microsoft Developer Center
Le Microsoft Developer Center est utilisé pour enregistrer les informations de société une seule fois. L’abonné doit être un représentant valide de la société et doit fournir des informations personnelles comme un moyen de valider leur identité. L’enregistrement de personne doit utiliser un compte de Microsoft qui est partagé de la société, **et le compte doit être utilisé dans le portail de publication Azure.** Vous devez vérifier que votre société ne possède pas déjà un compte Microsoft Developer Center avant d’essayer d’en créer un. Au cours du processus, nous collecter les informations d’adresse, les informations de compte bancaire et informations fiscales. Ils sont généralement obtenues à partir des contacts financiers ou des entreprises.

> [AZURE.IMPORTANT] Vous devez remplir les éléments du profil développeur pour progresser dans les différentes phases du déploiement et de création de l’offre.


| Profil de développeur | Pour démarrer le projet | Mise en attente | Publier libre et modèle de solution | Publication commerciale |
|----|----|----|----|----|
|Enregistrement de la société | Doit avoir | Doit avoir | Doit avoir | Doit avoir |
|ID de profil de taxe | Facultatif | Facultatif | Facultatif | Doit avoir |
|Compte bancaire | Facultatif | Facultatif | Facultatif | Doit avoir |


> [AZURE.NOTE] Mettre la licence votre propre (BYOL) est pris en charge uniquement pour les machines virtuelles et est considérée comme une offre **gratuite** .


### <a name="register-your-company-account"></a>Enregistrer votre compte société
1. Ouvrez un nouveau Internet Explorer InPrivate ou Incognito Chrome session de navigation pour vous assurer que vous n'êtes pas connecté à un compte personnel.

2. Accédez à [http://dev.windows.com/registration?accountprogram=azure](http://dev.windows.com/registration?accountprogram=azure) pour vous inscrire en tant que vendeur dans le centre de développement. Veuillez lire la Remarque importante ci-dessous avant de continuer.

    >[AZURE.IMPORTANT] Assurez-vous que la courrier électronique id ou liste de distribution (une liste de distribution est recommandée de supprimer la dépendance de personnes) que vous utilisez pour l’enregistrement dans le centre de développement à d’abord été enregistré en tant qu’un compte Microsoft. Si ce n’est pas le cas, puis enregistrez à l’aide de ce [lien](https://signup.live.com/signup?uaid=e479342fe2824efeb0c3d92c8f961fd3&lic=1). En outre, **toute électronique id sous le domaine de la société Microsoft, c'est-à-dire @microsoft.com ne peut pas être utilisé** pour l’inscription du centre de développement.

    ![dessin][img-signin]

3. Terminez l’Assistant « Nous aider à protéger votre compte », qui vérifie votre identité par téléphone numéro ou adresse e-mail.

    ![dessin][img-verify]

4. Dans la section « Informations de compte d’enregistrement », sélectionnez votre **pays/région du compte** dans le menu déroulant.

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_04.png)

    > [AZURE.WARNING] **« Donneur d’ordre-de » pays :** Avant de pouvoir vendre vos services sur le marché d’Azure, votre entité enregistrée doit être parmi les pays agréés « donneur d’ordre-de » ci-dessus. Cette restriction est pour des raisons de paiement et de la fiscalité. Nous recherchons activement à développer cette liste de pays dans un avenir proche, soyez donc attentifs. Pour plus d’informations, consultez les [règles de participation Marketplace](http://go.microsoft.com/fwlink/?LinkID=526833).

5. Sélectionnez votre Type de compte « » en tant que **société** , puis sur le bouton **suivant** .

    > [AZURE.IMPORTANT] Pour mieux comprendre les types de comptes et qui vous convient le mieux à choisir, consultez page de [frais, les emplacements et les types de compte](https://msdn.microsoft.com/library/windows/apps/jj863494.aspx)

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_05.png)

6. Permet d’entrer l' **éditeur de nom complet**, en général le nom de votre société.

    > [AZURE.TIP] Nom d’affichage de l’éditeur entré dans le centre de développement n’apparaît pas sur le marché d’Azure dès que votre offre est répertorié. Mais il est obligatoire pour terminer le processus d’inscription.

7. Entrez les **informations de Contact** pour la vérification du compte.

    > [AZURE.IMPORTANT] Vous devez fournir des informations de contact précises car il est utilisé dans notre processus de vérification de votre société à être approuvés dans le centre de développement.

8. Entrez les informations de contact de l' **Approbateur de la société**. Approbateur de la société est la personne qui peut vérifier que vous êtes autorisé à créer un compte dans le centre de développement pour le compte de votre organisation. Cliquez sur **suivant** pour passer à la **section « paiement »** une fois que vous avez terminé.

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_08.png)

9. Entrez vos informations de paiement pour votre compte. Si vous avez un code de promotion qui couvre le coût de l’enregistrement, vous pouvez le saisir ici. Dans le cas contraire, fournir vos informations de carte de crédit (ou PayPal pris en charge sur les marchés). Lorsque vous avez terminé, cliquez sur **suivant** pour passer à l' **« Écran de révision »**.

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgRegisterCo_09.png)

10. Passez en revue les informations de compte et de vérifier que tout est correct. Ensuite, lire et accepter les termes et conditions de l' [Accord de Publisher Microsoft Azure Marketplace](http://go.microsoft.com/fwlink/?LinkID=699560). Cochez la case pour indiquer l’avoir lu et accepté les termes du contrat.

11. Cliquez sur **Terminer** pour confirmer votre inscription. Nous vous enverrons un message de confirmation à votre adresse e-mail.

12. Si vous envisagez de publier uniquement les offres gratuites, cliquez sur **accéder au portail de publication Azure Marketplace** et vous pouvez ignorer la section 3 de ce document, [enregistrez votre compte sur le portail de publication](#3-register-your-account-in-the-publishing-portal).

Si vous envisagez de publier commercial offre (par exemple, les offres de Machine virtuelle avec un modèle de facturation horaire), cliquez sur **mettre à jour les informations de votre compte** où vous devez remplir la taxe et les informations bancaires dans votre compte Centre de développement.

Si vous préférez mettre à jour les informations fiscales et bancaires ultérieurement, puis vous pouvez passer à la section suivante, par exemple, la section 3 de ce document, [enregistrez votre compte sur le portail de publication](#3-register-your-account-in-the-publishing-portal)et se reconnecter ultérieurement à l’aide des liens dans le portail de publication Azure.

> [AZURE.IMPORTANT] Dans le cas des offres commerciales, vous ne serez pas en mesure de distribuer votre offre à la production sans les informations de taxe et le compte bancaire à la fin.

Si vous préférez mettre à jour les informations fiscales et bancaires ultérieurement, vous pouvez passer à la section 3, [enregistrez votre compte sur le portail de publication](#3-register-your-account-in-the-publishing-portal)et se reconnecter ultérieurement à l’aide des liens dans le portail de publication Azure.

### <a name="add-tax-and-banking-information"></a>Ajouter des taxes et des informations bancaires
 Si vous souhaitez publier des offres commerciales d’achat, vous devez également ajouter le paiement et les informations fiscales et soumettre pour validation dans le centre de développement. Si vous publiez les offres gratuites uniquement (ou offre de BYOL), vous n’avez pas besoin d’ajouter ces informations. Vous pouvez les ajouter ultérieurement, mais il prend un certain temps pour valider les informations de taxe. Si vous savez que vous proposera des offres commerciales à l’achat, nous vous recommandons d’ajouter dès que possible.

**Informations bancaires**

1. Connectez-vous au [Centre de développement Microsoft](http://dev.windows.com/registration?accountprogram=azure) avec votre compte Microsoft.

2. Cliquez sur **le compte de paiement** dans le menu de gauche, sous **méthode de paiement choisir** , cliquez sur **compte bancaire** ou **PayPal**.

    > [AZURE.IMPORTANT] Si vous avez des offres commerciales que le client achète sur le marché, c’est le compte où vous recevez le paiement pour ces achats.

3. Entrez les informations de paiement, puis cliquez sur **Enregistrer** lorsque vous êtes satisfait du résultat.

    > [AZURE.IMPORTANT] Si vous devez mettre à jour ou modifier votre compte de paiement, suivez les mêmes étapes ci-dessus, remplacer les informations en cours avec les nouvelles informations. La modification de votre compte de paiement peut différer de vos paiements par jusqu'à un cycle de paiement. Ce retard se produit, car nous devons vérifier la modification du compte, comme nous l’avons fait lorsque vous configurez le compte de paiement. Vous allez tout en étant payés pour le montant total après que votre compte a été vérifié ; tous les paiements dus pour le paiement en cours cycle sera ajouté à la suivante.

4. Cliquez sur **suivant**.

**Informations sur les taxes**

1. Vous connecter au [Centre de développement Microsoft](http://dev.windows.com/registration?accountprogram=azure) avec votre compte Microsoft (si nécessaire).

2. Dans le menu de gauche, cliquez sur **profil de taxe** .

3. Dans la page **configurer votre formulaire de taxe** , sélectionnez le pays ou la région où vous avez résidence permanente, puis sélectionnez le pays ou la région où vous maintenez citoyenneté principale. Cliquez sur **suivant**.

4. Entrez vos codes de taxes, puis cliquez sur **suivant**.

> [AZURE.WARNING] Vous ne serez pas en mesure de distribuer à votre commercial offre sans les informations de taxe et le compte bancaire de votre compte Microsoft Developer Center à la fin de la production.

Si vous rencontrez des problèmes avec l’inscription du Centre pour développeurs, veuillez vous connecter un ticket de support comme indiqué ci-dessous

1. Atteindre le lien de support [https://developer.microsoft.com/windows/support](https://developer.microsoft.com/windows/support)
2. Sous la section **Nous contacter** , cliquez sur le bouton **soumettre un incident** (comme illustré dans la capture d’écran ci-dessous)

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgAddTax_02.png)

3. Choisissez « Aide avec centre de développement « en tant que **type de problème** et « publier et gérer des applications » comme **catégorie**. Ensuite cliquez sur le bouton « Démarrer messagerie ».

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgAddTax_03.png)

4. Vous recevrez une page de connexion. Utilisez n’importe quel signe de compte Microsoft dans. Si vous ne disposez pas d’un compte Microsoft puis en créer un à l’aide de ce [lien](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1).
5. Remplissez les détails du problème et subit le ticket en cliquant sur le bouton **Envoyer** .

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgAddTax_05.png)

## <a name="3-register-your-account-in-the-publishing-portal"></a>3. Enregistrez votre compte sur le portail de publication
Le [portail de publication](http://publish.windowsazure.com) permet de publier et de gérer vos offres.

1. Ouvrez un nouveau Incognito de Chrome ou Internet Explorer InPrivate session de navigation pour vous assurer que vous n'êtes pas connecté à un compte personnel.

2. Accédez à [http://publish.windowsazure.com](http://publish.windowsazure.com).

3. Si vous êtes un nouvel utilisateur et la connexion à la publication portail pour la première fois, puis vous devez vous connecter avec le même id de courrier électronique à laquelle votre compte Centre de développement est enregistré. De cette manière votre compte Centre de développement et la publication de portail compte seront liés entre eux. Vous pourrez ajouter ultérieurement d’autres membres de la société, qui travaillent sur l’application, comme un co-admin dans la publication portail en suivant les étapes ci-dessous.

Si vous devenez un co-admin dans la publication portail, puis vous pouvez vous connecter avec votre compte co-admin.

  > [AZURE.TIP] Les stratégies de participation sont décrites sur le [site Web Azure](https://azure.microsoft.com/support/legal/marketplace/participation-policies/).

## <a name="4-steps-to-add-a-co-admin-in-the-publishing-portal"></a>4. les étapes pour ajouter une co-admin dans la publication portal
**qu’En supposant que vous êtes l’administrateur,** indiquées ci-dessous sont les étapes d’ajout d’un co-Admin.

>[AZURE.NOTE] **Pour les nouveaux utilisateurs,** avant d’ajouter une co-admin dans la publication portail, assurez-vous que vous avez créé au moins une application dans la publication portal. Il est nécessaire que l’onglet **éditeurs** apparaissent seulement après avoir créé au moins une application dans la publication portal.

1. Assurez-vous que l’id de courrier électronique co-admin est un account(MSA) de Microsoft. Si ce n’est pas le cas, enregistrer en tant qu’un MSA à l’aide de ce [lien](https://signup.live.com/signup?uaid=0089f09ccae94043a0f07c2aaf928831&lic=1).
2. Vérifiez qu’il existe au moins une application sous le compte administrateur avant d’essayer d’ajouter une co-Admin.
3. Une fois les étapes ci-dessus sont effectuées, connexion d’accès à la publication portail avec l’id de courrier électronique de co-admin, puis se déconnecter.
4. Maintenant la connexion à la publication portail avec l’id de courrier électronique admin.
5. Accédez aux éditeurs -> sélectionnez votre compte -> administrateurs -> ajouter le co-admin (capture d’écran ci-dessous)

  ![dessin](media/marketplace-publishing-accounts-creation-registration/imgAddAdmin_05.png)

## <a name="5-steps-to-delete-a-co-admin-in-the-publishing-portal"></a>5. comment supprimer un co-admin dans la publication portal
**qu’En supposant que vous êtes l’administrateur,** indiquées ci-dessous sont les étapes permettant de supprimer un co-Admin.

1. Connexion d’accès à la publication portail avec l’id de courrier électronique admin.
2. Accédez aux **éditeurs** -> sélectionnez votre compte -> **administrateurs** -> **Co-Admins**.
3. Cliquez sur le bouton **X** ensuite pour le co-admin que vous souhaitez supprimer d’assurances (capture d’écran ci-dessous).

    ![dessin](media/marketplace-publishing-accounts-creation-registration/imgDeleteAdmin_03.png)

## <a name="next-steps"></a>Étapes suivantes
Maintenant que votre compte est créé et enregistré, assurez-vous de vous répondre ou répondre à tous les composants requis pour publier votre offre en passant en revue les [conditions préalables non techniques](marketplace-publishing-pre-requisites.md)non techniques.

## <a name="see-also"></a>Voir aussi
- [Mise en route : comment publier une offre sur le marché d’Azure](marketplace-publishing-getting-started.md)

[img-msalive]:media/marketplace-publishing-accounts-creation-registration/creating-msa-account-msa-live.jpg
[img-email]:media/marketplace-publishing-accounts-creation-registration/creating-msa-account-msa-verifyemail.jpg
[img-sd-url]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-incognito.jpg
[img-signin]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-login.jpg
[img-verify]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-verify.jpg
[img-sd-top]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-acc-details.jpg
[img-sd-info]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal.jpg
[img-sd-type]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-acc-type.jpg
[img-sd-mktg1]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-det1.jpg
[img-sd-mktg2]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-det2.jpg
[img-sd-addr]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-comp-add.jpg
[img-sd-legal]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-personal-cmp.jpg
[img-sd-submit]:media/marketplace-publishing-accounts-creation-registration/seller-dashboard-approval.jpg

[link-msdndoc]: https://msdn.microsoft.com/library/jj552460.aspx
[link-sellerdashboard]: http://sellerdashboard.microsoft.com/
[link-pubportal]: https://publish.windowsazure.com
[link-single-vm]:marketplace-publishing-vm-image-creation.md
[link-single-vm-prereq]:marketplace-publishing-vm-image-creation-prerequisites.md
[link-multi-vm]:marketplace-publishing-solution-template-creation.md
[link-multi-vm-prereq]:marketplace-publishing-solution-template-creation-prerequisites.md
[link-datasvc]:marketplace-publishing-data-service-creation.md
[link-datasvc-prereq]:marketplace-publishing-data-service-creation-prerequisites.md
[link-devsvc]:marketplace-publishing-dev-service-creation.md
[link-devsvc-prereq]:marketplace-publishing-dev-service-creation-prerequisites.md
[link-pushstaging]:marketplace-publishing-push-to-staging.md
