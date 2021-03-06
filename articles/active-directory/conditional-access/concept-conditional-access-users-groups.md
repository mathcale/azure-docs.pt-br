---
title: Usuários e grupos na política de Acesso Condicional - Diretório Ativo do Azure
description: Quem são usuários e grupos em uma política de acesso condicional do Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 02/11/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 36898e75680771a9cb084fa142bb635ddbf51c70
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77192123"
---
# <a name="conditional-access-users-and-groups"></a>Acesso Condicional: Usuários e grupos

Uma política de acesso condicional deve incluir uma atribuição de usuário como um dos sinais no processo de decisão. Os usuários podem ser incluídos ou excluídos das políticas de Acesso Condicional. 

![Usuário como sinal nas decisões tomadas pelo Acesso Condicional](./media/concept-conditional-access-users-groups/conditional-access-users-and-groups.png)

## <a name="include-users"></a>Incluir usuários

Essa lista de usuários normalmente inclui todos os usuários que uma organização está direcionando em uma política de Acesso Condicional. 

As seguintes opções estão disponíveis para incluir ao criar uma política de acesso condicional.

- Nenhum
   - Nenhum usuário selecionado
- todos os usuários
   - Todos os usuários que existem no diretório, incluindo convidados B2B.
- Selecionar Usuários e grupos
   - Todos os usuários convidados e externos (visualização)
      - Esta seleção inclui quaisquer convidados B2B `user type` e usuários `guest`externos, incluindo qualquer usuário com o atributo definido para . Essa seleção também se aplica a qualquer usuário externo conectado de uma organização diferente, como um CSP (Cloud Solution Provider, provedor de soluções em nuvem). 
   - Funções de diretório (pré-visualização)
      - Permite que os administradores selecionem funções específicas de diretório Azure AD usadas para determinar a atribuição. Por exemplo, as organizações podem criar uma política mais restritiva sobre os usuários atribuídos à função de administrador global.
   - Usuários e grupos
      - Permite a segmentação de conjuntos específicos de usuários. Por exemplo, as organizações podem selecionar um grupo que contenha todos os membros do departamento de RH quando um aplicativo de RH é selecionado como o aplicativo em nuvem. Um grupo pode ser qualquer tipo de grupo no Azure AD, incluindo grupos de segurança e distribuição dinâmicos ou atribuídos.

## <a name="exclude-users"></a>Excluir usuários

Exclusões são comumente usadas para acesso de emergência ou contas de vidro de quebra. Mais informações sobre contas de acesso a emergências e por que elas são importantes podem ser encontradas nos seguintes artigos: 

* [Gerenciar contas de acesso de emergência no Microsoft Azure Active Directory](../users-groups-roles/directory-emergency-access.md)
* [Criar uma estratégia de gerenciamento de controle de acesso resiliente com o Azure Active Directory](../authentication/concept-resilient-controls.md)

As seguintes opções estão disponíveis para excluir ao criar uma política de acesso condicional.

- Todos os usuários convidados e externos (visualização)
   - Esta seleção inclui quaisquer convidados B2B `user type` e usuários `guest`externos, incluindo qualquer usuário com o atributo definido para . Essa seleção também se aplica a qualquer usuário externo conectado de uma organização diferente, como um CSP (Cloud Solution Provider, provedor de soluções em nuvem). 
- Funções de diretório (pré-visualização)
   - Permite que os administradores selecionem funções específicas de diretório Azure AD usadas para determinar a atribuição. Por exemplo, as organizações podem criar uma política mais restritiva sobre os usuários atribuídos à função de administrador global.
- Usuários e grupos
   - Permite a segmentação de conjuntos específicos de usuários. Por exemplo, as organizações podem selecionar um grupo que contenha todos os membros do departamento de RH quando um aplicativo de RH é selecionado como o aplicativo em nuvem. Um grupo pode ser qualquer tipo de grupo no Azure AD, incluindo grupos de segurança e distribuição dinâmicos ou atribuídos.

## <a name="next-steps"></a>Próximas etapas

- [Acesso Condicional: Aplicativos ou ações em nuvem](concept-conditional-access-cloud-apps.md)

- [Políticas comuns de Acesso Condicional](concept-conditional-access-policy-common.md)
