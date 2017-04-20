<properties 
    pageTitle="Créer des Modules personnalisés R dans l’apprentissage automatique Azure | Microsoft Azure" 
    description="Démarrage rapide pour la création de modules de R personnalisés dans l’apprentissage automatique de Azure." 
    services="machine-learning" 
    documentationCenter="" 
    authors="bradsev"  
    manager="jhubbard" 
    editor="cgronlun" />

<tags 
    ms.service="machine-learning" 
    ms.devlang="na" 
    ms.topic="article" 
    ms.tgt_pltfrm="na" 
    ms.workload="tbd" 
    ms.date="08/19/2016" 
    ms.author="bradsev;ankarloff" />


# <a name="author-custom-r-modules-in-azure-machine-learning"></a>Auteur R modules personnalisés dans Azure Machine Learning

Cette rubrique décrit comment créer et déployer un module R personnalisé dans l’apprentissage automatique de Azure. Il explique ce que sont les modules R personnalisés et les fichiers qui sont utilisés pour définir les. Il illustre comment construire les fichiers qui définissent un module et comment inscrire le module pour le déploiement dans un espace de travail d’apprentissage automatique. Les éléments et les attributs utilisés dans la définition du module personnalisé sont ensuite décrits plus en détail. Comment faire pour utiliser des fonctions auxiliaires, des fichiers et des sorties multiples est également abordée. 

[AZURE.INCLUDE [machine-learning-free-trial](../../includes/machine-learning-free-trial.md)]


## <a name="what-is-a-custom-r-module"></a>Quel est un module personnalisé de R ?

Un **module personnalisé** est un module défini par l’utilisateur qui peut être téléchargé vers votre espace de travail et exécuté dans le cadre d’une expérience d’apprentissage automatique de Azure. Un **module personnalisé de R** est un module personnalisé qui exécute une fonction définie par l’utilisateur de R. **R** est un langage de programmation informatique et des graphismes qui est largement utilisée par scientifiques statisticiens et de données pour la mise en œuvre d’algorithmes statistiques. R est actuellement le seul langage pris en charge dans les modules personnalisés, mais la prise en charge d’autres langues est prévue pour les versions futures.

Modules personnalisés ont un **état de première classe** dans Azure Machine apprentissage en ce sens qu’ils peuvent être utilisés comme tout autre module. Elles peuvent être exécutées avec d’autres modules, inclus dans les essais publiés ou dans les visualisations. Vous contrôlez l’algorithme implémenté par le module, l’entrée et à utiliser des ports de sortie, les paramètres de modélisation et autres comportements d’exécution différents. Une expérience qui contient des modules personnalisés peut également être publiée dans la galerie d’Intelligence Cortana pour un partage aisé.


## <a name="files-in-a-custom-r-module"></a>Fichiers dans un module de R personnalisé

Un module personnalisé de R est défini par un fichier .zip qui contient, au minimum, deux fichiers :

* Un **fichier source** qui implémente la fonction R exposée par le module
* Un **fichier de définition XML** qui décrit l’interface de module personnalisé

Autres fichiers auxiliaires peuvent également être inclus dans le fichier .zip qui fournit des fonctionnalités qui sont accessibles à partir du module personnalisé. Cette option est décrite dans la partie des **Arguments** de la section de référence **les éléments dans le fichier de définition XML** de l’exemple de démarrage rapide.


## <a name="quickstart-example-define-package-and-register-a-custom-r-module"></a>Exemple de démarrage rapide : définir, package et enregistrer un module personnalisé de R

Cet exemple illustre comment créer les fichiers requis par un module de R personnalisé, les empaqueter dans un fichier zip et puis d’inscrire le module dans votre espace de travail d’apprentissage automatique. Les fichiers de package et les exemples de zip exemple peuvent être téléchargés à partir du [fichier de téléchargement de CustomAddRows.zip](http://go.microsoft.com/fwlink/?LinkID=524916&clcid=0x409).

## <a name="the-source-file"></a>Le fichier source
Prenons l’exemple d’un module **Personnalisé Add Rows** qui modifie la mise en oeuvre standard du module **Add Rows** utilisé pour concaténer des lignes (observations) à partir de deux ensembles de données (trames de données). Le module **d’Ajouter des lignes** standard ajoute les lignes du deuxième groupe de données d’entrée à la fin de la première à l’aide de jeu de données d’entrée le `rbind` algorithme. Le texte personnalisé `CustomAddRows` fonction accepte les deux groupes de données de la même façon, mais accepte également un paramètre booléen swap comme une entrée supplémentaire. Si le paramètre de remplacement est défini sur **FALSE**, elle renvoie le même jeu de données que la mise en oeuvre standard. Mais, si le paramètre de remplacement a la **valeur TRUE**, la fonction ajoute des lignes du premier groupe de données d’entrée à la fin du deuxième groupe de données à la place. Le fichier CustomAddRows.R qui contient la mise en oeuvre de la R `CustomAddRows` fonction exposée par le module **Personnalisé Add Rows** a le code R suivant.

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) 
    {
        if (swap)
        {
            return (rbind(dataset2, dataset1));
        }
        else
        {
            return (rbind(dataset1, dataset2));
        } 
    } 

### <a name="the-xml-definition-file"></a>Le fichier de définition XML
Pour exposer cette `CustomAddRows` fonction sous la forme d’un module de formation de Machine d’Azure, un fichier de définition XML doit être créée pour définir le module **Personnalisé Add Rows** doit rechercher et le comportement. 

    <!-- Defined a module using an R Script -->
    <Module name="Custom Add Rows">
        <Owner>Microsoft Corporation</Owner>
        <Description>Appends one dataset to another. Dataset 2 is concatenated to Dataset 1 when Swap is FALSE, and vice versa when Swap is TRUE.</Description>
    
    <!-- Specify the base language, script file and R function to use for this module. -->      
        <Language name="R" 
         sourceFile="CustomAddRows.R" 
         entryPoint="CustomAddRows" />  
        
    <!-- Define module input and output ports -->
    <!-- Note: The values of the id attributes in the Input and Arg elements must match the parameter names in the R Function CustomAddRows defined in CustomAddRows.R. -->
        <Ports>
            <Input id="dataset1" name="Dataset 1" type="DataTable">
                <Description>First input dataset</Description>
            </Input>
            <Input id="dataset2" name="Dataset 2" type="DataTable">
                <Description>Second input dataset</Description>
            </Input>
            <Output id="dataset" name="Dataset" type="DataTable">
                <Description>The combined dataset</Description>
            </Output>
        </Ports>
        
    <!-- Define module parameters -->
        <Arguments>
            <Arg id="swap" name="Swap" type="bool" >
                <Description>Swap input datasets.</Description>
            </Arg>
        </Arguments>
    </Module>

 
Il est très important de noter que la valeur des attributs **id** des éléments dans le fichier XML **d’entrée** et **Arg** doit correspondre exactement les noms de paramètre de fonction du code dans le fichier CustomAddRows.R R : (*dataset1*, *dataset2*et *swap* dans l’exemple). De même, la valeur de l’attribut de **point d’entrée** de l’élément de **langage** doit correspondre exactement au nom de la fonction dans le script de R : (*CustomAddRows* dans l’exemple). 

En revanche, l’attribut **id** de l’élément de **sortie** ne correspond pas à toutes les variables dans le script de R. Lorsque plus d’une sortie est requise, retournez simplement une liste à partir de la fonction R avec les résultats, placés *dans le même ordre* que les **sorties** sont déclarés dans le fichier XML.

### <a name="package-and-register-the-module"></a>Créer un package et d’inscrire le module
Enregistrez ces deux fichiers comme *CustomAddRows.R* et *CustomAddRows.xml* , puis compresser les deux fichiers ensemble dans un fichier *CustomAddRows.zip* .

Pour les enregistrer dans votre espace de travail d’apprentissage automatique, accéder à votre espace de travail dans le Studio d’apprentissage Machine, cliquez sur le bouton **+ de nouveau** sur le bas et choisissez **MODULE -> à partir de PACKAGE ZIP** à télécharger le nouveau module **Personnalisé Add Rows** .

![Téléchargement Zip](./media/machine-learning-custom-r-modules/upload-from-zip-package.png)

Le module **Personnalisé Add Rows** est prêt à être accessible par vos expériences d’apprentissage automatique.


## <a name="elements-in-the-xml-definition-file"></a>Éléments dans le fichier de définition XML

### <a name="module-elements"></a>Éléments de module
L’élément de **Module** est utilisé pour définir un module personnalisé dans le fichier XML. Plusieurs modules peuvent être définis dans un fichier XML à l’aide de plusieurs éléments de **module** . Chaque module de votre espace de travail doit avoir un nom unique. Enregistrer un module personnalisé avec le même nom qu’un module personnalisé existant, et il remplace le module existant par le nouveau. Les modules personnalisés peuvent, toutefois, être enregistrés avec le même nom qu’un module de formation de Machine Azure existant. Dans ce cas, elles apparaissent dans la catégorie **personnalisée** de la palette de module.

    <Module name="Custom Add Rows" isDeterministic="false"> 
        <Owner>Microsoft Corporation</Owner>
        <Description>Appends one dataset to another...</Description>/> 


Dans l’élément **Module** , vous pouvez spécifier deux éléments facultatifs supplémentaires :

- un élément **propriétaire** qui est incorporé dans le module  
- un élément de **Description** qui contient le texte qui est affiché dans l’aide rapide pour le module, et lorsque vous survolez le module dans la formation de Machine interface utilisateur.


Règles pour les limites de caractères dans les éléments de Module :

* La valeur de l’attribut **name** dans l’élément de **Module** ne doit pas dépasser 64 caractères. 
* Le contenu de l’élément de **Description** ne doit pas dépasser 128 caractères.
* Le contenu de l’élément **propriétaire** ne doit pas dépasser 32 caractères.


Les résultats d’un module peuvent être déterministes ou nondeterministic.* * par défaut, tous les modules sont considérés comme étant déterministe. Autrement dit, le module avec un ensemble fixe de paramètres d’entrée et les données, doit retourner le même eacRAND de résultats ou une functionh de son exécution. Compte tenu de ce comportement, Azure Machine Learning Studio réexécute uniquement les modules marqués comme déterministe si un paramètre ou les données d’entrée a été modifiée. Renvoie les résultats mis en cache fournit également une exécution beaucoup plus rapide des expériences.

Il existe des fonctions non déterministes, telles que RAND ou une fonction qui renvoie la date ou l’heure. Si votre module utilise une fonction non déterministe, vous pouvez spécifier que le module n’est pas déterministe en définissant l’attribut facultatif **isDeterministic** sur **FALSE**. Cela garantit que le module est réexécuté à chaque exécution de l’essai, même si le module d’entrée et les paramètres n’ont pas changé. 

### <a name="language-definition"></a>Définition du langage
L’élément de **langage** dans votre fichier de définition XML est utilisé pour spécifier la langue du module personnalisé. R est actuellement le seul langage pris en charge. La valeur de l’attribut **sourceFile** doit être le nom de fichier R qui contient la fonction à appeler lorsque le module est exécuté. Ce fichier doit être dans le package zip. La valeur de l’attribut de **point d’entrée** est le nom de la fonction appelée et doit correspondre à une fonction valide associée dans le fichier source.

    <Language name="R" sourceFile="CustomAddRows.R" entryPoint="CustomAddRows" />


### <a name="ports"></a>Ports
Les ports d’entrée et de sortie pour un module personnalisé sont spécifiés dans les éléments enfants de la section **Ports** du fichier de définition XML. L’ordre de ces éléments détermine la mise en page expérimenté (UX) par les utilisateurs. Le premier enfant **d’entrée** ou **sortie** répertoriés dans l’élément de **Ports** du fichier XML est le port d’entrée de la plus à gauche dans la formation de Machine UX.
Chaque entrée et sortie port peut avoir un élément enfant **Description** facultatif qui spécifie le texte affiché lorsque vous placez le curseur de la souris sur le port dans l’interface utilisateur formation de Machine.

**Les règles de ports**:

* Nombre maximal de **ports d’entrée et de sortie** est 8 pour chacun.

### <a name="input-elements"></a>Éléments d’entrée
Ports d’entrée vous permettent de passer des données à votre fonction de R et un espace de travail. Les **types de données** pris en charge pour les ports d’entrée sont les suivantes : 

**DataTable :** Ce type est passé à votre fonction R comme un data.frame. En fait, tous les types (par exemple, des fichiers CSV ou des fichiers ARFF) qui sont pris en charge par l’apprentissage automatique et qui sont compatibles avec le **DataTable** sont convertis automatiquement à un data.frame. 

        <Input id="dataset1" name="Input 1" type="DataTable" isOptional="false">
            <Description>Input Dataset 1</Description>
        </Input>

L’attribut **id** associé à chaque port d’entrée de **table de données** doit avoir une valeur unique et cette valeur doit correspondre à son paramètre nommé correspondant dans votre fonction de R.
Ports en option **DataTable** qui ne sont pas transmis comme entrée d’une expérience ont la valeur **NULL** passée à la fonction R et ports de zip facultatifs sont ignorés si l’entrée n’est pas connectée. L’attribut **isOptional** est facultatif pour les types à la fois le **DataTable** et **le code postal** et est *false* par défaut.
       
**Zip :** Les modules personnalisés peuvent accepter un fichier zip en tant qu’entrée. Cette entrée est placée dans le répertoire de travail R de la fonction

        <Input id="zippedData" name="Zip Input" type="Zip" IsOptional="false">
            <Description>Zip files to be extracted to the R working directory.</Description>
        </Input>

Pour les modules R personnalisés, l’id d’un port de Zip ne faire correspondre les paramètres de la fonction de R. C’est parce que le fichier zip est automatiquement extraits vers le répertoire de travail de R.

**Règles de saisie :**

* La valeur de l’attribut **id** de l’élément **Input** doit être un nom de variable valide R.
* La valeur de l’attribut **id** de l’élément **Input** ne doit pas dépasser 64 caractères.
* La valeur de l’attribut **name** de l’élément **Input** ne doit pas dépasser 64 caractères.
* Le contenu de l’élément de **Description** ne doit pas être plu de 128 caractères
* La valeur de l’attribut **type** de l’élément **Input** doit être *le code postal* ou un *DataTable*.
* La valeur de l’attribut **isOptional** de l’élément **d’entrée** n’est pas requise (et s’il est *false* par défaut n’est ne pas spécifié) ; mais s’il est spécifié, il doit être *true* ou *false*.

### <a name="output-elements"></a>Éléments de sortie

**Standard des ports de sortie :** Ports de sortie sont mappées aux valeurs de retour à partir de votre fonction de R, qui peut ensuite être utilisée par les modules suivants. *Table de données* est le type de port de sortie standard uniquement pris en charge actuellement. (Prise en charge des *apprenants* et des *transformations* est à venir.) Une sortie de *DataTable* est définie comme suit :

    <Output id="dataset" name="Dataset" type="DataTable">
        <Description>Combined dataset</Description>
    </Output>

Pour les sorties dans les modules R personnalisés, la valeur de l’attribut **id** ne doit pas correspondre avec n’importe quoi dans le script de R, mais il doit être unique. Pour une sortie de module unique, la valeur de retour de la fonction de R doit être un *data.frame*. Pour exporter plusieurs objets d’un type de données pris en charge, les ports de sortie appropriée doivent être spécifiés dans le fichier de définition XML et les objets doivent être renvoyés sous forme de liste. Les objets de sortie sont affectés à la sortie des ports à partir de la gauche vers la droite, qui reflète l’ordre dans lequel les objets sont placés dans la liste retournée.

Par exemple, si vous souhaitez modifier le module **Personnalisé Add Rows** pour les deux jeux de données d’origine de la sortie, *dataset1* et *dataset2*, en plus de la nouvelle jointes dataset, le *dataset*, (dans l’ordre, de gauche à droite, sous la forme : *dataset*, *dataset1*, *dataset2*), puis définir les ports de sortie dans le fichier CustomAddRows.xml comme suit :

    <Ports> 
        <Output id="dataset" name="Dataset Out" type="DataTable"> 
            <Description>New Dataset</Description> 
        </Output> 
        <Output id="dataset1_out" name="Dataset 1 Out" type="DataTable"> 
            <Description>First Dataset</Description> 
        </Output> 
        <Output id="dataset2_out" name="Dataset 2 Out" type="DataTable"> 
            <Description>Second Dataset</Description> 
        </Output> 
        <Input id="dataset1" name="Dataset 1" type="DataTable"> 
            <Description>First Input Table</Description>
        </Input> 
        <Input id="dataset2" name="Dataset 2" type="DataTable"> 
            <Description>Second Input Table</Description> 
        </Input> 
    </Ports> 


Et renvoyer la liste des objets dans une liste dans l’ordre correct dans 'CustomAddRows.R' :

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) { 
        if (swap) { dataset <- rbind(dataset2, dataset1)) } 
        else { dataset <- rbind(dataset1, dataset2)) 
        } 
    return (list(dataset, dataset1, dataset2)) 
    } 
    
**Sortie de visualisation :** Vous pouvez également spécifier un port de sortie de type de *visualisation*, qui affiche la sortie dans le périphérique d’affichage R et la sortie de la console. Ce port ne fait pas partie de la sortie de la fonction R et n’interfère pas avec l’ordre des autres types de ports de sortie. Pour ajouter un port de visualisation pour les modules personnalisés, ajoutez un élément de **sortie** avec une valeur de *visualisation* pour l’attribut **type** :

    <Output id="deviceOutput" name="View Port" type="Visualization">
      <Description>View the R console graphics device output.</Description>
    </Output>
    
**Règles de sortie :**

* La valeur de l’attribut **id** de l’élément de **sortie** doit être un nom de variable valide R.
* La valeur de l’attribut **id** de l’élément de **sortie** ne doit pas dépasser 32 caractères.
* La valeur de l’attribut **name** de l’élément de **sortie** ne doit pas dépasser 64 caractères.
* La valeur de l’attribut **type** de l’élément de **sortie** doit être de *visualisation*.

### <a name="arguments"></a>Arguments
Données supplémentaires peuvent être passées à la fonction R via les paramètres de module qui sont définis dans l’élément **d’Arguments** . Ces paramètres s’affichent dans le volet Propriétés de plus à droite de l’interface utilisateur de formation Machine lorsque le module est sélectionné. Les arguments peuvent être des types pris en charge, ou vous pouvez créer une énumération personnalisée lorsque cela est nécessaire. Similaire aux éléments de **Ports** , les éléments **d’Arguments** peuvent avoir un élément **Description** facultatif qui spécifie le texte qui s’affiche lorsque vous placez la souris sur le nom du paramètre.
Les propriétés facultatives pour un module, comme defaultValue, minValue et maxValue peuvent être ajoutées à n’importe quel argument en tant qu’attributs à un élément de **Propriétés** . Les propriétés valides pour l’élément **Propriétés** dépendent du type d’argument et sont décrites avec les types d’arguments pris en charge dans la section suivante. Les arguments de la propriété **isOptional** a la valeur **« true »** ne nécessitent pas l’utilisateur à entrer une valeur. Si une valeur n’est pas fournie à l’argument, l’argument n’est pas passé à la fonction de point d’entrée. Arguments facultatifs de la fonction de point d’entrée doivent être explicitement gérées par la fonction, par exemple, assignée une valeur par défaut NULL dans la définition de fonction de point d’entrée. Un argument facultatif ne fera qu’appliquer les autres contraintes argument, par exemple, min ou max, si une valeur est fournie par l’utilisateur.
Comme c’est le cas avec les entrées et les sorties, il est essentiel que chacun des paramètres ont des valeurs de numéro d’identification unique associés. Dans notre exemple de démarrage rapide, le paramètre d’id/associé a été *swap*.

### <a name="arg-element"></a>Élément Arg
Un paramètre de module est défini à l’aide de l’élément enfant **Arg** de la section **Arguments** de fichier de définition XML. Comme avec les éléments enfants dans la section **Ports** , l’ordre des paramètres dans la section **Arguments** définit la disposition rencontrée dans l’expérience utilisateur. Les paramètres apparaissent à partir du haut vers le bas dans l’interface utilisateur dans le même ordre que celui dans lequel ils sont définis dans le fichier XML. Les types pris en charge par l’apprentissage automatique pour les paramètres sont répertoriés ici. 

**int** : un paramètre de type (32 bits) entier.

    <Arg id="intValue1" name="Int Param" type="int">
        <Properties min="0" max="100" default="0" />
        <Description>Integer Parameter</Description>
    </Arg>


* *Propriétés facultatives*: **min**, **max**, la **valeur par défaut** et **isOptional**

**double** – un paramètre de type double.

    <Arg id="doubleValue1" name="Double Param" type="double">
        <Properties min="0.000" max="0.999" default="0.3" />
        <Description>Double Parameter</Description>
    </Arg>


* *Propriétés facultatives*: **min**, **max**, la **valeur par défaut** et **isOptional**

**bool** : un paramètre booléen qui est représenté par une case à cocher-UX.

    <Arg id="boolValue1" name="Boolean Param" type="bool">
        <Properties default="true" />
        <Description>Boolean Parameter</Description>
    </Arg>



* *Propriétés facultatives*: **par défaut** - false si non définie

**chaîne**: une chaîne standard

    <Arg id="stringValue1" name="My string Param" type="string">
        <Properties isOptional="true" />
        <Description>String Parameter 1</Description>
    </Arg>  

* *Propriétés facultatives*: **isOptional** et **par défaut**

**ColumnPicker**: un paramètre de sélection de colonne. Ce type s’affiche dans l’expérience utilisateur sous la forme d’un sélecteur de colonne. L’élément de **propriété** est utilisé ici pour spécifier l’id du port à partir de laquelle les colonnes sont sélectionnées, où le type de port cible doit être de *DataTable*. Le résultat de la sélection de colonne est passé à la fonction R sous la forme d’une liste de chaînes contenant les noms de colonne sélectionnés. 

        <Arg id="colset" name="Column set" type="ColumnPicker">   
          <Properties portId="datasetIn1" allowedTypes="Numeric" default="NumericAll"/>
          <Description>Column set</Description>
        </Arg>


* *Propriétés requises*: **portId** - correspond à l’id d’un élément Input de type *DataTable*.
* *Propriétés facultatives*:
    * **allowedTypes** - filtre les types de colonne à partir de laquelle vous pouvez choisir. Les valeurs valides sont les suivantes : 
        *   Numérique
        *   Valeur booléenne
        *   Par catégorie
        *   Chaîne
        *   Étiquette
        *   Fonctionnalité
        *   Score de
        *   Tous les

    * **par défaut** - les sélections par défaut valide pour le sélecteur de colonne sont les suivantes : 
        * Aucun
        * NumericFeature
        * NumericLabel
        * NumericScore
        * NumericAll
        * BooleanFeature
        * BooleanLabel
        * BooleanScore
        * BooleanAll
        * CategoricalFeature
        * CategoricalLabel
        * CategoricalScore
        * CategoricalAll
        * StringFeature
        * StringLabel
        * StringScore
        * StringAll
        * AllLabel
        * AllFeature
        * AllScore
        * Tous les

                                                        
**Liste déroulante**: une liste spécifiée par l’utilisateur énuméré (liste déroulante). Les éléments de la liste déroulante sont spécifiés dans l’élément de **Propriétés** à l’aide **d’un élément** . L' **id** pour chaque **élément** doit être unique et une variable R valide. La valeur du **nom** d’un **élément** sert à la fois le texte qui s’affiche et la valeur qui est passée à la fonction de R.

    <Arg id="color" name="Color" type="DropDown">
      <Properties default="red">
        <Item id="red" name="Red Value"/>
        <Item id="green" name="Green Value"/>
        <Item id="blue" name="Blue Value"/>
      </Properties>
      <Description>Select a color.</Description>
    </Arg>  

* *Propriétés facultatives*:
    * **par défaut** - la valeur de la propriété par défaut doit correspondre à une valeur d’id de l’un des éléments **article** .


### <a name="auxiliary-files"></a>Fichiers auxiliaires

Tout fichier placé dans le fichier ZIP de module personnalisé doit être disponible pour une utilisation au moment de l’exécution. Présente les structures de répertoire sont conservées. Cela signifie que ce fichier works d’approvisionnement les mêmes localement et dans l’exécution de la formation de Machine Azure. 

> [AZURE.NOTE] Notez que tous les fichiers sont extraits vers le répertoire 'src' afin que tous les chemins doivent être ' src /' préfixe.

Par exemple, supposons que vous souhaitez supprimer toutes les lignes avec le NAs du dataset et également de supprimer des lignes en double, avant de les sortir dans CustomAddRows, et que vous avez déjà écrit une fonction R qui effectue cette opération dans un fichier RemoveDupNARows.R :

    RemoveDupNARows <- function(dataFrame) {
        #Remove Duplicate Rows:
        dataFrame <- unique(dataFrame)
        #Remove Rows with NAs:
        finalDataFrame <- dataFrame[complete.cases(dataFrame),]
        return(finalDataFrame)
    }
Vous pouvez source le fichier auxiliaire RemoveDupNARows.R dans la fonction CustomAddRows :

    CustomAddRows <- function(dataset1, dataset2, swap=FALSE) {
        source("src/RemoveDupNARows.R")
            if (swap) { 
                dataset <- rbind(dataset2, dataset1))
            } else { 
                dataset <- rbind(dataset1, dataset2)) 
            } 
        dataset <- removeDupNARows(dataset)
        return (dataset)
    }

Ensuite, téléchargez un fichier zip contenant 'CustomAddRows.R', 'CustomAddRows.xml' et 'RemoveDupNARows.R' sous la forme d’un module personnalisé de R.


## <a name="execution-environment"></a>Environnement d’exécution

L’environnement d’exécution du script de R utilise la même version de R en tant que le module **d’Exécution de Script de R** et pouvez utiliser les mêmes packages par défaut. Vous pouvez également ajouter des packages de R supplémentaires à votre module personnalisé en les incluant dans le paquet zip module personnalisé. Simplement les charger dans votre script de R comme vous le feriez dans votre propre environnement R. 

**Limitations de l’environnement d’exécution** sont les suivantes :

* Système de fichiers non persistants : fichiers écrits lors de l’exécution du module personnalisé ne sont pas conservées entre les exécutions du même module.
* Aucun accès réseau



 
