---
title: Amostras de análise de log do Azure Firewall
description: Os logs do Monitor do Azure podem ser usados para analisar o Firewall do Azure. Um arquivo de exemplo é incorporado no View Designer no Azure Monitor.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 01/23/2020
ms.author: victorh
ms.openlocfilehash: bc34afe82c1b73afb5f3d5d1a07f2a5059590146
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76705951"
---
# <a name="azure-firewall-log-analytics-samples"></a>Amostras de análise de log do Azure Firewall

As seguintes amostras de logs do Azure Monitor podem ser usadas para analisar seus registros do Firewall do Azure. O arquivo de exemplo é incorporado no View Designer no Azure Monitor, o artigo [Do View Designer no Azure Monitor](https://docs.microsoft.com/azure/log-analytics/log-analytics-view-designer) tem mais informações sobre o conceito de View Design.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="azure-monitor-logs-view"></a>Exibição de logs do Monitor do Azure

Veja como você pode configurar um exemplo de visualização de logs do Monitor Azure. Você pode baixar a visualização de exemplo do repositório [azure-docs-json-samples](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-firewall/AzureFirewall.omsview). A maneira mais fácil de fazer isso é clicar com o botão direito do mouse no hiperlink nesta página e escolher *Salvar como* e fornecer um nome como **AzureFirewall.omsview**. 

Execute as seguintes etapas para adicionar a exibição ao espaço de trabalho do Log Analytics:

1. Abra o espaço de trabalho do Log Analytics no portal Azure.
2. Abra **Designer de Exibição** abaixo de **Geral**.
3. Clique em **Importar**.
4. Procure e selecione o arquivo **AzureFirewall.omsview** baixado anteriormente.
5. Clique em **Salvar**.

Veja como é o modo de exibição para os dados de log de regra de aplicativo:

![Dados de log da regra de aplicativo](./media/log-analytics-samples/azurefirewall-applicationrulelogstats.png)

E para os dados de log da regra de rede:

![Dados de log da regra de rede]( ./media/log-analytics-samples/azurefirewall-networkrulelogstats.png)

O Azure Firewall registra dados abaixo do AzureDiagnostics com categoria como **AzureFirewallApplicationRule** ou **AzureFirewallNetworkRule**. Os dados que contêm os detalhes são armazenados no campo msg_s. Usando o operador [parse](https://docs.microsoft.com/azure/kusto/query/parseoperator), podemos extrair várias propriedades interessantes do campo msg_s. As consultas a seguir extraem as informações para ambas as categorias.

## <a name="application-rules-log-data-query"></a>Consulta de dados de log das regras de aplicativo

A consulta a seguir analisa os dados de log da regra de aplicativo. Em várias linhas de comentário, há algumas diretrizes sobre como a consulta foi criada:

```Kusto
AzureDiagnostics
| where Category == "AzureFirewallApplicationRule"
//using :int makes it easier to pars but later we'll convert to string as we're not interested to do mathematical functions on these fields
//this first parse statement is valid for all entries as they all start with this format
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " " TempDetails
//case 1: for records that end with: "was denied. Reason: SNI TLS extension was missing."
| parse TempDetails with "was " Action1 ". Reason: " Rule1
//case 2: for records that end with
//"to ocsp.digicert.com:80. Action: Allow. Rule Collection: RC1. Rule: Rule1"
//"to v10.vortex-win.data.microsoft.com:443. Action: Deny. No rule matched. Proceeding with default action"
| parse TempDetails with "to " FQDN ":" TargetPortInt:int ". Action: " Action2 "." *
//case 2a: for records that end with:
//"to ocsp.digicert.com:80. Action: Allow. Rule Collection: RC1. Rule: Rule1"
| parse TempDetails with * ". Rule Collection: " RuleCollection2a ". Rule:" Rule2a
//case 2b: for records that end with:
//for records that end with: "to v10.vortex-win.data.microsoft.com:443. Action: Deny. No rule matched. Proceeding with default action"
| parse TempDetails with * "Deny." RuleCollection2b ". Proceeding with" Rule2b
| extend 
SourcePort = tostring(SourcePortInt)
|extend
TargetPort = tostring(TargetPortInt)
| extend
//make sure we only have Allowed / Deny in the Action Field
Action1 = case(Action1 == "Deny","Deny","Unknown Action")
| extend
    Action = case(Action2 == "",Action1,Action2),
    Rule = case(Rule2a == "",case(Rule1 == "",case(Rule2b == "","N/A", Rule2b),Rule1),Rule2a), 
    RuleCollection = case(RuleCollection2b == "",case(RuleCollection2a == "","No rule matched",RuleCollection2a),RuleCollection2b),
    FQDN = case(FQDN == "", "N/A", FQDN),
    TargetPort = case(TargetPort == "", "N/A", TargetPort)
| project TimeGenerated, msg_s, Protocol, SourceIP, SourcePort, FQDN, TargetPort, Action ,RuleCollection, Rule
```

A mesma consulta em um formato mais condensado:

```Kusto
AzureDiagnostics
| where Category == "AzureFirewallApplicationRule"
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " " TempDetails
| parse TempDetails with "was " Action1 ". Reason: " Rule1
| parse TempDetails with "to " FQDN ":" TargetPortInt:int ". Action: " Action2 "." *
| parse TempDetails with * ". Rule Collection: " RuleCollection2a ". Rule:" Rule2a
| parse TempDetails with * "Deny." RuleCollection2b ". Proceeding with" Rule2b
| extend SourcePort = tostring(SourcePortInt)
| extend TargetPort = tostring(TargetPortInt)
| extend Action1 = case(Action1 == "Deny","Deny","Unknown Action")
| extend Action = case(Action2 == "",Action1,Action2),Rule = case(Rule2a == "", case(Rule1 == "",case(Rule2b == "","N/A", Rule2b),Rule1),Rule2a), 
RuleCollection = case(RuleCollection2b == "",case(RuleCollection2a == "","No rule matched",RuleCollection2a), RuleCollection2b),FQDN = case(FQDN == "", "N/A", FQDN),TargetPort = case(TargetPort == "", "N/A", TargetPort)
| project TimeGenerated, msg_s, Protocol, SourceIP, SourcePort, FQDN, TargetPort, Action ,RuleCollection, Rule
```

## <a name="network-rules-log-data-query"></a>Consulta de dados de log das regras de rede

A consulta a seguir analisa os dados de registro de regras da rede. Em várias linhas de comentário, há algumas diretrizes sobre como a consulta foi criada:

```Kusto
AzureDiagnostics
| where Category == "AzureFirewallNetworkRule"
//using :int makes it easier to pars but later we'll convert to string as we're not interested to do mathematical functions on these fields
//case 1: for records that look like this:
//TCP request from 10.0.2.4:51990 to 13.69.65.17:443. Action: Deny//Allow
//UDP request from 10.0.3.4:123 to 51.141.32.51:123. Action: Deny/Allow
//TCP request from 193.238.46.72:50522 to 40.119.154.83:3389 was DNAT'ed to 10.0.2.4:3389
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int *
//case 1a: for regular network rules
//TCP request from 10.0.2.4:51990 to 13.69.65.17:443. Action: Deny//Allow
//UDP request from 10.0.3.4:123 to 51.141.32.51:123. Action: Deny/Allow
| parse msg_s with * ". Action: " Action1a
//case 1b: for NAT rules
//TCP request from 193.238.46.72:50522 to 40.119.154.83:3389 was DNAT'ed to 10.0.2.4:3389
| parse msg_s with * " was " Action1b " to " NatDestination
//case 2: for ICMP records
//ICMP request from 10.0.2.4 to 10.0.3.4. Action: Allow
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2
| extend
SourcePort = tostring(SourcePortInt),
TargetPort = tostring(TargetPortInt)
| extend 
    Action = case(Action1a == "", case(Action1b == "",Action2,Action1b), Action1a),
    Protocol = case(Protocol == "", Protocol2, Protocol),
    SourceIP = case(SourceIP == "", SourceIP2, SourceIP),
    TargetIP = case(TargetIP == "", TargetIP2, TargetIP),
    //ICMP records don't have port information
    SourcePort = case(SourcePort == "", "N/A", SourcePort),
    TargetPort = case(TargetPort == "", "N/A", TargetPort),
    //Regular network rules don't have a DNAT destination
    NatDestination = case(NatDestination == "", "N/A", NatDestination)
| project TimeGenerated, msg_s, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action, NatDestination
```

A mesma consulta em um formato mais condensado:

```Kusto
AzureDiagnostics
| where Category == "AzureFirewallNetworkRule"
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int *
| parse msg_s with * ". Action: " Action1a
| parse msg_s with * " was " Action1b " to " NatDestination
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2
| extend SourcePort = tostring(SourcePortInt),TargetPort = tostring(TargetPortInt)
| extend Action = case(Action1a == "", case(Action1b == "",Action2,Action1b), Action1a),Protocol = case(Protocol == "", Protocol2, Protocol),SourceIP = case(SourceIP == "", SourceIP2, SourceIP),TargetIP = case(TargetIP == "", TargetIP2, TargetIP),SourcePort = case(SourcePort == "", "N/A", SourcePort),TargetPort = case(TargetPort == "", "N/A", TargetPort),NatDestination = case(NatDestination == "", "N/A", NatDestination)
| project TimeGenerated, msg_s, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action, NatDestination
```

## <a name="threat-intelligence-log-data-query"></a>Consulta de dados de log do Threat Intelligence

A consulta a seguir analisa os dados do registro de regras do Threat Intelligence:

```Kusto
AzureDiagnostics
| where OperationName  == "AzureFirewallThreatIntelLog"
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int *
| parse msg_s with * ". Action: " Action "." Message
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2
| extend SourcePort = tostring(SourcePortInt),TargetPort = tostring(TargetPortInt)
| extend Protocol = case(Protocol == "", Protocol2, Protocol),SourceIP = case(SourceIP == "", SourceIP2, SourceIP),TargetIP = case(TargetIP == "", TargetIP2, TargetIP),SourcePort = case(SourcePort == "", "N/A", SourcePort),TargetPort = case(TargetPort == "", "N/A", TargetPort)
| sort by TimeGenerated desc | project TimeGenerated, msg_s, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action,Message
```

## <a name="sample-logs"></a>Logs de exemplo

As seguintes amostras de registro mostram os dados incluídos em uma entrada de registro.

![entrada de registro](media/log-analytics-samples/log1.png)

![entrada de registro](media/log-analytics-samples/log2.png)

![entrada de registro](media/log-analytics-samples/log3.png)
## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre diagnóstico e monitoramento do Firewall do Azure, confira [Tutorial: Monitorar métricas e logs do Firewall do Azure](tutorial-diagnostics.md).
