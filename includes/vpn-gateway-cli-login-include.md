---
title: incluir arquivo
description: incluir arquivo
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 74d7bd087df4b00c0bafb5ec33fbbdfa5c57b379
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "67171762"
---
Inicie sessão na sua assinatura do Azure com o comando [az login](/cli/azure/) e siga as instruções na tela. Para obter mais informações sobre como conectar-se, confira [Introdução à CLI do Azure](/cli/azure/get-started-with-azure-cli).

```azurecli
az login
```

Se tiver mais de uma assinatura do Azure, liste as assinaturas para a conta.

```azurecli
az account list --all
```

Especifique a assinatura que você deseja usar.

```azurecli
az account set --subscription <replace_with_your_subscription_id>
```
