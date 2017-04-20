<properties
   pageTitle="Exemple de zone DMZ – créer une DMZ Simple avec NSGs | Microsoft Azure"
   description="Créer un réseau DMZ avec des groupes de sécurité réseau (NSG)"
   services="virtual-network"
   documentationCenter="na"
   authors="tracsman"
   manager="rossort"
   editor=""/>

<tags
   ms.service="virtual-network"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="02/01/2016"
   ms.author="jonor;sivae"/>

# <a name="example-1--build-a-simple-dmz-with-nsgs"></a>Exemple 1 – créer une DMZ Simple avec NSGs

[Revenir à la Page de pratiques meilleure sécurité limite][HOME]

Cet exemple crée une zone DMZ simple avec quatre groupes de sécurité de réseau et les serveurs windows. Vous y découvrirez également comment chacune des commandes utiles pour fournir une meilleure compréhension de chaque étape. Il existe également une section de scénario de trafic pour fournir une profondeur détaillées comment le trafic se poursuit à travers les couches de défense de la zone DMZ. Enfin, dans les références de la section est le code complet et les instructions pour créer cet environnement pour tester et expérimenter différents scénarios. 

![DMZ entrant avec NSG][1]

## <a name="environment-description"></a>Description de l’environnement
Dans cet exemple, il existe un abonnement qui contient les éléments suivants :

- Deux services en nuage : « FrontEnd001 » et « BackEnd001 »
- Un réseau virtuel, et « CorpNetwork », avec deux sous-réseaux ; « Frontal » et « Back-end »
- Un groupe de sécurité réseau qui est appliquée pour les deux sous-réseaux.
- Un serveur Windows qui représente un serveur web d’application (« IIS01 »)
- Serveurs de deux fenêtres qui représentent l’application back end serveurs (« AppVM01 », « AppVM02 »)
- Un serveur Windows qui représente un serveur DNS (« DNS01 »)

Dans la section références ci-dessous est un script PowerShell qui générera la majeure partie de l’environnement décrite ci-dessus. Créer les ordinateurs virtuels et les réseaux virtuels, bien que sont effectuées par l’exemple de script, pas décrits en détail dans ce document. 

Pour créer l’environnement ;

  1.    Enregistrez le fichier xml de configuration réseau inclus dans la section Références (mis à jour avec IP, l’emplacement et les noms qui correspondent à un scénario donné)
  2.    Mettre à jour les variables d’utilisateur dans le script pour correspondre à l’environnement, que le script doit être exécuté sur (les abonnements, les noms de service, etc.)
  3.    Exécutez le script dans PowerShell

**Remarque**: la région signifiée dans le script PowerShell doit correspondre à la région indiquée dans le fichier xml de configuration réseau.

Une fois que le script s’exécute correctement autres étapes facultatives peuvent être prises, dans la section références sont deux scripts pour configurer le serveur web et le serveur d’applications avec une application web simple permettant de tester cette configuration DMZ.

Les sections suivantes fournit une description détaillée des groupes de sécurité réseau et de leur fonctionnement pour que cet exemple en progressant dans les lignes principales du script PowerShell.

## <a name="network-security-groups-nsg"></a>Groupes de sécurité réseau (NSG)
Pour cet exemple, un groupe NSG est généré et ensuite chargé avec six règles. 

>[AZURE.TIP] En règle générale, vous devez d’abord créer vos règles « Autoriser » spécifiques, et enfin les règles plus génériques « Refuser ». Exigences imposées par la priorité affectée dont les règles sont évaluées premier. Une fois que le trafic est trouvé à appliquer à une règle spécifique, aucuns plus de règles ne sont évaluées. Règles NSG s’appliquent dans une ou l’autre dans la direction entrante ou sortante (du point de vue du sous-réseau).

De façon déclarative, les règles suivantes sont en cours de génération pour le trafic entrant :

1.  Le trafic DNS (port 53) interne est autorisé
2.  Le trafic RDP (le port 3389) à partir d’Internet à toute machine virtuelle est autorisé
3.  Le trafic HTTP (port 80) à partir d’Internet vers le serveur web (IIS01) est autorisé.
4.  Tout le trafic (tous les ports) à partir de IIS01 à AppVM1 est autorisé.
5.  Tout le trafic (tous les ports) à partir d’Internet à l’ensemble de VNet (les deux sous-réseaux) est refusé.
6.  Tout le trafic (tous les ports) à partir du sous-réseau frontal au sous-réseau back-end est refusé.

Avec ces règles lié à chaque sous-réseau, si une requête HTTP a été entrante provenant d’Internet vers le serveur web, les deux règles 3 (autoriser) et 5 (refuser) s’appliquent, mais dans la mesure où les règles 3 a une priorité plus élevée qu’il appliquerait et règle 5 ne serait pas entrent en jeu. Ainsi, la requête HTTP serait autorisée au serveur web. Si vous essayez le même trafic d’atteindre le serveur DNS01, règle 5 (Deny) est le premier à appliquer et le trafic ne serait pas autorisé à transmettre au serveur. Règle 6 (Deny) bloque le sous-réseau frontal de communiquer sur le sous-réseau du serveur principal (à l’exception du trafic autorisé dans les règles 1 et 4), cela protège le réseau back-end en cas un intrus découvre que l’application web sur le site Web frontal, l’attaquant aurait un accès limité au réseau protégé « back-end » (uniquement pour les ressources exposées sur le serveur de AppVM01).

Il existe une règle sortante par défaut qui autorise le trafic vers internet. Pour cet exemple, nous allons autorisant le trafic sortant et ne modifie ne pas les règles de trafic sortant. Pour verrouiller le trafic dans les deux directions, le routage défini par l’utilisateur est requise, il est exploré dans « Exemple 3 » ci-dessous.

Chaque règle est traitée plus en détail comme suit (**Remarque**: n’importe quel élément dans le dessous de la liste en commençant par un signe dollar (par exemple : $NSGName) est une variable définie par l’utilisateur à partir du script dans la section Références de ce document) :

1. Tout d’abord un groupe de sécurité du réseau doivent être généré pour les règles de blocage :

        New-AzureNetworkSecurityGroup -Name $NSGName `
            -Location $DeploymentLocation `
            -Label "Security group for $VNetName subnets in $DeploymentLocation"
 
2.  La première règle dans cet exemple permettra le trafic DNS entre tous les réseaux internes vers le serveur DNS sur le sous-réseau du serveur principal. La règle a certains paramètres importants :
  - « Type » indique dans quel sens du trafic de cette règle sera effective ; Il s’agit du point de vue de la Machine virtuelle ou de sous-réseau (selon où cette NSG est lié). Ainsi, si le Type est « Entrant » et le trafic est entrant le sous-réseau, la règle doit s’appliquer et trafic qui quitte le sous-réseau ne serait pas affecté par cette règle.
  - « Priorité » définit l’ordre dans lequel un flux de trafic sera évalué. Le nombre plus la priorité est faible. Dès qu’une règle s’applique à un flux de trafic spécifiques, aucune des règles additionnelles ne sont traitées. Ainsi, si une règle de priorité 1 permet le trafic et une règle de priorité 2 refuse le trafic et les deux règles s’appliquent au trafic, le trafic est autorisé à circuler (dans la mesure où la règle 1 a une priorité plus élevée qu’il a pris effet, et aucune règle n’a été appliqué).
  - « Action » indique si le trafic affecté par cette règle est bloqué ou autorisé.

            Get-AzureNetworkSecurityGroup -Name $NSGName | `
                Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" `
                -Type Inbound -Priority 100 -Action Allow `
                -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
                -DestinationAddressPrefix $VMIP[4] `
                -DestinationPortRange '53' `
                -Protocol *

3.  Cette règle autorise le trafic RDP de flux à partir d’internet pour le port de protocole RDP sur n’importe quel serveur sur un sous-réseau dans le VNET. Cette règle utilise deux types spéciaux de préfixes d’adresses ; « VIRTUAL_NETWORK » et « INTERNET ». Il s’agit d’un moyen facile de traiter une plus grande catégorie de préfixes d’adresses.

        Get-AzureNetworkSecurityGroup -Name $NSGName | `
            Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" `
            -Type Inbound -Priority 110 -Action Allow `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK `
            -DestinationPortRange '3389' `
            -Protocol *

4.  Cette règle autorise le trafic internet entrant sur le serveur web. Cela ne modifie pas le comportement de routage ; il autorise uniquement le trafic destiné à IIS01 à passer. Ainsi, si le trafic Internet était le serveur web comme destination cette règle devrait lui permettre et arrêter de traiter plus de règles. (Dans la règle de priorité 140 tout autre trafic internet entrant est bloqué). Si vous traitez uniquement le trafic HTTP, cette règle pourrait ensuite restreindre pour autoriser uniquement le Port de Destination 80.

        Get-AzureNetworkSecurityGroup -Name $NSGName | `
            Set-AzureNetworkSecurityRule -Name "Enable Internet to $VMName[0]" `
            -Type Inbound -Priority 120 -Action Allow `
            -SourceAddressPrefix Internet -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[0] `
            -DestinationPortRange '*' `
            -Protocol *

5.  Cette règle autorise le trafic du serveur IIS01 sur le serveur AppVM01, une règle ultérieure bloque toutes les autre frontaux au trafic de back-end. Pour améliorer cette règle, si le port est connu, qui doit être ajouté. Par exemple, si le serveur IIS accède uniquement à SQL Server sur AppVM01, la plage de ports de Destination doit être modifiée à partir de « * » (tous) à 1433 (port SQL), permettant ainsi une surface d’attaque entrante réduite sur AppVM01 doit l’application web est compromise.

        Get-AzureNetworkSecurityGroup -Name $NSGName | `
            Set-AzureNetworkSecurityRule -Name "Enable $VMName[1] to $VMName[2]" `
            -Type Inbound -Priority 130 -Action Allow `
        -SourceAddressPrefix $VMIP[1] -SourcePortRange '*' `
        -DestinationAddressPrefix $VMIP[2] `
        -DestinationPortRange '*' `
        -Protocol *
 
6.  Cette règle refuse le trafic provenant d’internet sur tous les serveurs sur le réseau. En combinaison avec la règle de priorité 110 et 120, autorise uniquement le trafic internet entrant sur le pare-feu et les ports RDP vers d’autres serveurs et les blocs de tout le reste. 

        Get-AzureNetworkSecurityGroup -Name $NSGName | `
            Set-AzureNetworkSecurityRule `
            -Name "Isolate the $VNetName VNet from the Internet" `
            -Type Inbound -Priority 140 -Action Deny `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK `
            -DestinationPortRange '*' `
            -Protocol *
 
7.  La règle finale refuse le trafic du sous-réseau frontal au sous-réseau back-end. Dans la mesure où il s’agit d’une seule règle entrante, le trafic inverse est autorisé (à partir du serveur principal pour le site Web frontal).

        Get-AzureNetworkSecurityGroup -Name $NSGName | `
            Set-AzureNetworkSecurityRule `
            -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" `
            -Type Inbound -Priority 150 -Action Deny `
            -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
            -DestinationAddressPrefix $BEPrefix `
            -DestinationPortRange '*' `
            -Protocol * 

## <a name="traffic-scenarios"></a>Scénarios de trafic
#### <a name="allowed-web-to-web-server"></a>(*Autorisé*) Web sur serveur Web
1.  Page de Internet utilisateur demandes HTTP à partir de FrontEnd001.CloudApp.Net (face Cloud Service Internet)
2.  Le trafic de passes de service via le point de terminaison ouvert sur le port 80 vers IIS01 le cloud (le serveur web)
3.  Site Web frontal sous-réseau commence le traitement de la règle de trafic entrant :
  1.    NSG règle 1 (DNS) ne s’appliquent pas, atteindre la règle suivante
  2.    NSG règle 2 (RDP) ne s’appliquent pas, atteindre la règle suivante
  3.    NSG règle 3 (Internet pour IIS01) s’applique, le trafic est le traitement des règles autorisées, arrêter
4.  Le trafic atteint l’adresse IP interne du serveur web IIS01 (10.0.1.5)
5.  IIS01 est à l’écoute du trafic web, reçoit cette demande et commence le traitement de la demande
6.  IIS01 demande le SQL Server sur AppVM01 pour plus d’informations
7.  Aucune règle sortante sur le sous-réseau du site Web frontal, le trafic est autorisé.
8.  Le sous-réseau de back-end commence le traitement de la règle de trafic entrant :
  1.    NSG règle 1 (DNS) ne s’appliquent pas, atteindre la règle suivante
  2.    NSG règle 2 (RDP) ne s’appliquent pas, atteindre la règle suivante
  3.    NSG règle 3 (Internet au pare-feu) ne s’appliquent, passer à la règle suivante
  4.    NSG règle 4 (IIS01 à AppVM01) s’applique, le trafic est autorisé, d’interrompre le traitement de la règle
9.  AppVM01 reçoit la requête SQL et répond
10. Étant donné qu’aucune règle de trafic sortant sur le back-end sous-réseau la réponse est autorisée.
11. Site Web frontal sous-réseau commence le traitement de la règle de trafic entrant :
  1.    Il n’y a aucune règle NSG qui s’applique à l’entrée le trafic du sous-réseau du serveur principal pour le sous-réseau frontal, donc aucun de du NSG règles s’appliquent
  2.    La règle du système par défaut autorisant le trafic entre sous-réseaux permettrait de ce trafic afin que le trafic est autorisé.
12. Le serveur IIS reçoit la réponse SQL et la fin de la réponse HTTP et envoie ensuite au demandeur
13. Dans la mesure où il n’y a aucune règle sortante sur le sous-réseau frontal la réponse n’est autorisée, et l’utilisateur Internet reçoit la page web demandée.

#### <a name="allowed-rdp-to-backend"></a>(*Autorisé*) RDP pour le back-end
1.  Administrateur de serveur sur internet demande la session RDP à AppVM01 sur BackEnd001.CloudApp.Net:xxxxx, où xxxxx est le numéro de port attribué de façon aléatoire pour RDP pour AppVM01 (le port affecté se trouve sur le portail Azure ou via PowerShell)
2.  Sous-réseau de back-end commence le traitement de la règle de trafic entrant :
  1.    NSG règle 1 (DNS) ne s’appliquent pas, atteindre la règle suivante
  2.    NSG règle 2 (RDP) s’applique, le trafic est le traitement des règles autorisées, arrêter
3.  Qu’aucune règle de trafic sortant, les règles par défaut s’appliquent et le trafic de retour est autorisé
4.  Session RDP est activée.
5.  AppVM01 demande de mot de passe utilisateur

#### <a name="allowed-web-server-dns-lookup-on-dns-server"></a>(*Autorisé*) Recherche de DNS du serveur Web sur le serveur DNS
1.  Web serveur, IIS01, besoins de flux de données à www.data.gov, mais doit résoudre l’adresse.
2.  La configuration du réseau pour les listes de VNet DNS01 (10.0.2.4 sur le sous-réseau du back-end) comme serveur DNS principal, IIS01 envoie la requête DNS pour DNS01
3.  Aucune règle sortante sur le sous-réseau du site Web frontal, le trafic est autorisé.
4.  Sous-réseau de back-end commence le traitement de la règle de trafic entrant :
  1.     NSG règle 1 (DNS) s’applique, le trafic est le traitement des règles autorisées, arrêter
5.  Serveur DNS reçoit la requête
6.  Serveur DNS ne possède pas l’adresse mise en cache et demande à un serveur DNS racine sur internet
7.  Aucune règle sortante sur le sous-réseau du back-end, le trafic est autorisé.
8.  Serveur DNS Internet répond, dans la mesure où cette session a été initialisée en interne, la réponse est autorisée.
9.  Serveur DNS met en cache la réponse et répond à la demande initiale à IIS01
10. Aucune règle sortante sur le sous-réseau du back-end, le trafic est autorisé.
11. Site Web frontal sous-réseau commence le traitement de la règle de trafic entrant :
  1.    Il n’y a aucune règle NSG qui s’applique à l’entrée le trafic du sous-réseau du serveur principal pour le sous-réseau frontal, donc aucun de du NSG règles s’appliquent
  2.    La règle du système par défaut autorisant le trafic entre sous-réseaux permettrait à ce trafic afin que le trafic est autorisé.
12. IIS01 reçoit la réponse DNS01

#### <a name="allowed-web-server-access-file-on-appvm01"></a>(*Autorisé*) Fichier d’accès de serveur Web sur AppVM01
1.  IIS01 demande un fichier sur AppVM01
2.  Aucune règle sortante sur le sous-réseau du site Web frontal, le trafic est autorisé.
3.  Le sous-réseau de back-end commence le traitement de la règle de trafic entrant :
  1.    NSG règle 1 (DNS) ne s’appliquent pas, atteindre la règle suivante
  2.    NSG règle 2 (RDP) ne s’appliquent pas, atteindre la règle suivante
  3.    NSG règle 3 (Internet pour IIS01) ne s’appliquent, passer à la règle suivante
  4.    NSG règle 4 (IIS01 à AppVM01) s’applique, le trafic est autorisé, d’interrompre le traitement de la règle
4.  AppVM01 reçoit la demande et répond avec un fichier (en supposant que l’accès est autorisé)
5.  Étant donné qu’aucune règle de trafic sortant sur le back-end sous-réseau la réponse est autorisée.
6.  Site Web frontal sous-réseau commence le traitement de la règle de trafic entrant :
  1.    Il n’y a aucune règle NSG qui s’applique à l’entrée le trafic du sous-réseau du serveur principal pour le sous-réseau frontal, donc aucun de du NSG règles s’appliquent
  2.    La règle du système par défaut autorisant le trafic entre sous-réseaux permettrait de ce trafic afin que le trafic est autorisé.
7.  Le serveur IIS reçoit le fichier

#### <a name="denied-web-to-backend-server"></a>(*Refusé*) Site Web sur le serveur principal
1.  Utilisateur Internet tente d’accéder à un fichier sur AppVM01 via le service de BackEnd001.CloudApp.Net
2.  Dans la mesure où il n’y a aucun point de terminaison ouvert pour le partage de fichiers, cela ne serait pas passer le Service en nuage et n’aurait pas d’atteindre le serveur
3.  Si les points de terminaison ont été ouverts pour une raison quelconque, les règles NSG 5 (Internet à VNet) serait bloquer ce trafic

#### <a name="denied-web-dns-lookup-on-dns-server"></a>(*Refusé*) Recherche DNS Web sur le serveur DNS
1.  Utilisateur Internet essaie de rechercher un enregistrement DNS interne sur DNS01 via le service de BackEnd001.CloudApp.Net
2.  Dans la mesure où il n’y a aucun point de terminaison ouvert pour DNS, cela ne serait pas passer le Service en nuage et n’aurait pas d’atteindre le serveur
3.  Si les points de terminaison ont été ouverts pour une raison quelconque, les règles NSG 5 (Internet à VNet) serait bloquer ce trafic (Remarque : cette règle 1 (DNS) ne s’appliquent pas pour deux raisons, tout d’abord l’adresse source est internet, cette règle ne s’applique qu’à la VNet local comme source, aussi il s’agit d’une règle d’autorisation, afin qu’il ne serait jamais refuser le trafic)

#### <a name="denied-web-to-sql-access-through-firewall"></a>(*Refusé*) Web Access SQL par l’intermédiaire de pare-feu
1.  Utilisateur Internet demande les données SQL à partir de FrontEnd001.CloudApp.Net (face Cloud Service Internet)
2.  Dans la mesure où il n’y a aucun point de terminaison ouvert pour SQL, cela ne serait pas passer le Service en nuage et n’aboutiraient le pare-feu
3.  Si les points de terminaison ont été ouverts pour une raison quelconque, le sous-réseau frontal commence le traitement de la règle de trafic entrant :
  1.    NSG règle 1 (DNS) ne s’appliquent pas, atteindre la règle suivante
  2.    NSG règle 2 (RDP) ne s’appliquent pas, atteindre la règle suivante
  3.    NSG règle 3 (Internet pour IIS01) s’applique, le trafic est le traitement des règles autorisées, arrêter
4.  Le trafic atteint une adresse IP interne de la IIS01 (10.0.1.5)
5.  IIS01 n’est pas à l’écoute sur le port 1433, donc aucune réponse à la demande

## <a name="conclusion"></a>Conclusion
Il s’agit d’un moyen relativement simple et direct d’isoler le sous-réseau back-end du trafic entrant.

Plus d’exemples et une vue d’ensemble des limites de sécurité réseau, vous pouvez trouver [ici][HOME].

## <a name="references"></a>Références
### <a name="main-script-and-network-config"></a>Script principal et la configuration du réseau
Enregistrer le Script complet dans un fichier de script PowerShell. Enregistrer la configuration du réseau dans un fichier nommé « NetworkConf1.xml ».
Modifiez les variables définies par l’utilisateur si nécessaire. Exécutez le script, puis suivez les instructions de configuration de règle de pare-feu contenues dans la section 1 de l’exemple ci-dessus.

#### <a name="full-script"></a>Script complet
Ce script va, basé sur les variables définies par l’utilisateur ;

1.  Se connecter à un abonnement Azure
2.  Créer un nouveau compte de stockage
3.  Créer un nouveau VNet et deux sous-réseaux tel que défini dans le fichier de configuration de réseau
4.  Créer des ordinateurs virtuels du serveur 4 windows
5.  Configurer NSG, y compris :
  - Création d’un NSG
  - Remplissage avec des règles
  - Liaison du NSG aux sous-réseaux appropriés

Ce script PowerShell doit être exécuté localement sur qu'un internet connecté PC ou serveur.

>[AZURE.IMPORTANT] Lorsque ce script est exécuté, il peut être avertissements et autres messages d’information qui apparaît dans PowerShell. Seuls les messages erreur en rouge sont un motif de préoccupation.


    <# 
     .SYNOPSIS
      Example of Network Security Groups in an isolated network (Azure only, no hybrid connections)
    
     .DESCRIPTION
      This script will build out a sample DMZ setup containing:
       - A default storage account for VM disks
       - Two new cloud services
       - Two Subnets (FrontEnd and BackEnd subnets)
       - One server on the FrontEnd Subnet
       - Three Servers on the BackEnd Subnet
       - Network Security Groups to allow/deny traffic patterns as declared
      
      Before running script, ensure the network configuration file is created in
      the directory referenced by $NetworkConfigFile variable (or update the
      variable to reflect the path and file name of the config file being used).
    
     .Notes
      Security requirements are different for each use case and can be addressed in a
      myriad of ways. Please be sure that any sensitive data or applications are behind
      the appropriate layer(s) of protection. This script serves as an example of some
      of the techniques that can be used, but should not be used for all scenarios. You
      are responsible to assess your security needs and the appropriate protections
      needed, and then effectively implement those protections.
    
      FrontEnd Service (FrontEnd subnet 10.0.1.0/24)
       IIS01      - 10.0.1.5
     
      BackEnd Service (BackEnd subnet 10.0.2.0/24)
       DNS01      - 10.0.2.4
       AppVM01    - 10.0.2.5
       AppVM02    - 10.0.2.6
    
    #>
    
    # Fixed Variables
        $LocalAdminPwd = Read-Host -Prompt "Enter Local Admin Password to be used for all VMs"
        $VMName = @()
        $ServiceName = @()
        $VMFamily = @()
        $img = @()
        $size = @()
        $SubnetName = @()
        $VMIP = @()
    
    # User Defined Global Variables
      # These should be changes to reflect your subscription and services
      # Invalid options will fail in the validation section
    
      # Subscription Access Details
        $subID = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    
      # VM Account, Location, and Storage Details
        $LocalAdmin = "theAdmin"
        $DeploymentLocation = "Central US"
        $StorageAccountName = "vmstore02"
    
      # Service Details
        $FrontEndService = "FrontEnd001"
        $BackEndService = "BackEnd001"
    
      # Network Details
        $VNetName = "CorpNetwork"
        $FESubnet = "FrontEnd"
        $FEPrefix = "10.0.1.0/24"
        $BESubnet = "BackEnd"
        $BEPrefix = "10.0.2.0/24"
        $NetworkConfigFile = "C:\Scripts\NetworkConf1.xml"
    
      # VM Base Disk Image Details
        $SrvImg = Get-AzureVMImage | Where {$_.ImageFamily -match 'Windows Server 2012 R2 Datacenter'} | sort PublishedDate -Descending | Select ImageName -First 1 | ForEach {$_.ImageName}
      
      # NSG Details
        $NSGName = "MyVNetSG"
    
    # User Defined VM Specific Config
        # Note: To ensure proper NSG Rule creation later in this script:
        #       - The Web Server must be VM 0
        #       - The AppVM1 Server must be VM 1
        #       - The DNS server must be VM 3
        #
        #       Otherwise the NSG rules in the last section of this
        #       script will need to be changed to match the modified
        #       VM array numbers ($i) so the NSG Rule IP addresses
        #       are aligned to the associated VM IP addresses.
    
        # VM 0 - The Web Server
          $VMName += "IIS01"
          $ServiceName += $FrontEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $FESubnet
          $VMIP += "10.0.1.5"
    
        # VM 1 - The First Application Server
          $VMName += "AppVM01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.5"
    
        # VM 2 - The Second Application Server
          $VMName += "AppVM02"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.6"
    
        # VM 3 - The DNS Server
          $VMName += "DNS01"
          $ServiceName += $BackEndService
          $VMFamily += "Windows"
          $img += $SrvImg
          $size += "Standard_D3"
          $SubnetName += $BESubnet
          $VMIP += "10.0.2.4"

    # ----------------------------- #
    # No User Defined Varibles or   #
    # Configuration past this point #
    # ----------------------------- #   

      # Get your Azure accounts
        Add-AzureAccount
        Set-AzureSubscription –SubscriptionId $subID -ErrorAction Stop
        Select-AzureSubscription -SubscriptionId $subID -Current -ErrorAction Stop
    
      # Create Storage Account
        If (Test-AzureName -Storage -Name $StorageAccountName) { 
            Write-Host "Fatal Error: This storage account name is already in use, please pick a diffrent name." -ForegroundColor Red
            Return}
        Else {Write-Host "Creating Storage Account" -ForegroundColor Cyan 
              New-AzureStorageAccount -Location $DeploymentLocation -StorageAccountName $StorageAccountName}
    
      # Update Subscription Pointer to New Storage Account
        Write-Host "Updating Subscription Pointer to New Storage Account" -ForegroundColor Cyan 
        Set-AzureSubscription –SubscriptionId $subID -CurrentStorageAccountName $StorageAccountName -ErrorAction Stop
    
    # Validation
    $FatalError = $false
    
    If (-Not (Get-AzureLocation | Where {$_.DisplayName -eq $DeploymentLocation})) {
         Write-Host "This Azure Location was not found or available for use" -ForegroundColor Yellow
         $FatalError = $true}
    
    If (Test-AzureName -Service -Name $FrontEndService) { 
        Write-Host "The FrontEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The FrontEndService service name is valid for use." -ForegroundColor Green}
    
    If (Test-AzureName -Service -Name $BackEndService) { 
        Write-Host "The BackEndService service name is already in use, please pick a different service name." -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The BackEndService service name is valid for use." -ForegroundColor Green}
    
    If (-Not (Test-Path $NetworkConfigFile)) { 
        Write-Host 'The network config file was not found, please update the $NetworkConfigFile variable to point to the network config xml file.' -ForegroundColor Yellow
        $FatalError = $true}
    Else { Write-Host "The network config file was found" -ForegroundColor Green
            If (-Not (Select-String -Pattern $DeploymentLocation -Path $NetworkConfigFile)) {
                Write-Host 'The deployment location was not found in the network config file, please check the network config file to ensure the $DeploymentLocation varible is correct and the netowrk config file matches.' -ForegroundColor Yellow
                $FatalError = $true}
            Else { Write-Host "The deployment location was found in the network config file." -ForegroundColor Green}}
    
    If ($FatalError) {
        Write-Host "A fatal error has occured, please see the above messages for more information." -ForegroundColor Red
        Return}
    Else { Write-Host "Validation passed, now building the environment." -ForegroundColor Green}
    
    # Create VNET
        Write-Host "Creating VNET" -ForegroundColor Cyan 
        Set-AzureVNetConfig -ConfigurationPath $NetworkConfigFile -ErrorAction Stop
    
    # Create Services
        Write-Host "Creating Services" -ForegroundColor Cyan
        New-AzureService -Location $DeploymentLocation -ServiceName $FrontEndService -ErrorAction Stop
        New-AzureService -Location $DeploymentLocation -ServiceName $BackEndService -ErrorAction Stop
    
    # Build VMs
        $i=0
        $VMName | Foreach {
            Write-Host "Building $($VMName[$i])" -ForegroundColor Cyan
            New-AzureVMConfig -Name $VMName[$i] -ImageName $img[$i] –InstanceSize $size[$i] | `
                Add-AzureProvisioningConfig -Windows -AdminUsername $LocalAdmin -Password $LocalAdminPwd  | `
                Set-AzureSubnet  –SubnetNames $SubnetName[$i] | `
                Set-AzureStaticVNetIP -IPAddress $VMIP[$i] | `
                Set-AzureVMMicrosoftAntimalwareExtension -AntimalwareConfiguration '{"AntimalwareEnabled" : true}' | `
                Remove-AzureEndpoint -Name "PowerShell" | `
                New-AzureVM –ServiceName $ServiceName[$i] -VNetName $VNetName -Location $DeploymentLocation
                # Note: A Remote Desktop endpoint is automatically created when each VM is created.
            $i++
        }
        # Add HTTP Endpoint for IIS01
        Get-AzureVM -ServiceName $ServiceName[0] -Name $VMName[0] | Add-AzureEndpoint -Name HTTP -Protocol tcp -LocalPort 80 -PublicPort 80 | Update-AzureVM
    
    # Configure NSG
        Write-Host "Configuring the Network Security Group (NSG)" -ForegroundColor Cyan
        
      # Build the NSG
        Write-Host "Building the NSG" -ForegroundColor Cyan
        New-AzureNetworkSecurityGroup -Name $NSGName -Location $DeploymentLocation -Label "Security group for $VNetName subnets in $DeploymentLocation"
    
      # Add NSG Rules
        Write-Host "Writing rules into the NSG" -ForegroundColor Cyan
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internal DNS" -Type Inbound -Priority 100 -Action Allow `
            -SourceAddressPrefix VIRTUAL_NETWORK -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[3] -DestinationPortRange '53' `
            -Protocol *
    
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable RDP to $VNetName VNet" -Type Inbound -Priority 110 -Action Allow `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '3389' `
            -Protocol *
    
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable Internet to $($VMName[0])" -Type Inbound -Priority 120 -Action Allow `
            -SourceAddressPrefix Internet -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[0] -DestinationPortRange '*' `
            -Protocol *
    
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Enable $($VMName[0]) to $($VMName[1])" -Type Inbound -Priority 130 -Action Allow `
            -SourceAddressPrefix $VMIP[0] -SourcePortRange '*' `
            -DestinationAddressPrefix $VMIP[1] -DestinationPortRange '*' `
            -Protocol *
        
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $VNetName VNet from the Internet" -Type Inbound -Priority 140 -Action Deny `
            -SourceAddressPrefix INTERNET -SourcePortRange '*' `
            -DestinationAddressPrefix VIRTUAL_NETWORK -DestinationPortRange '*' `
            -Protocol *
    
        Get-AzureNetworkSecurityGroup -Name $NSGName | Set-AzureNetworkSecurityRule -Name "Isolate the $FESubnet subnet from the $BESubnet subnet" -Type Inbound -Priority 150 -Action Deny `
            -SourceAddressPrefix $FEPrefix -SourcePortRange '*' `
            -DestinationAddressPrefix $BEPrefix -DestinationPortRange '*' `
            -Protocol *
    
        # Assign the NSG to the Subnets
            Write-Host "Binding the NSG to both subnets" -ForegroundColor Cyan
            Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $FESubnet -VirtualNetworkName $VNetName
            Set-AzureNetworkSecurityGroupToSubnet -Name $NSGName -SubnetName $BESubnet -VirtualNetworkName $VNetName
    
    # Optional Post-script Manual Configuration
      # Install Test Web App (Run Post-Build Script on the IIS Server)
      # Install Backend resource (Run Post-Build Script on the AppVM01)
      Write-Host
      Write-Host "Build Complete!" -ForegroundColor Green
      Write-Host
      Write-Host "Optional Post-script Manual Configuration Steps" -ForegroundColor Gray
      Write-Host " - Install Test Web App (Run Post-Build Script on the IIS Server)" -ForegroundColor Gray
      Write-Host " - Install Backend resource (Run Post-Build Script on the AppVM01)" -ForegroundColor Gray
      Write-Host
      

#### <a name="network-config-file"></a>Fichier de configuration de réseau
Enregistrer ce fichier xml avec l’emplacement de mise à jour et ajouter le lien vers ce fichier à la variable $NetworkConfigFile dans le script ci-dessus.
    
    <NetworkConfiguration xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
      <VirtualNetworkConfiguration>
        <Dns>
          <DnsServers>
            <DnsServer name="DNS01" IPAddress="10.0.2.4" />
            <DnsServer name="Level3" IPAddress="209.244.0.3" />
          </DnsServers>
        </Dns>
        <VirtualNetworkSites>
          <VirtualNetworkSite name="CorpNetwork" Location="Central US">
            <AddressSpace>
              <AddressPrefix>10.0.0.0/16</AddressPrefix>
            </AddressSpace>
            <Subnets>
              <Subnet name="FrontEnd">
                <AddressPrefix>10.0.1.0/24</AddressPrefix>
              </Subnet>
              <Subnet name="BackEnd">
                <AddressPrefix>10.0.2.0/24</AddressPrefix>
              </Subnet>
            </Subnets>
            <DnsServersRef>
              <DnsServerRef name="DNS01" />
              <DnsServerRef name="Level3" />
            </DnsServersRef>
          </VirtualNetworkSite>
        </VirtualNetworkSites>
      </VirtualNetworkConfiguration>
    </NetworkConfiguration>

#### <a name="sample-application-scripts"></a>Exemples de Scripts Application
Si vous souhaitez installer un exemple d’application pour ce, ainsi que d’autres exemples de DMZ, une a été fournie à l’adresse suivante : [Exemple de Script de l’Application][SampleApp]

<!--Image References-->
[1]: ./media/virtual-networks-dmz-nsg-asm/example1design.png "DMZ entrant avec NSG"

<!--Link References-->
[HOME]: ../best-practices-network-security.md
[SampleApp]: ./virtual-networks-sample-app.md

