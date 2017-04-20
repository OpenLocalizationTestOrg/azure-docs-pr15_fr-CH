<properties
    pageTitle="Connecter les ordinateurs virtuels de Linux dans un service en nuage | Microsoft Azure"
    description="Connecter une machine virtuelle Linux créée avec le modèle classique de déploiement d’un service en nuage Azure ou d’un réseau virtuel."
    services="virtual-machines-linux"
    documentationCenter=""
    authors="cynthn"
    manager="timlt"
    editor=""
    tags="azure-service-management"/>

<tags
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="07/06/2016"
    ms.author="cynthn"/>

# <a name="connect-linux-virtual-machines-created-with-the-classic-deployment-model-with-a-virtual-network-or-cloud-service"></a>Se connecter à une machine virtuelle Linux créée avec le modèle de déploiement classique avec un service de nuage ou de réseau virtuel

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]

Linux une machine virtuelle créée avec le modèle classique de déploiement sont toujours placée dans un service en nuage. Le service en nuage agit comme un conteneur et fournit un ensemble de points de terminaison pour accéder à l’ordinateur virtuel via Internet, une adresse IP publique et un nom DNS publique unique. Le service en nuage peut être dans un réseau virtuel, mais qui n’est pas une obligation. Vous pouvez également [connecter des machines virtuelles de Windows avec un service de nuage ou de réseau virtuel](virtual-machines-windows-classic-connect-vms.md).

Si un service en nuage n’est pas dans un réseau virtuel, il s’agit d’un service en nuage *autonome* . Les ordinateurs virtuels d’un service en nuage autonome peut communiquer uniquement avec d’autres machines virtuelles à l’aide des noms DNS publics des autres machines virtuelles, et que le trafic transite sur Internet. Si un réseau virtuel est un service en nuage, les ordinateurs virtuels de ce service en nuage peut communiquer avec tous les autres ordinateurs virtuels dans le réseau virtuel sans envoyer tout le trafic sur Internet.

Si vous placez vos ordinateurs virtuels dans le même service de cloud autonome, vous servira d’équilibrage de charge et de jeux de disponibilité. Pour plus d’informations, consultez [Gérer la disponibilité des ordinateurs virtuels](virtual-machines-linux-manage-availability.md)et les [ordinateurs virtuels d’équilibrage de charge](virtual-machines-linux-load-balance.md) . Toutefois, vous ne peut pas organiser les ordinateurs virtuels sur des sous-réseaux ou un service en nuage autonome de se connecter à votre réseau local. Voici un exemple :

[AZURE.INCLUDE [virtual-machines-common-classic-connect-vms](../../includes/virtual-machines-common-classic-connect-vms.md)]

## <a name="next-steps"></a>Étapes suivantes

Après avoir créé un ordinateur virtuel, il est judicieux [d’Ajouter un disque de données](virtual-machines-linux-classic-attach-disk.md) pour vos services et vos charges de travail ont un emplacement pour stocker des données. 



