<properties
   pageTitle="Conseils de conception d’API | Microsoft Azure"
   description="Conseils sur la création d’un puits conçu l’API."
   services=""
   documentationCenter="na"
   authors="dragon119"
   manager="christb"
   editor=""
   tags=""/>

<tags
   ms.service="best-practice"
   ms.devlang="rest-api"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="07/13/2016"
   ms.author="masashin"/>

# <a name="api-design-guidance"></a>Conseils de conception d’API

[AZURE.INCLUDE [pnp-header](../includes/guidance-pnp-header-include.md)]

## <a name="overview"></a>Vue d’ensemble

De nombreuses solutions basées sur le web modernes rendent l’utilisation de services web, hébergés par des serveurs web, pour fournir des fonctionnalités de client à distance les applications. Les opérations de service web expose constituent une API web. Une API de web bien conçu doit viser pour prendre en charge :

- **Indépendance des plates-formes**. Les applications clientes doivent être en mesure d’utiliser l’API du service web fournit sans nécessiter la façon dont les données ou les opérations qui expose les API sont physiquement implémentées. Cette opération nécessite que l’API respecte des normes communes qui permettent à un service web et une application de client d’accord sur les formats de données à utiliser et la structure des données qui sont échangées entre les applications client et le service web.

- **Évolution du Service**. Le service web doit pouvoir évoluer et ajouter (ou supprimer) la fonctionnalité indépendamment des applications clientes. Les applications clientes existantes doivent être en mesure de continuer à fonctionner en non modifié comme les fonctionnalités fournies par la modification du service web. Toutes les fonctionnalités doivent également être détectable, afin que les applications clientes peuvent utiliser pleinement il.

L’objectif de ce guide est de décrire les problèmes que vous devez considérer lorsque vous créez un site web API.

## <a name="introduction-to-representational-state-transfer-rest"></a>Introduction à transfert d’état Representational (reste)

Dans son dissertation en 2000, Roy fielding a proposé une autre approche architecturale à structurer les opérations exposées par les services web ; RESTE. REST est un style architectural pour créer des systèmes distribués selon hypermédia. Le principal avantage du modèle reste est qu’il est basé sur des normes ouvertes et qu’elle ne lie pas l’implémentation du modèle ou les applications clientes qui y accéder à l’implémentation spécifique. Par exemple, un service web de repos peut être implémenté à l’aide de l’API de Web Microsoft ASP.NET, et les applications clientes pourraient être développées à l’aide de n’importe quel langage et un ensemble d’outils qui peut générer des requêtes HTTP et analyser les réponses HTTP.

> [AZURE.NOTE] REST est réellement indépendante de tout protocole sous-jacent et n’est pas nécessairement lié à HTTP. Toutefois, les implémentations les plus fréquentes des systèmes basés sur le reste utilisent HTTP comme le protocole d’application pour l’envoi et la réception des demandes. Ce document se concentre sur les principes du reste de mappage conçus pour fonctionner à l’aide de HTTP.

Le modèle reste utilise un schéma de navigation pour représenter des objets et des services sur un réseau (appelé _ressources_). De nombreux systèmes qui implémentent généralement les autres utilisent le protocole HTTP pour transmettre des demandes d’accès à ces ressources. Dans ces systèmes, une application cliente soumet une requête sous la forme d’un URI qui identifie une ressource et une méthode HTTP (les plus courants étant GET, POST, PUT ou DELETE) qui indique l’opération à effectuer sur cette ressource.  Le corps de la requête HTTP contient les données nécessaires pour effectuer l’opération. Le point important à comprendre est que reste définit un modèle de demande sans état. Requêtes HTTP doivent être indépendants et peuvent se produire dans n’importe quel ordre, afin d’essayer de conserver les informations d’état transitoire entre des demandes n’est pas possible.  Le seul endroit où les informations sont stockées est dans les ressources elles-mêmes, et chaque demande doit être une opération atomique. En effet, un modèle reste implémente une machine à états finis où une demande passe à une ressource, d’un état non transitoires bien définie à un autre.

> [AZURE.NOTE] La nature sans état des requêtes individuelles du modèle reste active un système construit en suivant ces principes pour être hautement évolutive. Il n’est pas nécessaire de conserver une affinité entre une application cliente effectue une série de demandes et les serveurs web spécifique gère ces demandes.

Un autre point essentiel dans la mise en œuvre d’un modèle reste efficace est de comprendre les relations entre les diverses ressources auquel le modèle fournit un accès. Ces ressources sont généralement organisés en collections et les relations. Par exemple, supposons qu’une analyse rapide d’un système de commerce électronique indique qu’il y a deux collections dans laquelle les applications sont susceptibles d’être intéressées : clients et commandes. Chaque commande et le client doivent avoir sa propre clé unique à des fins d’identification. L’URI pour accéder à la collection de commandes peut être quelque chose d’aussi simple que _/orders_, et de même URI pour l’extraction de tous les clients peut être _/customers_. Émission d’une requête HTTP GET à l’URI _/orders_ doit retourner la liste représentant toutes les commandes dans la collection codée sous la forme d’une réponse HTTP :

```HTTP
GET http://adventure-works.com/orders HTTP/1.1
...
```

La réponse ci-dessous encode les commandes sous la forme d’une structure de liste JSON :

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
[{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1},{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2},{"orderId":3,"orderValue":16.60,"productId":2,"quantity":4},{"orderId":4,"orderValue":25.90,"productId":3,"quantity":1},{"orderId":5,"orderValue":99.90,"productId":1,"quantity":1}]
```
Pour extraire un ordre nécessite de spécifier l’identificateur pour la commande à partir de la ressource de _commandes_ , comme _/orders/2_:

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
```

```HTTP
HTTP/1.1 200 OK
...
Date: Fri, 22 Aug 2014 08:49:02 GMT
Content-Length: ...
{"orderId":2,"orderValue":10.00,"productId":4,"quantity":2}
```

> [AZURE.NOTE] Pour plus de simplicité, ces exemples illustrent les informations dans les réponses renvoyées sous forme de données de texte JSON. Toutefois, il n’existe aucune raison pourquoi ressources ne doivent pas contenir tout autre type de données pris en charge par HTTP, telles que le binaire ou crypté ; le type de contenu dans la réponse HTTP doit spécifier le type. En outre, un modèle reste peut être en mesure de retourner les mêmes données dans différents formats, tels que XML ou JSON. Dans ce cas, le service web doit être en mesure d’effectuer la négociation de contenu avec le client qui effectue la demande. La demande peut inclure un en-tête _Accept_ qui spécifie le format par défaut que le client souhaite recevoir et le service web doit tenter de respecter ce format si possible.

Notez que la réponse à une demande de repos, l’utilisation de la norme HTTP codes d’état. Par exemple, une requête qui retourne des données valides doit inclure le code de réponse HTTP 200 (OK), lors d’une demande qui ne parvient pas à trouver ou de supprimer une ressource spécifiée doit renvoyer une réponse qui inclut le code d’état HTTP 404 (introuvable).

## <a name="design-and-structure-of-a-restful-web-api"></a>Conception et la structure d’un site web RESTful API

Les clés à la conception d’une API web réussie sont simplicité et cohérence. Une API Web qui présente ces deux facteurs rend plus facile de créer des applications clientes qui doivent utiliser l’API.

Une API de web RESTful se concentre sur l’exposition d’un ensemble de ressources connectées et de fournir des opérations essentielles qui permettent à une application de manipuler ces ressources et de naviguer facilement entre les. Pour cette raison, les URI qui constituent une API web RESTful classique doivent être orientées vers les données qu’il expose et les fonctionnalités fournies par HTTP permet d’intervenir sur ces données. Cette approche nécessite une vision différente de celle généralement utilisée lors de la création d’un ensemble de classes dans une API orientée objet qui tend à être plus motivé par le comportement d’objets et de classes. En outre, une API de web RESTful doit être sans état et ne dépendent pas des opérations appelées dans une séquence particulière. Les sections suivantes résument les points à étudier lors de la conception d’un site web API RESTful.

### <a name="organizing-the-web-api-around-resources"></a>Organisation de l’API web autour de ressources

> [AZURE.TIP] Les URI exposées par un service web de repos doivent être basé sur les noms (les données auxquelles l’API web fournit l’accès) et pas les verbes (ce qu’une application peut faire avec les données).

Se concentrer sur les entités métier qui expose les API web. Par exemple, dans un site web API prend en charge le système de commerce électronique décrit précédemment, les entités principales sont customers et orders. Processus, tels que le fait de passer une commande est possible grâce à une opération HTTP POST qui accepte les informations de commande et l’ajoute à la liste des commandes pour le client. En interne, cette opération POST peut effectuer des tâches telles que la vérification des niveaux de stock et de la facturation. La réponse HTTP peut indiquer si la commande a été effectuée avec succès ou non. Notez également qu’une ressource n’a pas à être basée sur un élément de données physique unique. Par exemple, une ressource de commande peut être implémentée en interne à l’aide des informations agrégées à partir de nombreuses lignes réparties sur plusieurs tables dans une base de données relationnelle, mais présentée au client comme une entité unique.

> [AZURE.TIP] Évitez la conception d’une interface reste qui reflète ou dépend de la structure interne des données qu’il expose. RESTE est plus de mise en œuvre de simples opérations CRUD (Create, Retrieve, Update, Delete) sur des tables séparées dans une base de données relationnelle. L’objectif de repos est pour mapper les entités métier et les opérations qu’une application peut effectuer sur ces entités pour l’implémentation physique de ces entités, mais un client ne doit pas être exposé à ces détails physiques.

Les entités métier individuelles rarement existent isolément (bien que certains objets singleton peuvent exister), mais plutôt tendance à être regroupés dans des collections. Dans d’autres termes, chaque entité et chaque collection sont des ressources. Dans une API web RESTful, chaque collection a sa propre URI dans le service web et l’exécution d’une requête HTTP GET sur un URI pour une collection récupère une liste d’éléments de la collection. Chaque élément individuel possède également son propre URI et une application peut soumettre une autre demande HTTP GET à l’aide de cet URI pour récupérer les détails de cet élément. Vous devez organiser les URI pour les collections et les éléments de manière hiérarchique. Dans le système de commerce électronique, l' URI _/customers_ désigne les collection du client, et le _/customers/5_ récupère les détails pour le seul client avec l’ID 5 à partir de cette collection. Cette approche permet de conserver l’API web intuitive.

> [AZURE.TIP] Adopter une convention d’affectation de noms cohérente dans les URI ; en général, il faut pour utiliser des noms au pluriel pour les URI que collections de référence.

Vous devez également examiner les relations entre les différents types de ressources et comment vous pouvez exposer ces associations. Par exemple, les clients peuvent placer zéro, une ou plusieurs commandes. Un moyen naturel pour représenter cette relation serait via un URI tel que _/customers/5/orders_ , pour rechercher toutes les commandes pour le client 5. Vous pouvez également envisager de représentant l’association d’une commande à un client spécifique via un URI tel que _/orders/99/customer_ pour trouver le client pour l’ordre de 99, mais étend ce modèle plus loin peut devenir fastidieux à implémenter. Une meilleure solution consiste à fournir des liens de navigation vers associé ressources, telles que le client, dans le corps du message de réponse HTTP renvoyé lorsque l’ordre est interrogée. Ce mécanisme est décrite plus en détail dans la section Utilisation de l’approche HATEOAS pour activer la Navigation aux ressources connexes plus loin dans ce guide.

Dans des systèmes plus complexes, il peut être beaucoup plus de types d’entité, et il peut être tentant de fournir l’URI qui permettent à une application cliente naviguer à travers plusieurs niveaux de relations, telles que _/customers/1/orders/99/products_ , pour obtenir la liste des produits dans l’ordre de 99 passées par le client 1. Toutefois, ce niveau de complexité peut être difficile à gérer et fixe, si les relations entre les ressources changer à l’avenir. Au lieu de cela, vous devez rechercher garder un URI relativement simple. L’esprit qu’une fois qu’une application a une référence à une ressource, il doit être possible d’utiliser cette référence pour rechercher des éléments liés à cette ressource. La requête précédente peut être remplacée par l' URI _/customers/1/orders_ pour rechercher toutes les commandes pour le client 1 et ensuite interroger l' URI _/orders/99/products_ pour trouver les produits dans l’ordre (en supposant que 99 de commande par le client 1).

> [AZURE.TIP] Évitez nécessitant plus complexe que la _collection/élément/collection_d’URI de ressource.

Un autre point à prendre en compte est que toutes les requêtes web imposent une charge sur le serveur web, et plus le nombre de demandes plus la charge. Vous devez tenter de définir vos ressources pour éviter les « bavardages » web API qui exposent un grand nombre de ressources small. Une telle API peut-être nécessiter une application client d’envoyer plusieurs demandes pour rechercher toutes les données dont il a besoin. Il peut être avantageux de dénormaliser données et combiner des informations en plus grands ressources qui peuvent être extraites par l’émission d’une demande unique. Toutefois, vous devez équilibrer cette approche par rapport à la charge de l’extraction des données qui ne peuvent pas être fréquemment requises par le client. Récupération d’objets volumineux peut augmenter la latence d’une demande et de supporter des coûts de bande passante supplémentaire pour un petit avantage si des données supplémentaires ne sont pas souvent utilisées.

Éviter d’introduire des dépendances entre le API web à la structure, le type ou l’emplacement des sources de données sous-jacentes. Par exemple, si vos données se trouvent dans une base de données relationnelle, l’API web n’a pas besoin d’exposer chaque table sous la forme d’une collection de ressources. Considérez le web API comme une abstraction de la base de données et si nécessaire introduire une couche de mappage entre la base de données et de l’API web. De cette façon, si la conception ou l’implémentation de la base de données est modifiée (par exemple, vous déplacez à partir d’une base de données relationnelle contenant une collection de tables normalisées à un système de stockage NoSQL dénormalisé comme une base de données de document) les applications clientes sont isolées à partir de ces modifications.
> [AZURE.TIP] La source de données qui sous-tend une API web ne doit pas être un magasin de données ; Il peut être un autre service ou line-of-business application ou même une application héritée en cours d’exécution sur site d’une organisation.

Enfin, il peut être pas possible de mapper chaque opération implémentée par une API web à une ressource spécifique. Vous pouvez gérer ces scénarios _non-ressource_ par le biais de demandes HTTP GET, une partie des fonctionnalités d’appel et renvoient les résultats sous la forme d’un message de réponse HTTP. Un site web API qui implémente des opérations de style calculatrice simple comme ajouter et soustraire peut fournir l’URI qui exposent ces opérations en tant que ressources de pseudo et utiliser la chaîne de requête pour spécifier les paramètres requis. Par exemple une requête GET à l’URI _/add?operand1 = 99 & operand2 = 1_ peut renvoyer un message de réponse auprès de l’organisme qui contient la valeur 100 et obtenir la demande à l’URI _/subtract?operand1 = 50 & operand2 = 20_ Impossible de renvoyer un message de réponse avec le corps contenant la valeur 30. Toutefois, n’utilisez ces formes d’URI avec modération.

### <a name="defining-operations-in-terms-of-http-methods"></a>Définition des opérations en fonction des méthodes HTTP

Le protocole HTTP définit un certain nombre de méthodes qui affecter la signification sémantique à une demande. Les méthodes HTTP communes utilisées par les API de web RESTful plus sont :

- **Obtenir**, pour récupérer une copie de la ressource à l’URI spécifié. Le corps du message de réponse contient les détails de la ressource demandée.

- **POST**, pour créer une nouvelle ressource à l’URI spécifié. Le corps du message de requête fournit les détails de la nouvelle ressource. Notez que POST peut également servir à déclencher des opérations qui en réalité ne créez pas de ressources.

- **Placer**, de remplacer ou de mettre à jour la ressource au niveau de l’URI spécifié. Le corps du message de demande spécifie la ressource à modifier et les valeurs à appliquer.

- **Supprimer**, pour supprimer la ressource au niveau de l’URI spécifié.

> [AZURE.NOTE] Le protocole HTTP définit également d’autres méthodes moins couramment utilisées, telles que des correctifs qui est utilisé pour demander des mises à jour sélectives à une ressource, qui est utilisé pour demander une description d’une ressource, les OPTIONS qui permet à un client obtenir des informations sur les options de communication pris en charge par le serveur, des informations d’en-tête et de TRACE qui permet à un client pour demander des informations qu’il peut utiliser à des fins de test et les tests de diagnostic.

L’effet d’une demande spécifique doit dépendre soit de la ressource à laquelle elle est appliquée sur une collection ou un élément individuel. Le tableau suivant résume les conventions courantes adoptées par des implémentations plus RESTful à l’aide de l’exemple du commerce électronique. Notez que certaines de ces demandes peuvent être implémentée ; il dépend du scénario spécifique.

| **Ressources** | **Publier** | **Télécharger** | **PUT** | **SUPPRIMER** |
|--------------|----------|---------|---------|------------|
| /Customers. | Créer un nouveau client | Récupérer tous les clients | Mise à jour globale des clients (_si implémenté_) | Supprimer tous les clients |
| clients/1 | Erreur | Récupérer les détails pour le client 1 | Mettre à jour des détails de client 1 s’il existe, sinon retour une erreur | Supprimer le client 1 |
| /Customers/1/Orders | Créer une nouvelle commande pour le client 1 | Extrait toutes les commandes pour le client 1 | Mettre à jour des commandes pour le client 1 (_si implémenté_) | Supprimer toutes les commandes pour le client 1 (_si implémenté_) |

L’objectif de demandes GET et DELETE sont relativement simples, mais il est portée de confusion concernant le rôle et les effets des requêtes POST et PUT.

Une demande POST doit créer une ressource avec les données fournies dans le corps de la demande. Dans le modèle reste, vous appliquez fréquemment demandes POST aux ressources qui sont des collections ; la nouvelle ressource est ajoutée à la collection.

> [AZURE.NOTE] Vous pouvez également définir les demandes POST qui déclenche certaines fonctionnalités (et qui ne renvoient pas nécessairement données), et ces types de requête peuvent être appliquées aux collections. Par exemple, vous pouvez utiliser une requête POST pour passer d’une feuille de temps à un service de traitement de paie et obtenir les taxes calculées en tant que réponse.

Une requête PUT vise à modifier une ressource existante. Si la ressource spécifiée n’existe pas, la requête PUT peut renvoyer une erreur (dans certains cas, il risque de créer la ressource). Les requêtes PUT sont plus souvent appliquées aux ressources qui sont des éléments individuels (par exemple, un client ou un ordre), bien qu’elles puissent être appliquées à des collections, bien que moins fréquemment, cela est implémenté. Notez que PUT demande sont idempotents, alors que les requêtes POST ne le sont pas ; Si une application soumet le même placer demande plusieurs fois les résultats doivent toujours être la même (la même ressource est modifiée avec les mêmes valeurs), mais si une application répète la requête POST même le résultat sera la création de plusieurs ressources.

> [AZURE.NOTE] En principe, une demande HTTP PUT remplace une ressource existante avec la ressource spécifiée dans le corps de la demande. Si l’intention est de modifier une sélection de propriétés d’une ressource tout en conservant les autres propriétés inchangées, il doit être implémenté à l’aide d’une requête HTTP des correctifs. Toutefois, les nombreuses implémentations RESTful assouplir cette règle et utilisent l’instruction PUT dans les deux cas.

### <a name="processing-http-requests"></a>Traitement des demandes HTTP
Les données incluses par une application cliente de nombreuses requêtes HTTP et les messages de réponse correspondants à partir du serveur web, peuvent être présentées dans une variété de formats (ou des types de médias). Par exemple, les données qui spécifie les détails d’une commande ou un client pourraient fournies en tant que XML, JSON ou un autre format encodé et compressé. Une API de web RESTful doit prendre en charge différents types de supports comme demandé par l’application cliente qui envoie une demande.

Lorsqu’une application cliente envoie une requête qui retourne des données dans le corps d’un message, elle peut spécifier les types de médias qu’il peut traiter dans l’en-tête Accept de la requête. Le code suivant illustre une demande HTTP GET qui Récupère les détails du client 1 et demande le résultat à retourner en tant que JSON (le client doit examiner toujours le type de média des données dans la réponse pour vérifier le format des données retournées) :

```HTTP
GET http://adventure-works.com/orders/2 HTTP/1.1
...
Accept: application/json
...
```

Si le serveur web prend en charge ce type de média, il peut répondre par une réponse qui inclut l’en-tête Content-Type qui spécifie le format des données dans le corps du message :

> [AZURE.NOTE] Pour une interopérabilité maximale, le support types référencés dans les en-têtes Accept et Content-Type doivent être reconnu des types MIME plutôt qu’un support personnalisé de type.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":2,"productID":4,"quantity":2,"orderValue":10.00}
```

Si le serveur web ne prend pas en charge le type de média requis, il peut envoyer les données dans un format différent. DANS tous les cas, il doit spécifier le type de média (par exemple _application/json_) dans l’en-tête Content-Type. Il est de la responsabilité de l’application cliente pour analyser le message de réponse et d’interpréter de manière appropriée les résultats dans le corps du message.

Notez que dans cet exemple, le serveur web correctement extrait les données demandées et indique la réussite en passant de nouveau un code d’état 200 dans l’en-tête de réponse. Si aucune donnée n’est trouvée, elle doit renvoyer un code d’état de 404 (introuvable) et le corps du message de réponse peut contenir des informations supplémentaires. Le format de ces informations est spécifié par l’en-tête Content-Type, comme illustré dans l’exemple suivant :

```HTTP
GET http://adventure-works.com/orders/222 HTTP/1.1
...
Accept: application/json
...
```

Commande 222 n’existe pas, donc le message de réponse se présente comme suit :

```HTTP
HTTP/1.1 404 Not Found
...
Content-Type: application/json; charset=utf-8
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"message":"No such order"}
```

Lorsqu’une application envoie une requête HTTP PUT pour mettre à jour d’une ressource, il spécifie l’URI de la ressource et fournit les données à modifier dans le corps du message de demande. Elle doit également spécifier le format de ces données à l’aide de l’en-tête Content-Type. Un format commun utilisé pour les informations de texte est _application/x-www-form-urlencoded_, qui comprend un ensemble de paires nom/valeur séparées par le caractère &. L’exemple suivant montre une requête PUT HTTP qui modifie les informations de la commande 1 :

```HTTP
PUT http://adventure-works.com/orders/1 HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
ProductID=3&Quantity=5&OrderValue=250
```

Si la modification est réussie, il doit idéalement répond avec un code d’état HTTP 204, indiquant que le processus a été traité correctement, mais que le corps de la réponse ne contient aucune information supplémentaire. L’en-tête d’emplacement de la réponse contient l’URI de la ressource qui vient d’être mis à jour :

```HTTP
HTTP/1.1 204 No Content
...
Location: http://adventure-works.com/orders/1
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

> [AZURE.TIP] Si les données dans un message de requête HTTP PUT comprennent les informations de date et d’heure, assurez-vous que votre service web accepte les dates et les heures au format suivant la norme ISO 8601.

Si la ressource à mettre à jour n’existe pas, le serveur web peut répondre avec une réponse introuvable comme décrit précédemment. Ou bien, si le serveur crée l’objet lui-même il peut retourner des codes de l’état HTTP 200 (OK) ou HTTP 201 (créé) et le corps de la réponse peuvent contenir les données pour la nouvelle ressource. Si l’en-tête Content-Type de la demande spécifie un format de données que le serveur web ne peut pas gérer, il doit répondre avec un code d’état HTTP 415 (Type de média non pris en charge).

> [AZURE.TIP] Envisagez d’implémenter des opérations en bloc HTTP PUT qui peuvent les mises à jour plusieurs ressources dans une collection de lot. La requête PUT doit spécifier l’URI de la collection, et le corps de la demande doit spécifier les détails des ressources à modifier. Cette approche peut aider à réduire la longueur et d’améliorer les performances.

Le format d’une demande HTTP POST qui créent de nouvelles ressources sont similaires à celles des demandes PUT ; le corps du message contient les détails de la nouvelle ressource à ajouter. Toutefois, l’URI spécifie généralement la collection à laquelle la ressource doit être ajoutée. L’exemple suivant crée un nouvel ordre et l’ajoute à la collection de commandes :

```HTTP
POST http://adventure-works.com/orders HTTP/1.1
...
Content-Type: application/x-www-form-urlencoded
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
productID=5&quantity=15&orderValue=400
```

Si la demande réussit, le serveur web doit répondre avec un code de message avec le code d’état HTTP 201 (créé). L’en-tête Location doit contenir l’URI de la ressource nouvellement créée, et le corps de la réponse doit contenir une copie de la nouvelle ressource. l’en-tête Content-Type spécifie le format de ces données :

```HTTP
HTTP/1.1 201 Created
...
Content-Type: application/json; charset=utf-8
Location: http://adventure-works.com/orders/99
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
Content-Length: ...
{"orderID":99,"productID":5,"quantity":15,"orderValue":400}
```

> [AZURE.TIP] Si les données fournies par une requête PUT ou POST ne sont pas valides, le serveur web doit répondre avec un message avec le code d’état HTTP 400 (requête incorrecte). Le corps de ce message peut contenir des informations supplémentaires sur le problème lié à la demande et les formats attendus, ou il peut contenir un lien vers une URL qui fournit plus de détails.

Pour supprimer une ressource, une demande HTTP DELETE fournit simplement l’URI de la ressource à supprimer. L’exemple suivant tente de supprimer l’ordre de 99 :

```HTTP
DELETE http://adventure-works.com/orders/99 HTTP/1.1
...
```

Si l’opération de suppression réussit, le serveur web doit répondre avec un code d’état HTTP 204, indiquant que le processus a été traité correctement, mais que le corps de la réponse ne contient aucune information supplémentaire (c’est la même réponse retournée par une opération PUT réussie, mais sans un en-tête d’emplacement que la ressource n’existe plus.) Il est également possible pour une demande de suppression renvoyer le code d’état HTTP 200 (OK) ou 202 (accepté) si la suppression est exécutée de façon asynchrone.

```HTTP
HTTP/1.1 204 No Content
...
Date: Fri, 22 Aug 2014 09:18:37 GMT
```

Si la ressource n’est pas trouvée, le serveur web doit retourner un message de 404 (introuvable).

> [AZURE.TIP] Si toutes les ressources dans une collection doivent être supprimées, activer une requête DELETE HTTP pour l’URI de la collection, plutôt que de forcer l’application à supprimer de chaque ressource à tour de rôle à partir de la collection.

### <a name="filtering-and-paginating-data"></a>Le filtrage et la pagination des données

Vous efforcer de conserver les URI simple et intuitive. Une collection de ressources via une seule URI aide à cet égard, mais il peut entraîner l’extraction de grandes quantités de données lorsque seul un sous-ensemble des informations est requis. Génération d’un grand volume de trafic affecte non seulement les performances et l’évolutivité du serveur web, mais également affecter le temps de réponse des applications client demande les données.

Par exemple, si les commandes contiennent le prix payé pour la commande, une application cliente qui a besoin de récupérer toutes les commandes qui ont un coût sur une valeur spécifique peut-être récupérer toutes les commandes à partir de l’URI _/orders_ et filtrer ces commandes localement. Clairement, ce processus est extrêmement inefficace ; Il fait perdre de puissance de traitement et de la bande passante réseau sur le serveur qui héberge le site web API.

Une solution peut être à fournir un schéma d’URI tel que _/orders/ordervalue_greater_than_n_ , où _n_ est le prix de la commande, mais pour tous, mais un nombre limité de prix une telle approche n’est pas pratique. En outre, si vous avez besoin pour les commandes de requête en fonction d’autres critères, vous pouvez vous retrouver confrontée avec offrant une longue liste d’URI avec des noms parfois non intuitives.

Une meilleure stratégie pour le filtrage des données est de fournir des critères de filtre dans la chaîne de requête qui est passé sur le web, API, telles que _/orders?ordervaluethreshold = n_. Dans cet exemple, l’opération correspondante dans l’API de web est responsable de l’analyse et la gestion de la `ordervaluethreshold` paramètre dans la chaîne de requête et en retournant les résultats filtrés dans la réponse HTTP.

Certaines demandes HTTP GET simples sur les ressources de collection peuvent très bien renvoyer un grand nombre d’éléments. Pour lutter contre le risque de ce problème vous devez concevoir le web API pour limiter la quantité de données retournées par une requête unique. Vous pouvez y parvenir en prenant en charge les chaînes de requête qui permettent à l’utilisateur de spécifier le nombre maximal d’éléments à récupérer (qui peut lui-même être soumis à une limite basse afin d’empêcher les attaques par déni de Service) et un offset de début dans la collection. Par exemple, la chaîne de requête dans l’URI _/orders?limit = 25 & offset = 50_ doit récupérer 25 commandes commençant par la commande 50e trouvée dans la collection de commandes. Comme les filtres de données, l’opération qui implémente la requête GET dans l’API de web est responsable de l’analyse et la gestion de la `limit` et `offset` les paramètres dans la chaîne de requête. Pour aider les applications clientes, obtenir des requêtes qui retournent des données paginées doivent également inclure des métadonnées qui indiquent le nombre total des ressources disponibles dans la collection. Vous pouvez également envisager d’autres stratégies de pagination intelligent ; Pour plus d’informations, consultez [API Design Notes : échange Smart](http://bizcoder.com/api-design-notes-smart-paging)

Vous pouvez suivre une stratégie similaire pour le tri des données telle qu’elle est lue ; Vous pouvez fournir un paramètre de tri qui prend un nom de champ la valeur, tel que _/orders?sort = ProductID_. Toutefois, notez que cette approche peut avoir un effet nuisible sur la mise en cache (paramètres de chaîne font partie de l’identificateur de ressource utilisé par de nombreuses implémentations de cache que la clé de données en mémoire cache de requête).

Vous pouvez étendre cette approche pour limiter (projet) les champs renvoyés si un élément de ressource unique contient une grande quantité de données. Par exemple, vous pouvez utiliser un paramètre de chaîne de requête qui accepte une liste délimitée par des virgules des champs, tel que _/orders?fields = ProductID, quantité_.

> [AZURE.TIP] Donnez à tous les paramètres facultatifs dans les chaînes de requête par défaut explicite. Par exemple, définir la `limit` paramètre 10 et le `offset` paramètre sur 0 si vous implémentez la pagination, définissez le paramètre de tri à la clé de la ressource si vous implémentez le tri et la valeur la `fields` paramètre à tous les champs dans la ressource si vous prenez en charge des projections.

### <a name="handling-large-binary-resources"></a>Gestion des ressources binaires volumineux

Une seule ressource peut contenir les champs binaires de grande taille, tels que des fichiers ou des images. Pour surmonter la transmission des problèmes causés par des connexions non fiables et intermittentes et pour améliorer les temps de réponse, envisagez de fournir des opérations qui permettent à ces ressources à récupérer en segments par l’application cliente. Pour ce faire, l’API web doit prend en charge l’en-tête Accept-Ranges pour les requêtes GET pour les ressources de grande taille et idéalement implémenter les demandes HTTP HEAD pour ces ressources. L’en-tête Accept-Ranges indique que l’opération d’obtention prend en charge les résultats partiels et qu’une application cliente peut envoyer des demandes GET qui renvoient un sous-ensemble d’une ressource spécifiée sous la forme d’une plage d’octets. Une requête HEAD est similaire à une demande GET, excepté qu’elle renvoie uniquement un en-tête qui décrit la ressource et un corps de message vide. Une application cliente peut émettre une requête HEAD pour déterminer s’il faut extraire une ressource à l’aide des requêtes GET partielles. L’exemple suivant montre une requête HEAD qui obtienne des informations sur une image du produit :

```HTTP
HEAD http://adventure-works.com/products/10?fields=productImage HTTP/1.1
...
```

Le message de réponse contient un en-tête qui inclut la taille de la ressource (4580 octets) et l’en-tête Accept-Ranges que l’opération GET correspondante prend en charge les résultats partiels :

```HTTP
HTTP/1.1 200 OK
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 4580
...
```

L’application cliente peut utiliser ces informations pour construire une série d’opérations GET pour extraire l’image en segments plus petits. La première requête extrait les premiers octets de 2500 à l’aide de l’en-tête de la plage :

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=0-2499
...
```

Le message de réponse indique qu’il s’agit d’une réponse partielle en renvoyant le code d’état HTTP 206. L’en-tête Content-Length indique le nombre réel d’octets renvoyés dans le corps du message (pas la taille de la ressource) et l’en-tête Content-Range indique quelle partie de la ressource il s’agit (octets 0-2 499 de 4580) :

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2500
Content-Range: bytes 0-2499/4580
...
_{binary data not shown}_
```

Une demande ultérieure de l’application cliente peut récupérer le reste de la ressource à l’aide d’un en-tête de plage approprié :

```HTTP
GET http://adventure-works.com/products/10?fields=productImage HTTP/1.1
Range: bytes=2500-
...
```

Le message de résultat correspondant doit ressembler à ceci :

```HTTP
HTTP/1.1 206 Partial Content
...
Accept-Ranges: bytes
Content-Type: image/jpeg
Content-Length: 2080
Content-Range: bytes 2500-4580/4580
...
```

## <a name="using-the-hateoas-approach-to-enable-navigation-to-related-resources"></a>Pour activer la navigation vers d’autres ressources à l’aide de l’approche HATEOAS

Une des principales motivations reste est qu’il doit être possible de naviguer dans l’ensemble des ressources sans nécessiter une connaissance préalable du modèle URI. Chaque demande HTTP GET doit retourner les informations nécessaires pour trouver les ressources liées directement à l’objet demandé par le biais de liens hypertexte inclus dans la réponse, et il doit l’être également des informations qui décrit les opérations disponibles sur chacune de ces ressources. Ce principe est appelé HATEOAS, ou Hypertext comme le moteur d’état de l’Application. Le système est effectivement une machine à états finis, et la réponse à chaque demande contient les informations nécessaires pour passer d’un état à l’autre ; aucune autre information n’est nécessaire.

> [AZURE.NOTE] Actuellement il n’y aucune normes ou spécifications qui définissent comment le principe HATEOAS de modèle. Les exemples indiqués dans cette section illustrent une solution possible.

Par exemple, pour gérer la relation entre les clients et les commandes, les données retournées dans la réponse pour une commande spécifique doivent contenir l’URI sous la forme d’un lien hypertexte qui identifie le client qui a passé la commande et les opérations pouvant être effectuées sur ce client.

```HTTP
GET http://adventure-works.com/orders/3 HTTP/1.1
Accept: application/json
...
```

Le corps du message de réponse contient un `links` tableau (mis en surbrillance dans l’exemple de code) qui spécifie la nature de la relation (_client_), l’URI du client (_http://adventure-works.com/customers/3_), comment récupérer les détails de ce client (_GET_) et les types MIME que le serveur web prend en charge de la récupération d’informations (_text/xml_ et _application/json_). Il s’agit d’une application cliente doit être en mesure d’extraire les détails du client toutes les informations. En outre, le tableau de liens inclut également des liens vers d’autres opérations qui peuvent être effectuées, telles que PUT (pour modifier le client, ainsi que le format que le serveur web s’attend le client à fournir) et supprimer.

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[(some links omitted){"rel":"customer","href":" http://adventure-works.com/customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":"
customer","href":" http://adventure-works.com /customers/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"customer","href":" http://adventure-works.com /customers/3","action":"DELETE","types":[]}]}
```

Par souci d’exhaustivité, le tableau de liens doit également inclure auto-référencement des informations se rapportant à la ressource qui a été extrait. Ces liens ont été omis de l’exemple précédent, mais sont mis en surbrillance dans le code suivant. Notez que, dans ces liens, la relation _automatique_ a été utilisée pour indiquer qu’il s’agit d’une référence à la ressource en cours retournée par l’opération :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"orderID":3,"productID":2,"quantity":4,"orderValue":16.60,"links":[{"rel":"self","href":" http://adventure-works.com/orders/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" self","href":" http://adventure-works.com /orders/3", "action":"PUT","types":["application/x-www-form-urlencoded"]},{"rel":"self","href":" http://adventure-works.com /orders/3", "action":"DELETE","types":[]},{"rel":"customer",
"href":" http://adventure-works.com /customers/3", "action":"GET","types":["text/xml","application/json"]},{"rel":" customer" (customer links omitted)}]}
```

Pour que cette approche soit efficace, les applications clientes doivent être préparées pour extraire et analyser ces informations supplémentaires.

## <a name="versioning-a-restful-web-api"></a>Gestion des versions une API RESTful web

Il est très improbable que dans l’ensemble, mais la plus simple des cas qu’une API web reste statique. Que les impératifs commerciaux évoluent de nouvelles collections de ressources peuvent être ajoutés, les relations entre les ressources peuvent être modifiées et la structure des données de ressources peut être modifiée. Lors de la mise à jour d’un site web pour gérer les demandes nouvelles ou différentes API est un processus relativement simple, vous devez prendre en compte les effets que ces changements auront sur les applications clientes utilisant l’API web. Le problème est que bien que le développeur de concevoir et d’implémenter une API web a un contrôle total sur cette API, le développeur n’a pas le même degré de contrôle sur les applications clientes qui peuvent être créés par des organisations tierces d’exploitation à distance. L’impératif principal est de permettre à des applications clientes existantes de continuer à fonctionner inchangé tout en autorisant à nouveau client applications pour tirer parti des nouvelles fonctionnalités et des ressources.

Gestion des versions permet un web API pour indiquer les fonctionnalités et les ressources qu’il expose, et une application cliente peut envoyer des demandes qui sont dirigées vers une version spécifique d’une fonctionnalité ou une ressource. Les sections suivantes décrivent les différentes approches, dont chacune a ses propres avantages et inconvénients.

### <a name="no-versioning"></a>Aucun contrôle de version

Ceci est l’approche la plus simple et peut-être être acceptable pour certains API internes. Principaux changements peuvent être représentés en tant que nouvelles ressources ou de nouveaux liens.  Ajout de contenu aux ressources existantes peut ne pas présente une modification avec rupture que les applications client qui n’attendent pas à voir que ce contenu simplement l’ignore.

Par exemple, une demande à l' URI _http://adventure-works.com/customers/3_ doit renvoyer les détails d’un client unique contenant des `id`, `name`, et `address` champs attendus par l’application cliente :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

> [AZURE.NOTE] Pour les besoins de simplicité et de clarté, les réponses de l’exemple illustrées dans cette section n’incluent pas les liens HATEOAS.

Si le `DateCreated` champ est ajouté dans le schéma de la ressource client, puis la réponse se présente comme suit :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":"1 Microsoft Way Redmond WA 98053"}
```

Les applications clientes existantes peuvent continuer de fonctionner correctement s’ils sont capables d’ignorer les champs non reconnus, tandis que de nouvelles applications clientes peuvent être conçues pour gérer ce nouveau champ. Toutefois, si des modifications plus radicales au schéma de ressources se produisent (par exemple, supprimer ou renommer des champs) ou les relations entre les ressources modifier ces peuvent constituer des modifications avec rupture qui empêchent les applications clientes existantes de fonctionner correctement. Dans ces situations, vous devez envisager une des approches suivantes.

### <a name="uri-versioning"></a>Gestion des versions de l’URI

Chaque fois que vous modifiez le web API ou modifiez le schéma des ressources, vous ajoutez un numéro de version à l’URI pour chaque ressource. Les URI existantes doivent continuer à fonctionner comme avant, retourner des ressources qui sont conformes à leur schéma d’origine.

Extension de l’exemple précédent, si le `address` champ est restructuré dans de sous-champs contenant chaque élément constitutif de l’adresse (tel que `streetAddress`, `city`, `state`, et `zipCode`), cette version de la ressource peut être exposée via un URI contenant un numéro de version, par exemple http://adventure-works.com/v2/customers/3 :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Ce mécanisme de gestion des versions est très simple, mais dépend du serveur de routage de la demande vers le point de terminaison approprié. Toutefois, il peut devenir difficile à manipuler que le site web QU'API arrivera à échéance à travers plusieurs itérations et le serveur doit prendre en charge d’un certain nombre de versions différentes. En outre, du point de vue d’une préférence, dans tous les cas les applications clientes extraire les mêmes données (client 3), afin que l’URI ne doit pas vraiment être différent en fonction de la version. Ce régime complique également la mise en oeuvre de HATEOAS comme tous les liens doit inclure le numéro de version dans leurs URI.

### <a name="query-string-versioning"></a>Version de chaîne de requête

Plutôt que de fournir plusieurs URI, vous pouvez spécifier la version de la ressource à l’aide d’un paramètre au sein de la chaîne de requête ajoutée à la requête HTTP, par exemple _http://adventure-works.com/customers/3?version=2_. Le paramètre de version par défaut une valeur significative par exemple 1 s’il est omis par les applications clientes plus anciennes.

Cette approche présente l’avantage de sémantique qui la même ressource est toujours extrait de l’URI même, mais il dépend du code qui gère la demande d’analyser la chaîne de requête et de renvoyer la réponse HTTP appropriée. Cette approche se caractérise également par les complications mêmes pour l’implémentation de HATEOAS comme mécanisme de contrôle de version URI.

> [AZURE.NOTE] Certains anciens navigateurs web et les proxys web ne seront pas mettre en cache les réponses pour les demandes qui incluent une chaîne de requête dans l’URL. Ceci peut avoir un impact négatif sur les performances pour les applications web qui utilisent une API web et qui s’exécutent à partir de ce type de navigateur web.

### <a name="header-versioning"></a>Version de l’en-tête

Plutôt que d’ajouter le numéro de version sous la forme d’un paramètre de chaîne de requête, vous pouvez implémenter un en-tête personnalisé qui indique la version de la ressource. Cette approche nécessite que l’application cliente ajoute l’en-tête approprié pour toutes les demandes, bien que le code qui traite la demande client peut utiliser une valeur par défaut (version 1) si l’en-tête de version est omis. Les exemples suivants utilisent un en-tête personnalisé nommé _Custom-Header_. La valeur de cet en-tête indique la version de l’API web.

Version 1 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=1
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Version 2 :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Custom-Header: api-version=2
...
```

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","dateCreated":"2014-09-04T12:11:38.0376089Z","address":{"streetAddress":"1 Microsoft Way","city":"Redmond","state":"WA","zipCode":98053}}
```

Notez que comme avec les deux approches précédentes, mise en œuvre de HATEOAS, y compris l’en-tête personnalisé approprié dans tous les liens.

### <a name="media-type-versioning"></a>Type de support de gestion des versions

Lorsqu’une application cliente envoie une requête HTTP GET à un serveur web, il devrait mentionner le format du contenu qu’elle peut gérer à l’aide d’une en-tête Accept, comme décrit plus haut dans ce guide. Fréquemment de l’en-tête _Accept_ vise à permettre à l’application cliente pour spécifier si le corps de la réponse doit être XML, JSON ou tout autre format commun le client peut analyser. Toutefois, il est possible de définir des types de médias personnalisé comprenant des informations sur l’activation de l’application cliente indiquer quelle version d’une ressource attendu. L’exemple suivant illustre une demande qui spécifie un en-tête _Accept_ avec valeur _application/vnd.adventure-works.v1+json_. L’élément _vnd.adventure-works.v1_ indique au serveur web qu’elle doit retourner la version 1 de la ressource, tandis que l’élément _json_ Spécifie que le format du corps de la réponse doit être JSON :

```HTTP
GET http://adventure-works.com/customers/3 HTTP/1.1
...
Accept: application/vnd.adventure-works.v1+json
...
```

Le code qui gère la demande est responsable de l’en-tête _Accept_ de traitement et il en respectant autant que possible (dans l’en-tête _Accept_ , dans lequel cas le serveur web peut choisir le format le plus approprié pour le corps de réponse, l’application cliente peut spécifier plusieurs formats). Le serveur web confirme le format des données dans le corps de la réponse à l’aide de l’en-tête Content-Type :

```HTTP
HTTP/1.1 200 OK
...
Content-Type: application/vnd.adventure-works.v1+json; charset=utf-8
...
Content-Length: ...
{"id":3,"name":"Contoso LLC","address":"1 Microsoft Way Redmond WA 98053"}
```

Si l’en-tête Accept ne spécifie pas de tous les types de médias connus, le serveur web peut générer un message de réponse HTTP 406 (non Acceptable) ou renvoyer un message avec un type de média par défaut.

Cette approche est sans aucun doute les sons des mécanismes de contrôle de version et se prête naturellement à HATEOAS, incluant le type MIME des données connexes dans les liens de ressources.

> [AZURE.NOTE] Lorsque vous sélectionnez une stratégie de gestion des versions, vous devez également envisager les implications sur les performances, en particulier la mise en cache sur le serveur web. Le versioning d’URI et les schémas de version de chaîne de requête sont facilement dans la mesure où la même combinaison de chaîne de requête/URI fait référence aux mêmes données chaque heure.

> Le versioning de l’en-tête et les mécanismes de contrôle de version de Type de média nécessitent généralement une logique supplémentaire pour examiner les valeurs de l’en-tête personnalisé ou l’en-tête Accept. Dans un environnement à grande échelle, de nombreux clients à l’aide de différentes versions d’une API web peuvent entraîner une quantité importante de données en double dans un cache côté serveur. Ce problème peut devenir aiguë si une application cliente communique avec un serveur web via un proxy qui implémente la mise en cache, et qui transmet une demande au serveur web si elle ne contient pas actuellement une copie des données demandées dans son cache.

## <a name="more-information"></a>Plus d’informations

- Le [Livre de recettes RESTful](http://restcookbook.com/) contient une introduction à la génération d’une API RESTful.
- [L’API de liste de contrôle](https://mathieu.fenniak.net/the-api-checklist/) Web contient une liste utile des éléments à prendre en compte lors de la conception et l’implémentation d’une API Web.
