---
title: Configure a política de firewall de aplicativos web de filtragem geográfica para o serviço de porta frontal do Azure
description: Neste tutorial, você aprende como criar uma política de geofiltragem e associar a política com o host frontend existente da Front Door
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: victorh
ms.reviewer: tyao
ms.openlocfilehash: abcef61d478eccb4e979b60eb845ac8d398a49f9
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79135863"
---
# <a name="set-up-a-geo-filtering-waf-policy-for-your-front-door"></a>Configure uma política WAF de filtragem geográfica para sua porta da frente

Este tutorial mostra como usar o Azure PowerShell para criar uma política de filtragem geográfica de exemplo e associar a política a seu host de front-end Front Door existente. Essa política de filtragem geográfica de exemplo bloqueará as solicitações de todos os outros países/regiões, exceto dos Estados Unidos.

Caso não tenha uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) agora.

## <a name="prerequisites"></a>Pré-requisitos

Antes de configurar uma política de filtragem geográfica, configure o ambiente do PowerShell e crie um perfil de Front Door.
### <a name="set-up-your-powershell-environment"></a>Configurar o ambiente do PowerShell
O Azure PowerShell fornece um conjunto de cmdlets que usam o modelo [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) para gerenciar os recursos do Azure. 

Você pode instalar o [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) no computador local e usá-lo em qualquer sessão do PowerShell. Siga as instruções na página para entrar com suas credenciais Azure e instale o módulo Az PowerShell.

#### <a name="connect-to-azure-with-an-interactive-dialog-for-sign-in"></a>Conecte-se ao Azure com um diálogo interativo para login

```
Install-Module -Name Az
Connect-AzAccount
```
Verifique se tem a versão atual do PowerShellGet instalada. Execute o comando abaixo e reabra o PowerShell.

```
Install-Module PowerShellGet -Force -AllowClobber
``` 
#### <a name="install-azfrontdoor-module"></a>Instalar o módulo Az.FrontDoor 

```
Install-Module -Name Az.FrontDoor
```

### <a name="create-a-front-door-profile"></a>Criar um perfil de Front Door

Crie um perfil da Porta da Frente seguindo as instruções descritas no [Quickstart: Crie um perfil da porta da frente](../../frontdoor/quickstart-create-front-door.md).

## <a name="define-geo-filtering-match-condition"></a>Definir condição de correspondência da filtragem geográfica

Crie uma condição de correspondência de exemplo que seleciona as solicitações não provenientes de "US" usando [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) nos parâmetros, ao criar uma condição de correspondência. Dois códigos de país de letras para mapeamento de países são fornecidos no [que é geofiltragem em um domínio para a Porta Frontal do Azure?](waf-front-door-geo-filtering.md).

```azurepowershell-interactive
$nonUSGeoMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable RemoteAddr `
-OperatorProperty GeoMatch `
-NegateCondition $true `
-MatchValue "US"
```
 
## <a name="add-geo-filtering-match-condition-to-a-rule-with-action-and-priority"></a>Adicionar condição de correspondência de filtragem geográfica a uma regra com Ação e Prioridade

Crie um objeto CustomRule `nonUSBlockRule` baseado na condição de correspondência, uma Ação e uma Prioridade, usando [New-AzFrontDoorWafCustomRuleObject](/powershell/module/az.frontdoor/new-azfrontdoorwafcustomruleobject).  Uma CustomRule pode ter várias MatchCondition.  Neste exemplo, a Ação está definida como Bloquear e a Prioridade como 1, a prioridade mais alta.

```
$nonUSBlockRule = New-AzFrontDoorWafCustomRuleObject `
-Name "geoFilterRule" `
-RuleType MatchRule `
-MatchCondition $nonUSGeoMatchCondition `
-Action Block `
-Priority 1
```

## <a name="add-rules-to-a-policy"></a>Adicionar regras a uma política

Encontre o nome do grupo de recursos que contém o perfil de Front Door usando `Get-AzResourceGroup`. Em seguida, crie um objeto de política `geoPolicy` contendo `nonUSBlockRule` usando [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy) no grupo de recursos especificado que contém o perfil de Front Door. Você deve fornecer um nome exclusivo para a política geográfica. 

O exemplo a seguir usa o nome do Grupo de Recursos *myResourceGroupFD1* com a suposição de que você criou o perfil da Porta da Frente usando instruções fornecidas no [artigo Quickstart: Create a Front Door.](../../frontdoor/quickstart-create-front-door.md) No exemplo abaixo, substitua o nome da política *geoPolicyAllowUSOnly* por um nome de política exclusivo.

```
$geoPolicy = New-AzFrontDoorWafPolicy `
-Name "geoPolicyAllowUSOnly" `
-resourceGroupName myResourceGroupFD1 `
-Customrule $nonUSBlockRule  `
-Mode Prevention `
-EnabledState Enabled
```

## <a name="link-waf-policy-to-a-front-door-frontend-host"></a>Vincular a política WAF a um host de front-end do Front Door

Vincule o objeto de política WAF ao host de front-end do Front Door e atualize as propriedades do Front Door. 

Para fazer isso, primeiro recupere o objeto de Front Door usando [Get-AzFrontDoor](/powershell/module/az.frontdoor/get-azfrontdoor). 

```
$geoFrontDoorObjectExample = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD1
$geoFrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $geoPolicy.Id
```

Em seguida, defina a propriedade de front-end WebApplicationFirewallPolicyLink para a ID do recurso do `geoPolicy`usando [AzFrontDoor](/powershell/module/az.frontdoor/set-azfrontdoor).

```
Set-AzFrontDoor -InputObject $geoFrontDoorObjectExample[0]
```

> [!NOTE] 
> Você só precisará definir a propriedade WebApplicationFirewallPolicyLink uma vez para vincular uma política WAF a um host de front-end do Front Door. As atualizações subsequentes da política serão aplicadas automaticamente ao host de front-end.

## <a name="next-steps"></a>Próximas etapas

- Saiba mais sobre o [Firewall do aplicativo Web do Azure](../overview.md).
- Saiba como [criar um Front Door](../../frontdoor/quickstart-create-front-door.md).
