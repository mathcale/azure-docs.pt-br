---
title: incluir arquivo
description: incluir arquivo
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 08/02/2019
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: 3788fdb954917f28f64a0dfe035bed4ded0932d5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "69642130"
---
O gatilho de blob fornece várias propriedades de metadados. Essas propriedades podem ser usadas como parte de expressões de associação em outras associações ou como parâmetros em seu código. Esses valores têm a mesma semântica que o tipo [CloudBlob](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.cloudblob?view=azure-dotnet).

|Propriedade  |Type  |Descrição  |
|---------|---------|---------|
|`BlobTrigger`|`string`|O caminho do blob de gatilho.|
|`Uri`|`System.Uri`|A URI do blob para o local principal.|
|`Properties` |[BlobProperties](https://docs.microsoft.com/dotnet/api/microsoft.azure.storage.blob.blobproperties)|As propriedades do sistema do blob. |
|`Metadata` |`IDictionary<string,string>`|Os metadados definidos pelo usuário para o blob.|

Por exemplo, o script C# e exemplos de JavaScript a seguir registram o caminho para o blob disparando, incluindo o contêiner:

```csharp
public static void Run(string myBlob, string blobTrigger, ILogger log)
{
    log.LogInformation($"Full blob path: {blobTrigger}");
} 
```
