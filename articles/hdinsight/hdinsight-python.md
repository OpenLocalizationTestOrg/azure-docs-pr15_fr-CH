<properties
    pageTitle="Utilisez les Python ruche et porc HDInsight | Microsoft Azure"
    description="Apprenez à utiliser les Python utilisateur définie par fonctions (UDF) à partir de la ruche et de porcins dans HDInsight, la pile de technologie Hadoop sur Azure."
    services="hdinsight"
    documentationCenter=""
    authors="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"/>

<tags
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="python"
    ms.topic="article"
    ms.date="09/07/2016" 
    ms.author="larryfr"/>

#<a name="use-python-with-hive-and-pig-in-hdinsight"></a>Utilisez les Python avec la ruche et porc dans HDInsight

Ruche et porc sont très utiles pour travailler avec des données dans HDInsight, mais parfois vous avez besoin d’un plus grand objectif de langage. Ruche et porcins permettent de créer des fonctions définies par l’utilisateur (UDF) à l’aide d’une variété de langages de programmation. Dans cet article, vous apprendrez comment utiliser un fichier UDF les Python à partir de la ruche et de porcins.

##<a name="requirements"></a>Configuration requise

* Un cluster de HDInsight (Windows ou Linux)

* Un éditeur de texte

    > [AZURE.IMPORTANT] Si vous utilisez un serveur basé sur Linux de HDInsight, mais créez les Python de fichiers sur un client Windows, vous devez utiliser un éditeur qui utilise LF comme une fin de ligne. Si vous ne savez pas si votre éditeur utilise le saut de ligne ou CRLF, consultez la section [dépannage](#troubleshooting) pour obtenir la procédure sur la suppression du caractère CR à l’aide des utilitaires sur le cluster de HDInsight.
    
##<a name="python"></a>Python sur HDInsight

Python2.7 est installé par défaut sur HDInsight 3.0 et version ultérieure clusters. Ruche peut être utilisé avec cette version de Python pour le traitement de flux de données (les données sont passées entre la ruche et les Python à l’aide de STDOUT/STDIN).

HDInsight inclut également Jython, qui est une implémentation de Python écrite en Java. Porcs comprend comment communiquer avec Jython sans avoir recours à la diffusion en continu, donc c’est préférable lorsque vous utilisez des porcs. Toutefois, vous pouvez également utiliser les Python normales (Python de C), avec des porcs ainsi.

##<a name="hivepython"></a>Ruche et les Python

Python peut être utilisé comme un fichier UDF à partir de la ruche par le biais de la HiveQL **transformer** instruction. Par exemple, le HiveQL suivant appelle un script Python stocké dans le fichier **streaming.py** .

**Basé sur Linux de HDInsight**

    add file wasbs:///streaming.py;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
      USING 'python streaming.py' AS
      (clientid string, phoneLable string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;

**HDInsight de basées sur Windows**

    add file wasbs:///streaming.py;

    SELECT TRANSFORM (clientid, devicemake, devicemodel)
      USING 'D:\Python27\python.exe streaming.py' AS
      (clientid string, phoneLable string, phoneHash string)
    FROM hivesampletable
    ORDER BY clientid LIMIT 50;

> [AZURE.NOTE] Sur les clusters basés sur Windows de HDInsight, la clause **USING** doit spécifier le chemin d’accès complet à python.exe. C’est toujours le `D:\Python27\python.exe`.

Voici ce que fait cet exemple :

1. L’instruction **d’Ajouter le fichier** au début du fichier ajoute le fichier **streaming.py** dans le cache distribué, afin qu’il soit accessible par tous les nœuds du cluster.

2. La transformation **SELECT... À l’aide de** instruction sélectionne des données de la **hivesampletable**et transmet le clientid, devicemake et devicemodel pour le script **streaming.py** .

3. La clause **AS** décrit les champs renvoyés à partir de **streaming.py**

<a name="streamingpy"></a>Voici le fichier **streaming.py** utilisé par l’exemple HiveQL.

    #!/usr/bin/env python

    import sys
    import string
    import hashlib

    while True:
      line = sys.stdin.readline()
      if not line:
        break

      line = string.strip(line, "\n ")
      clientid, devicemake, devicemodel = string.split(line, "\t")
      phone_label = devicemake + ' ' + devicemodel
      print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])

Puisque nous utilisons la diffusion en continu, ce script doit effectuer les opérations suivantes :

1. Données lues à partir de STDIN. Cela est accompli à l’aide de `sys.stdin.readline()` dans cet exemple.

2. Le caractère de saut de ligne à droite est supprimé à l’aide de `string.strip(line, "\n ")`, dans la mesure où nous voulons simplement les données de texte et pas l’indicateur de fin de ligne.

2. Lorsque vous effectuez le traitement de flux, une seule ligne contient toutes les valeurs avec un caractère de tabulation entre chaque valeur. Par conséquent, `string.split(line, "\t")` peut être utilisé pour diviser l’entrée à chaque onglet, en retournant uniquement les champs.

3. Lorsque le traitement est terminé, la sortie doit être écrite dans STDOUT sous la forme d’une seule ligne, avec une tabulation entre chaque champ. Cela est accompli à l’aide de `print "\t".join([clientid, phone_label, hashlib.md5(phone_label).hexdigest()])`.

4. Tout ceci se produit dans une `while` boucle, qui se répétera jusqu'à ce que no `line` est lu à partir de laquelle `break` quitte la boucle et le script se termine.

En outre, le script concatène simplement les valeurs d’entrée pour `devicemake` et `devicemodel`et calcule un hachage de la valeur concaténée. Assez simple, mais il décrit les notions de base du fonctionne de tout script de Python appelé à partir de la ruche : boucle, lecture d’entrée jusqu'à ce qu’il soit non plus, séparez chaque ligne d’entrée dans les onglets, les processus, d’écrire une seule ligne de sortie de délimités par des tabulations.

Consultez [les exemples en cours d’exécution](#running) pour l’exécution de cet exemple sur votre cluster de HDInsight.

##<a name="pigpython"></a>Porcs et les Python

Un script de Python peut être utilisé sous la forme d’un fichier UDF de porc par le biais de l’instruction de **Générer** . Il existe deux façons d’y parvenir ; à l’aide de Jython (les Python implémentées sur la Machine virtuelle Java) et C Python (Python régulière). 

La principale différence entre ces sont que Jython s’exécute sur la machine virtuelle Java et en mode natif peut être appelée qu’à partir du porc (également en cours d’exécution sur la machine virtuelle Java.) Les Python C est un processus externe (écrit en C.) Ainsi, les données à partir de porc sur la machine virtuelle Java sont envoyées pour le script qui s’exécute dans un processus de Python, puis la sortie qui est envoyée vers le porc.

Pour déterminer si les porcs utilise Jython ou les Python C pour exécuter le script, utilisez __Enregistrer__ lorsque vous référencez le script Python de porc Latin. Cette option indique à porc quel interpréteur à utiliser et les alias à créer pour le script. Les exemples suivants enregistrement des scripts avec des porcs en tant que __myfuncs__:

* __Pour utiliser Jython__:`register '/path/to/pig_python.py' using jython as myfuncs;`
* __Pour utiliser les Python C__:`register '/path/to/pig_python.py' using streaming_python as myfuncs;`

> [AZURE.IMPORTANT] Lorsque vous utilisez Jython, le chemin d’accès vers le fichier pig_jython peut être un chemin local ou un WASB : / / chemin d’accès. Toutefois, lorsque vous utilisez les Python C, vous devez référencer un fichier sur le système de fichiers local du nœud que vous utilisez pour soumettre le travail de porc.

Une fois après l’inscription, le Latin de porc pour cet exemple est le même pour les deux :

    LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
    LOG = FILTER LOGS by LINE is not null;
    DETAILS = FOREACH LOG GENERATE myfuncs.create_structure(LINE);
    DUMP DETAILS;

Voici ce que fait cet exemple :

1. La première ligne charge l’exemple de fichier de données, **exemple.log** dans les **journaux**. Dans la mesure où ce fichier journal n’a pas un schéma cohérent, il définit également chaque enregistrement (**ligne** dans ce cas,) sous la forme d’un **chararray**. Pour l’essentiel, charArray est une chaîne.

2. La ligne suivante filtre toutes les valeurs null, contenant le résultat de l’opération dans le **journal**.

3. Ensuite, il effectue une itération sur les enregistrements de **journal** et utilise les **Générer** pour appeler la méthode **create_structure** contenue dans le script de Python/Jython chargé en tant que **myfuncs**.  **Ligne** permet de passer de l’enregistrement en cours à la fonction.

4. Enfin, les sorties sont vidés à STDOUT à l’aide de la commande **DUMP** . Il s’agit simplement d’afficher immédiatement les résultats une fois l’opération terminée ; dans un script réel vous le faites normalement de **magasin de** données dans un nouveau fichier.

Le fichier de script de Python réel est également similaire entre les Python C et Jython, la seule différence étant que vous devez importer à partir de __porc\_util__ lorsque vous utilisez les Python C. Voici le __porc\_python.py__ script :

<a name="streamingpy"></a>

    # Uncomment the following if using C Python
    #from pig_util import outputSchema

    @outputSchema("log: {(date:chararray, time:chararray, classname:chararray, level:chararray, detail:chararray)}")
    def create_structure(input):
    if (input.startswith('java.lang.Exception')):
        input = input[21:len(input)] + ' - java.lang.Exception'
    date, time, classname, level, detail = input.split(' ', 4)
    return date, time, classname, level, detail

> [AZURE.NOTE] 'pig_util' n’est pas quelque chose que vous devez à vous soucier de l’installation ; Il est automatiquement disponible pour le script.

N’oubliez pas que nous avons précédemment définie la **ligne** d’entrée comme un chararray car il n’a aucun schéma cohérent pour l’entrée ? Quelles sont les Python du script est de transformer les données dans un schéma cohérent pour la sortie. Il fonctionne comme suit :

1. Le **@outputSchema** instruction définit le format des données qui seront renvoyés aux porcs. Dans ce cas, il s’agit d’un **sac de données**, qui est un type de données de porc. Le sac contient les champs suivants, qui sont tous des chararray (chaînes) :

    * date - date de que l’entrée de journal a été créée.
    * heure-heure à laquelle que l’entrée de journal a été créée.
    * nom de classe - le nom de classe l’entrée a été créé pour
    * niveau - le niveau de journalisation
    * détail - détails commentés pour l’entrée du journal

2. Ensuite, la **définition create_structure(input)** définit la fonction que porc passera de lignes de facturation.

3. Les données d’exemple, **exemple.log**, est essentiellement conforme au date, heure, classname, niveau, schéma et au détail que nous voulons renvoyer. Mais il contient aussi quelques lignes qui commencent par la chaîne '*java.lang.Exception*' qui doivent être modifiés pour correspondre au schéma. L’instruction **if** vérifie les, puis transforme les données d’entrée pour déplacer la chaîne '*java.lang.Exception*' à la fin, mettre la données en ligne notre schéma de production prévue.

4. Ensuite, la commande **Fractionner** permet de fractionner les données sur les premiers caractères de l’espace de quatre. Ainsi, cinq valeurs, qui sont affectées en ** **date**, **heure**, **classname**et **Détail****.

5. Enfin, les valeurs sont renvoyées pour les porcs.

Lorsque les données sont renvoyées au porc, il aura un schéma cohérent tel que défini dans le **@outputSchema** instruction.

##<a name="running"></a>Les exemples en cours d’exécution

Si vous utilisez un cluster basé sur Linux de HDInsight, suivez les étapes **SSH** ci-dessous. Si vous utilisez un cluster de HDInsight de basées sur Windows et un client Windows, suivez les étapes de **PowerShell** .

###<a name="ssh"></a>SSH

Pour plus d’informations sur l’utilisation du protocole SSH, consultez <a href="../hdinsight-hadoop-linux-use-ssh-unix/" target="_blank">Utiliser SSH avec basé sur Linux d’Hadoop sur HDInsight de Linux, Unix ou de OS X</a> ou <a href="../hdinsight-hadoop-linux-use-ssh-windows/" target="_blank">Utiliser SSH avec basé sur Linux d’Hadoop sur HDInsight à partir de Windows</a>.

1. Les Python exemples [streaming.py](#streamingpy) et [pig_python.py](#jythonpy), de créer des copies locales des fichiers sur votre ordinateur de développement.

2. Utilisez `scp` pour copier les fichiers à votre cluster HDInsight. Par exemple, les éléments suivants seraient copiez les fichiers vers un cluster nommé **mon_cluster**.

        scp streaming.py pig_python.py myuser@mycluster-ssh.azurehdinsight.net:

3. Utiliser le protocole SSH pour se connecter au cluster. Par exemple, le texte suivant connecte à un cluster nommé **mon_cluster** en tant qu’utilisateur **myuser**.

        ssh myuser@mycluster-ssh.azurehdinsight.net

4. À partir de la session SSH, ajouter les python téléchargés les fichiers précédemment dans le stockage WASB pour le cluster.

        hdfs dfs -put streaming.py /streaming.py
        hdfs dfs -put pig_python.py /pig_python.py

Après avoir téléchargé les fichiers, procédez comme suit pour exécuter les tâches de la ruche et de porcins.

####<a name="hive"></a>La ruche

1. Utilisez la `hive` une commande pour démarrer l’environnement de la ruche. Vous devriez voir un `hive>` demander une fois que le shell a chargé.

2. Entrez la commande suivante à le `hive>` invite.

        add file wasbs:///streaming.py;
        SELECT TRANSFORM (clientid, devicemake, devicemodel)
          USING 'python streaming.py' AS
          (clientid string, phoneLabel string, phoneHash string)
        FROM hivesampletable
        ORDER BY clientid LIMIT 50;

3. Après avoir entré la dernière ligne, la tâche doit démarrer. Finalement il retournera sortie semblable à la suivante.

        100041  RIM 9650    d476f3687700442549a83fac4560c51c
        100041  RIM 9650    d476f3687700442549a83fac4560c51c
        100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9
        100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9
        100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9

####<a name="pig"></a>Porcs

1. Utilisez la `pig` une commande pour démarrer l’interpréteur de commandes. Vous devriez voir un `grunt>` demander une fois que le shell a chargé.

2. Entrez les instructions suivantes sur le `grunt>` invite pour exécuter le script de Python à l’aide de l’interpréteur Jython.

        Register wasbs:///pig_python.py using jython as myfuncs;
        LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
        LOG = FILTER LOGS by LINE is not null;
        DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
        DUMP DETAILS;

3. Après avoir tapé la ligne suivante, la tâche doit démarrer. Finalement il retournera sortie semblable à la suivante.

        ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
        ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
        ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
        ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
        ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

4. Utilisez `quit` pour quitter le shell Grunt, puis utilisez ce qui suit pour modifier le fichier pig_python.py sur le système de fichiers local :

    NANO pig_python.py

5. Une fois dans l’éditeur, ne commentez pas la ligne suivante en supprimant le `#` caractère depuis le début de la ligne :

        #from pig_util import outputSchema

    Une fois que la modification a été apportée, utilisez Ctrl + X pour quitter l’éditeur. Sélectionnez Y, puis sur ENTRÉE pour enregistrer les modifications.

6. Utilisez le `pig` commande pour redémarrer le shell. Une fois que vous êtes à la `grunt>` invite, utilisez ce qui suit pour exécuter le script de Python à l’aide de l’interpréteur les Python C.

        Register 'pig_python.py' using streaming_python as myfuncs;
        LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);
        LOG = FILTER LOGS by LINE is not null;
        DETAILS = foreach LOG generate myfuncs.create_structure(LINE);
        DUMP DETAILS;

    Une fois cette tâche terminée, vous devriez voir la même sortie que lorsque vous avez précédemment exécuté le script à l’aide de Jython.

###<a name="powershell"></a>PowerShell

Ces étapes utilisent Azure PowerShell. Si ce n’est pas déjà installé et configuré sur votre ordinateur de développement, reportez-vous à la section [comment installer et configurer Azure PowerShell](../powershell-install-configure.md) avant d’utiliser les étapes suivantes.

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

1. Les Python exemples [streaming.py](#streamingpy) et [pig_python.py](#jythonpy), de créer des copies locales des fichiers sur votre ordinateur de développement.

2. Utilisez le script PowerShell suivant pour télécharger le **streaming.py** et **porc\_python.py** les fichiers sur le serveur. Remplacez par le nom de votre cluster Azure HDInsight et le chemin d’accès à la **streaming.py** et de **porc\_python.py** les fichiers sur les trois premières lignes du script.

        $clusterName = YourHDIClusterName
        $pathToStreamingFile = "C:\path\to\streaming.py"
        $pathToJythonFile = "C:\path\to\pig_python.py"

        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value

        #Create a storage content and upload the file
        $context = New-AzureStorageContext `
            -StorageAccountName $storageAccountName `
            -StorageAccountKey $storageAccountKey
        
        Set-AzureStorageBlobContent `
            -File $pathToStreamingFile `
            -Blob "streaming.py" `
            -Container $container `
            -Context $context
        
        Set-AzureStorageBlobContent `
            -File $pathToJythonFile `
            -Blob "pig_python.py" `
            -Container $container `
            -Context $context

    Ce script récupère les informations de votre cluster HDInsight, extrait le compte et la clé pour le compte de stockage par défaut, puis télécharge les fichiers à la racine du conteneur.

    > [AZURE.NOTE] Vous trouverez d’autres méthodes de téléchargement de scripts dans le document à [télécharger des données pour les travaux d’Hadoop dans HDInsight](hdinsight-upload-data.md) .

Après avoir téléchargé les fichiers, utilisez les scripts PowerShell suivantes pour démarrer les travaux. Lorsque le travail est terminé, le résultat doit être écrit dans la console PowerShell.

####<a name="hive"></a>La ruche

Le script suivant exécute le script __streaming.py__ . Avant d’exécuter, il vous invite pour les informations de compte pour votre cluster HDInsight HTTPs/Admin.

    # Replace 'YourHDIClusterName' with the name of your cluster
    $clusterName = YourHDIClusterName
    $creds=Get-Credential
    #Get the cluster info so we can get the resource group, storage, etc.
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
    $container=$clusterInfo.DefaultStorageContainer
    $storageAccountKey=(Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
    #Create a storage content and upload the file
    $context = New-AzureStorageContext `
        -StorageAccountName $storageAccountName `
        -StorageAccountKey $storageAccountKey
    
    # If using a Windows-based HDInsight cluster, change the USING statement to:
    # "USING 'D:\Python27\python.exe streaming.py' AS " +
    $HiveQuery = "add file wasbs:///streaming.py;" +
                 "SELECT TRANSFORM (clientid, devicemake, devicemodel) " +
                   "USING 'python streaming.py' AS " +
                   "(clientid string, phoneLabel string, phoneHash string) " +
                 "FROM hivesampletable " +
                 "ORDER BY clientid LIMIT 50;"

    $jobDefinition = New-AzureRmHDInsightHiveJobDefinition `
        -Query $HiveQuery

    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
    Write-Host "Wait for the Hive job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -JobId $job.JobId `
        -ClusterName $clusterName `
        -HttpCredential $creds
    # Uncomment the following to see stderr output
    # Get-AzureRmHDInsightJobOutput `
    #   -Clustername $clusterName `
    #   -JobId $job.JobId `
    #   -DefaultContainer $container `
    #   -DefaultStorageAccountName $storageAccountName `
    #   -DefaultStorageAccountKey $storageAccountKey `
    #   -HttpCredential $creds `
    #   -DisplayOutputType StandardError
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -HttpCredential $creds

La sortie de la tâche de **la ruche** doit ressembler à ce qui suit :

    100041  RIM 9650    d476f3687700442549a83fac4560c51c
    100041  RIM 9650    d476f3687700442549a83fac4560c51c
    100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9
    100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9
    100042  Apple iPhone 4.2.x  375ad9a0ddc4351536804f1d5d0ea9b9

####<a name="pig-jython"></a>Porcs (Jython)

Celui-ci utilisera le script __pig_python.py__ , à l’aide de l’interpréteur Jython. Avant d’exécuter, il vous demandera les informations de HTTPs/Admin pour le cluster HDInsight.

> [AZURE.NOTE] Lors de l’envoi d’un travail à l’aide de PowerShell à distance, il n’est pas possible d’utiliser les Python C l’interpréteur.

    # Replace 'YourHDIClusterName' with the name of your cluster
    $clusterName = YourHDIClusterName

    $creds = Get-Credential
    #Get the cluster info so we can get the resource group, storage, etc.
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
    $container=$clusterInfo.DefaultStorageContainer
    $storageAccountKey=(Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value
    
    #Create a storage content and upload the file
    $context = New-AzureStorageContext `
        -StorageAccountName $storageAccountName `
        -StorageAccountKey $storageAccountKey
            
    $PigQuery = "Register wasbs:///jython.py using jython as myfuncs;" +
                "LOGS = LOAD 'wasbs:///example/data/sample.log' as (LINE:chararray);" +
                "LOG = FILTER LOGS by LINE is not null;" +
                "DETAILS = foreach LOG generate myfuncs.create_structure(LINE);" +
                "DUMP DETAILS;"

    $jobDefinition = New-AzureRmHDInsightPigJobDefinition -Query $PigQuery

    $job = Start-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobDefinition $jobDefinition `
        -HttpCredential $creds
        
    Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
    Wait-AzureRmHDInsightJob `
        -Job $job.JobId `
        -ClusterName $clusterName `
        -HttpCredential $creds
    # Uncomment the following to see stderr output
    # Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -HttpCredential $creds

La sortie de la tâche de **porc** doit apparaître semblable à la suivante :

    ((2012-02-03,20:11:56,SampleClass5,[TRACE],verbose detail for id 990982084))
    ((2012-02-03,20:11:56,SampleClass7,[TRACE],verbose detail for id 1560323914))
    ((2012-02-03,20:11:56,SampleClass8,[DEBUG],detail for id 2083681507))
    ((2012-02-03,20:11:56,SampleClass3,[TRACE],verbose detail for id 1718828806))
    ((2012-02-03,20:11:56,SampleClass3,[INFO],everything normal for id 530537821))

##<a name="troubleshooting"></a>Résolution des problèmes

###<a name="errors-when-running-jobs"></a>Erreurs lors de l’exécution des travaux

Lors de l’exécution de la tâche de la ruche, vous pouvez rencontrer une erreur similaire à ce qui suit :

    Caused by: org.apache.hadoop.hive.ql.metadata.HiveException: [Error 20001]: An error occurred while reading or writing to your custom script. It may have crashed with an error.
    
Ce problème peut être provoqué par les fins de ligne dans le fichier streaming.py. De nombreux Windows éditeurs utilisent par défaut le CRLF comme la fin de ligne, mais les applications Linux généralement attendre LF.

Si vous utilisez un éditeur qui ne peut pas créer les fins de ligne saut de ligne, ou n’êtes pas sûr, les fins de ligne sont utilisées, utilisez les instructions PowerShell suivantes pour supprimer les caractères CR avant de télécharger le fichier à HDInsight :

    $original_file ='c:\path\to\streaming.py'
    $text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
    [IO.File]::WriteAllText($original_file, $text)

###<a name="powershell-scripts"></a>Scripts PowerShell

Les deux scripts PowerShell exemple utilisés pour exécuter les exemples contiennent une ligne de commentaire qui affichera la sortie d’erreur de la tâche. Si vous n’obtenez pas le résultat attendu pour le travail, la ligne suivante et voir si les informations d’erreur indiquent un problème.

    # Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $job.JobId `
            -DefaultContainer $container `
            -DefaultStorageAccountName $storageAccountName `
            -DefaultStorageAccountKey $storageAccountKey `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

Les informations d’erreur (STDERR) et le résultat de la tâche (STDOUT), sont également enregistrés dans le conteneur d’objet blob par défaut pour vos clusters aux emplacements suivants.

Pour cette tâche...|Examinez ces fichiers dans le conteneur de l’objet blob
---|---
La ruche|/ HivePython/stderr<p>/ HivePython/stdout
Porcs|/ PigPython/stderr<p>/ PigPython/stdout

##<a name="next"></a>Étapes suivantes

Si vous devez charger les Python les modules qui ne sont pas fournis par défaut, reportez-vous à la section [comment déployer un module pour Azure HDInsight](http://blogs.msdn.com/b/benjguin/archive/2014/03/03/how-to-deploy-a-python-module-to-windows-azure-hdinsight.aspx) pour obtenir un exemple de cette procédure.

Pour les autres façons d’utiliser de porc, de la ruche et pour en savoir plus sur l’utilisation de MapReduce, consultez la rubrique suivante.

* [Utilisez la ruche avec HDInsight](hdinsight-use-hive.md)

* [Utilisez des porcs avec HDInsight](hdinsight-use-pig.md)

* [Utilisez MapReduce avec HDInsight](hdinsight-use-mapreduce.md)
