---
title: Práticas recomendadas do operador – recursos do agendador avançado no Serviço de Kubernetes do Azure (AKS)
description: Conheça as práticas recomendadas de operador do cluster para usar recursos avançados de agendador como taints and tolerations, seletores de nó e afinidade, ou afinidade entre pods e antiafinidade no Serviço de Kubernetes do Azure (AKS)
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: 546c1d6ae25a33c6df93469ccf8c230b4b1c474b
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79252890"
---
# <a name="best-practices-for-advanced-scheduler-features-in-azure-kubernetes-service-aks"></a>Práticas recomendadas para os recursos do agendador avançado no Serviço de Kubernetes do Azure (AKS)

À medida que você gerencia clusters no Serviço de Kubernetes do Azure (AKS), geralmente é necessário isolar equipes e cargas de trabalho. O agendador Kubernetes fornece recursos avançados que permitem controlar quais pods podem ser agendados em determinados nódulos ou como aplicativos multi-pod podem ser distribuídos adequadamente em todo o cluster. 

Este artigo sobre práticas recomendadas se concentra em recursos de agendamento de Kubernetes avançados para os operadores do cluster. Neste artigo, você aprenderá como:

> [!div class="checklist"]
> * Use taints e tolerations para limitar quais pods podem ser agendados em nós
> * Dê preferência ao pods para executar em determinados nós com seletores de nó ou afinidade de nó
> * Dividir a distância ou pods juntos de grupo com a afinidade entre pods ou antiafinidade

## <a name="provide-dedicated-nodes-using-taints-and-tolerations"></a>Fornecer nós dedicados usando taints e tolerations

**Diretrizes de práticas recomendadas** - limitar o acesso para aplicativos de uso intensivo de recursos, como controladores de entrada a nós específicos. Manter os recursos de nó disponíveis para cargas de trabalho os exigem e não permitir o agendamento de outras cargas de trabalho em nós.

Quando você cria um cluster do AKS, você pode implantar nós com suporte de GPU ou um grande número de CPUs avançadas. Esses nós geralmente são usados para cargas de trabalho de processamento de dados grandes, como aprendizado de máquina (ML) ou a inteligência artificial (AI). Como esse tipo de hardware geralmente é um recurso de nó caro de implantar, limite as cargas de trabalho que podem ser agendadas em nós. Em vez disso, talvez você queira dedicar alguns nós do cluster para executar serviços de entrada e impedir outras cargas de trabalho.

Este suporte para diferentes nós é fornecido usando vários grupos de nó. Um cluster AKS fornece uma ou mais piscinas de nós.

O Agendador Kubernetes pode usar taints e tolerations para restringir quais cargas de trabalho podem ser executados em nós.

* Um **taint** é aplicado a um nó que indica que apenas os pods específicos podem ser agendados neles.
* Um **toleration**, em seguida, é aplicado a um pod que lhes permite *tolerar* um taint de nó.

Quando você implanta um pod em um cluster do AKS, os Kubernetes apenas agendam pods em nós onde um toleration é alinhado com o taint. Como exemplo, suponha que você tenha um pool de nó suscitado pelo cluster AKS para nomes com suporte a GPU. Você define o nome, como *gpu*, em seguida, um valor para o agendamento. Se você definir esse valor como *NoSchedule*, o Agendador Kubernetes não poderá agendar pods no nó, se o pod não definir o toleration apropriado.

```console
kubectl taint node aks-nodepool1 sku=gpu:NoSchedule
```

Com um taint aplicado a nós, você, em seguida, define um toleration na especificação de pod que permite a programação nos nós. O exemplo a seguir define o `sku: gpu` e `effect: NoSchedule` para tolerar o taint aplicado ao nó na etapa anterior:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Quando esse pod é implantado, como o uso de `kubectl apply -f gpu-toleration.yaml`, os Kubernetes podem agendar com êxito o pod em nós com o taint aplicado. Esse isolamento lógico permite que você controle o acesso a recursos dentro de um cluster.

Ao aplicar taints, trabalhe com seus desenvolvedores de aplicativos e proprietários para permitir que definam os tolerations necessários em suas implantações.

Para obter mais informações sobre taints e tolerations, consulte [Aplicar taints e tolerations][k8s-taints-tolerations].

Para obter mais informações sobre como usar vários pools de nós no AKS, consulte [Criar e gerenciar vários pools de nós para um cluster em AKS][use-multiple-node-pools].

### <a name="behavior-of-taints-and-tolerations-in-aks"></a>Comportamento de manchas e tolerâncias em AKS

Quando você atualiza um pool de nó em AKS, as manchas e tolerâncias seguem um padrão definido à medida que são aplicados a novos nodes:

- **Clusters padrão que usam conjuntos de escala de máquinas virtuais**
  - Vamos supor que você tem um cluster de dois nós - *nó1* e *nó2*. Você atualiza a piscina de nó.
  - Dois nós adicionais são criados, *node3* e *node4*, e as manchas são transmitidas, respectivamente.
  - O *nó original1* e *o nó2* são excluídos.

- **Clusters sem suporte de conjunto de escala de máquina virtual**
  - Novamente, vamos supor que você tem um cluster de dois nós - *nó1* e *nó2*. Quando você atualiza, um nó adicional *(nó3*) é criado.
  - As manchas do *nó1* são aplicadas ao *nó3,* em seguida, *o nó 1* é então excluído.
  - Outro novo nó é criado (chamado *nó1*, uma vez que o *nó anterior1* foi excluído), e as manchas de *nó2* são aplicadas ao novo *nó1*. Em seguida, *o nó2* é excluído.
  - Em *essência, o nó1* torna-se *nó3*, e *o nó2* torna-se *nó1*.

Quando você escala uma piscina de nó em AKS, manchas e tolerâncias não são ressoadas por design.

## <a name="control-pod-scheduling-using-node-selectors-and-affinity"></a>Controle o agendando de pod usando os seletores de nó e afinidade

**Diretrizes de práticas recomendadas** – controle o agendamento de pods em nós usando os seletores de nó, afinidade de nó, ou afinidade entre pods. Essas configurações permitem que o agendador Kubernetes isole logicamente cargas de trabalho, como pelo hardware no nó.

Taints e tolerations são usados para isolar logicamente recursos com um corte de disco rígido - se o pod não tolerar um taint de nó, não é agendado no nó. Uma abordagem alternativa é usar os seletores de nó. Você marca os nós, como para indicar o armazenamento SSD anexado localmente ou uma grande quantidade de memória e, em seguida, define na especificação do pod um seletor de nós. O Kubernetes agenda, em seguida, os pods em um nó correspondente. Ao contrário de tolerations, pods sem um seletor de nó correspondente podem ser agendados em nós rotulados. Esse comportamento permite os recursos não utilizados em nós para consumir, mas dá prioridade a pods que definem o seletor de nó correspondente.

Vamos examinar um exemplo de nós com uma grande quantidade de memória. Esses nós podem dar preferência ao pods que solicitam uma grande quantidade de memória. Para garantir que os recursos não fiquem ocioso, eles também permitem que outros pods sejam executados.

```console
kubectl label node aks-nodepool1 hardware:highmem
```

Uma especificação de pod, em seguida, adiciona a `nodeSelector` propriedade para definir um seletor de nó que corresponde ao rótulo definido em um nó:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
    nodeSelector:
      hardware: highmem
```

Ao usar essas opções de agendador, trabalhe com seus desenvolvedores de aplicativos e proprietários para permitir que eles definam corretamente as especificações de pod.

Para obter mais informações sobre o uso de seletores de nó, consulte [Atribuir Pods a Nós][k8s-node-selector].

### <a name="node-affinity"></a>Afinidade de nó

Um seletor de nó é uma maneira básica para atribuir os pods para um determinado nó. Mais flexibilidade do que está disponível usando a *afinidade de nó*. Com afinidade de nó, você define o que acontece se o pod não puder ser correspondido com um nó. Você pode *exigir* que o agendador Kubernetes corresponda a um pod com um host rotulado. Ou, você pode *preferir* uma correspondência, mas permitir que o pod seja agendado em um host diferente, se nenhuma correspondência estiver disponível.

O exemplo a seguir define a afinidade de nó para *requiredDuringSchedulingIgnoredDuringExecution*. Este afinidade exige o agendamento de Kubernetes para usar um nó com um rótulo correspondente. Se nenhum nó estiver disponível, o pod precisa esperar o agendamento para continuar. Para permitir que o pod seja agendado em um nó diferente, em vez disso, você pode definir o valor *preferredDuringScheduledIgnoreDuringExecution*:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values: highmem
```

A parte *IgnoredDuringExecution* da configuração indica que, se o nó de rótulos mudar, o pod não deve ser removido a partir do nó. O agendador Kubernetes usa apenas os rótulos de nó atualizado para novos pods serem agendados, não os pods já agendados em nós.

Para obter mais informações, consulte [Afinidade e antiafinidade][k8s-affinity].

### <a name="inter-pod-affinity-and-anti-affinity"></a>Afinidade entre pods e antiafinidade

Uma abordagem final para o agendador Kubernetes isolar logicamente as cargas de trabalho está usando a afinidade entre pods ou antiafinidade. As configurações definem esse pods *não devem* ser agendadas em um nó que tem uma correspondência existente pod, ou que eles *devem* ser agendados. Por padrão, o agendador Kubernetes tenta agendar vários pods em uma réplica definida entre os nós. Você pode definir regras mais específicas alternativas para esse comportamento.

Um bom exemplo é um aplicativo web que também usa um Azure Cache para Redis. Você pode usar regras de antiafinidade de pod para solicitar que o agendador Kubernetes distribua réplicas entre os nós. Em seguida, você pode usar regras de afinidade para garantir que cada componente do aplicativo web seja agendado no mesmo host que um cache correspondente. A distribuição de pods entre nós é semelhante ao exemplo a seguir:

| **Nó 1** | **Nó 2** | **Nó 3** |
|------------|------------|------------|
| webapp-1   | webapp-2   | webapp-3   |
| cache-1    | cache-2    | cache-3    |

Este exemplo é uma implantação mais complexa do que o uso de seletores de nó ou afinidade de nó. A implantação fornece a você o controle sobre os pods de agendas Kubernetes em nós e pode isolar os recursos logicamente. Para obter um exemplo completo deste aplicativo web com o exemplo do Azure Cache for Redis, consulte [Co-localizar pods no mesmo nó][k8s-pod-affinity].

## <a name="next-steps"></a>Próximas etapas

Este artigo se concentra nos recursos avançados de agendador Kubernetes. Para obter mais informações sobre operações de cluster no AKS, consulte as seguintes práticas recomendadas:

* [Isolamento de multi locação e cluster][aks-best-practices-scheduler]
* [Recursos básicos de Agendador Kubernetes][aks-best-practices-scheduler]
* [Autenticação e autorização][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-taints-tolerations]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[k8s-node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[k8s-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
[k8s-pod-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#always-co-located-in-the-same-node

<!-- INTERNAL LINKS -->
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[use-multiple-node-pools]: use-multiple-node-pools.md
