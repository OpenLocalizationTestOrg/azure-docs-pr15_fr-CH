<properties 
    pageTitle="Comment faire pour effectuer une diffusion en continu live avec des codeurs sur site à l’aide de .NET | Microsoft Azure" 
    description="Cette rubrique montre comment utiliser .NET pour effectuer l ' encodage live avec les codeurs sur place." 
    services="media-services" 
    documentationCenter="" 
    authors="Juliako" 
    manager="erikre" 
    editor=""/>

<tags 
    ms.service="media-services" 
    ms.workload="media" 
    ms.tgt_pltfrm="na" 
    ms.devlang="ne" 
    ms.topic="article" 
    ms.date="08/31/2016"  
    ms.author="cenkdin;juliako"/>

#<a name="how-to-perform-live-streaming-with-on-premise-encoders-using-net"></a>Comment faire pour effectuer une diffusion en continu live avec des codeurs sur site à l’aide de .NET

> [AZURE.SELECTOR]
- [Portail]( media-services-portal-live-passthrough-get-started.md)
- [.NET]( media-services-dotnet-live-encode-with-onpremises-encoders.md)
- [RESTE]( https://msdn.microsoft.com/library/azure/dn783458.aspx)

Ce didacticiel vous guide à travers les étapes de l’utilisation du SDK de .NET Azure Media Services pour créer un **canal** qui est configuré pour une livraison directe. 


##<a name="prerequisites"></a>Conditions préalables

Les éléments suivants sont nécessaires pour terminer le didacticiel :

- Un compte Azure.
- Un compte de Services de support. Pour créer un compte de Services de support, reportez-vous à la section [comment créer un compte de Services de support](media-services-portal-create-account.md).
- Configurer votre environnement de développement. Pour plus d’informations, voir [configurer votre environnement](media-services-set-up-computer.md).
- Une webcam. Par exemple, [le codeur Telestream Wirecast](http://www.telestream.net/wirecast/overview.htm).

Recommandé de consulter les articles suivants :

- [Services multimédias Azure RTMP prennent en charge et codeurs en direct](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/)
- [Live en continu avec des codeurs sur site qui créent des flux de multi-débit](media-services-live-streaming-with-onprem-encoders.md)


##<a name="example"></a>Exemple

L’exemple de code suivant montre comment effectuer les tâches suivantes :

- Se connecter à des Services multimédias
- Création d’un canal
- Mise à jour du canal
- Récupérer un point de terminaison du canal d’entrée. Le point de terminaison d’entrée doit être fournie à l’encodeur live sur site. Les signaux de convertit codeur direct de la caméra dans des flux qui sont envoyés à l’entrée du canal (acquisition) point de terminaison.
- Récupérer le point de terminaison du canal aperçu
- Créer et lancer un programme
- Créer un localisateur nécessaire pour accéder au programme
- Créer et lancer une StreamingEndpoint
- Mise à jour du point de terminaison en continu
- Obtenir des localisateurs pour tous vos points de terminaison en continu
- Arrêt des ressources

Pour plus d’informations sur la configuration d’un codeur live, consultez [prise en charge du protocole RTMP Azure Media Services et les codeurs de Live](https://azure.microsoft.com/blog/2014/09/18/azure-media-services-rtmp-support-and-live-encoders/).
                
    using System;
    using System.Collections.Generic;
    using System.Configuration;
    using System.IO;
    using System.Linq;
    using System.Net;
    using System.Security.Cryptography;
    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.WindowsAzure.MediaServices.Client;
    using Newtonsoft.Json.Linq;
    
    namespace AMSLiveTest
    {
        class Program
        {
            private const string StreamingEndpointName = "streamingendpoint001";
            private const string ChannelName = "channel001";
            private const string AssetlName = "asset001";
            private const string ProgramlName = "program001";
            
            // Read values from the App.config file.
            private static readonly string _mediaServicesAccountName =
            ConfigurationManager.AppSettings["MediaServicesAccountName"];
            private static readonly string _mediaServicesAccountKey =
            ConfigurationManager.AppSettings["MediaServicesAccountKey"];
            
            // Field for service context.
            private static CloudMediaContext _context = null;
            private static MediaServicesCredentials _cachedCredentials = null;
            
            static void Main(string[] args)
            {
                // Create and cache the Media Services credentials in a static class variable.
                _cachedCredentials = new MediaServicesCredentials(
                _mediaServicesAccountName,
                _mediaServicesAccountKey);
                // Used the cached credentials to create CloudMediaContext.
                _context = new CloudMediaContext(_cachedCredentials);
                
                IChannel channel = CreateAndStartChannel();
                
                // Set the Live Encoder to point to the channel's input endpoint:
                string ingestUrl = channel.Input.Endpoints.FirstOrDefault().Url.ToString();
                
                // Use the previewEndpoint to preview and verify
                // that the input from the encoder is actually reaching the Channel.
                string previewEndpoint = channel.Preview.Endpoints.FirstOrDefault().Url.ToString();
                
                IProgram program = CreateAndStartProgram(channel);
                ILocator locator = CreateLocatorForAsset(program.Asset, program.ArchiveWindowLength);
                IStreamingEndpoint streamingEndpoint = CreateAndStartStreamingEndpoint();
                GetLocatorsInAllStreamingEndpoints(program.Asset);
                
                // Once you are done streaming, clean up your resources.
                Cleanup(streamingEndpoint, channel);
            }
            
            public static IChannel CreateAndStartChannel()
            {
                //If you want to change the Smooth fragments to HLS segment ratio, you would set the ChannelCreationOptions’s Output property.
                
                IChannel channel = _context.Channels.Create(
                new ChannelCreationOptions
                {
                Name = ChannelName,
                Input = CreateChannelInput(),
                Preview = CreateChannelPreview()
                });
                
                //Starting and stopping Channels can take some time to execute. To determine the state of operations after calling Start or Stop, query the IChannel.State .
                
                channel.Start();
                
                return channel;
            }
            
            private static ChannelInput CreateChannelInput()
            {
                return new ChannelInput
                {
                    StreamingProtocol = StreamingProtocol.RTMP,
                    AccessControl = new ChannelAccessControl
                    {
                        IPAllowList = new List<IPRange>
                                {
                                new IPRange
                            {
                                Name = "TestChannelInput001",
                                // Setting 0.0.0.0 for Address and 0 for SubnetPrefixLength
                                // will allow access to IP addresses.
                                Address = IPAddress.Parse("0.0.0.0"),
                                SubnetPrefixLength = 0
                            }
                        }
                    }
                };
            }
    
            private static ChannelPreview CreateChannelPreview()
            {
                return new ChannelPreview
                {
                    AccessControl = new ChannelAccessControl
                    {
                        IPAllowList = new List<IPRange>
                        {
                            new IPRange
                            {
                                Name = "TestChannelPreview001",
                                // Setting 0.0.0.0 for Address and 0 for SubnetPrefixLength
                                // will allow access to IP addresses.
                                Address = IPAddress.Parse("0.0.0.0"),
                                SubnetPrefixLength = 0
                            }
                        }
                    }
                };
            }
    
            public static void UpdateCrossSiteAccessPoliciesForChannel(IChannel channel)
            {
                var clientPolicy =
                    @"<?xml version=""1.0"" encoding=""utf-8""?>
                <access-policy>
                    <cross-domain-access>
                        <policy>
                            <allow-from http-request-headers=""*"" http-methods=""*"">
                                <domain uri=""*""/>
                            </allow-from>
                            <grant-to>
                               <resource path=""/"" include-subpaths=""true""/>
                            </grant-to>
                        </policy>
                    </cross-domain-access>
                </access-policy>";
    
                var xdomainPolicy =
                    @"<?xml version=""1.0"" ?>
                <cross-domain-policy>
                    <allow-access-from domain=""*"" />
                </cross-domain-policy>";
    
                channel.CrossSiteAccessPolicies.ClientAccessPolicy = clientPolicy;
                channel.CrossSiteAccessPolicies.CrossDomainPolicy = xdomainPolicy;
    
                channel.Update();
            }
    
            public static IProgram CreateAndStartProgram(IChannel channel)
            {
                IAsset asset = _context.Assets.Create(AssetlName, AssetCreationOptions.None);
    
                // Create a Program on the Channel. You can have multiple Programs that overlap or are sequential;
                // however each Program must have a unique name within your Media Services account.
                IProgram program = channel.Programs.Create(ProgramlName, TimeSpan.FromHours(3), asset.Id);
                program.Start();
    
                return program;
            }
    
            public static ILocator CreateLocatorForAsset(IAsset asset, TimeSpan ArchiveWindowLength)
            {
                // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.            

                var locator = _context.Locators.CreateLocator
                    (
                        LocatorType.OnDemandOrigin,
                        asset,
                        _context.AccessPolicies.Create
                        (
                            "Live Stream Policy",
                            ArchiveWindowLength,
                            AccessPermissions.Read
                        )
                    );
    
                return locator;
            }
    
            public static IStreamingEndpoint CreateAndStartStreamingEndpoint()
            {
                var options = new StreamingEndpointCreationOptions
                {
                    Name = StreamingEndpointName,
                    ScaleUnits = 1,
                    AccessControl = GetAccessControl(),
                    CacheControl = GetCacheControl()
                };
    
                IStreamingEndpoint streamingEndpoint = _context.StreamingEndpoints.Create(options);
                streamingEndpoint.Start();
    
                return streamingEndpoint;
            }
    
            private static StreamingEndpointAccessControl GetAccessControl()
            {
                return new StreamingEndpointAccessControl
                {
                    IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "Allow all",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    },
    
                    AkamaiSignatureHeaderAuthenticationKeyList = new List<AkamaiSignatureHeaderAuthenticationKey>
                    {
                        new AkamaiSignatureHeaderAuthenticationKey
                        {
                            Identifier = "My key",
                            Expiration = DateTime.UtcNow + TimeSpan.FromDays(365),
                            Base64Key = Convert.ToBase64String(GenerateRandomBytes(16))
                        }
                    }
                };
            }
    
            private static byte[] GenerateRandomBytes(int length)
            {
                var bytes = new byte[length];
                using (var rng = new RNGCryptoServiceProvider())
                {
                    rng.GetBytes(bytes);
                }
    
                return bytes;
            }
    
            private static StreamingEndpointCacheControl GetCacheControl()
            {
                return new StreamingEndpointCacheControl
                {
                    MaxAge = TimeSpan.FromSeconds(1000)
                };
            }
    
            public static void UpdateCrossSiteAccessPoliciesForStreamingEndpoint(IStreamingEndpoint streamingEndpoint)
            {
                var clientPolicy =
                    @"<?xml version=""1.0"" encoding=""utf-8""?>
                <access-policy>
                    <cross-domain-access>
                        <policy>
                            <allow-from http-request-headers=""*"" http-methods=""*"">
                                <domain uri=""*""/>
                            </allow-from>
                            <grant-to>
                               <resource path=""/"" include-subpaths=""true""/>
                            </grant-to>
                        </policy>
                    </cross-domain-access>
                </access-policy>";
    
                var xdomainPolicy =
                    @"<?xml version=""1.0"" ?>
                <cross-domain-policy>
                    <allow-access-from domain=""*"" />
                </cross-domain-policy>";
    
                streamingEndpoint.CrossSiteAccessPolicies.ClientAccessPolicy = clientPolicy;
                streamingEndpoint.CrossSiteAccessPolicies.CrossDomainPolicy = xdomainPolicy;
    
                streamingEndpoint.Update();
            }
    
            public static void GetLocatorsInAllStreamingEndpoints(IAsset asset)
            {
                var locators = asset.Locators.Where(l => l.Type == LocatorType.OnDemandOrigin);
                var ismFile = asset.AssetFiles.AsEnumerable().FirstOrDefault(a => a.Name.EndsWith(".ism"));
                var template = new UriTemplate("{contentAccessComponent}/{ismFileName}/manifest");
                var urls = locators.SelectMany(l =>
                            _context
                                .StreamingEndpoints
                                .AsEnumerable()
                                .Where(se => se.State == StreamingEndpointState.Running)
                                .Select(
                                    se =>
                                        template.BindByPosition(new Uri("http://" + se.HostName),
                                        l.ContentAccessComponent,
                                            ismFile.Name)))
                            .ToArray();
    
            }
    
            public static void Cleanup(IStreamingEndpoint streamingEndpoint,
                                        IChannel channel)
            {
                if (streamingEndpoint != null)
                {
                    streamingEndpoint.Stop();
                    streamingEndpoint.Delete();
                }
    
                IAsset asset;
                if (channel != null)
                {
    
                    foreach (var program in channel.Programs)
                    {
                        asset = _context.Assets.Where(se => se.Id == program.AssetId)
                                                .FirstOrDefault();
    
                        program.Stop();
                        program.Delete();
    
                        if (asset != null)
                        {
                            foreach (var l in asset.Locators)
                                l.Delete();
    
                            asset.Delete();
                        }
                    }
    
                    channel.Stop();
                    channel.Delete();
                }
            }
        }
    }

##<a name="next-step"></a>Étape suivante

Révision Media Services, voies de formation

[AZURE.INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

##<a name="provide-feedback"></a>Fournir des commentaires

[AZURE.INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]
