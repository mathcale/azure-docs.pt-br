---
title: Visão geral de alertas e monitoramento de notificações do Azure
description: Visão geral de alertas no Azure. Alertas, alertas clássicos e a interface de alertas.
ms.subservice: alerts
ms.topic: conceptual
ms.date: 01/28/2018
ms.openlocfilehash: 7ca77531ed3e1fae8ec297e430597452c7512aea
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79274782"
---
# <a name="overview-of-alerts-in-microsoft-azure"></a>Visão geral dos alertas no Microsoft Azure 

Este artigo descreve o que são alertas, seus benefícios e como começar a usá-los.  

## <a name="what-are-alerts-in-microsoft-azure"></a>O que são os alertas no Microsoft Azure?
Os alertas trabalham de forma proativa, mandando notificações quando encontram condições importante em seus dados de monitoramento. Eles permitem que você identifique e resolva problemas antes que os usuários do seu sistema os percebam. 

Este artigo discute a experiência de alerta unificada no Azure Monitor, que inclui alertas que foram gerenciados anteriormente pelo Log Analytics e pelo Application Insights. A [experiência anterior de alerta](alerts-classic.overview.md) e os tipos de alerta são chamados de *alertas clássicos*. Você pode visualizar essa experiência mais antiga e o tipo de alerta mais antigo selecionando **Exibir alertas clássicos** na parte superior da página de alerta. 

## <a name="overview"></a>Visão geral

O diagrama a seguir representa o fluxo de alertas. 

![Diagrama do fluxo de alerta](media/alerts-overview/Azure-Monitor-Alerts.svg)

As regras de alerta são separadas dos alertas e das ações tomadas quando um alerta dispara. A regra de alerta captura o alvo e os critérios para alertar. A regra de alerta pode estar em estado habilitado ou desabilitado. Os alertas disparam apenas quando estão habilitados. 

A seguir estão os principais atributos de uma regra de alerta:

**Recurso de destino**: Define o escopo e os sinais disponíveis para alerta. Um destino pode ser um recurso do Azure. Destinos de exemplo: uma máquina virtual, uma conta de armazenamento, um conjunto de dimensionamento de máquinas virtuais, um espaço de trabalho do Log Analytics ou um recurso do Application Insights. Para certos recursos (como máquinas virtuais), você pode especificar vários recursos como o alvo da regra de alerta.

**Sinal:** Emitido pelo recurso de destino. Os sinais podem ser dos seguintes tipos: métrica, registro de atividade, insights de aplicativos e log.

**Critérios**: Uma combinação de sinal e lógica aplicada em um recurso de destino. Exemplos: 

- Porcentagem de CPU > 70%
- Tempo de resposta do servidor > 4 ms 
- Contagem de resultados de uma consulta de log > 100

**Nome de alerta**: Um nome específico para a regra de alerta configurada pelo usuário.

**Descrição do alerta**: Uma descrição da regra de alerta configurada pelo usuário.

**Gravidade**: A gravidade do alerta após os critérios especificados na regra de alerta é atendida. A gravidade pode variar de 0 a 4.

- Sev 0 = Crítico
- Sev 1 = Erro
- Sev 2 = Aviso
- Sev 3 = Informativo
- Sev 4 = Verbose 

**Ação**: Uma ação específica tomada quando o alerta é disparado. Para obter mais informações, consulte [Grupos de Ação](../../azure-monitor/platform/action-groups.md).

## <a name="what-you-can-alert-on"></a>Sobre o que você pode alertar

Você pode alertar sobre métricas e registros, conforme descrito em [fontes de dados de monitoramento](../../azure-monitor/platform/data-sources.md). Elas incluem, mas sem limitação:

- Valores métricos
- Consultas da pesquisa de logs
- Eventos de log de atividades
- Integridade da plataforma subjacente do Azure
- Testes de disponibilidade do site

Antes, as métricas do Azure Monitor, o Application Insights, o Log Analytics e a Integridade do Serviço tinham recursos de alerta separados. Com o tempo, o Azure aprimorou e combinou a interface de usuário e os diferentes métodos de alerta. Essa consolidação ainda está em processo. Como resultado, ainda há alguns recursos de alerta que não estão no novo sistema de alertas.  

| **Fonte do monitor** | **Tipo de sinal**  | **Descrição** |
|-------------|----------------|-------------|
| Integridade do serviço | Log de atividades  | Sem suporte. Consulte [Criar alertas do log de atividades em notificações de serviço](../../azure-monitor/platform/alerts-activity-log-service-notifications.md).  |
| Application Insights | Testes de disponibilidade na Web | Sem suporte. Consulte [Alertas de teste da Web](../../azure-monitor/app/monitor-web-app-availability.md). Disponível para qualquer site que seja instrumentado para enviar dados ao Application Insights. Receba uma notificação quando a disponibilidade ou capacidade de resposta de um site estiver abaixo das expectativas. |

## <a name="manage-alerts"></a>Gerenciar alertas
Você pode definir o estado de um alerta para especificar onde ele está no processo de resolução. Quando os critérios especificados na regra de alerta são atendidos, um alerta é criado ou acionado, e ele tem um status de *Novo*. É possível alterar o status ao reconhecer um alerta e ao fechá-lo. Todas as alterações de estado são armazenadas no histórico do alerta.

Os seguintes estados de alerta são compatíveis.

| Estado | Descrição |
|:---|:---|
| Novo | O problema acabou de ser detectado e ainda não foi revisto. |
| Confirmado | Um administrador examinou o alerta e começou a trabalhar nele. |
| Fechado | O problema foi resolvido. Depois que um alerta for fechado, será possível reabri-lo, alterando-o para outro estado. |

O *estado de alerta* é diferente e independente da *condição do monitor*. O estado de alerta é definido pelo usuário. A condição do monitor é definida pelo sistema. Quando um alerta é acionado, a condição do monitor do alerta é definida como *acionado*. Quando a condição subjacente que causou o acionamento do alerta desaparece, a condição do monitor é definida como *resolvida*. O estado de alerta não é alterado até que o usuário o altere. Saiba [como alterar o estado dos seus alertas e grupos inteligentes](https://aka.ms/managing-alert-smart-group-states).

## <a name="smart-groups"></a>Grupos inteligentes 

Grupos inteligentes são agregações de alertas baseados em algoritmos de aprendizado de máquina, que podem ajudar a reduzir o ruído de alerta e ajudar na solução de problemas. [Saiba mais sobre Grupos inteligentes](https://aka.ms/smart-groups) e [como gerenciar seus grupos inteligentes](https://aka.ms/managing-smart-groups).


## <a name="alerts-experience"></a>Experiência de alertas 
A página Alertas padrão fornece um resumo dos alertas criados dentro de um determinado intervalo de tempo. Ele exibe os alertas totais para cada gravidade, com colunas que identificam o número total de alertas em cada estado para cada gravidade. Selecione qualquer uma das gravidades para abrir a página [Todos os Alertas](#all-alerts-page) filtrada por tal gravidade.

Alternativamente, você pode [enumerar programicamente as instâncias de alerta geradas em suas assinaturas usando APIs REST](#manage-your-alert-instances-programmatically).

> [!NOTE]
   >  Você só pode acessar os alertas gerados nos últimos 30 dias.

Não mostra nem rastreia alertas clássicos. Você pode alterar as assinaturas ou parâmetros para atualizar a página. 

![Captura de tela da página Alertas](media/alerts-overview/alerts-page.png)

Você pode filtrar essa exibição selecionando valores nos menus suspensos na parte superior da página.

| Coluna | Descrição |
|:---|:---|
| Subscription | Selecione as assinaturas do Azure para as quais deseja visualizar os alertas. Você pode optar opcionalmente por selecionar todas as suas assinaturas. Apenas os alertas aos que você tem acesso nas assinaturas selecionadas estão incluídos na exibição. |
| Resource group | Selecione um único grupo de recursos. Somente alertas com destinos no grupo de recursos selecionado são incluídos na exibição. |
| Intervalo de horas | Apenas os alertas disparados dentro do intervalo de tempo selecionado estão incluídos na exibição. Os valores com suporte são a última hora, as últimas 24 horas, os últimos 7 dias e os últimos 30 dias. |

Selecione os seguintes valores na parte superior da página Alertas para abrir outra página:

| Valor | Descrição |
|:---|:---|
| Total de alertas | O número total de alertas que correspondem aos critérios selecionados. Selecione esse valor para abrir a exibição Todos os Alertas sem filtro. |
| Grupos inteligentes | O número total de grupos inteligentes que foram criados a partir dos alertas que correspondem aos critérios selecionados. Selecione esse valor para abrir a lista de grupos inteligentes na exibição Todos os Alertas.
| Total de regras de alerta | O número total de regras de alerta na assinatura e no grupo de recursos selecionados. Selecione esse valor para abrir a exibição Regras filtrada na assinatura e no grupo de recursos selecionados.


## <a name="manage-alert-rules"></a>Gerenciar regras de alerta
Para mostrar a página **Regras,** **selecione Gerenciar regras de alerta**. A página Regras é um único lugar para gerenciar todas as regras de alerta em suas assinaturas do Azure. Ela lista todas as regras de alerta e estas podem ser classificadas com base em grupos de recursos, recursos de destino, nome da regra ou status. Você também pode editar, ativar ou desativar regras de alerta a partir desta página.  

 ![Captura de tela da página Regras](./media/alerts-overview/alerts-preview-rules.png)


## <a name="create-an-alert-rule"></a>Criar uma regra de alerta
Você pode criar alertas de forma consistente, independentemente do serviço de monitoramento ou tipo de sinal. Todos os alertas disparados e os detalhes relacionados ficam disponíveis em uma mesma página.
 
Veja como criar uma nova regra de alerta:
1. Escolha o _destino_ para o alerta.
1. Selecione o _sinal_ entre os sinais disponíveis para o destino.
1. Especifique a _lógica_ a ser aplicada aos dados do sinal.
 
Esse processo de criação simplificado não exige mais que você conheça a origem de monitoramento ou sinais que tenham suporte antes de selecionar um recurso do Azure. A lista de sinais disponíveis é filtrada automaticamente com base no recurso de destino selecionado. Também com base nesse destino, você será guiado pela definição da lógica da regra de alerta automaticamente.  

Saiba mais sobre como criar regras de alerta em [Criar, exibir e gerenciar alertas usando o Azure Monitor](../../azure-monitor/platform/alerts-metric.md).

Os alertas estão disponíveis em vários serviços de monitoramento do Azure. Para obter informações sobre como e quando usar cada um desses serviços, consulte [Monitorar aplicativos e recursos do Azure](../../azure-monitor/overview.md). 


## <a name="all-alerts-page"></a>Página Todos os Alertas 
Para ver a página **Todos os Alertas,** selecione **Alertas Totais**. Aqui você pode ver uma lista de alertas criados dentro do tempo selecionado. É possível exibir uma lista dos alertas individuais ou uma lista dos grupos inteligentes que contêm os alertas. Selecione a faixa na parte superior da página para alternar entre as exibições.

![Captura de tela da página Todos os Alertas](media/alerts-overview/all-alerts-page.png)

Você pode filtrar a exibição selecionando os seguintes valores nos menus suspensos na parte superior da página:

| Coluna | Descrição |
|:---|:---|
| Subscription | Selecione as assinaturas do Azure para as quais deseja visualizar os alertas. Você pode optar opcionalmente por selecionar todas as suas assinaturas. Apenas os alertas aos que você tem acesso nas assinaturas selecionadas estão incluídos na exibição. |
| Resource group | Selecione um único grupo de recursos. Somente alertas com destinos no grupo de recursos selecionado são incluídos na exibição. |
| Tipo de recurso | Selecione um ou mais tipos de recurso. Somente alertas com destinos do tipo selecionado são incluídos na exibição. Essa coluna somente estará disponível depois que um grupo de recursos for especificado. |
| Recurso | Selecione um recurso. Apenas alertas com esse recurso como um destino são incluídos na exibição. Essa coluna somente estará disponível depois que um tipo de recurso for especificado. |
| Severity | Selecione uma gravidade de alerta ou selecione **Tudo** para incluir alertas de todas as gravidades. |
| Monitorar condição | Selecione uma condição de monitor ou selecione **Tudo** para incluir alertas de todas as condições. |
| Estado de alerta | Selecione um estado de alerta ou selecione **Tudo** para incluir alertas de todos os estados. |
| Monitorar serviço | Selecione um serviço ou selecione **Tudo** para incluir todos os serviços. Apenas alertas criados por regras que usam o serviço como um destino são incluídos. |
| Intervalo de horas | Apenas os alertas disparados dentro do intervalo de tempo selecionado estão incluídos na exibição. Os valores com suporte são a última hora, as últimas 24 horas, os últimos 7 dias e os últimos 30 dias. |

Selecione **Colunas** na parte superior da página para selecionar quais colunas mostrar. 

## <a name="alert-details-page"></a>Página de detalhes de alerta
Quando você seleciona um alerta, esta página fornece detalhes do alerta e permite que você altere seu estado.

![Captura de tela da página detalhes do alerta](media/alerts-overview/alert-detail2.png)

A página de detalhes do Alerta inclui as seguintes seções:

| Seção | Descrição |
|:---|:---|
| Resumo | Exibe as propriedades e outras informações significativas sobre o alerta. |
| Histórico | Lista cada ação realizada pelo alerta e todas as alterações feitas no alerta. Atualmente limitado a alterações de estado. |
| Diagnósticos | Informações sobre o grupo inteligente no qual o alerta está incluído. A *contagem de alerta* refere-se ao número de alertas incluídos no grupo inteligente. Inclui outros alertas no mesmo grupo inteligente que foram criados nos últimos 30 dias, independentemente do filtro de tempo na página da lista de alertas. Selecione um alerta para exibir os detalhes. |

## <a name="role-based-access-control-rbac-for-your-alert-instances"></a>RBAC (Role-based Access Control, controle de acesso baseado em função) para suas instâncias de alerta

O consumo e o gerenciamento de instâncias de alerta exigem que o usuário tenha as funções de RBAC incorporadas de [monitoramento do colaborador](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#monitoring-contributor) ou do leitor de [monitoramento](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#monitoring-reader). Essas funções são suportadas em qualquer escopo do Azure Resource Manager, desde o nível de assinatura até atribuições granulares em um nível de recurso. Por exemplo, se um usuário só tem `ContosoVM1`o monitoramento do acesso do contribuinte `ContosoVM1`para a máquina virtual, esse usuário pode consumir e gerenciar apenas os alertas gerados em .

## <a name="manage-your-alert-instances-programmatically"></a>Gerencie suas instâncias de alerta de forma programática

Você pode querer consultar programáticamente os alertas gerados contra sua assinatura. Isso pode ser para criar visualizações personalizadas fora do portal Azure, ou para analisar seus alertas para identificar padrões e tendências.

Você pode consultar os alertas gerados contra suas assinaturas usando a [API De gerenciamento de alerta](https://aka.ms/alert-management-api) ou usando o Gráfico de Recursos do [Azure](../../governance/resource-graph/overview.md) e a [API REST for Resources](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources).

A API rest do Gráfico de recursos permite consultar instâncias de alerta em escala. Isso é recomendado quando você tem que gerenciar alertas gerados em muitas assinaturas. 

A seguinte solicitação de amostra para a API Rest do Gráfico de Recursos retorna a contagem de alertas dentro de uma assinatura:

```json
{
  "subscriptions": [
    <subscriptionId>
  ],
  "query": "AlertsManagementResources | where type =~ 'Microsoft.AlertsManagement/alerts' | summarize count()"
}
```

Você também pode ver o resultado desta consulta do Resource Graph no portal com o Azure Resource Graph Explorer: [portal.azure.com](https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/AlertsManagementResources%20%7C%20where%20type%20%3D~%20%27Microsoft.AlertsManagement%2Falerts%27%20%7C%20summarize%20count())

Você pode consultar os alertas para seus campos [essenciais.](alerts-common-schema-definitions.md#essentials)

Use a [API DE GERENCIAMENTO DE ALERTA REST](https://aka.ms/alert-management-api) para obter mais informações sobre alertas específicos, incluindo seus campos de contexto de [alerta.](alerts-common-schema-definitions.md#alert-context)

## <a name="next-steps"></a>Próximas etapas

- [Saiba mais sobre os grupos inteligentes](https://aka.ms/smart-groups)
- [Saiba mais sobre grupos de ação](../../azure-monitor/platform/action-groups.md)
- [Gerenciar suas instâncias de alertas no Azure](https://aka.ms/managing-alert-instances)
- [Gerenciar grupos inteligentes](https://aka.ms/managing-smart-groups)
- [Saiba mais sobre preços de alertas do Azure](https://azure.microsoft.com/pricing/details/monitor/)






