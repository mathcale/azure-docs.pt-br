---
title: Replicação geográfica de um registro
description: Comece a criar e gerenciar um registro de contêiner Azure geo-replicado, que permite que o registro atenda várias regiões com réplicas regionais multi-mestre.
author: stevelas
ms.topic: article
ms.date: 08/16/2019
ms.author: stevelas
ms.openlocfilehash: d238de30e458261a11c941c03ac127c732ca8d3d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74456451"
---
# <a name="geo-replication-in-azure-container-registry"></a>Replicação geográfica no Registro de Contêiner do Azure

As empresas que desejam ter uma presença local ou um backup dinâmico optam por executar os serviços de várias regiões do Azure. Como prática recomendada, colocar um Registro de contêiner em cada região em que as imagens são executadas permite operações perto da rede, possibilitando transferências de camada de imagem rápidas e confiáveis. A replicação geográfica permite que um Registro de contêiner do Azure funcione como um único Registro, atendendo a várias regiões com Registros regionais multimestre. 

Um Registro com replicação geográfica oferece os seguintes benefícios:

* Nomes de marca/imagem/Registro únicos podem ser usados em várias regiões
* Acesso ao Registro perto da rede das implantações regionais
* Nenhuma taxa de saída adicional, uma vez que o pull das imagens são efetuadas de um Registro replicado local na mesma região que seu host de contêiner
* Gerenciamento único de um Registro entre várias regiões

> [!NOTE]
> Se você precisar manter cópias de imagens de contêiner em mais de um registro de contêiner do Azure, o Registro de Contêiner do Azure também dará suporte à [importação de imagens](container-registry-import-images.md). Por exemplo, em um fluxo de trabalho de DevOps, você pode importar uma imagem de um registro de desenvolvimento para um registro de produção, sem a necessidade de usar comandos do Docker.
>

## <a name="example-use-case"></a>Caso de uso de exemplo
A Contoso executa um site com presença pública localizado nos EUA, Canadá e Europa. Para atender a esses mercados com conteúdo local e perto da rede, a Contoso executa os clusters do [Serviço de Kubernetes do Azure](/azure/aks/) (AKS) no Oeste dos EUA, Leste dos EUA, Canadá Central e Europa Ocidental. O aplicativo do site, implantado como uma imagem do Docker, utiliza os mesmos código e imagem em todas as regiões. O conteúdo, local para essa região, é recuperado de um banco de dados, provisionado exclusivamente em cada região. Cada implantação regional tem sua configuração exclusiva para recursos como o banco de dados local.

A equipe de desenvolvimento está localizada em Seattle WA, utilizando o data center do Oeste dos EUA.

![Efetuando push para vários Registros](media/container-registry-geo-replication/before-geo-replicate.png)<br />*Efetuando push para vários Registros*

Antes de usar os recursos de replicação geográfica, a Contoso tinha um Registro no Oeste dos EUA, com um Registro adicional na Europa Ocidental. Para atender a essas regiões diferentes, a equipe de desenvolvimento precisava efetuar push de imagens para dois registros diferentes.

```bash
docker push contoso.azurecr.io/public/products/web:1.2
docker push contosowesteu.azurecr.io/public/products/web:1.2
```
![Efetuando pull de vários Registros](media/container-registry-geo-replication/before-geo-replicate-pull.png)<br />*Efetuando pull de vários Registros*

Os desafios comuns de vários Registros incluem:

* Todos os clusters do Leste dos EUA, Oeste dos EUA e Canadá Central efetuam push do Registro do Oeste dos EUA, incorrendo taxas de saída à medida que esses hosts do contêiner remoto efetuam pull de imagens de data centers do Oeste dos EUA.
* A equipe de desenvolvimento deve efetuar push de imagens para Registros do Oeste dos EUA e da Europa Ocidental.
* A equipe de desenvolvimento deve configurar e manter cada implantação regional com nomes de imagem que referenciam o Registro local.
* O acesso ao Registro deve ser configurado para cada região.

## <a name="benefits-of-geo-replication"></a>Benefícios da replicação geográfica

![Efetuando pull de um Registro com replicação geográfica](media/container-registry-geo-replication/after-geo-replicate-pull.png)

Usando o recurso de replicação geográfica do Registro de Contêiner do Azure, estes benefícios são realizados:

* Gerenciar um único Registro em todas as regiões:`contoso.azurecr.io`
* Gerenciar uma única configuração de implantações de imagem, porque todas as regiões usavam a mesma URL de imagem:`contoso.azurecr.io/public/products/web:1.2`
* Enviar por push para um único registro, enquanto o ACR gerencia a replicação geográfica. Você pode configurar [webhooks](container-registry-webhook.md) regionais para receber notificações sobre eventos em réplicas específicas.

## <a name="configure-geo-replication"></a>Configurar a replicação geográfica

Configurar a replicação geográfica é tão fácil quanto clicar em regiões em um mapa. Você também pode gerenciar a replicação geográfica usando ferramentas, incluindo os comandos [de replicação az acr](/cli/azure/acr/replication) no Cli do Azure, ou implantar um registro habilitado para replicação geográfica com um [modelo do Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-container-registry-geo-replication).

A replicação geográfica é um recurso de [Registros Premium](container-registry-skus.md) somente. Se seu Registro ainda não é Premium, é possível alterar de Básico e Standard para Premium no [Portal do Azure](https://portal.azure.com):

![Alternando SKUs no Portal do Azure](media/container-registry-skus/update-registry-sku.png)

Para configurar a replicação geográfica para o registro do Premium, faça logon no Portal do Azure em https://portal.azure.com.

Navegue até o Registro de Contêiner do Azure e selecione **Replicações**:

![Replicações na interface do usuário de Registro de contêiner do Portal do Azure](media/container-registry-geo-replication/registry-services.png)

Um mapa é exibido mostrando todas as regiões atuais do Azure:

 ![Mapa de região no Portal do Azure](media/container-registry-geo-replication/registry-geo-map.png)

* Hexágonos azuis representam réplicas atuais
* Hexágonos verdes representam regiões de réplica possíveis
* Hexágonos cinza representam regiões do Azure ainda não disponíveis para replicação

Para configurar uma réplica, selecione um hexágono verde e, em seguida, **Criar**:

 ![Criar a interface do usuário de replicação no Portal do Azure](media/container-registry-geo-replication/create-replication.png)

Para configurar réplicas adicionais, selecione os hexágonos verdes para outras regiões e, em seguida, clique em **Criar**.

O ACR começa a sincronizar imagens em réplicas configurados. Depois de concluído, o portal reflete *Pronto*. O status da réplica no portal não é atualizado automaticamente. Use o botão Atualizar para ver o status atualizado.

## <a name="considerations-for-using-a-geo-replicated-registry"></a>Considerações sobre o uso de um registro com replicação geográfica

* Cada região em um registro com replicação geográfica é independente após a configuração. Os SLAs de Registro de Contêiner do Azure se aplicam a cada região geográfica replicada.
* Quando você envia imagens por push ou pull de um registro com replicação geográfica, o Gerenciador de Tráfego do Azure em segundo plano envia a solicitação para o registro localizado na região mais próxima de você.
* Depois que você envia uma atualização de imagem ou marca por push para a região mais próxima, demora algum tempo até o Registro de Contêiner do Azure replicar as camadas e manifestos para as demais regiões que você aceitou. As imagens maiores demoram mais tempo para replicar do que as menores. As imagens e marcas são sincronizadas em todas as regiões de replicação com um modelo de consistência eventual.
* Para gerenciar fluxos de trabalho que dependem de atualizações de push para uma geo-replicada, recomendamos que você configure [webhooks](container-registry-webhook.md) para responder aos eventos push. Você pode configurar webhooks regionais dentro de um registro com replicação geográfica para acompanhar eventos por push, conforme eles são concluídos em todas as regiões com replicação geográfica.

## <a name="delete-a-replica"></a>Excluir uma réplica

Depois de configurar uma réplica para o seu registro, você pode excluí-la a qualquer momento se ela não for mais necessária. Exclua uma réplica usando o portal Azure ou outras ferramentas, como o comando [az acr replication delete](/cli/azure/acr/replication#az-acr-replication-delete) no Azure CLI.

Para excluir uma réplica no portal Azure:

1. Navegue até o registro de contêineres do Azure e selecione **Replicações**.
1. Selecione o nome de uma réplica e **selecione Excluir**. Confirme se deseja excluir a réplica.

> [!NOTE]
> Você não pode excluir a réplica de registro na *região inicial* do registro, ou seja, o local onde você criou o registro. Você só pode excluir a réplica de casa excluindo o próprio registro.

## <a name="geo-replication-pricing"></a>Preços da replicação geográfica

A replicação geográfica é um recurso do [SKU Premium](container-registry-skus.md) do Registro de Contêiner do Azure. Quando você replica um Registro para as regiões desejadas, incorre em taxas de Registro do Premium para cada região.

No exemplo anterior, a Contoso consolidou dois registros inoperantes em um, adicionado réplicas ao Oeste dos EUA, Canadá Central e Europa Ocidental. A Contoso pagaria o Premium quatro vezes por mês, sem nenhuma configuração ou gerenciamento adicional. Agora cada região efetua pull de suas imagens localmente, melhorando o desempenho e a confiabilidade sem taxas de saída de rede do Oeste dos EUA para o Canadá e o Leste dos EUA.

## <a name="troubleshoot-push-operations-with-geo-replicated-registries"></a>Solução de problemas de operações de push com registros com replicação geográfica
 
Um cliente do Docker que efetua push de uma imagem para um registro com replicação geográfica pode não efetuar push de todas as camadas de imagem e seu manifesto para uma única região replicada. Isso pode ocorrer porque o Gerenciador de Tráfego do Azure roteia as solicitações de registro para o registro replicado mais próximo da rede. Se o registro tiver duas regiões de replicação *próximas*, as camadas de imagem e o manifesto poderão ser distribuídos para os dois sites, e a operação de push falhará quando o manifesto for validado. Esse problema ocorre devido à maneira como o nome DNS do registro é resolvido em alguns hosts Linux. Esse problema não ocorre no Windows, que fornece um cache DNS do lado do cliente.
 
Se esse problema ocorrer, uma solução será aplicar um cache DNS do lado do cliente, como `dnsmasq`, no host Linux. Isso ajuda a garantir que o nome do registro seja resolvido de forma consistente. Caso esteja usando uma VM do Linux no Azure para efetuar push para um registro, confira as opções em [Opções de resolução de nomes DNS para máquinas virtuais do Linux no Azure](../virtual-machines/linux/azure-dns.md).

Para otimizar a resolução DNS para a réplica mais próxima ao efetuar push de imagens, configure um registro com replicação geográfica nas mesmas regiões do Azure da origem das operações de push ou a região mais próxima ao trabalhar fora do Azure.

## <a name="next-steps"></a>Próximas etapas

Confira a série de tutoriais em três partes, [Replicação geográfica no Registro de Contêiner do Azure](container-registry-tutorial-prepare-registry.md). Percorra a criação de um Registro com replicação geográfica, criando um contêiner e, em seguida, implantando-o com um único comando `docker push` em várias instâncias regionais dos Aplicativos Web para Contêineres.

> [!div class="nextstepaction"]
> [Replicação geográfica no Registro de Contêiner do Azure](container-registry-tutorial-prepare-registry.md)
