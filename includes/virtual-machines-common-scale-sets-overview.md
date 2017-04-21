

Pour les applications nécessitant une mise à l’échelle les ressources de calcul et en échelle opérations sont implicitement équilibrées entre des domaines de panne et de la mise à jour. Pour une introduction à l’échelle de l’ordinateur virtuel jeux de faire référence à récente [annonce de blog Azure](https://azure.microsoft.com/blog/azure-vm-scale-sets-public-preview/).

Regardez ces vidéos pour en savoir plus sur les jeux d’échelle de machine virtuelle :

 - [Mark Russinovich traite des jeux d’échelle Azure](https://channel9.msdn.com/Blogs/Regular-IT-Guy/Mark-Russinovich-Talks-Azure-Scale-Sets/)  

 - [Échelle de l’ordinateur virtuel définit avec Guy Bowerman](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-191-Virtual-Machine-Scale-Sets-with-Guy-Bowerman)

## <a name="creating-and-managing-vm-scale-sets"></a>Création et gestion d’échelle de la machine virtuelle définit

Échelle de machine virtuelle jeux peuvent être définies et déploiement à l’aide de modèles JSON et [Autres API](https://msdn.microsoft.com/library/mt589023.aspx) simplement comme des ordinateurs virtuels de gestionnaire de ressources Azure. Par conséquent, les méthodes de déploiement standard Azure le Gestionnaire de ressources peuvent être utilisés. Pour plus d’informations sur les modèles, consultez [modèles de création d’un gestionnaire de ressources Azure](../articles/resource-group-authoring-templates.md).

Vous trouverez un ensemble de modèles d’exemple pour les jeux d’échelle de machine virtuelle dans le référentiel de GitHub de modèles de démarrage rapide d’Azure ici :

[https://github.com/Azure/azure-quickstart-templates](https://github.com/Azure/azure-quickstart-templates) - recherchez des modèles avec _vmss_ dans le titre.

Dans les pages de détail de ces modèles, vous verrez un bouton qui est lié à la fonctionnalité de déploiement de portail. Pour déployer l’ensemble d’échelle de machine virtuelle, cliquez sur le bouton, puis renseignez à tous les paramètres qui sont requis dans le portail. Si vous ne savez pas si une ressource prend en charge les cas supérieur ou mixte, il est plus sûr de toujours utiliser les valeurs de paramètre de minuscules. Il existe également une dissection vidéo pratique d’un modèle VM échelle jeu ici :

[Dissection du modèle la valeur Echelle de machine virtuelle](https://channel9.msdn.com/Blogs/Windows-Azure/VM-Scale-Set-Template-Dissection/player)

## <a name="scaling-a-vm-scale-set-out-and-in"></a>Mise à l’échelle d’une échelle de machine virtuelle définie et dans

Pour augmenter ou diminuer le nombre de machines virtuelles dans un ensemble d’échelle de machine virtuelle, il suffit de modifier la propriété _capacity_ et de redéployer le modèle. Cette simplicité la rend facile à écrire votre propre couche mise à l’échelle personnalisée si vous souhaitez définir des événements échelle personnalisée qui ne sont pas pris en charge par autoscale Azure.

Si vous redéployez un modèle pour modifier la capacité, vous pouvez définir un modèle beaucoup plus petit inclut uniquement le point de stock et de la capacité de mise à jour. Voici un exemple de ce : [https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-scale-in-or-out/azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-linux-nat/azuredeploy.json).

Pour les différentes étapes qui crée un ensemble d’échelle qui est automatiquement mis à l’échelle, consultez [Automatiquement Machines à l’échelle dans un jeu d’échelle de Machine virtuelle](../articles/virtual-machines/virtual-machines-windows-vmss-powershell-creating.md)

## <a name="monitoring-your-vm-scale-set"></a>Surveillance de votre échelle de machine virtuelle définie

Il est actuellement recommandé de qu'utiliser l' [Explorateur de ressources Azure](https://resources.azure.com) pour afficher les jeux d’échelle VM. Jeux d’échelle VM sont une ressource sous Microsoft.Compute, à partir de ce site vous pouvez les voir en développant les liens suivants :

    subscriptions -> your subscription -> resourceGroups -> providers -> Microsoft.Compute -> virtualMachineScaleSets -> your VM scale set -> etc.

## <a name="vm-scale-set-scenarios"></a>Échelle de machine virtuelle définie des scénarios

Cette section répertorie quelques scénarios de jeu échelle VM classiques. Certains services Azure de niveau supérieur (par exemple, lot, Service de tissu, Azure conteneur Service) utilise ces scénarios.

 - **RDP / instances la valeur SSH à l’échelle de l’ordinateur virtuel** - jeu d’échelle A VM est créé à l’intérieur d’un VNET et VM individuelles dans n’est pas alloués des adresses IP publiques. C’est une bonne chose parce que vous ne voulez généralement que la surcharge de gestion et dépenses d’attribution des adresses IP distinctes à toutes les ressources sans état dans votre grille de calcul, et vous pouvez facilement vous connecter à ces ordinateurs virtuels à partir d’autres ressources dans votre VNET, y compris celles qui ont des adresses IP publiques telles que les équilibreurs de charge ou les machines virtuelles d’autonome.

 - **Se connecter à des ordinateurs virtuels à l’aide de règles NAT** - vous pouvez créer une adresse IP publique, assignez-le à un équilibreur de charge et définir les règles de trafic entrant NAT qui mappent un port sur l’adresse IP à un port sur un ordinateur virtuel dans le jeu d’échelle de machine virtuelle. Par exemple

    Adresse IP publique -> Port 50000 -> vmss\_0 -> Port 22 publique IP - > Port 50001 -> vmss\_1 -> Port 22 publique IP - > voie 50002 -> vmss\_2 -> Port 22

    Voici un exemple de création d’un ensemble d’échelle de machine virtuelle qui utilise des règles NAT pour activer des connexions SSH pour chaque ordinateur virtuel dans une échelle définie à l’aide d’un seul IP publique : [https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-linux-nat)

    Voici un exemple de faire la même chose avec RDP et Windows : [https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-nat](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-windows-nat)

 - La valeur de **se connecter à des ordinateurs virtuels à l’aide d’un « jumpbox »** - si vous créez une échelle de machine virtuelle et répond à une machine virtuelle dans la même VNET, la machine virtuelle autonome et le jeu d’échelle VM qu'ordinateurs virtuels peuvent se connecter à un autre à l’aide de son adresse IP interne autonome tel que défini par le VNET/sous-réseau. Si vous créez une adresse IP publique et l’assignez à la machine virtuelle autonome vous pouvez RDP ou SSH à la machine virtuelle autonome puis connecter vos instances de jeu échelle machine virtuelle à partir de cet ordinateur. À ce stade, vous pouvez remarquer qu’un simple ensemble d’échelle VM est intrinsèquement plus sûr qu’une machine virtuelle autonome simple avec une adresse IP publique dans sa configuration par défaut.

    Pour obtenir un exemple de cette approche, ce modèle permet de créer un cluster simple composé d’un système autonome VM maître qui gère un cluster VM ensemble à l’échelle en fonction des ordinateurs virtuels à Mesos : [https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json](https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json)

 - **L’équilibrage de charge Round robin à l’échelle de l’ordinateur virtuel défini instances** - si vous souhaitez remettre le travail à un cluster de calcul des ordinateurs virtuels à l’aide d’une approche de « tourniquet », vous pouvez configurer un équilibreur de charge Azure avec équilibrage de la charge en conséquence les règles. Vous pouvez également définir des sondes pour vérifier l’exécution de votre application en exécutant la commande ping ports avec un chemin d’accès de protocole, intervalle et demande spécifié.

    Voici un exemple qui crée un jeu d’échelle de machine virtuelle des ordinateurs virtuels exécutant le serveur web IIS et utilise un équilibreur de charge pour équilibrer la charge qui reçoit de chaque machine virtuelle. Il utilise également le protocole HTTP à une URL spécifique sur chaque machine virtuelle : [https://github.com/gbowerman/azure-myriad/blob/master/vmss-win-iis-vnet-storage-lb.json](https://github.com/gbowerman/azure-myriad/blob/master/vmss-win-iis-vnet-storage-lb.json) - examiner le type de ressource de Microsoft.Network/loadBalancers et networkProfile et extensionProfile dans le virtualMachineScaleSet.

 - **Déploiement d’une échelle de l’ordinateur virtuel est définie comme un cluster de calcul dans un gestionnaire de cluster PaaS** - échelle VM jeux sont parfois décrits comme un rôle worker de nouvelle génération. Il s’agit d’une description valide mais il exécute également le risque de confusion échelle jeu de fonctionnalités avec les fonctionnalités de rôle de travail PaaS v1. Dans un sens, échelle de machine virtuelle jeux offrent un véritable « rôle de travail » ou de ressource de travail, car ils fournissent une ressource de calcul généralisée qui est la plate-forme/runtime indépendant, personnalisable et s’intègre dans Azure le Gestionnaire de ressources IaaS.

    Un rôle de travail v1 PaaS, tandis que la limitée en termes de prise en charge de plate-forme/runtime (images de plate-forme Windows) inclut également des services tels que l’échange de VIP, les paramètres de mise à niveau configurables, runtime/app déploiement des paramètres spécifiques qui se sont pas _encore_ disponibles dans la machine virtuelle mise à l’échelle les jeux, ou seront livrés par d’autres services PaaS de niveau supérieurs comme Service de Fabric. C’est dans cette vous pouvez regarder jeux d’échelle de machine virtuelle comme une infrastructure qui prend en charge les services PaaS. Par exemple Solutions de PaaS comme Service de Fabric ou responsables de cluster comme Mesos pouvez créer sur la machine virtuelle mise à l’échelle définit comme une couche de calcul évolutif.

    Voici un exemple d’un cluster de Mesos qui déploie une échelle de machine virtuelle définie dans la même VNET sous la forme d’une machine virtuelle autonome. La machine virtuelle autonome est une forme de base de Mesos, et le jeu d’échelle de machine virtuelle représente un ensemble de nœuds de l’esclave : [https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json](https://github.com/gbowerman/azure-myriad/blob/master/mesos-vmss-simple-cluster.json). Les versions futures du [Service de conteneur Azure](https://azure.microsoft.com/blog/azure-container-service-now-and-the-future/) déploiera des versions plus complexes/renforcée de ce scénario en fonction des jeux d’échelle de machine virtuelle.

## <a name="vm-scale-set-performance-and-scale-guidance"></a>Échelle de l’ordinateur virtuel définie les performances et mettre à l’échelle d’orientation

- Au cours de la présentation publique, ne créez pas plus de 500 VMs dans plusieurs jeux d’échelle de machine virtuelle à la fois.
- Plan pour les machines virtuelles de pas plus de 40 par compte de stockage.
- Répartir les premières lettres du nom de compte de stockage autant que possible.  Les modèles VMSS exemple dans [Azure Quickstart modèles](https://github.com/Azure/azure-quickstart-templates/) fournissent des exemples de cette procédure.
- Si vous utilisez des ordinateurs virtuels personnalisés, prévoir des VM pas plus de 40 par ensemble d’échelle de machine virtuelle, dans un compte de stockage unique.  Vous devez l’image préalablement copié dans le compte de stockage avant de commencer le déploiement de jeu échelle de machine virtuelle. Consultez le Forum aux questions pour plus d’informations.
- Plan pour les machines virtuelles de ne plus de 2 048 par VNET.  Cette limite est augmentée à l’avenir.
- Le nombre de machines virtuelles que vous pouvez créer est limité par votre quota de base Compute dans une région. Vous devrez peut-être contacter le support technique afin d’augmenter votre limite de quota de calcul augmenté même si vous avez une limite haute de cœurs pour une utilisation avec les services en nuage ou IaaS v1 aujourd'hui. Interroger votre quota, vous pouvez exécuter la commande CLI d’Azure suivante : _azure vm - utilisation de la liste_et la commande PowerShell suivante : _Get-AzureRmVMUsage_ (si vous utilisez une version de PowerShell ci-dessous 1.0 _Get-AzureVMUsage_).

## <a name="vm-scale-set-frequently-asked-questions"></a>Échelle de machine virtuelle définie fréquemment posées

**Q.** Le nombre de machines virtuelles peuvent avoir dans un jeu d’échelle VM ?

**A.** 100 si vous utilisez des images de plate-forme qui peuvent être réparties entre plusieurs comptes de stockage. Si vous utilisez des images personnalisées, 40, dans la mesure où les images personnalisées sont limités à un compte de stockage unique lors de l’aperçu.

**Q** les autres limites de ressource existent pour les jeux d’échelle de machine virtuelle ?

**A.** Vous êtes limité à la création d’ordinateurs virtuels ne dépassant pas 500 dans plusieurs jeux d’échelle par région au cours de la période d’aperçu. Les fichiers [limites de Service d’abonnement Azure /](../articles/azure-subscription-service-limits.md) s’appliquent.

**Q.** Sont les disques de données pris en charge dans les ensembles d’échelle VM ?

**A.** Pas dans la version initiale. Vos options de stockage des données sont les suivantes :

- Fichiers Azure (lecteurs partagé de SMB)

- Lecteur du système d’exploitation

- Lecteur TEMP (local, ne bénéficient ne pas du stockage Azure)

- (Base de données à distance, les tables Azure, BLOB Azure) du service de données externes

**Q.** Les régions Azure prend en charge pour les jeux d’échelle VM ?

**A.** N’importe quelle région qui prend en charge le Gestionnaire de ressources Azure prend en charge les jeux d’échelle de machine virtuelle.

**Q.** Comment créer une échelle de l’ordinateur virtuel à l’aide d’une image personnalisée ?

**A.** Rien dans la propriété vhdContainers, par exemple :

    "storageProfile": {
        "osDisk": {
            "name": "vmssosdisk",
            "caching": "ReadOnly",
            "createOption": "FromImage",
            "image": {
                "uri": [https://mycustomimage.blob.core.windows.net/system/Microsoft.Compute/Images/mytemplates/template-osDisk.vhd](https://mycustomimage.blob.core.windows.net/system/Microsoft.Compute/Images/mytemplates/template-osDisk.vhd)
            },
            "osType": "Windows"
        }
    },


**Q.** Si réduire mes échelle VM définie la capacité de 20 à 15, ce qui les ordinateurs virtuels seront supprimées ?

**A.** L’échelle uniformément la valeur sur la mise à niveau des domaines et des domaines d’erreur pour optimiser la disponibilité des ordinateurs virtuels sont supprimés.

**Q.** Pourquoi il si je puis augmentez la capacité de 15 à 18 ?

**A.** Si vous augmentez à 18, les ordinateurs virtuels avec index 15, 16, 17 sera créé. Dans les deux cas, les ordinateurs virtuels sont réparties sur FDs et UDs.

**Q.** Lorsque vous utilisez plusieurs extensions dans un jeu d’échelle VM, puis-je appliquer une séquence d’exécution ?

**A.** Pas directement, mais l’extension customScript, votre script peut attendre une autre extension exécuter (par exemple par le suivi de l’extension du journal - voir [https://github.com/Azure/azure-quickstart-templates/blob/master/201-vmss-lapstack-autoscale/install\_lap.sh](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vmss-lapstack-autoscale/install_lap.sh)).

**Q.** Jeux d’échelle VM fonctionnent avec des jeux de disponibilité Azure ?

**A.** Oui. Un ensemble d’échelle VM est une disponibilité implicite avec 3 FDs et 5 UDs. Vous n’avez pas besoin de configurer quoi que ce soit sous virtualMachineProfile. Dans les futures versions, jeux d’échelle de machine virtuelle sont susceptibles de couvrir plusieurs locataires, mais maintenant une échelle définie est un ensemble unique de disponibilité.
