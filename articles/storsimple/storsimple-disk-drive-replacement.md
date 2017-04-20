<properties 
   pageTitle="Remplacer un disque sur un périphérique StorSimple | Microsoft Azure"
   description="Explique comment remplacer un disque dans un boîtier principal StorSimple ou un boîtier EBOD."
   services="storsimple"
   documentationCenter=""
   authors="alkohli"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="08/17/2016"
   ms.author="alkohli" />

# <a name="replace-a-disk-drive-on-your-storsimple-device"></a>Remplacer un lecteur de disque sur le périphérique StorSimple

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel explique comment vous pouvez supprimer et remplacer un lecteur de disque dur défectueux ou défaillant sur un périphérique Microsoft Azure StorSimple. Pour remplacer un lecteur de disque, vous devez :

- Dégager le verrou antitamper

- Retirez le lecteur de disque

- Installez le lecteur de disque de remplacement

>[AZURE.IMPORTANT] Avant le retrait et le remplacement d’un disque dur, passez en revue les informations de sécurité dans le [remplacement de composants matériels StorSimple](storsimple-hardware-component-replacement.md).

## <a name="disengage-the-antitamper-lock"></a>Dégager le verrou antitamper

Cette procédure explique comment les verrous antitamper sur votre périphérique de StorSimple peuvent être embrayés ou débrayés lorsque vous remplacez les lecteurs de disque. Les verrous antitamper sont montés dans les poignées du support de lecteur, et ils sont accessibles via une petite ouverture dans la section du loquet de la poignée. Les lecteurs sont fournis avec les verrous en position verrouillée.

#### <a name="to-unlock-the-antitamper-lock"></a>Pour déverrouiller le verrouillage antitamper

1. Soigneusement insérer la touche de verrouillage (un tournevis « contre les falsifications » T10 fourni par Microsoft) à l’ouverture de la poignée et le socket. 

    >[AZURE.NOTE] Si le verrou antitamper est activé, l’indicateur rouge est visible dans l’ouverture de l’objectif.

    ![Disque verrouillé](./media/storsimple-disk-drive-replacement/IC741056.png)

    **Figure 1** Verrou anti fraude engagé

  	|Étiquette|Description|
  	|:----|:----------|
  	|1|Ouverture de l’indicateur|
  	|2|Verrouillage antitamper|

2. Rotation de la clé dans une direction sens antihoraire jusqu'à ce que le voyant rouge n’est pas visible dans l’ouverture au-dessus de la touche.

3. Supprimez la clé.

    ![Lecteur de disquette non verrouillée](./media/storsimple-disk-drive-replacement/IC741057.png)

    **Figure 2** Lecteur de disquette non verrouillée

4. Le lecteur de disque peut maintenant être enlevé.

Suivez les étapes dans l’ordre inverse de s’engager le verrou.

## <a name="remove-the-disk-drive"></a>Retirez le lecteur de disque

Votre périphérique StorSimple prend en charge une configuration d’espaces de stockage de type 10 RAID. Cela signifie qu’il peut fonctionner normalement avec un disque défectueux, le lecteur à l’état solide (SSD), ou le lecteur de disque dur (HDD). 

>[AZURE.IMPORTANT]
>
>- Si votre système dispose de plus d’un disque défectueux, ne supprimez pas plusieurs SSD ou disque dur du système à n’importe quel point dans le temps. Cela peut entraîner une perte de données.
>
>- Assurez-vous que vous placez un remplacement SSD dans un emplacement qui contenait auparavant une SSD. De même, placez un disque dur de remplacement dans un emplacement contenant déjà un disque dur.
>
>- Dans le portail Azure classique, les emplacements sont numérotés de 0 – 11. Par conséquent, si le portail montre que dans l’emplacement 2 Échec d’un disque, sur le périphérique, recherchez le disque défectueux dans le troisième emplacement à partir de l’angle supérieur gauche.

Les lecteurs peuvent être enlevés et remplacés pendant que le système fonctionne.

#### <a name="to-remove-a-drive"></a>Pour supprimer un lecteur

1. Pour identifier le disque défaillant, le portail classique Azure, accédez aux **périphériques** > **Maintenance** > **État du matériel**. Dans la mesure où un disque peut échouer dans le boîtier principal ou d’un boîtier EBOD (si vous utilisez un modèle 8600), vérifiez l’état des disques sous **Composants partagés** et sous **Composants partagés d’enceinte de EBOD**. Un disque défaillant dans un boîtier est affiché avec un état rouge.

2. Recherchez les disques situé à l’avant du boîtier principal ou le boîtier EBOD. 

3. Si le disque est déverrouillé, passez à l’étape suivante. Si le disque est verrouillé, déverrouillez-le en suivant la procédure de [dégager le verrou antitamper](#disengage-the-antitamper-lock).

4. Appuyez sur le loquet de noir sur le module de support du lecteur et tirez la poignée du support de lecteur de retirer à l’avant du châssis. 

    ![Relâcher la poignée du lecteur de disque](./media/storsimple-disk-drive-replacement/IC741051.png)

    **Figure 3** Libération du handle du lecteur

5. Lors de la poignée du support de lecteur est entièrement développée, faites glisser le support de disque du châssis. 

    ![Glissement du disque sur le lecteur de disque](./media/storsimple-disk-drive-replacement/IC741052.png)
    
    **Figure 4** Faisant glisser le disque dur sur le support

## <a name="install-the-replacement-disk-drive"></a>Installez le lecteur de disque de remplacement

Une fois un lecteur a échoué dans votre appareil StorSimple et que vous l’avez supprimé, procédez comme suit pour le remplacer par un nouveau lecteur.

#### <a name="to-insert-a-drive"></a>Pour insérer un lecteur

1. Assurez-vous que la poignée du support de lecteur est entièrement développée, comme illustré dans l’image suivante.

    ![Lecteur de disque avec poignée étendue](./media/storsimple-disk-drive-replacement/IC741044.png)

    **Figure 5** Lecteur avec poignée étendue

2. Faites glisser le support de disque dans le châssis. 

    ![Disque décalée dans le support de disque](./media/storsimple-disk-drive-replacement/IC741045.png)

    **Figure 6**  Glisser le support de disque dans le châssis

3. Avec le support du disque inséré, fermez la poignée du support de disque tout en continuant à pousser le support de disque dans le châssis, jusqu'à ce que la poignée du support de lecteur s’aligne en position verrouillée.

4. Utilisez la touche qui a été fournie par Microsoft (tournevis de Torx contre les falsifications) pour sécuriser la poignée du support en place en activant l’un quart de tour la vis de verrouillage dans le sens horaire.

5. Vérifiez que le remplacement a réussi et que le lecteur est opérationnel en l’accès au portail classique Azure et en accédant à la **Maintenance** > **État du matériel**. Sous **Composants partagés** ou des **Composants partagés d’enceinte de EBOD**, l’état du disque doit être vert, indiquant qu’elle est saine.

    >[AZURE.NOTE] Elle peut prendre plusieurs heures pour l’état du disque deviennent verts après le remplacement.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le [remplacement de composants matériels StorSimple](storsimple-hardware-component-replacement.md).
