---
title: Atualize a configuração de um cluster independente
description: Aprenda a atualizar a configuração que executa um cluster autônomo do Service Fabric.
author: dkkapur
ms.topic: conceptual
ms.date: 11/09/2018
ms.author: dekapur
ms.openlocfilehash: 8e7e01dac29cb9ba91c83270dac4e46c73b2089e
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75610107"
---
# <a name="upgrade-the-configuration-of-a-standalone-cluster"></a>Atualize a configuração de um cluster independente 

Para qualquer sistema moderno, a capacidade de atualização é fundamental para o sucesso de seu produto a longo prazo. Um cluster do Azure Service Fabric é um recurso que pertence a você. Este artigo descreve como atualizar as definições de configuração do cluster autônomo do Service Fabric.

## <a name="customize-cluster-settings-in-the-clusterconfigjson-file"></a>Personalizar configurações de cluster no arquivo ClusterConfig.json
Os clusters autônomos são configurados através do arquivo *ClusterConfig.json.* Para saber mais sobre as diferentes configurações, consulte [Definições de configuração para um cluster autônomo do Windows](service-fabric-cluster-manifest.md).

Você pode adicionar, atualizar ou remover `fabricSettings` configurações na seção na seção [Propriedades Cluster](./service-fabric-cluster-manifest.md#cluster-properties) em *ClusterConfig.json*. 

Por exemplo, o JSON a seguir adiciona uma nova configuração *MaxDiskQuotaInMB* para a *seção de diagnóstico * em `fabricSettings`:

```json
      {
        "name": "Diagnostics",
        "parameters": [
          {
            "name": "MaxDiskQuotaInMB",
            "value": "65536"
          }
        ]
      }
```

Depois de modificar as configurações no arquivo ClusterConfig.json, [teste a configuração do cluster](#test-the-cluster-configuration) e, em seguida, [atualize a configuração do cluster](#upgrade-the-cluster-configuration) para aplicar as configurações ao cluster. 

## <a name="test-the-cluster-configuration"></a>Testar a configuração de cluster
Antes de iniciar a atualização de configuração, você pode testar seu novo JSON de configuração de cluster executando o seguinte script do PowerShell no pacote autônomo:

```powershell
TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File>
```

Ou use este script:

```powershell
TestConfiguration.ps1 -ClusterConfigFilePath <Path to the new Configuration File> -OldClusterConfigFilePath <Path to the old Configuration File> -FabricRuntimePackagePath <Path to the .cab file which you want to test the configuration against>
```

Algumas configurações não podem ser atualizadas, como pontos finais, nome do cluster, IP de nó, etc. A nova configuração de cluster JSON é testada contra a antiga e lança erros na janela PowerShell se houver um problema.

## <a name="upgrade-the-cluster-configuration"></a>Atualizar a configuração do cluster
Para atualizar a atualização da configuração do cluster, execute [Start-ServiceFabricClusterConfigurationUpgrade](https://docs.microsoft.com/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade). A atualização de configuração é processada domínio de atualização por domínio de atualização.

```powershell
Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
```

## <a name="upgrade-cluster-certificate-configuration"></a>Atualizar configuração de certificado do cluster
Um certificado de cluster é usado para autenticação entre os nós de cluster. A substituição do certificado deve ser executada com muito cuidado, pois a falha bloqueia a comunicação entre os nós de cluster.

Há suporte para quatro opções:  

* Atualização de um certificado: o caminho de atualização é Certificado A (Primário)-> Certificado B (Primário)-> Certificado C (Primário)->....

* Atualização de dois certificados: o caminho de atualização é Certificado A (Primário) - > Certificado A (Primário) e B (Secundário) -> Certificado B (Primário) -> Certificado B (Primário) e C (Secundário) -> Certificado C (Primário)->....

* Atualização do tipo de certificado: configuração de certificados com base em CommonName configuração <-> com base em impressão digital do certificado. Por exemplo, impressão digital do certificado (primária) e a impressão digital B (secundários) -> certificado CommonName C.

* Atualização da impressão digital do emissor do certificado: o caminho de atualização é Certificado CN=A,IssuerThumbprint=IT1 (Principal) -> Certificado CN=A,IssuerThumbprint=IT1,IT2 (Principal) -> Certificado CN=A,IssuerThumbprint=IT2 (Principal).


## <a name="next-steps"></a>Próximas etapas
* Saiba como personalizar algumas das [configurações de cluster do Service Fabric](service-fabric-cluster-fabric-settings.md).
* Saiba como [reduzir e escalar horizontalmente seu cluster](service-fabric-cluster-scale-up-down.md).
* Saiba mais sobre [atualizações de aplicativo](service-fabric-application-upgrade.md).

<!--Image references-->
[getfabversions]: ./media/service-fabric-cluster-upgrade-windows-server/getfabversions.PNG
