
<properties
pageTitle="Guide de programmation SCP.NET | Azure"
description="Découvrez comment utiliser SCP.NET pour créer. Topologies de Storm basée sur le réseau pour utilisent avec tempête sur HDInsight."
services="hdinsight"
documentationCenter=""
authors="raviperi"
manager="jhubbard"
editor="cgronlun"/>

<tags
ms.service="hdinsight"
ms.devlang="dotnet"
ms.topic="article"
ms.tgt_pltfrm="na"
ms.workload="big-data"
ms.date="05/16/2016"
ms.author="raviperi"/>

#<a name="scp-programming-guide"></a>Guide de programmation SCP

SCP est une plate-forme à créer en temps réel, application de traitement de données fiable, cohérente et hautement performantes. Il est construit sur [Apache tempête](http://storm.incubator.apache.org/) --un système conçu par les Communautés OSS de traitement du flux de données. Tempête est conçu par Nathan Marz et ouvrir alimenté par Twitter. Il s’appuie sur [Apache soigneur](http://zookeeper.apache.org/), un autre projet d’Apache pour permettre la coordination de distribuées très fiable et de la gestion d’état. 

Non seulement le projet SCP porté tempête sous Windows, mais également d’ajouter des extensions et personnalisation pour l’écosystème de Windows dans le projet. Les extensions incluent l’expérience des développeurs .NET et des bibliothèques, la personnalisation inclut le déploiement basé sur Windows. 

L’extension et la personnalisation s’effectue de telle sorte que nous n’avez pas besoin de branche dans les projets de systèmes d’exploitation et que nous pouvons tirer parti des écosystèmes dérivées basées sur tempête.

## <a name="processing-model"></a>Modèle de traitement 

Les données contenues dans le SCP sont modélisées en tant que flux continus de tuples. Généralement les tuples flux dans une file d’attente tout d’abord, chercher puis transformé par la logique d’application hébergées à l’intérieur d’une topologie de tempête, enfin la sortie peut être transmise sous forme de tuples à un autre système SCP, ou être validée comme système de fichiers distribués, les banques ou de bases de données comme SQL Server.

![Un diagramme d’une file d’attente de traitement, qui alimente un magasin de données alimentant en données](media/hdinsight-storm-scp-programming-guide/queue-feeding-data-to-processing-to-data-store.png)

Dans la tempête, une topologie de l’application définit un graphique de calcul. Chaque nœud dans une topologie contient la logique de traitement et des liens entre les nœuds indiquent le flux de données. Les nœuds d’injecter des données d’entrée dans la topologie sont appelés des becs verseurs, qui peuvent être utilisés pour les données de la séquence. Les données d’entrée peuvent résider dans des fichiers journaux, de la base de données transactionnelle, le compteur de performance système etc.. Les nœuds avec les deux flux de données d’entrée et de sortie sont appelés des boulons, lequel effectuer le filtrage de données réelles et sélections et agrégation.

SCP prend en charge les efforts déployés, au moins une et exactement-traitement des données en une seule fois. Dans une application distribuée de traitement en continu, les diverses erreurs peuvent se produire pendant le traitement de données, par exemple de panne réseau, la défaillance de l’ordinateur ou erreur de code utilisateur etc.. Au moins une transformation garantit que toutes les données seront traitées au moins une fois par relecture automatiquement les données mêmes lorsque l’erreur se produit. Traitement d’au moins une est simple et fiable et mieux à bien dans de nombreuses applications. Toutefois, lorsque l’application nécessite un inventaire exact, par exemple, au moins une est insuffisante dans la mesure où les mêmes données peuvent potentiellement être lu dans la topologie de l’application. Dans ce cas, exactement-une fois que le traitement est conçu pour vous assurer que le résultat est correcte même lorsque les données peuvent être relues et traitées plusieurs fois.

SCP permet aux développeurs de .NET développer des applications de processus de données en temps réel tandis que le levier de la Machine virtuelle Java (JVM) basé tempête sous le capot. Le .NET et JVM communiquent via le socket TCP local. En gros chaque bec/du boulon est une paire de processus .net/Java, où la logique utilisateur s’exécute dans le processus .net sous la forme d’un plug-in.

Pour générer une application de traitement de données sur le SCP, plusieurs étapes sont nécessaires :

-   Concevoir et implémenter les becs verseurs pour extraire les données de file d’attente.

-   Concevoir et mettre en œuvre des boulons pour traiter les données d’entrée et enregistrer les données dans les magasins externes comme base de données.

-   Conception de la topologie, puis les soumettre et exécuter la topologie. La topologie définit les sommets et les données de flux entre les sommets. SCP prendre la spécification de la topologie et le déployer sur un cluster de tempête, où chaque sommet s’exécute sur un nœud de logique. Le basculement et la mise à l’échelle va être prises en charge par le Planificateur de tâches de tempête.

Ce document utilise des exemples simples de la génération d’applications de traitement de données avec le SCP.

## <a name="scp-plugin-interface"></a>Interface de plug-in SCP 

Plug-ins de SCP (ou applications) sont exécutables autonomes qui peuvent tous deux exécuter à l’intérieur de Visual Studio au cours de la phase de développement et le brancher sur le pipeline de tempête après le déploiement en production. Le plug-in SCP est d’écrire tout de même que les autres applications Windows standard console. Plate-forme SCP.NET déclare une interface pour bec/du boulon et le code du plug-in utilisateur doit implémenter ces interfaces. L’objectif principal de cette conception est que l’utilisateur puisse se concentrer sur leur propre logique métier et laisser les autres éléments à être traités par la plate-forme SCP.NET.

Code utilisateur du plug-in doit implémenter une des interfaces suivantes, dépend de si la topologie est transactionnelle ou non transactionnelle, et si le composant est un bec ou boulon.

-   ISCPSpout

-   ISCPBolt

-   ISCPTxSpout

-   ISCPBatchBolt

### <a name="iscpplugin"></a>ISCPPlugin

ISCPPlugin est l’interface commune pour tous les types de plug-ins. Actuellement, il s’agit d’une interface factice.

    public interface ISCPPlugin 
    {
    }

### <a name="iscpspout"></a>ISCPSpout

ISCPSpout est l’interface de bec non transactionnelle.

     public interface ISCPSpout : ISCPPlugin                    
     {
         void NextTuple(Dictionary<string, Object> parms);         
         void Ack(long seqId, Dictionary<string, Object> parms);   
         void Fail(long seqId, Dictionary<string, Object> parms);  
     }

Lors de la `NextTuple()` est appelée, la C\# code utilisateur peut émettre un ou plusieurs tuples. S’il n’a rien à émettre, cette méthode doit retourner toute émission de quoi que ce soit. Il convient de noter que `NextTuple()`, `Ack()`, et `Fail()` appelée dans une boucle serrée dans un seul thread à la C\# processus. Lorsqu’il n’y a aucun tuples à émettre, elle est courtois pour que la mise en veille de le NextTuple pour une courte période de temps (par exemple, 10 millisecondes) pour autant perdre trop de ressources CPU.

`Ack()`et `Fail()` est appelé uniquement lorsque le mécanisme d’accusé de réception est activée dans le fichier spec. Le `seqId` est utilisé pour identifier le tuple qui est reçu ou a échoué. Si l’accusé de réception est activée dans la topologie non transactionnelle, la fonction d’émission suivante doit donc être utilisée dans un bec :

    public abstract void Emit(string streamId, List<object> values, long seqId); 

Si l’accusé de réception n’est pas pris en charge dans une topologie non transactionnelle, la `Ack()` et `Fail()` peut être laissée en tant que fonction vide.

Les `parms` les paramètres d’entrée de ces fonctions sont simplement vider le dictionnaire, ils sont réservés pour une utilisation ultérieure.

### <a name="iscpbolt"></a>ISCPBolt

ISCPBolt est l’interface pour boulon non transactionnelle.

    public interface ISCPBolt : ISCPPlugin 
    {
    void Execute(SCPTuple tuple);           
    }

Lorsque le nouveau tuple est disponible, le `Execute()` fonction sera appelée pour la traiter.

### <a name="iscptxspout"></a>ISCPTxSpout

ISCPTxSpout est l’interface de bec transactionnel.

    public interface ISCPTxSpout : ISCPPlugin
    {
        void NextTx(out long seqId, Dictionary<string, Object> parms);  
        void Ack(long seqId, Dictionary<string, Object> parms);         
        void Fail(long seqId, Dictionary<string, Object> parms);        
    }

Comme leur part contradictoire non transactionnelles, `NextTx()`, `Ack()`, et `Fail()` appelée dans une boucle serrée dans un seul thread à la C\# processus. Lorsqu’il n’y a pas de données à émettre, elle est courtois avoir `NextTx` la mise en veille pour une courte durée (10 millisecondes) pour autant perdre trop de ressources processeur.

`NextTx()`est appelé pour démarrer une nouvelle transaction, le paramètre de sortie `seqId` est utilisé pour identifier la transaction, qui est également utilisée dans `Ack()` et `Fail()`. Dans `NextTx()`, utilisateur peut émettre des données à côté de Java. Les données doivent être stockées dans soigneur pour prendre en charge de la relecture. Étant donné que la capacité de soigneur est très limitée, utilisateur doit uniquement émettre des métadonnées, pas en bloc les données dans un bec transactionnelle.

Tempête est lues automatiquement une transaction en cas d’échec, alors `Fail()` ne doit pas être appelée dans le cas normal. Mais si le SCP peut vérifier les métadonnées émise par un bec transactionnel, il peut appeler `Fail()` lorsque les métadonnées ne sont pas valide.

Les `parms` les paramètres d’entrée de ces fonctions sont simplement vider le dictionnaire, ils sont réservés pour une utilisation ultérieure.

### <a name="iscpbatchbolt"></a>ISCPBatchBolt

ISCPBatchBolt est l’interface pour boulon transactionnelle.

    public interface ISCPBatchBolt : ISCPPlugin           
    {
        void Execute(SCPTuple tuple);
        void FinishBatch(Dictionary<string, Object> parms);  
    }

`Execute()`est appelé lorsqu’il y a de nouveau tuple arrivant dans le boulon. `FinishBatch()`est appelé lorsque cette transaction est terminée. Le `parms` paramètre d’entrée est réservé pour une utilisation ultérieure.

Dans une topologie transactionnelle, il est un concept important – `StormTxAttempt`. Il possède deux champs, `TxId` et `AttemptId`. `TxId`est utilisé pour identifier une transaction spécifique, et pour une transaction donnée, il peut y avoir plusieurs tentative si la transaction échoue et est relu. SCP.NET sera à nouveau un objet différent de ISCPBatchBolt pour traiter chaque `StormTxAttempt`, exactement comme si le tempête côté de Java. L’objectif de cette conception est de prendre en charge les transactions parallèles de traitement. Utilisateur il Gardez à l’esprit que si la tentative de la transaction est terminée, l’objet ISCPBatchBolt correspondant est détruit et la mémoire nettoyée.

## <a name="object-model"></a>Modèle d’objet

SCP.NET fournit également un ensemble de pour les développeurs à programmer avec les objets de clé simple. Ils sont le **contexte**, **StateStore**et **SCPRuntime**. Ils seront abordées dans la partie du reste de cette section.

### <a name="context"></a>Contexte

Contexte fournit un environnement en cours d’exécution pour l’application. Chaque instance de ISCPPlugin (ISCPSpout/ISCPBolt/ISCPTxSpout/ISCPBatchBolt) est une instance de contexte correspondante. La fonctionnalité fournie par le contexte peut être divisée en deux parties : (1) la partie statique qui est disponible dans l’ensemble C\# traiter, (2) la partie dynamique qui est uniquement disponible pour l’instance de contexte spécifique.

### <a name="static-part"></a>Partie statique

    public static ILogger Logger = null;
    public static SCPPluginType pluginType;                      
    public static Config Config { get; set; }                    
    public static TopologyContext TopologyContext { get; set; }  

`Logger`est fourni à des fins de journal.

`pluginType`est utilisé pour indiquer le type de plug-in de la C\# processus. Si le C\# processus est exécuté en mode de test local (sans Java), le type de plug-in est `SCP_NET_LOCAL`.

    public enum SCPPluginType 
    {
        SCP_NET_LOCAL = 0,       
        SCP_NET_SPOUT = 1,       
        SCP_NET_BOLT = 2,        
        SCP_NET_TX_SPOUT = 3,   
        SCP_NET_BATCH_BOLT = 4  
    }

`Config`est fourni pour obtenir les paramètres de configuration du côté de Java. Les paramètres sont passés de côté de Java lorsque C\# plug-in est initialisé. Le `Config` paramètres sont divisés en deux parties : `stormConf` et `pluginConf`.

    public Dictionary<string, Object> stormConf { get; set; }  
    public Dictionary<string, Object> pluginConf { get; set; }  

`stormConf`est définis par une tempête de paramètres et de `pluginConf` est les paramètres définis par le SCP. Par exemple :

    public class Constants
    {
        … …

        // constant string for pluginConf
        public static readonly String NONTRANSACTIONAL_ENABLE_ACK = "nontransactional.ack.enabled";  

        // constant string for stormConf
        public static readonly String STORM_ZOOKEEPER_SERVERS = "storm.zookeeper.servers";           
        public static readonly String STORM_ZOOKEEPER_PORT = "storm.zookeeper.port";                 
    }

`TopologyContext`est fourni pour obtenir le contexte de la topologie, il est particulièrement utile pour les composants avec plusieurs parallélisme. Voici un exemple :

    //demo how to get TopologyContext info
    if (Context.pluginType != SCPPluginType.SCP_NET_LOCAL)                      
    {
        Context.Logger.Info("TopologyContext info:");
        TopologyContext topologyContext = Context.TopologyContext;                    
        Context.Logger.Info("taskId: {0}", topologyContext.GetThisTaskId());          
        taskIndex = topologyContext.GetThisTaskIndex();
        Context.Logger.Info("taskIndex: {0}", taskIndex);
        string componentId = topologyContext.GetThisComponentId();                    
        Context.Logger.Info("componentId: {0}", componentId);
        List<int> componentTasks = topologyContext.GetComponentTasks(componentId);  
        Context.Logger.Info("taskNum: {0}", componentTasks.Count);                    
    }

### <a name="dynamic-part"></a>Partie dynamique

Les interfaces suivantes sont pertinentes pour une certaine instance de contexte. L’instance de contexte est créé par plate-forme SCP.NET et passé au code utilisateur :

    // Declare the Output and Input Stream Schemas

    public void DeclareComponentSchema(ComponentStreamSchema schema);   

    // Emit tuple to default stream.
    public abstract void Emit(List<object> values);                   

    // Emit tuple to the specific stream.
    public abstract void Emit(string streamId, List<object> values);  

Pour la prise en charge d’accusé de réception de bec non transactionnelle, la méthode suivante est fournie :

    // for non-transactional Spout which supports ack
    public abstract void Emit(string streamId, List<object> values, long seqId);  

Pour la prise en charge d’accusé de réception de boulon non transactionnelle, il doit explicitement `Ack()` ou `Fail()` le tuple reçue. Et lors de l’émission de nouveau tuple, elle doit également spécifier les ancres du nouveau tuple. Les méthodes suivantes sont fournies.

    public abstract void Emit(string streamId, IEnumerable<SCPTuple> anchors, List<object> values); 
    public abstract void Ack(SCPTuple tuple);
    public abstract void Fail(SCPTuple tuple);

### <a name="statestore"></a>StateStore

`StateStore`Fournit des services de métadonnées, la génération de séquence monotone et coordination d’attente libres. Abstractions de niveau supérieur d’accès concurrentiel distribué peuvent être générées sur `StateStore`, y compris les verrous distribués, des files d’attente distribuées, barrières et les services de transaction.

SCP les applications peuvent utiliser le `State` objet à rendre persistantes des informations dans soigneur, en particulier pour la topologie transactionnelle. Ce faisant, si transactionnel bec tombe en panne et redémarrer, il peut récupérer les informations nécessaires à partir de soigneur et redémarrer le pipeline.

Le `StateStore` objet a essentiellement de ces méthodes :

    /// <summary>
    /// Static method to retrieve a state store of the given path and connStr 
    /// </summary>
    /// <param name="storePath">StateStore Path</param>
    /// <param name="connStr">StateStore Address</param>
    /// <returns>Instance of StateStore</returns>
    public static StateStore Get(string storePath, string connStr);

    /// <summary>
    /// Create a new state object in this state store instance
    /// </summary>
    /// <returns>State from StateStore</returns>
    public State Create();

    /// <summary>
    /// Retrieve all states that were previously uncommitted, excluding all aborted states 
    /// </summary>
    /// <returns>Uncommited States</returns>
    public IEnumerable<State> GetUnCommitted();

    /// <summary>
    /// Get all the States in the StateStore
    /// </summary>
    /// <returns>All the States</returns>
    public IEnumerable<State> States();

    /// <summary>
    /// Get state or registry object
    /// </summary>
    /// <param name="info">Registry Name(Registry only)</param>
    /// <typeparam name="T">Type, Registry or State</typeparam>
    /// <returns>Return Registry or State</returns>
    public T Get<T>(string info = null);

    /// <summary>
    /// List all the committed states
    /// </summary>
    /// <returns>Registries contain the Committed State </returns> 
    public IEnumerable<Registry> Commited();

    /// <summary>
    /// List all the Aborted State in the StateStore
    /// </summary>
    /// <returns>Registries contain the Aborted State</returns>
    public IEnumerable<Registry> Aborted();

    /// <summary>
    /// Retrieve an existing state object from this state store instance 
    /// </summary>
    /// <returns>State from StateStore</returns>
    /// <typeparam name="T">stateId, id of the State</typeparam>
    public State GetState(long stateId)

Le `State` objet a essentiellement de ces méthodes :

    /// <summary>
    /// Set the status of the state object to commit 
    /// </summary>
    public void Commit(bool simpleMode = true); 

    /// <summary>
    /// Set the status of the state object to abort 
    /// </summary>
    public void Abort();

    /// <summary>
    /// Put an attribute value under the give key 
    /// </summary>
    /// <param name="key">Key</param> 
    /// <param name="attribute">State Attribute</param> 
    public void PutAttribute<T>(string key, T attribute); 

    /// <summary>
    /// Get the attribute value associated with the given key 
    /// </summary>
    /// <param name="key">Key</param> 
    /// <returns>State Attribute</returns>               
    public T GetAttribute<T>(string key);                    

Pour le `Commit()` la méthode simpleMode est définie sur true, il sera simplement supprimer la ZNode correspondante dans soigneur. Dans le cas contraire, il supprime le ZNode en cours et l’ajout d’un nouveau nœud dans le validé\_chemin d’accès.

### <a name="scpruntime"></a>SCPRuntime

SCPRuntime fournit le des deux méthodes suivantes.

    public static void Initialize();
    
    public static void LaunchPlugin(newSCPPlugin createDelegate);  

`Initialize()`est utilisé pour initialiser l’environnement d’exécution SCP. Dans cette méthode, la C\# processus se connectera le côté de Java et obtient les paramètres de configuration et de contexte de la topologie.

`LaunchPlugin()`permet de lancer la boucle de traitement de message. Dans cette boucle, le C\# plug-in reçoit les messages formulaire côté Java (y compris les signaux de contrôle et de tuples) et ensuite traiter les messages, par exemple l’appel de la méthode d’interface fournissent par le code utilisateur. Le paramètre d’entrée pour la méthode `LaunchPlugin()` est un délégué qui peut retourner un objet qui implémente l’interface de ISCPSpout/IScpBolt/ISCPTxSpout/ISCPBatchBolt.

    public delegate ISCPPlugin newSCPPlugin(Context ctx, Dictionary\<string, Object\> parms); 

Pour ISCPBatchBolt, nous pouvons obtenir des `StormTxAttempt` de `parms`et l’utiliser pour déterminer s’il s’agit d’une tentative de relus. Cela est généralement effectué à boulon commit, et il est démontré dans les `HelloWorldTx` exemple.

En règle générale, les plug-ins SCP peuvent s’exécuter dans deux modes ici :

1. Mode de Test local : Dans ce mode, les plug-ins SCP (C\# code utilisateur) s’exécutent à l’intérieur de Visual Studio au cours de la phase de développement. `LocalContext`peut être utilisé dans ce mode, qui fournit la méthode pour sérialiser les tuples émis à des fichiers locaux et de les lire dans la mémoire.

        public interface ILocalContext
        {
            List\<SCPTuple\> RecvFromMsgQueue();
            void WriteMsgQueueToFile(string filepath, bool append = false);  
            void ReadFromFileToMsgQueue(string filepath);                    
        }

2. Mode standard : Dans ce mode, les plug-ins SCP sont lancées par les processus java de tempête.

    Voici un exemple de plug-in de SCP de lancement :

        namespace Scp.App.HelloWorld
        {
        public class Generator : ISCPSpout
        {
            … …
            public static Generator Get(Context ctx, Dictionary<string, Object> parms)
            {
            return new Generator(ctx);
            }
        }
    
        class HelloWorld
        {
            static void Main(string[] args)
            {
            /* Setting the environment variable here can change the log file name */
            System.Environment.SetEnvironmentVariable("microsoft.scp.logPrefix", "HelloWorld");
    
            SCPRuntime.Initialize();
            SCPRuntime.LaunchPlugin(new newSCPPlugin(Generator.Get));
            }
        }
        }


## <a name="topology-specification-language"></a>Langage de spécification de topologie 

Spécification de topologie SCP est un langage de description et la configuration des topologies de SCP spécifique du domaine. Il est basé sur la Clojure DSL de tempête (<http://storm.incubator.apache.org/documentation/Clojure-DSL.html>) et est étendu par le SCP.

Spécifications de la topologie peuvent être soumises directement à un cluster de storm pour exécution via la commande ***runspec*** .

SCP.NET a ajouter les fonctions suivantes pour définir la topologie transactionnelle :

|**Nouvelles fonctions**|**Paramètres**|**Description**|
|---|---|---|
|**Tx-topolopy**|nom de la topologie<br />plan de BEC VERSEUR<br />plan de boulon|Définition d’une topologie transactionnelle avec le nom de la topologie, &nbsp;mappage de définition et de la table de définition des boulons des becs verseurs amovibles|
|**SCP-tx-BEC VERSEUR**|EXEC-nom<br />args<br />champs|Définir un bec transactionnel. Il exécutera l’application avec le ***nom-exec*** à l’aide des ***arguments***.<br /><br />Les ***champs*** sont les champs de sortie de BEC VERSEUR|
|**SCP-tx-lot-boulon**|EXEC-nom<br />args<br />champs|Définir un boulon de lot transactionnelle. Il exécute l’application à l’aide de ***exec-nom*** ***args.***<br /><br />Les champs sont les champs de sortie pour le boulon.|
|**SCP-tx-validation-boulon**|EXEC-nom<br />args<br />champs|Définir un boulon Committer transactionnelle. Il exécutera l’application avec le ***nom-exec*** à l’aide des ***arguments***.<br /><br />Les ***champs*** sont les champs de sortie de boulon|
|**nontx-topolopy**|nom de la topologie<br />plan de BEC VERSEUR<br />plan de boulon|Définition d’une topologie non transactionnelles avec le nom de la topologie,&nbsp; mappage de définition et de la table de définition des boulons des becs verseurs amovibles|
|**SCP-bec**|EXEC-nom<br />args<br />champs<br />paramètres|Définir un bec non transactionnelles. Il exécutera l’application avec le ***nom-exec*** à l’aide des ***arguments***.<br /><br />Les ***champs*** sont les champs de sortie de BEC VERSEUR<br /><br />Les ***paramètres*** sont facultatif, l’utiliser pour spécifier des paramètres, tels que « nontransactional.ack.enabled ».|
|**SCP-boulon**|EXEC-nom<br />args<br />champs<br />paramètres|Définir un boulon non transactionnelles. Il exécutera l’application avec le ***nom-exec*** à l’aide des ***arguments***.<br /><br />Les ***champs*** sont les champs de sortie de boulon<br /><br />Les ***paramètres*** sont facultatif, l’utiliser pour spécifier des paramètres, tels que « nontransactional.ack.enabled ».|

SCP.NET a suivre touches de mots définis :

|**Mots clés**|**Description**|
|---|---|
|**: nom**|Définir le nom de la topologie|
|**: topologie**|Pour définir la topologie à l’aide des fonctions ci-dessus et dans ceux de build.|
|**: p**|Définir l’indicateur de parallélisme pour chaque bec ou le boulon.|
|**: configuration**|Définir les paramètre de configurer ou de mettre à jour d’existants|
|**: schéma**|Définir le schéma du flux de données.|

Et les paramètres fréquemment utilisés :

|**Paramètre**|**Description**|
|---|---|
|**« plugin.name »**|nom du fichier exe du plug-in C#|
|**« plugin.args »**|arguments de plug-in|
|**« output.schema »**|Schéma de sortie|
|**« nontransactional.ack.enabled »**|Si l’accusé de réception est activé pour les topologies non transactionnelles|

La commande runspec doit être déployée avec les bits, l’utilisation est similaire à :

    .\bin\runSpec.cmd
    usage: runSpec [spec-file target-dir [resource-dir] [-cp classpath]]
    ex: runSpec examples\HelloWorld\HelloWorld.spec specs examples\HelloWorld\Target

Le paramètre de ***ressource-dir*** est facultatif, vous devez le spécifier lorsque vous souhaitez connecter un C\# application et ce répertoire contiendra l’application, les dépendances et les configurations.

Le paramètre ***classpath*** est également facultatif. Il est utilisé pour spécifier le chemin de classe Java si le fichier spec contient Java BEC VERSEUR ou le boulon.

## <a name="miscellaneous-features"></a>Fonctionnalités diverses

### <a name="input-and-output-schema-declaration"></a>Entrée et sortie déclaration de schéma

L’utilisateur peut émettre tuple dans C\# processus, la plate-forme doit sérialiser le tuple en byte [], transférer au côté de Java et tempête transfère ce tuple dans les cibles. En même temps dans le composant en aval, la C\# processus de recevoir le tuple de côté de java et convertir les types d’origine par plate-forme, toutes ces opérations sont masquées par la plate-forme.

Pour prendre en charge de la sérialisation et la désérialisation, le code utilisateur doit déclarer le schéma des entrées et des sorties.

Le schéma d’entrée/sortie de flux de données est défini comme un dictionnaire, la clé est le StreamId et la valeur est le type des colonnes. Le composant peut avoir plusieurs flux de données déclarées.

    public class ComponentStreamSchema
    {
        public Dictionary<string, List<Type>> InputStreamSchema { get; set; }
        public Dictionary<string, List<Type>> OutputStreamSchema { get; set; }
        public ComponentStreamSchema(Dictionary<string, List<Type>> input, Dictionary<string, List<Type>> output)
        {
            InputStreamSchema = input;
            OutputStreamSchema = output;
        }
    }


Dans l’objet de contexte, nous avons l’API suivante est ajoutée :

    public void DeclareComponentSchema(ComponentStreamSchema schema)

Code utilisateur devez vous assurer que les tuples émis respecter le schéma défini pour ce flux de données, ou le système lève une exception runtime.

### <a name="multi-stream-support"></a>Prise en charge des flux multiples

SCP prend en charge le code utilisateur pour envoyer ou recevoir de plusieurs flux de données distinctes en même temps. La prise en charge se reflète dans l’objet de contexte, la méthode Emit prend un paramètre d’ID facultatif de flux.

Deux méthodes de l’objet de contexte de SCP.NET ont été ajoutés. Ils sont utilisés pour émettre de Tuple ou n-uplets pour spécifier StreamId. Le StreamId est une chaîne et doit être cohérente dans les deux C\# et spécification de définition de topologie.

        /* Emit tuple to the specific stream. */
        public abstract void Emit(string streamId, List<object> values);

        /* for non-transactional Spout only */
        public abstract void Emit(string streamId, List<object> values, long seqId);

L’émission d’un flux non existant entraîne des exceptions runtime.

### <a name="fields-grouping"></a>Regroupement de champs

Le regroupement de champs dans génération dans Strom ne fonctionne pas correctement dans SCP.NET. Sur le côté du Proxy de Java, tous les types de données des champs sont réellement byte [], et les champs de regroupement utilise le code de hachage de byte [] objet pour effectuer le regroupement. Le code de hachage objet byte [] est l’adresse de cet objet dans la mémoire. Ainsi, le regroupement sera incorrect pour les objets [] deux octets qui partagent le même contenu, mais pas la même adresse.

SCP.NET ajoute une méthode personnalisée de regroupement, et il utilise le contenu de l’octet pour effectuer le regroupement. Dans le fichier de **spécifications** , la syntaxe ressemble à :

    (bolt-spec
        {
            "spout_test" (scp-field-group :non-tx [0,1])
        }
        …
    )


Ici,

1.  « champ-scp-group » signifie « Groupement de champ personnalisée implémentée par SCP ».

2.  « : tx « ou » : non-tx » signifie que si elle est transactionnelle topologie. Nous avons besoin de ces informations dans la mesure où l’index de départ est différent dans tx et les topologies non-tx.

3.  [0,1] signifie un hashset de codes de champ, à partir de 0.

### <a name="hybrid-topology"></a>Topologie de hybride

La tempête native est écrit en Java. Et SCP.Net a amélioré pour activer notre douanier écrire C\# code permettant de gérer la logique métier. Mais nous prennent également en charge les topologies hybrides, qui contient non seulement les C\# becs verseurs/boulons, mais également Java bec/boulons.

### <a name="specify-java-spoutbolt-in-spec-file"></a>Spécifier de VERSEUR Java/du boulon dans le fichier spec

Dans le fichier spec, « scp-bec » et « boulon scp » permet également spécifier becs verseurs amovibles de Java et de boulons, Voici un exemple :

    (spout-spec 
      (microsoft.scp.example.HybridTopology.Generator.)           
      :p 1)

Ici `microsoft.scp.example.HybridTopology.Generator` est le nom de la classe Java bec.

### <a name="specify-java-classpath-in-runspec-command"></a>Spécifiez le chemin de classe Java dans runSpec, commande

Si vous souhaitez soumettre la topologie contenant des boulons ou des becs verseurs amovibles de Java, vous devez d’abord compiler le Java becs verseurs amovibles ou les boulons et obtenir les fichiers Jar. Ensuite, vous devez spécifier le chemin de classe java qui contient les fichiers Jar lors de l’envoi de topologie. Voici un exemple :

    bin\runSpec.cmd examples\HybridTopology\HybridTopology.spec specs examples\HybridTopology\net\Target -cp examples\HybridTopology\java\target\*

Ici **exemples de\\HybridTopology\\java\\cible\\ ** est le dossier contenant le fichier Jar de Java bec/du boulon.

### <a name="serialization-and-deserialization-between-java-and-c"></a>Sérialisation et désérialisation entre Java et C\#

Notre composant SCP inclut côté de Java et C\# côté. Pour interagir avec des becs verseurs/boulons de Java natives, la sérialisation/désérialisation doit être effectuée entre la côte de Java et C\# côté, comme illustré dans le graphique suivant.

![diagramme de composant java envoi au composant SCP envoyer au composant de Java](media/hdinsight-storm-scp-programming-guide/java-compent-sending-to-scp-component-sending-to-java-component.png)

1.  **Sérialisation de côté de Java et de la désérialisation dans C\# côté**
    
    Nous avons d’abord fournir l’implémentation par défaut pour la sérialisation de côté de Java et de la désérialisation dans C\# côté. La méthode de sérialisation de côté de Java peut être spécifiée dans le fichier de spécifications :
    
        (scp-bolt
            {
                "plugin.name" "HybridTopology.exe"
                "plugin.args" ["displayer"]
                "output.schema" {}
                "customized.java.serializer" ["microsoft.scp.storm.multilang.CustomizedInteropJSONSerializer"]
            })
    
    La méthode de désérialisation dans C\# côté doit être spécifié dans C\# code utilisateur :
    
        Dictionary<string, List<Type>> inputSchema = new Dictionary<string, List<Type>>();
        inputSchema.Add("default", new List<Type>() { typeof(Person) });
        this.ctx.DeclareComponentSchema(new ComponentStreamSchema(inputSchema, null));
        this.ctx.DeclareCustomizedDeserializer(new CustomizedInteropJSONDeserializer());            
    
    Cette implémentation par défaut doit gérer la plupart des cas si le type de données n’est pas trop complexe. Dans certains cas, soit parce que le type de données utilisateur est trop complexe, la performance de notre implémentation par défaut ne répond pas aux besoins de l’utilisateur, plug-in peut leur propre implémentation.

    L’interface de serialize de côté de java est définie comme suit :
    
        public interface ICustomizedInteropJavaSerializer {
            public void prepare(String[] args);
            public List<ByteBuffer> serialize(List<Object> objectList);
        }
    
    L’interface de désérialisation dans C\# côté est définie comme :
    
    interface publique ICustomizedInteropCSharpDeserializer
    
        public interface ICustomizedInteropCSharpDeserializer
        {
            List<Object> Deserialize(List<byte[]> dataList, List<Type> targetTypes);
        }

2.  **La sérialisation dans C\# côté et la désérialisation de la côte de Java**

    La méthode de sérialisation dans C\# côté doit être spécifié dans C\# code utilisateur :
    
        this.ctx.DeclareCustomizedSerializer(new CustomizedInteropJSONSerializer()); 
    
    La méthode de désérialisation du côté Java doit être spécifiée dans le fichier de spécifications :
    
      (scp-bec {« plugin.name » « HybridTopology.exe » « plugin.args » [ » Générateur «] « output.schema » {« par défaut » [« personne »]} « customized.java.deserializer » [« microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer » « microsoft.scp.example.HybridTopology.Person »]})
    
    « Microsoft.scp.storm.multilang.CustomizedInteropJSONDeserializer » est le nom du désérialiseur, et « microsoft.scp.example.HybridTopology.Person » est que la classe de cible les données est désérialisée à.
    
    L’utilisateur peut également plug-in leur propre implémentation de C\# sérialiseur et désérialiseur de Java. Il s’agit de l’interface pour C\# sérialiseur :
    
        public interface ICustomizedInteropCSharpSerializer
        {
            List<byte[]> Serialize(List<object> dataList);
        }
    
    Il s’agit de l’interface pour le désérialiseur de Java :
    
        public interface ICustomizedInteropJavaDeserializer {
            public void prepare(String[] targetClassNames);
            public List<Object> Deserialize(List<ByteBuffer> dataList);
        }

## <a name="scp-host-mode"></a>Mode SCP hôte

Dans ce mode, utilisateur peut compiler leurs codes à la DLL et SCPHost.exe fournis par SCP permet de soumettre la topologie. Le fichier spec ressemble à ceci :

    (scp-spout
      {
        "plugin.name" "SCPHost.exe"
        "plugin.args" ["HelloWorld.dll" "Scp.App.HelloWorld.Generator" "Get"]
        "output.schema" {"default" ["sentence"]}
      })

Ici, `plugin.name` est spécifié en tant que `SCPHost.exe` fournis par le Kit de développement logiciel SCP. SCPHost.exe qui accepte trois paramètres :

1.  Le premier est le nom de la DLL, qui est `"HelloWorld.dll"` dans cet exemple.

2.  Le second est le nom de classe, qui est `"Scp.App.HelloWorld.Generator"` dans cet exemple.

3.  La troisième est le nom d’une méthode statique publique, qui peut être appelée pour obtenir une instance de ISCPPlugin.

En mode hôte, le code utilisateur est compilé en tant que DLL et est appelé par plate-forme de SCP. Plate-forme de SCP peut donc prendre le contrôle de la logique de traitement complet. Par conséquent, nous vous recommandons à topologie en mode hôte de SCP de soumettre, car elle peut simplifier l’expérience de développement et nous apporter plus de souplesse et d’une meilleure compatibilité descendante pour une version ultérieure ainsi que nos clients.

## <a name="scp-programming-examples"></a>Exemples de programmation SCP 

### <a name="helloworld"></a>HelloWorld

**HelloWorld** est un exemple très simple pour afficher un aperçu de la SCP.Net. Il utilise une topologie non transactionnelle, avec un orifice appelé **Générateur**et deux boulons appelée **séparateur** et **compteur**. Le bec **Générateur** aléatoire génère certaines phrases et l’émission de ces phrases à **séparateur**. Le **fractionnement** du boulon sera fractionner les phrases à mots et l’émission de ces mots pour boulon de **compteur** . Le boulon « counter » utilise un dictionnaire pour enregistrer le nombre d’occurrences de chaque mot.

Il existe deux fichiers specs, **HelloWorld.spec** et **HelloWorld\_EnableAck.spec** pour cet exemple. Dans la C\# code, peuvent se trouver si l’accusé de réception est activée en obtenant le pluginConf du côté de Java.

    /* demo how to get pluginConf info */
    if (Context.Config.pluginConf.ContainsKey(Constants.NONTRANSACTIONAL_ENABLE_ACK))
    {
        enableAck = (bool)(Context.Config.pluginConf[Constants.NONTRANSACTIONAL_ENABLE_ACK]);
    }
    Context.Logger.Info("enableAck: {0}", enableAck);

Dans le bec, si l’accusé de réception est activée, un dictionnaire est utilisé pour mettre en cache les tuples qui n’ont pas été reçu. Si Fail() est appelée, le tuple ayant échoué sera relu :

    public void Fail(long seqId, Dictionary<string, Object> parms)
    {
        Context.Logger.Info("Fail, seqId: {0}", seqId);
        if (cachedTuples.ContainsKey(seqId))
        {
            /* get the cached tuple */
            string sentence = cachedTuples[seqId];

            /* replay the failed tuple */
            Context.Logger.Info("Re-Emit: {0}, seqId: {1}", sentence, seqId);
            this.ctx.Emit(Constants.DEFAULT_STREAM_ID, new Values(sentence), seqId);
        }
        else
        {
            Context.Logger.Warn("Fail(), can't find cached tuple for seqId {0}!", seqId);
        }
    }

### <a name="helloworldtx"></a>HelloWorldTx

L’exemple **HelloWorldTx** montre comment implémenter une topologie transactionnelle. Il possède un bec appelé **Générateur**, un lot de boulons appelée **nombre partielle**et une validation boulon appelée **somme**. Il existe également trois fichiers txt prédéfinis : **DataSource0.txt**, **DataSource1.txt** et **DataSource2.txt**.

Dans chaque transaction, le bec **Générateur** sera aléatoirement choisissez les deux fichiers dans les trois fichiers prédéfinis et émettre les noms des deux fichiers à boulon **partielle-nombre** . Le boulon **partielle-nombre** tout d’abord obtenir le nom de fichier du tuple reçu, puis ouvrir le fichier et compter le nombre de mots contenus dans ce fichier et enfin émettre le numéro word au boulon **somme** . Le boulon de **somme** résume le nombre total.

Pour atteindre la sémantique **exactement une fois** , le boulon de validation **- somme** doivent évaluer s’il s’agit d’une transaction relue. Dans cet exemple, il a une variable membre static :

    public static long lastCommittedTxId = -1; 

Lorsqu’une instance de ISCPBatchBolt est créée, elle obtient le `txAttempt` à partir des paramètres d’entrée :

    public static CountSum Get(Context ctx, Dictionary<string, Object> parms)
    {
        /* for transactional topology, we can get txAttempt from the input parms */
        if (parms.ContainsKey(Constants.STORM_TX_ATTEMPT))
        {
            StormTxAttempt txAttempt = (StormTxAttempt)parms[Constants.STORM_TX_ATTEMPT];
            return new CountSum(ctx, txAttempt);
        }
        else
        {
            throw new Exception("null txAttempt");
        }
    }

Lors de la `FinishBatch()` est appelée, le `lastCommittedTxId` sera mise à jour si elle n’est pas une transaction relue.

    public void FinishBatch(Dictionary<string, Object> parms)
    {
        /* judge whether it is a replayed transaction? */
        bool replay = (this.txAttempt.TxId <= lastCommittedTxId);
 
        if (!replay)
        {
            /* If it is not replayed, update the toalCount and lastCommittedTxId vaule */
            totalCount = totalCount + this.count;
            lastCommittedTxId = this.txAttempt.TxId;
        }
        … …
    }


### <a name="hybridtopology"></a>HybridTopology

Cette topologie contient un bec de Java et un C\# boulon. Il utilise l’implémentation de sérialisation et de désérialisation par défaut fournie par la plate-forme de SCP. Ref, veuillez le **HybridTopology.spec** dans **exemples de\\HybridTopology** dossier des détails de fichier spec et **SubmitTopology.bat** pour savoir comment spécifier le chemin de classe Java.

### <a name="scphostdemo"></a>SCPHostDemo

Cet exemple est fondamentalement identique HelloWorld. La seule différence est que le code utilisateur est compilé en tant que DLL et la topologie est envoyée à l’aide de SCPHost.exe. Veuillez ref de la section « Mode d’hôte SCP » pour des explications plus détaillées.

##<a name="next-steps"></a>Étapes suivantes

Pour obtenir des exemples de topologies de tempête créés à l’aide de SCP, consultez les rubriques suivantes :

* [Développer des topologies de C# pour tempête Apache sur HDInsight à l’aide de Visual Studio](hdinsight-storm-develop-csharp-visual-studio-topology.md)
* [Traiter les événements de concentrateurs d’événement Azure avec tempête sur HDInsight](hdinsight-storm-develop-csharp-event-hub-topology.md)
* [Créer des flux de données multiples dans une topologie de tempête C#](hdinsight-storm-twitter-trending.md)
* [Alimentation Bi permet de visualiser les données d’une topologie de tempête](hdinsight-storm-power-bi-topology.md)
* [Traiter des données de capteur de véhicule de concentrateurs d’événement à l’aide de la tempête sur HDInsight](https://github.com/hdinsight/hdinsight-storm-examples/tree/master/IotExample)
* [Extraire, transformer et charger (ETL) de concentrateurs d’événement Azure pour HBase](https://github.com/hdinsight/hdinsight-storm-examples/blob/master/RealTimeETLExample)
* [Mettre en corrélation des événements à l’aide de la tempête et HBase sur HDInsight](hdinsight-storm-correlation-topology.md)
