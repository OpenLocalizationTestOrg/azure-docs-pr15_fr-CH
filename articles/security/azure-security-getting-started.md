<properties
   pageTitle="Mise en route de la sécurité de Microsoft Azure | Microsoft Azure"
   description="Cet article fournit une vue d’ensemble des fonctionnalités de sécurité de Microsoft Azure et considérations générales pour les organisations qui effectuent une migration leurs actifs à un fournisseur de nuage."
   services="security"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>

<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/19/2016"
   ms.author="yuridio"/>

#<a name="getting-started-with-microsoft-azure-security"></a>Mise en route de la sécurité de Microsoft Azure

Lorsque vous générez ou migrez des ressources informatiques à un fournisseur de nuage, vous comptez sur les capacités de cette organisation pour protéger les applications et les données vous confier à leurs services et les contrôles de sécurité qu’ils fournissent pour contrôler la sécurité de vos ressources en nuage.

D’Azure est conçue à partir de la fonction pour les applications pour l’hébergement des millions de clients simultanément et fournissant une base fiable sur laquelle les entreprises peuvent répondre à leurs besoins de sécurité. En outre, Azure vous offre un large éventail d’options de sécurité configurables et la possibilité de les contrôler de sorte que vous pouvez personnaliser la sécurité afin de répondre aux besoins uniques de votre déploiement.

Dans cet article de vue d’ensemble sur la sécurité Azure, nous allons étudier :

-   Fonctionnalités et services azure que vous pouvez utiliser pour aider à sécuriser vos services et vos données dans Azure

-   Comment Microsoft protège l’infrastructure Azure pour aider à protéger vos données et applications

##<a name="identity-and-access-management"></a>Gestion des identités et des accès


Il est essentiel de contrôler l’accès à l’infrastructure informatique, des données et des applications. Dans Microsoft Azure, ces fonctionnalités sont remies par les services comme Azure Active Directory, le stockage Azure et la prise en charge de nombreuses normes et des API.

[Azure Active Directory](../active-directory/active-directory-whatis.md) (AD azure) est un référentiel d’identité et d’un moteur qui fournit l’authentification, l’autorisation et contrôle d’accès pour les utilisateurs, les groupes et les objets d’une organisation. Annonce Azure offre également aux développeurs un moyen efficace pour intégrer la gestion des identités dans leurs applications. Protocoles standard tels que [SAML 2.0](https://en.wikipedia.org/wiki/SAML_2.0), [WS-Federation](https://msdn.microsoft.com/library/bb498017.aspx)et [OpenID connecter](http://openid.net/connect/) rend la connexion sur un grand nombre de plates-formes telles que PHP, Node.js, .NET et Java.

L’API basée sur REST de graphique permet aux développeurs de lire et d’écrire dans le répertoire à partir de n’importe quelle plate-forme. Prise en charge pour [OAuth 2.0](http://oauth.net/2/), les développeurs peuvent créer mobile et sécurisent les applications web qui s’intègrent avec Microsoft et tiers web API et créer leur propre web API. Les bibliothèques clientes Open source sont disponibles pour .net, Windows Store, iOS et Android avec des bibliothèques supplémentaires en cours de développement.

### <a name="how-azure-enables-identity-and-access-management"></a>Comment Azure permet la gestion des identités et des accès

Annonce Azure peut servir sous la forme d’un répertoire de nuage autonome pour votre organisation ou une solution intégrée avec sur site Active Directory existant. Certaines fonctionnalités d’intégration incluent la synchronisation d’annuaire et d’authentification unique (SSO). Ces étendent la portée de vos identités sur site existant dans le nuage et améliorent l’expérience de l’administrateur et l’utilisateur final.

Certaines autres fonctionnalités pour la gestion des identités et des accès :

-   AD Azure permet [l’authentification unique](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/) aux applications SaaS, quel que soit l’endroit où elles sont hébergées. Certaines applications fédérées avec AD Azure, et d’autres utilisent le mot de passe l’authentification unique. Applications fédérées peuvent prennent également en charge la mise en service de l’utilisateur et le stockage du mot de passe.

-   Accès aux données dans le [Stockage Azure](https://azure.microsoft.com/services/storage/) est contrôlé via l’authentification. Chaque compte de stockage a une clé primaire ([Clé de compte de stockage](https://msdn.microsoft.com/library/azure/ee460785.aspx), ou SAK) et une clé secrète secondaire (Signature de l’accès partagé ou SAS).

-   Annonce Azure fournit l’identité sous la forme d’un Service par le biais de fédération (à l’aide de la réplication, la synchronisation et [Les Services de fédération Active Directory](../active-directory/fundamentals-identity.md)avec les annuaires sur site.

-   [Azure plusieurs facteurs d’authentification (AMF)](../multi-factor-authentication/multi-factor-authentication.md) est le service d’authentification à plusieurs facteurs qui demande aux utilisateurs de vérifier également les connexions à l’aide d’une application mobile, un appel téléphonique ou un message texte. Il est disponible pour utilisation avec AD Azure, pour sécuriser les ressources locales avec le serveur d’AMF Azure et les applications personnalisées et les répertoires à l’aide du SDK.

-   [Les Services de domaine Active Directory de Azure](https://azure.microsoft.com/services/active-directory-ds/) vous permet de joindre des ordinateurs virtuels Azure à un domaine sans avoir besoin de déployer des contrôleurs de domaine. Les utilisateurs peuvent se connecter à ces ordinateurs virtuels à l’aide de leurs informations d’identification Active Directory d’entreprise et administrer à un domaine d’ordinateurs virtuels à l’aide de la stratégie de groupe pour mettre en œuvre des bases de sécurité sur tous vos ordinateurs virtuels Azure.

-   [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) vous fournit un service de gestion d’identité globale, hautement disponible pour les applications pour les consommateurs qui s’adapte à des centaines de millions d’identités. Il peut être intégré via mobile et web des plates-formes. Vos consommateurs peuvent ouvrir une session pour toutes vos applications grâce à des expériences personnalisables à l’aide de leurs comptes sociaux existants ou en créant de nouvelles informations d’identification.

##<a name="data-access-control-and-encryption"></a>Contrôle d’accès aux données et le cryptage

Microsoft utilise les principes de la séparation des droits et [Privilèges](https://en.wikipedia.org/wiki/Principle_of_least_privilege) tout au long des opérations Azure. Accès aux données par le personnel de support Azure requiert une autorisation explicite et est accordé sur une base « juste-à-temps » qui est connectée et auditée, puis révoqué après la fin de la mission.

En outre, Azure fournit plusieurs fonctionnalités de protection des données en transit et au repos, y compris le chiffrement de données, des fichiers, des applications, des services, des communications et des lecteurs. Vous avez la possibilité de chiffrer les informations avant de les placer dans Azure, mais aussi stockant des clés dans vos centres de données sur site.

![Logiciel anti-programme malveillant de Microsoft dans Azure](./media/azure-security-getting-started/sec-azgsfig1.PNG)

### <a name="azure-encryption-technologies"></a>Technologies de cryptage Azure

Vous pouvez rassembler des détails sur l’accès d’administration à votre environnement d’abonnement à l’aide [des rapports de publicité Azure](../active-directory/active-directory-reporting-audit-events.md). Vous avez la possibilité de configurer le [Chiffrement de lecteur BitLocker](https://technet.microsoft.com/library/cc732774.aspx) sur des disques durs virtuels contenant des informations sensibles dans Azure.

Autres fonctionnalités dans Azure qui vous aideront à maintenir la sécurité de vos données :

-   Les développeurs peuvent créer cryptage dans les applications qu’elles déploient dans Azure à l’aide de.NET Framework et Windows [CryptoAPI](https://msdn.microsoft.com/library/ms867086.aspx) .

- Chiffrement du côté client pour le stockage blob Microsoft permettent de contrôler complètement les clés.  Le service de stockage jamais voit les clés et qu’il est incapable de décrypter les données.

-   [RMS Azure](https://technet.microsoft.com/library/jj585026.aspx) (le [Kit de développement logiciel RMS](https://msdn.microsoft.com/library/dn758244.aspx) doté de fichier et de cryptage au niveau des données et de prévention des fuites de données grâce à la gestion de l’accès basé sur la stratégie.

-   Azure prend en charge le [chiffrement au niveau table et colonne (TDE/effacer)](http://blogs.msdn.com/b/sqlsecurity/archive/2015/05/12/recommendations-for-using-cell-level-encryption-in-azure-sql-database.aspx) dans SQL Server Virtual Machines et tiers prend en charge les serveurs de gestion des clés dans les centres de données clients sur site.

-   Stockage compte, Signatures de l’accès partagé, certificats de gestion et autres touches sont uniques à chaque client d’Azure.

-   Stockage de hybride Azure [StorSimple](http://www.microsoft.com/server-cloud/products/storsimple/overview.aspx) crypte les données via une 128 bits public / privée paire de clés avant de télécharger sur le stockage Azure.

-   Azure prend en charge et utilise de nombreux mécanismes de cryptage, y compris AES, SSL/TLS et IPsec, selon les types de données et les conteneurs et les transports.

##<a name="virtualization"></a>Virtualisation

La plateforme Azure utilise un environnement virtualisé. Les instances utilisateur fonctionnent comme des machines virtuelles autonomes qui n’ont pas accès à un serveur hôte physique et l’isolation est appliquée à l’aide de la physique [des niveaux de privilège de (ring-0/anneau-3) processeur](https://en.wikipedia.org/wiki/Protection_ring).

Anneau 0 est le plus privilégié et 3 est le moins. La système d’exploitation invité s’exécute dans le moindre privilège Ring 1 et applications dans le dernier privilège Ring 3. Cette virtualisation des ressources physiques entraîne une séparation claire entre le système d’exploitation invité et hyperviseur, ce qui entraîne une séparation de sécurité supplémentaire entre les deux.

Hyperviseur d’Azure agit comme un noyau de micro et passe toutes les demandes d’accès matériel à partir d’ordinateurs virtuels invités à l’hôte pour le traitement à l’aide d’une interface de mémoire partagée appelée VMBus. Cela empêche les utilisateurs d’obtenir un accès en lecture/écriture/exécution brut au système et réduit le risque de partage des ressources système.

![Logiciel anti-programme malveillant de Microsoft dans Azure](./media/azure-security-getting-started/sec-azgsfig2.PNG)

### <a name="how-azure-implements-virtualization"></a>Comment Azure implémente la virtualisation

Azure utilise un pare-feu hyperviseur (filtre de paquets), qui est implémenté dans l’hyperviseur et configuré par un agent contrôleur de fabric. Cela permet de protéger les locataires contre tout accès non autorisé. Par défaut, lorsqu’une machine virtuelle est créée, tout le trafic est bloqué et l’agent contrôleur de fabric configure le filtre de paquets pour ajouter des *règles et des exceptions* afin d’autoriser le trafic autorisé.

Il existe deux catégories de règles qui sont programmés ici :

-   **Configuration de l’ordinateur ou les règles de l’Infrastructure**: par défaut, toutes les communications sont bloquée. Il existe des exceptions pour permettre à un ordinateur virtuel envoyer et recevoir le trafic DHCP et DNS. Ordinateurs virtuels peuvent également envoyer le trafic vers l’internet « public » et acheminer le trafic vers les autres ordinateurs virtuels dans le cluster et le serveur d’Activation du système d’exploitation. Les ordinateurs virtuels autorisé la liste de destinations sortantes n’inclut pas les sous-réseaux de routeur Azure, back-end Azure management et d’autres propriétés de Microsoft.

-   **Fichier de Configuration de rôle**: définit les ACL entrants basés sur le modèle de service des locataires. Par exemple, si un client a un site Web frontal sur le port 80 sur un certain VM, Azure ouvre le port TCP 80 pour toutes les adresses IP si vous configurez un point de terminaison dans le modèle de [Gestion des services Azure](../resource-manager-deployment-model.md) . Si la machine virtuelle a un rôle principal ou le travail en cours d’exécution, puis nous allons ouvrir le rôle de travail qu’à la machine virtuelle dans le même client.

##<a name="isolation"></a>Isolation

Maintenance de séparation pour empêcher accidentelle ou non autorisé de transfert d’informations entre les déploiements dans une architecture mutualisée partagée est une autre exigence de sécurité important de nuage.

Azure implémente [un contrôle d’accès réseau](https://azure.microsoft.com/blog/network-isolation-options-for-machines-in-windows-azure-virtual-networks/) et répartition des tâches via l’isolation de VLAN, ACL, chargent équilibreurs et les filtres IP. Externe de trafic entrant pour vos ordinateurs virtuels est restreinte aux ports et les protocoles que vous définissez. Filtrage du réseau est mis en oeuvre pour empêcher le trafic usurpé et limite le trafic entrant et sortant pour les composants de plateforme sécurisée. Par défaut, stratégies des flux de trafic sont implémentés sur des dispositifs de protection de frontière interdisant le trafic.

![Logiciel anti-programme malveillant de Microsoft dans Azure](./media/azure-security-getting-started/sec-azgsfig3.PNG)

Traduction d’adresses réseau (NAT) est utilisé pour séparer le trafic de réseau internes du trafic externe. Le trafic interne n’est pas routable en externe. Les [adresses IP virtuelles](http://blogs.msdn.com/b/cloud_solution_architect/archive/2014/11/08/vips-dips-and-pips-in-microsoft-azure.aspx) sont routables en externe sont traduites en adresses [IP dynamiques internes](http://blogs.msdn.com/b/cloud_solution_architect/archive/2014/11/08/vips-dips-and-pips-in-microsoft-azure.aspx) qui ne sont pas routables dans Azure.

Le trafic externe vers des machines virtuelles Azure est protégée par un pare-feu via les listes de contrôle d’accès (ACL) sur les routeurs, les équilibreurs de charge et les commutateurs de couche 3. Seuls les protocoles spécifiques connus sont autorisés. Les ACL sont en place pour limiter le trafic en provenance des ordinateurs virtuels invités d’autres VLAN utilisé pour la gestion. En outre, le trafic filtré via les filtres IP sur l’hôte du système d’exploitation, de limiter le trafic sur les deux couches de réseau et de la liaison de données.

### <a name="how-azure-implements-isolation"></a>Comment Azure implémente l’isolation

Le contrôleur de tissu Azure est chargé d’allouer les ressources de l’infrastructure pour le client des charges de travail et gère la communication unidirectionnelle à partir de l’hôte d’ordinateurs virtuels. L’hyperviseur Azure applique la mémoire et le processus de séparation entre les machines virtuelles, et en toute sécurité les itinéraires le trafic réseau aux locataires de système d’exploitation invité. Azure implémente également d’isolation pour les locataires, stockage et réseaux virtuels :

-   Chaque client AD Azure est isolé logiquement à l’aide de limites de sécurité.

-   Les comptes de stockage Azure sont uniques pour chaque abonnement, et l’accès doit être authentifié à l’aide d’une clé de compte de stockage.

-   Les réseaux virtuels sont isolés de façon logique grâce à une combinaison unique des adresses IP privées, les pare-feu et les ACL IP. Charge équilibrage acheminer le trafic vers les locataires appropriés en fonction des définitions de point de terminaison.

##<a name="virtual-network-and-firewall"></a>Réseau virtuel et du pare-feu

Les [réseaux virtuels et distribuées](http://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx) dans Azure garantissent que le trafic de votre réseau privé est isolé logiquement le trafic sur d’autres réseaux virtuels Azure.

![Logiciel anti-programme malveillant de Microsoft dans Azure](./media/azure-security-getting-started/sec-azgsfig4.PNG)

Votre abonnement peut contenir plusieurs réseaux privés isolés (et inclure des pare-feu, équilibrage de la charge et la traduction d’adresses réseau).

Azure fournit trois niveaux principaux de répartition réseau dans chaque cluster Azure pour logiquement répartir le trafic. [Réseaux locaux virtuels](https://azure.microsoft.com/services/virtual-network/) (Réseaux locaux virtuels) sont utilisés pour séparer le trafic du client du reste du réseau Azure. L’accès au réseau Azure depuis l’extérieur du cluster est limité via les équilibreurs de charge.

Le trafic réseau vers et à partir d’ordinateurs virtuels doit passer par le commutateur virtuel hyperviseur. Le composant de filtre IP dans le système d’exploitation racine isole la racine virtuelle à partir d’ordinateurs virtuels et des ordinateurs virtuels à partir d’un autre. Il effectue un filtrage du trafic de limiter les communications entre les nœuds du locataire et l’Internet public (selon la configuration du service du client), les séparer des autres locataires.

Le filtre IP permet d’empêcher des ordinateurs virtuels entre :

- Générer un trafic usurpé

- Recevoir du trafic non qui leur sont adressé

- Acheminement du trafic à la terminaison d’infrastructure protégés

- Envoyer ou recevoir du trafic de diffusion inapproprié

Vous pouvez placer vos machines virtuelles sur [Des réseaux virtuels Azure](https://azure.microsoft.com/documentation/services/virtual-network/). Ces réseaux virtuels est similaires aux réseaux que vous configurez dans les environnements locaux, où ils sont généralement associés à un switch virtuel. Les ordinateurs virtuels connectés au même réseau virtuel Azure peuvent communiquer entre eux sans configuration supplémentaire. Vous avez également la possibilité de configurer différents sous-réseaux au sein de votre réseau virtuel Azure.

Vous pouvez utiliser les technologies de réseau virtuel d’Azure suivantes pour aider à sécuriser les communications sur votre réseau virtuel Azure :

-   [**Les groupes de sécurité réseau (NSG)**](../virtual-network/virtual-networks-nsg.md). Vous pouvez utiliser un NSG pour contrôler le trafic à une ou plusieurs instances de la machine virtuelle (VM) dans votre réseau virtuel. Un NSG contient les règles de contrôle d’accès qui autorisent ou refusent le trafic en fonction de la direction du trafic, protocole, adresse source et de port et adresse de destination et port.

-   [**Routage défini par l’utilisateur**](../virtual-network/virtual-networks-udr-overview.md). Vous pouvez contrôler le routage des paquets via une appliance virtuelle en créant des chemins définis par l’utilisateur qui spécifient le saut suivant pour les paquets entrants à un sous-réseau spécifique à une appliance de sécurisation de votre réseau virtuel.

-   [**Le routage IP**](../virtual-network/virtual-networks-udr-overview.md). Une appliance de sécurisation réseau virtuel doit être en mesure de recevoir le trafic entrant qui n’est pas adressé à lui-même. Pour permettre à un ordinateur virtuel recevoir le trafic adressé vers d’autres destinations, vous activez le routage IP de la machine virtuelle.

-   [**Forcé de tunneling**](../vpn-gateway/vpn-gateway-about-forced-tunneling.md). Permet de tunneling forcé rediriger ou « forcer » lié aux Internet du trafic généré par vos ordinateurs virtuels dans un virtuel Azure revenir à votre emplacement sur site via un tunnel VPN de site à site d’inspection et d’audit

-   [Les ACL de **point de terminaison** ](../virtual-machines/virtual-machines-windows-classic-setup-endpoints.md). Vous pouvez contrôler quels ordinateurs sont autorisées les connexions entrantes à partir d’Internet, à un ordinateur virtuel sur votre réseau virtuel Azure en définissant des ACL de point de terminaison.

-   [**Solutions de sécurité réseau de partenaires**](https://azure.microsoft.com/marketplace/). Il existe un certain nombre de solution de sécurité réseau partenaire auquel vous pouvez accéder à partir de Azure Marketplace.

### <a name="how-azure-implements-virtual-networks-and-firewall"></a>Comment Azure implémente des pare-feu et des réseaux virtuels

Azure implémente des pare-feu à filtrage de paquets sur tous les hôtes et invités ordinateurs virtuels par défaut. Images de système d’exploitation Windows à partir de la galerie d’Azure ont également le pare-feu Windows est activé par défaut. Équilibreurs de charge au niveau du périmètre public en vis-à-vis des réseaux contrôler les communications du Azure basées sur les ACL IP gérés par les administrateurs de clients.

Si Azure déplace les données d’un client dans le cadre d’opérations normales ou durant un incident, il le fait sur des canaux de communications cryptées privé. Autres fonctionnalités exploitées par Azure à utiliser dans des pare-feu et des réseaux virtuels sont les suivantes :

-   **Pare-feu d’hôte natif**: tissu Azure et stockage s’exécutent sur un système d’exploitation natif qui a sans hyperviseur et, par conséquent, le pare-feu windows est configuré avec les deux ensembles de règles ci-dessus. Stockage exécute natif pour optimiser les performances.

-   **L’hôte de pare-feu**: le pare-feu d’hôte est de protéger le système d’exploitation hôte qui s’exécute de l’hyperviseur. Les règles sont programmées pour autoriser uniquement le contrôleur de tissu et passer les cases pour communiquer avec l’hôte du système d’exploitation sur un port spécifique. Les autres exceptions sont pour permettre la réponse DHCP et les réponses DNS. Azure utilise une configuration de machine de fichier qu’il possède le modèle de règles de pare-feu pour le système d’exploitation hôte. L’hôte lui-même est protégé contre les attaques externes par un pare-feu Windows configuré pour autoriser uniquement les communications provenant de sources connues et authentifiés.

-   **Pare-feu de l’invité**: réplique les règles dans le filtre de paquets de commutateur de machine virtuelle mais programmés dans différents logiciels (c'est-à-dire l’élément pare-feu Windows du système d’exploitation invité). Le pare-feu d’ordinateur virtuel invité peut être configuré pour restreindre les communications vers ou à partir de l’invité ordinateur virtuel, même si la communication est autorisée par la configuration au niveau de l’hôte IP Filter. Par exemple, vous pouvez choisir d’utiliser le pare-feu de la machine virtuelle guest pour limiter la communication entre deux de vos VNets qui ont été configurés pour se connecter à un autre.

-   **Stockage pare-feu (FW)**: le pare-feu sur le stockage frontal filtre le trafic uniquement sur les ports 80/443 et autres utilitaire nécessaire. Le pare-feu sur le back-end de stockage restreint communications proviennent uniquement de serveurs front-end de stockage.

-   **Passerelle de réseau virtuel**: [Les passerelles réseau virtuel Azure](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md) servent de passerelles de connexion de réseau virtuel d’Azure, vos charges de travail sur vos sites locaux sur des locaux de la croix. Il est nécessaire de se connecter à des sites locaux par le biais de [tunnels VPN de site à site IPsec](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)ou par le biais de circuits de [ExpressRoute](../expressroute/expressroute-introduction.md) . Les tunnels IPsec/IKE VPN, les passerelles effectuent les négociations IKE et établissent les tunnels IPsec S2S VPN entre réseaux virtuels et sur les sites locaux. Les passerelles de réseau virtuel arrêter également [les VPN site-à-point](../vpn-gateway/vpn-gateway-point-to-site-create.md).

##<a name="secure-remote-access"></a>Accès à distance sécurisé

Les données stockées dans le nuage doivent être activés pour empêcher les exploits et d’assurer la confidentialité et l’intégrité pendant le transit des garanties suffisantes. Cela inclut les contrôles de réseau qui se conformer basée sur des règles et vérifiables, des identités et des accès aux mécanismes de gestion d’une organisation.

Technologie de cryptage intégrée vous permet de crypter les communications au sein et entre les déploiements, entre les régions d’Azure et d’Azure aux centres de données sur site. Accès administrateur aux machines virtuelles via le portail de gestion Azure, [À distance de Windows PowerShell](http://blogs.technet.com/b/heyscriptingguy/archive/2013/09/07/weekend-scripter-remoting-the-cloud-with-windows-azure-and-powershell.aspx)et [sessions Bureau à distance](../virtual-machines/virtual-machines-windows-classic-connect-logon.md)est toujours chiffré.

Pour étendre en toute sécurité votre centre de données local dans le nuage, Azure fournit à la fois [VPN de site à site](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md) et [point-à-site VPN](../vpn-gateway/vpn-gateway-point-to-site-create.md), ainsi que les liaisons dédiées avec [ExpressRoute](../expressroute/expressroute-introduction.md) (connexions à des réseaux virtuels Azure via connexion VPN sont cryptées).

### <a name="how-azure-implements-secure-remote-access"></a>Comment Azure implémente un accès distant sécurisé

Connexions à Azure Portal doivent toujours être authentifiées et nécessitent SSL/TLS. Vous pouvez configurer les certificats de gestion pour permettre une gestion sécurisée. Industrie standard protocoles sécurisés tels que [IPsec](https://en.wikipedia.org/wiki/IPsec) et de [SSTP](https://technet.microsoft.com/magazine/2007.06.cableguy.aspx) sont entièrement pris en charge.

[ExpressRoute d’Azure](../expressroute/expressroute-introduction.md) permet de créer des connexions entre des centres de données Azure et l’infrastructure dans vos locaux ou dans un environnement de colocalisation. ExpressRoute les connexions ne passent pas sur l’Internet public. Ils offrent plus la fiabilité, vitesses plus rapides, latences inférieurs et une plus grande sécurité que les liaisons par défaut de basées sur Internet. Dans certains cas, à l’aide de connexions de ExpressRoute pour transférer des données entre des locaux et Azure peut également produire des économies significatives.

##<a name="logging-and-monitoring"></a>Journalisation et la surveillance

Azure fournit authentifié d’enregistrement des événements liés à la sécurité qui génèrent un journal d’audit et est conçu pour résister aux manipulations. Cela inclut les informations système, telles que les journaux des événements de sécurité dans l’infrastructure Azure VM et Azure AD. Surveillance des événements de sécurité inclut la collecte des événements comme des modifications dans DNS ou DHCP adresses IP du serveur, tentative d’accès à des ports, les protocoles ou les adresses IP qui sont bloquées par des modifications de conception, dans les paramètres de pare-feu ou de la stratégie de sécurité, création de compte ou de groupe, des processus inattendus ou l’installation du pilote.

![Logiciel anti-programme malveillant de Microsoft dans Azure](./media/azure-security-getting-started/sec-azgsfig5.PNG)

Des journaux d’audit enregistrement des accès utilisateur privilégié et activités, les tentatives d’accès autorisés et non autorisés, les exceptions de système et les événements de sécurité sont conservées pour un laps de temps défini. La conservation de vos journaux est à votre discrétion car vous configurez journal collecte et conservation à vos propres besoins.

### <a name="how-azure-implements-logging-and-monitoring"></a>Comment Azure implémente la journalisation et la surveillance

Azure déploie des agents Agents de gestion (MA) et le moniteur de sécurité Azure (ASM) pour chaque calcul, de stockage ou nœud de tissu sous gestion native ou virtuel. Chaque Agent de gestion est configuré pour l’authentification d’un compte de service de stockage équipe avec un certificat obtenu à partir du magasin de certificats Azure et transférer préconfigurée diagnostic et les données d’événement pour le compte de stockage. Ces agents ne sont pas déployés vers des machines virtuelles de clients.

Les administrateurs Azure accéder aux journaux via un portail web pour un accès authentifié et contrôlé dans les journaux. Un administrateur peut analyser, filtrer, mettre en corrélation et analyser les journaux. Les comptes de stockage Azure service team pour les journaux sont protégés contre les accès d’administrateur direct à protéger contre la falsification du journal.

Microsoft collecte des journaux à partir de périphériques réseau utilisant le protocole Syslog et à partir des serveurs utilisant Microsoft Audit Collection Services (ACS). Ces journaux est placés dans une base de données de journaux à partir de laquelle les alertes sont générées pour des événements suspects directement à un administrateur de Microsoft. L’administrateur peut accéder et analyser ces fichiers journaux.

[Diagnostics d’Azure](https://msdn.microsoft.com/library/azure/gg433048.aspx) est une fonctionnalité d’Azure qui vous permet de collecter des données de diagnostic à partir d’une application qui s’exécute dans Azure. Il s’agit de données de diagnostic pour le débogage et résolution des problèmes, mesurer les performances, l’utilisation des ressources, l’analyse de trafic et planification de la capacité de surveillance et d’audit. Une fois que les données de diagnostic sont collectées, il peut être transféré vers un compte de stockage Azure pour la persistance. Transferts peuvent être planifiées ou à la demande.

##<a name="threat-mitigation"></a>Atténuation des menaces

En plus de l’isolation, le cryptage et le filtrage, Azure utilise un certain nombre de mécanismes d’atténuation des menaces et des processus afin de protéger l’infrastructure et des services. Citons notamment les contrôles internes et technologies utilisés pour détecter et corriger les menaces telles que les [10 premiers OWASP avoir](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)DDoS et escalade des privilèges.

Les contrôles de sécurité et le processus de gestion des risques que Microsoft a mis en place pour sécuriser son infrastructure cloud réduisent le risque d’incidents de sécurité. Mais, en cas d’incident, l’équipe de gestion de la sécurité (SIM – Security Incident Management) au sein de l’équipe Microsoft Online Services de sécurité et de conformité (OSSC) est de 24 x 7 prêt à répondre.

### <a name="how-azure-implements-threat-mitigation"></a>Comment Azure implémente atténuation des menaces

Sécurité Azure a des contrôles en place pour implémenter l’atténuation des menaces et également à aider les clients à atténuer les menaces potentielles dans leurs environnements. La liste ci-dessous récapitule les fonctionnalités de réduction des risques de menace offertes par Azure :

-   [Azure Anti-Malware](../security/azure-security-antimalware.md) est activé par défaut sur tous les serveurs d’infrastructure. Vous pouvez éventuellement activer dans vos propres ordinateurs virtuels.

-   Microsoft tient à jour continue de surveillance entre les serveurs, les réseaux et les applications de détecter les menaces et d’empêcher les exploits. Alertes automatisées avertissent les administrateurs des comportements anormaux, ce qui leur permet de prendre les mesures correctives contre les menaces internes et externes.

-   Vous avez la possibilité de déployer des solutions de sécurité de 3 rd-party dans vos abonnements, tels que des pare-feu d’application web de [Barracuda](https://techlib.barracuda.com/ng54/deployonazure).

-   L’approche de Microsoft pour le test de pénétration inclut «[Association de rouge](http://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf)», qui implique des professionnels de la sécurité Microsoft attaquer les systèmes de production en direct (non-client) dans Azure pour tester des défenses contre les menaces permanentes de réelles et avancées.

-   Systèmes de déploiement intégré gérer la distribution et l’installation des correctifs de sécurité sur la plateforme Azure.

##<a name="next-steps"></a>Étapes suivantes

[Centre d’approbation Azure](https://azure.microsoft.com/support/trust-center/)

[Blog de l’équipe sécurité Azure](http://blogs.msdn.com/b/azuresecurity/)

[Microsoft Security Response Center](https://technet.microsoft.com/library/dn440717.aspx)

[Blog de Active Directory](http://blogs.technet.com/b/ad/)
