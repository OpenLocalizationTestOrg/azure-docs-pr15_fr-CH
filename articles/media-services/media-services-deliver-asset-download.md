<properties 
    pageTitle="Télécharger des ressources multimédias" 
    description="Obtenir des informations sur le point de télécharger actifs sur votre ordinateur. Exemples de code sont écrits en C# et utilisent le Kit de développement de Services de support pour .NET." 
    services="media-services" 
    documentationCenter="" 
    authors="juliako" 
    manager="erikre" 
    editor=""/>

<tags 
    ms.service="media-services" 
    ms.workload="media" 
    ms.tgt_pltfrm="na" 
    ms.devlang="na" 
    ms.topic="article"
    ms.date="09/26/2016"
    ms.author="juliako"/>

#<a name="how-to-deliver-an-asset-by-download"></a>Comment : fournir une immobilisation par téléchargement

Cette rubrique traite des options pour la livraison de ressources multimédias téléchargés vers les Services de support. Vous pouvez fournir un contenu de Media Services dans nombreux scénarios d’applications. Vous pouvez télécharger les ressources multimédias, ou d’y accéder à l’aide d’un repère. Vous pouvez envoyer le contenu multimédia vers une autre application ou à un autre fournisseur de contenu. Pour améliorer les performances et l’évolutivité, vous pouvez également fournir le contenu à l’aide d’un réseau de livraison de contenu (CDN).

Cet exemple montre comment télécharger des ressources multimédias à partir des Services de support sur votre ordinateur local. Le code interroge les travaux associés avec le compte de Services de support par ID de tâche et accède à la collection **OutputMediaAssets** (qui est l’ensemble d’un ou plusieurs actifs de média de sortie qui résulte de l’exécution d’une tâche). Cet exemple montre comment télécharger des contenus multimédia de sortie à partir d’un projet, mais vous pouvez appliquer la même approche pour télécharger d’autres actifs.

    
    // Download the output asset of the specified job to a local folder.
    static IAsset DownloadAssetToLocal( string jobId, string outputFolder)
    {
        // This method illustrates how to download a single asset. 
        // However, you can iterate through the OutputAssets
        // collection, and download all assets if there are many. 
    
        // Get a reference to the job. 
        IJob job = GetJob(jobId);
    
        // Get a reference to the first output asset. If there were multiple 
        // output media assets you could iterate and handle each one.
        IAsset outputAsset = job.OutputMediaAssets[0];
    
        // Create a SAS locator to download the asset
        IAccessPolicy accessPolicy = _context.AccessPolicies.Create("File Download Policy", TimeSpan.FromDays(30), AccessPermissions.Read);
        ILocator locator = _context.Locators.CreateLocator(LocatorType.Sas, outputAsset, accessPolicy);
    
        BlobTransferClient blobTransfer = new BlobTransferClient
        {
            NumberOfConcurrentTransfers = 20,
            ParallelTransferThreadCount = 20
        };
    
        var downloadTasks = new List<Task>();
        foreach (IAssetFile outputFile in outputAsset.AssetFiles)
        {
            // Use the following event handler to check download progress.
            outputFile.DownloadProgressChanged += DownloadProgress;
    
            string localDownloadPath = Path.Combine(outputFolder, outputFile.Name);
    
            Console.WriteLine("File download path:  " + localDownloadPath);
    
            downloadTasks.Add(outputFile.DownloadAsync(Path.GetFullPath(localDownloadPath), blobTransfer, locator, CancellationToken.None));
    
            outputFile.DownloadProgressChanged -= DownloadProgress;
        }
    
        Task.WaitAll(downloadTasks.ToArray());
    
        return outputAsset;
    }
    
    static void DownloadProgress(object sender, DownloadProgressChangedEventArgs e)
    {
        Console.WriteLine(string.Format("{0} % download progress. ", e.Progress));
    }



##<a name="media-services-learning-paths"></a>Cursus Media Services

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##<a name="provide-feedback"></a>Fournir des commentaires

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

   
##<a name="see-also"></a>Voir aussi 

[Fournir un contenu de diffusion en continu](media-services-deliver-streaming-content.md)

