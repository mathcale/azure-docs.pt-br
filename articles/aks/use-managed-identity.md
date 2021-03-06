---
title: Use identidades gerenciadas no Serviço Azure Kubernetes
description: Saiba como usar identidades gerenciadas no Azure Kubernetes Service (AKS)
services: container-service
author: saudas
manager: saudas
ms.topic: article
ms.date: 03/10/2019
ms.author: saudas
ms.openlocfilehash: 85efc6d9d203ca06c5f7566376993b4c13950788
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80369964"
---
# <a name="use-managed-identities-in-azure-kubernetes-service"></a>Use identidades gerenciadas no Serviço Azure Kubernetes

Atualmente, um cluster Azure Kubernetes Service (AKS) (especificamente, o provedor de nuvem Kubernetes) requer um *diretor de serviço* para criar recursos adicionais, como balanceadores de carga e discos gerenciados no Azure. Ou você deve fornecer um diretor de serviço ou a AKS cria um em seu nome. Os diretores de serviço normalmente têm uma data de validade. Os clusters eventualmente atingem um estado no qual o principal de serviço deve ser renovado para manter o cluster funcionando. Gerenciar os diretores de serviços adiciona complexidade.

*As identidades gerenciadas* são essencialmente um invólucro em torno dos diretores de serviço, e tornam sua gestão mais simples. Para saber mais, leia sobre [identidades gerenciadas para os recursos do Azure.](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)

A KS cria duas identidades gerenciadas:

- **Identidade gerenciada atribuída ao sistema**: A identidade que o provedor de nuvem Kubernetes usa para criar recursos do Azure em nome do usuário. O ciclo de vida da identidade atribuída ao sistema está ligado ao do cluster. A identidade é excluída quando o cluster é excluído.
- **Identidade gerenciada atribuída pelo usuário**: A identidade usada para autorização no cluster. Por exemplo, a identidade atribuída pelo usuário é usada para autorizar a AKS a usar AcRs (ACRs) ou autorizar o kubelet a obter metadados do Azure.

Os complementos também autenticam usando uma identidade gerenciada. Para cada complemento, uma identidade gerenciada é criada pela AKS e dura para a vida útil do complemento. Para criar e usar seu próprio VNet, endereço IP estático ou disco Azure conectado onde os recursos estão fora do grupo de recursos MC_*, use o PrincipalID do cluster para executar uma atribuição de função. Para obter mais informações sobre a atribuição de papéis, consulte [O acesso do Delegado a outros recursos do Azure](kubernetes-service-principal.md#delegate-access-to-other-azure-resources).

## <a name="before-you-begin"></a>Antes de começar

Você deve ter o seguinte recurso instalado:

- O Azure CLI, versão 2.2.0 ou posterior

## <a name="create-an-aks-cluster-with-managed-identities"></a>Crie um cluster AKS com identidades gerenciadas

Agora você pode criar um cluster AKS com identidades gerenciadas usando os seguintes comandos CLI.

Primeiro, crie um grupo de recursos do Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Em seguida, crie um cluster AKS:

```azurecli-interactive
az aks create -g MyResourceGroup -n MyManagedCluster --enable-managed-identity
```

Uma criação de cluster bem-sucedida usando identidades gerenciadas contém essas informações de perfil principal do serviço:

```json
"servicePrincipalProfile": {
    "clientId": "msi",
    "secret": null
  }
```

Finalmente, obtenha credenciais para acessar o cluster:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name MyManagedCluster
```

O cluster será criado em poucos minutos. Em seguida, você pode implantar suas cargas de trabalho de aplicativos no novo cluster e interagir com ele, assim como você fez com os clusters AKS baseados no principal de serviço.

> [!IMPORTANT]
>
> - Clusters AKS com identidades gerenciadas só podem ser habilitados durante a criação do cluster.
> - Os clusters AKS existentes não podem ser atualizados ou atualizados para habilitar identidades gerenciadas.
