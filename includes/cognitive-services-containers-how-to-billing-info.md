---
author: IEvangelist
ms.author: dapine
ms.date: 02/19/2020
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 2ac93f5aba722eea78267a512999a5581a887b99
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77474212"
---
Consultas ao contêiner são cobradas pelo tipo de preço do recurso do Azure usado para `ApiKey`.

Os contêineres azure Cognitive Services não são licenciados para funcionar sem estarem conectados ao ponto final de medição/faturamento. Você precisa permitir que os contêineres comuniquem as informações de cobrança com o ponto de extremidade de cobrança em todos os momentos. Os contêineres dos Serviços Cognitivos não enviam dados do cliente, como imagem ou texto que está sendo analisado, para a Microsoft.

### <a name="connect-to-azure"></a>Conectar-se ao Azure

O contêiner precisa dos valores de argumento de cobrança para ser executado. Esses valores permitem que o contêiner se conecte ao ponto de extremidade de cobrança. O contêiner relata o uso a cada 10 a 15 minutos. Se o contêiner não se conectar ao Azure dentro da janela de tempo permitida, ele continuará sendo executado, mas não atenderá a consultas até que o ponto de extremidade de cobrança seja restaurado. Serão realizadas 10 tentativas de conexão no mesmo intervalo de tempo de 10 a 15 minutos. Se ele não puder se conectar ao ponto final de cobrança dentro das 10 tentativas, o contêiner pára de atender solicitações.

### <a name="billing-arguments"></a>Argumentos de cobrança

<a href="https://docs.docker.com/engine/reference/commandline/run/" target="_blank"> `docker run` O <span class="docon docon-navigate-external x-hidden-focus"></span> </a> comando iniciará o contêiner quando todas as três opções forem fornecidas com valores válidos:

| Opção | Descrição |
|--------|-------------|
| `ApiKey` | A chave de API do recurso dos Serviços Cognitivos usado para rastrear informações de cobrança.<br/>O valor dessa opção deve ser definido como uma chave de API para o recurso provisionado especificado em `Billing`. |
| `Billing` | O ponto de extremidade do recurso dos Serviços Cognitivos usado para rastrear informações de cobrança.<br/>O valor dessa opção deve ser definido como o URI do ponto de extremidade de um recurso do Azure provisionado.|
| `Eula` | Indica que você aceitou a licença do contêiner.<br/>O valor dessa opção deve ser definido como **aceitar**. |
