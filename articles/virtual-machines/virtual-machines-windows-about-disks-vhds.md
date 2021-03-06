<properties
    pageTitle="Sur les disques et les disques durs virtuels pour Windows VMs | Microsoft Azure"
    description="Découvrez les notions de base des disques et des machines virtuelles de disques durs virtuels pour Windows dans Azure."
    services="virtual-machines-windows"
    documentationCenter=""
    authors="cynthn"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager,azure-service-management"/>

<tags
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/27/2016"
    ms.author="cynthn"/>

# <a name="about-disks-and-vhds-for-azure-virtual-machines"></a>Sur les disques et les disques durs virtuels pour les machines virtuelles Azure

Tout comme n’importe quel autre ordinateur, les ordinateurs virtuels dans Azure utiliser des disques comme un emplacement pour stocker un système d’exploitation, des applications et des données. Tous les ordinateurs virtuels Azure ont au moins deux disques – un disque du système d’exploitation Windows et un disque temporaire. Le disque du système d’exploitation est créé à partir d’une image, et que le disque du système d’exploitation et l’image sont des disques durs virtuels (VHD) stockées dans un compte de stockage Azure. Machines virtuelles peut également avoir d’un ou plusieurs disques de données, qui sont également stockés en tant que disques durs virtuels. Cet article est également disponible pour les [ordinateurs virtuels Linux](virtual-machines-linux-about-disks-vhds.md).

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]



## <a name="operating-system-disk"></a>Disque du système d’exploitation

Chaque machine virtuelle dispose d’un disque du système d’exploitation attaché. Il a enregistré sous la forme d’un disque SATA et étiqueté comme le lecteur C: par défaut. Ce disque a une capacité maximale de 1 023 gigaoctets (Go). 

##<a name="temporary-disk"></a>Disque temporaire

Le disque temporaire est automatiquement créé pour vous. Le disque temporaire est appelé le lecteur D: par défaut et il est utilisé pour stocker le fichier pagefile.sys. 

La taille du disque temporaire varie en fonction de la taille de la machine virtuelle. Pour plus d’informations, consultez les [machines virtuelles de tailles pour Windows](virtual-machines-windows-sizes.md).

>[AZURE.WARNING] Ne pas stocker les données sur le disque temporaire. Il fournit un stockage temporaire pour les applications et les processus et est destiné à stocker uniquement des données tels que les fichiers de page ou de remplacement. Pour remapper ce disque à une lettre de lecteur différente, voir [modification de la lettre de lecteur du disque temporaire Windows](virtual-machines-windows-classic-change-drive-letter.md).

Pour plus d’informations sur la façon dont Azure utilise le disque temporaire, reportez-vous à la section [comprendre le lecteur temporaire Microsoft Azure machines virtuelles en fonctionnement](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

## <a name="data-disk"></a>Disque de données

Un disque de données est lié à une machine virtuelle pour stocker des données d’application ou d’autres données que vous souhaitez conserver un disque dur virtuel. Disques de données sont enregistrés en tant que disques SCSI et sont étiquetés avec une lettre que vous choisissez.  Chaque disque de données a une capacité maximale de 1 023 go. La taille de la machine virtuelle détermine combien de disques données vous pouvez joindre et le type de stockage que vous pouvez utiliser pour héberger les disques.

>[AZURE.NOTE] Pour plus d’informations sur les capacités des ordinateurs virtuels, voir [machines virtuelles tailles pour Windows](virtual-machines-windows-sizes.md).

Azure crée un disque de système d’exploitation lorsque vous créez un ordinateur virtuel à partir d’une image. Si vous utilisez une image qui inclut des disques de données, Azure crée également les disques de données lorsqu’il crée l’ordinateur virtuel. Dans le cas contraire, vous ajoutez des disques de données après avoir créé l’ordinateur virtuel.

Vous pouvez ajouter des disques de données sur une machine virtuelle à tout moment, en **attachant** le disque sur l’ordinateur virtuel. Vous pouvez utiliser un disque dur virtuel que vous avez téléchargés ou copiés sur votre compte de stockage, ou un qui Azure crée pour vous. Attacher un disque de données associe le fichier de disque dur virtuel avec la machine virtuelle en plaçant un « bail » sur le disque dur virtuel et ne peut pas être supprimé à partir du stockage pendant qu’il est encore attaché.

## <a name="about-vhds"></a>Sur les disques durs virtuels

Les disques durs virtuels utilisés dans Azure sont des fichiers .vhd stockés en tant que BLOB de page dans un compte de stockage standard ou premium dans Azure. Pour plus d’informations sur les blobs de page, consultez [blobs de bloc de compréhension et de blobs de page](https://msdn.microsoft.com/library/ee691964.aspx). Pour plus d’informations sur le stockage de la prime, consultez [stockage de prime : stockage hautes performances pour les charges de travail Azure virtual machine](../storage/storage-premium-storage.md).

Azure prend en charge le format de disque dur virtuel de disque fixe. Le format fixe dispose le disque logique linéaire dans le fichier, afin que cet offset de disque X est stocké à l’offset du blob X. Un pied de page à la fin de l’objet blob décrit les propriétés de ce dernier. Souvent, les déchets de format fixe d’espace, car la plupart des disques ont de grandes plages inutilisés dans les. Cependant, Azure stocke les fichiers .vhd dans un format incomplet, vous recevez les avantages des deux disques fixes et dynamiques en même temps. Pour plus d’informations, consultez [mise en route avec les disques durs virtuels](https://technet.microsoft.com/library/dd979539.aspx).

Tous les fichiers .vhd Azure que vous souhaitez utiliser comme source pour créer des disques ou des images sont en lecture seule. Lorsque vous créez un disque ou une image, Azure effectue des copies des fichiers .vhd. Ces copies peuvent être en lecture seule ou en lecture-écriture, selon l’utilisation du disque dur virtuel.

Lorsque vous créez un ordinateur virtuel à partir d’une image, Azure crée un disque de l’ordinateur virtuel qui est une copie du fichier .vhd source. Afin de protéger contre la suppression accidentelle, Azure place un bail sur n’importe quel fichier .vhd de source qui est utilisé pour créer une image, d’un disque du système d’exploitation ou d’un disque de données.

Avant de supprimer un fichier .vhd de source, vous devez supprimer le bail en supprimant le disque ou l’image. Vous pouvez supprimer la machine virtuelle, le disque du système d’exploitation, et le fichier .vhd de source en une seule fois par suppression de l’ordinateur virtuel et supprimer tous les disques associés. Toutefois, la suppression d’un fichier .vhd qui est une source pour un disque de données requiert plusieurs étapes dans un ordre de jeu. Tout d’abord vous détachez le disque à partir de l’ordinateur virtuel, puis supprimez le disque, puis supprimez le fichier .vhd.

>[AZURE.WARNING] Si vous supprimez un fichier .vhd de source de stockage, ou que vous supprimez votre compte de stockage, Microsoft ne peut pas récupérer ces données pour vous.



## <a name="next-steps"></a>Étapes suivantes
-  [Attacher un disque](virtual-machines-windows-attach-disk-portal.md) à ajouter un stockage supplémentaire pour votre ordinateur virtuel.
-  [Télécharger une image de machine virtuelle de Windows Azure](virtual-machines-windows-upload-image.md) à utiliser lors de la création d’une nouvelle machine virtuelle.
-  [Modifier la lettre de lecteur du disque temporaire Windows](virtual-machines-windows-classic-change-drive-letter.md) afin que votre application peut utiliser le lecteur D: pour les données.
