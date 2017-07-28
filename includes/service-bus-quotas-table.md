Le tableau suivant répertorie les informations de quota spécifiques pour le Bus des services de messagerie. Limites de concentrateurs d’événement sont inclus dans cette table, mais pour plus d’informations sur les concentrateurs d’événement, consultez [Tarification des concentrateurs d’événement](https://azure.microsoft.com/pricing/details/event-hubs/). Pour informations sur la tarification et autres quotas de Bus des services, consultez la vue d’ensemble de [Tarification de Bus de Service](https://azure.microsoft.com/pricing/details/service-bus/) .

|Nom de quota|Champ d’application|Type de|Comportement en cas de dépassement|Valeur|
|---|---|---|---|---|
| Nombre maximal d’espaces de noms par abonnement Azure|Namespace|Statique|Les demandes suivantes des espaces de noms supplémentaires seront rejetées par le portail.|100|
|Taille de la file d’attente/rubrique.|Entité|Défini lors de la création de la file d’attente/rubrique.|Les messages entrants seront rejetées et une exception sera reçue par le code appelant.|1, 2, 3, 4 ou 5 Go.<br /><br />Si [le partitionnement](service-bus-partitioning.md) est activé, la taille de la file d’attente/rubrique maximale est de 80 Go.|
|Nombre de connexions simultanées sur un espace de noms|Namespace|Statique|Les demandes suivantes pour les connexions supplémentaires seront rejetées et une exception sera reçue par le code appelant. Opérations de repos n’entrent pas dans les connexions TCP simultanées.|NetMessaging : 1 000<br /><br />AMQP : 5 000|
|Nombre de connexions simultanées sur une entité de la file d’attente/rubrique/abonnement|Entité|Statique|Les demandes suivantes pour les connexions supplémentaires seront rejetées et une exception sera reçue par le code appelant. Opérations de repos n’entrent pas dans les connexions TCP simultanées.|Limitée par la limite de connexions simultanées par espace de noms.|
|Numéro de concurrent recevoir des demandes sur une entité de la file d’attente/rubrique/abonnement|Entité|Statique|Suivantes reçoivent les demandes seront rejetées et une exception sera reçue par le code appelant. Ce quota s’applique à la combinaison numéro de concurrent recevoir des opérations sur tous les abonnements sur un sujet.|5 000|
|Nombre de rubriques/files d’attente par l’espace de noms de service|Au niveau du système|Statique|Les demandes suivantes pour la création d’une nouvelle rubrique ou de la file d’attente dans l’espace de noms de service seront rejetées. Par conséquent, si configuré via le [portail Azure][], un message d’erreur sera généré. Si elle est appelée à partir de l’API de gestion, une exception sera reçue par le code appelant.|10 000<br /><br />Le nombre total de rubriques et de files d’attente dans un espace de noms du service doit être inférieur ou égal à 10 000.<br/>Ce n’est pas applicable à la prime que toutes les entités sont partitionnées.|
|Nombre de rubriques/files d’attente partitionnées par espace de noms de service|Au niveau du système|Statique|Les demandes suivantes pour la création d’une nouvelle rubrique partitionnée ou d’une file d’attente dans l’espace de noms de service seront rejetées. Par conséquent, si configuré via le [portail Azure][], un message d’erreur sera généré. Si elle est appelée à partir de l’API de gestion, une exception **QuotaExceededException** est reçue par le code appelant.|Couches de base et Standard - 100<br />Premium - 1 000<br/><br />Chaque file d’attente partitionnée ou la rubrique compte vers le quota de 10 000 entités par espace de noms.|
|Taille maximale d’un chemin d’entité messagerie : file d’attente ou rubrique|Entité|Statique|-|260 caractères|
|Taille maximale de n’importe quel nom d’entité messagerie : espace de noms, abonnement, règle d’abonnement ou concentrateur d’événements|Entité|Statique|-|50 caractères|
|Taille maximale de l’événement de l’événement concentrateurs|Au niveau du système|Statique|-|256 KO|
|Taille des messages pour une entité de la file d’attente/rubrique/abonnement|Au niveau du système|Statique|Les messages entrants qui dépassent les quotas seront rejetées et une exception sera reçue par le code appelant.|Taille de message maximale : 256 Ko ([couche Standard](../articles/service-bus/service-bus-premium-messaging.md)) / 1 Mo ([niveau de prime](../articles/service-bus/service-bus-premium-messaging.md)). <br /><br />**Remarque** En raison de la surcharge du système, cette limite est généralement légèrement inférieur.<br /><br />Taille de l’en-tête maximum : 64 Ko<br /><br />Nombre maximal de propriétés d’en-tête dans le sac de propriétés : **octets/int. MaxValue**<br /><br />Taille maximale de la propriété dans le sac de propriétés : aucune limite explicite. Limité par la taille de l’en-tête de maximum.|
|Taille de la propriété message pour une entité de la file d’attente/rubrique/abonnement|Au niveau du système|Statique|Une exception **exception SerializationException** est générée.|Taille maximale de message de la propriété pour chaque propriété est 32 Ko. Taille cumulée de toutes les propriétés ne peuvent pas dépasser 64 Ko. Cela s’applique à l’intégralité de l’en-tête de la [BrokeredMessage](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.aspx), qui a deux propriétés de l’utilisateur ainsi que les propriétés système (telles que le [numéro de séquence](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.sequencenumber.aspx), [Label](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.label.aspx), [MessageId](https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx)et ainsi de suite).|
|Nombre d’abonnements par rubrique|Au niveau du système|Statique|Les demandes suivantes pour la création des abonnements supplémentaires pour la rubrique seront rejetées. Par conséquent, si configuré via le portail, un message d’erreur s’affichera. Si elle est appelée à partir de l’API de gestion d’exception est reçue par le code appelant.|2 000|
|Nombre de filtres SQL par rubrique|Au niveau du système|Statique|Les demandes suivantes pour la création de filtres supplémentaires sur le sujet seront rejetées et une exception sera reçue par le code appelant.|2 000|
|Nombre de filtres de corrélation par rubrique|Au niveau du système|Statique|Les demandes suivantes pour la création de filtres supplémentaires sur le sujet seront rejetées et une exception sera reçue par le code appelant.|100 000|
|Taille de filtres et les actions SQL|Au niveau du système|Statique|Les demandes suivantes pour la création de filtres supplémentaires seront rejetées et une exception sera reçue par le code appelant.|Longueur maximale de chaîne de condition de filtre : 1024 (1 Ko).<br /><br />Longueur maximale de chaîne d’action de règle : 1024 (1 Ko).<br /><br />Nombre maximal d’expressions par l’action de la règle : 32.|
|Nombre de règles de [SharedAccessAuthorizationRule](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sharedaccessauthorizationrule.aspx) par l’espace de noms, file d’attente ou rubrique|Entité, espace de noms|Statique|Les demandes suivantes pour la création de règles supplémentaires seront rejetées et une exception sera reçue par le code appelant.|Nombre maximal de règles : 12. <br /><br /> Les règles qui sont configurées dans un espace de noms du Bus de Service s’appliquent à toutes les files d’attente et les rubriques de cet espace de noms.

[Azure portal]: https://portal.azure.com