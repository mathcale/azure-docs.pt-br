---
title: Gerenciar usuários excluídos das políticas de acesso condicional - Azure AD
description: Saiba como usar as avaliações de acesso do Azure Active Directory (Azure AD) para gerenciar usuários que foram excluídos das políticas de acesso condicional
services: active-directory
documentationcenter: ''
author: msaburnley
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 09/25/2018
ms.author: ajburnle
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: d1d7bce940f2b614c239e8b5e5719d96da10a6c0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75422704"
---
# <a name="use-azure-ad-access-reviews-to-manage-users-excluded-from-conditional-access-policies"></a>Use avaliações de acesso a Azure AD para gerenciar usuários excluídos das políticas de acesso condicional

Em um mundo ideal, todos os usuários seguiriam as políticas de acesso para proteger o acesso aos recursos de sua organização. No entanto, às vezes, há casos comerciais em que é necessário fazer exceções. Este artigo descreve alguns exemplos em que exclusões podem ser necessárias e descreve como você, no papel do administrador de TI, pode gerenciar essa tarefa, evitar negligência com relação às exceções à política e fornecer aos auditores comprovação de que essas exceções são revisadas regularmente usando as revisões de acesso do Azure AD (Azure Active Directory).

> [!NOTE]
> Para usar as revisões de acesso do Azure AD, é necessário ter uma licença válida do Azure AD Premium P2, uma licença paga do Enterprise Mobility + Security E5 ou uma licença de avaliação. Para obter mais informações, consulte [Edições do Active Directory do Azure](../fundamentals/active-directory-whatis.md).

## <a name="why-would-you-exclude-users-from-policies"></a>Por que excluir usuários das políticas?

Como administrador de TI, você pode usar [o Azure AD Conditional Access](../conditional-access/overview.md) para exigir que os usuários se autentiquem usando a autenticação multifatorial (MFA) ou faça login em uma rede ou dispositivo confiável. Durante o planejamento de implantação, você descobre que alguns desses requisitos não podem ser atendidos por todos os usuários. Por exemplo, há usuários que trabalham em um escritório remoto que não faz parte de sua rede interna ou há um executivo que usa um telefone antigo que não é compatível. A empresa exige que esses usuários possam fazer login e fazer seu trabalho, portanto, eles são excluídos das políticas de Acesso Condicional.

Como outro exemplo, você pode usar [locais nomeados](../conditional-access/location-condition.md) no Acesso Condicional para configurar um conjunto de condados e regiões das quais você não deseja permitir que os usuários acessem seu inquilino.

![Locais nomeados no Acesso Condicional](./media/conditional-access-exclusion/named-locations.png)

No entanto, em alguns casos, os usuários podem ter uma razão legítima para fazer login a partir desses países/regiões bloqueados. Por exemplo, os usuários podem estar viajando por motivos pessoais ou a trabalho. Neste exemplo, a política de acesso condicional para bloquear esses países/regiões poderia ter um grupo de segurança na nuvem dedicado para os usuários que são excluídos da política. Usuários que precisam de acesso quando estão viajando podem adicionar a si mesmos ao grupo usando o [Gerenciamento de Grupos de Autoatendimento do Azure AD](../users-groups-roles/groups-self-service-management.md).

Outro exemplo pode ser que você tenha uma política de acesso condicional que [bloqueia a autenticação de legado para a grande maioria dos seus usuários](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/07/azure-ad-conditional-access-support-for-blocking-legacy-auth-is-in-public-preview/). A Microsoft recomenda enfaticamente que você bloqueie o uso de protocolos herdados em seu locatário para melhorar sua postura de segurança. No entanto, se houver alguns usuários que realmente precisam usar métodos de autenticação herdados para acessar recursos por meio do Office 2010 ou de clientes baseados em SMTP/IMAP/POP, você poderá excluir esses usuários da política que bloqueia métodos de autenticação herdados.

## <a name="why-are-exclusions-challenging"></a>Por que as exclusões são um desafio?

No Azure AD, você pode escopo de uma política de Acesso Condicional a um conjunto de usuários. Você também pode excluir alguns desses usuários selecionando funções azure AD, usuários individuais ou convidados de usuários. É importante se lembrar de que, quando as exclusões são configuradas, a intenção da política não pode ser imposta para os usuários em questão. Se essas exclusões tiverem sido configuradas como uma lista de usuários individuais ou por meio de um grupo de segurança local herdado, a visibilidade da lista de exclusões será limitada (os usuários talvez não saibam de sua existência), bem como o controle do administrador de TI sobre ela (os usuários poderão ingressar no grupo de segurança para ignorar a política). Além disso, usuários que se qualificavam para a exclusão em um determinado momento podem deixar de precisar ou de estar qualificados para ela.

No início de uma exclusão, há uma pequena lista de usuários que ignoram a política. Com o passar do tempo, cada mais usuários são excluídos e a lista aumenta. Em algum momento, é necessário analisar a lista e confirmar que cada um desses usuários ainda precisa ser excluído. Do ponto de vista técnico, o gerenciamento da lista pode ser relativamente fácil, mas quem toma as decisões de negócios e como você garante que todo o processo seja auditável?

No entanto, se você configurar a exclusão à diretiva de acesso condicional usando um grupo AD do Azure, então você poderá usar as avaliações de acesso como um controle compensador, para aumentar a visibilidade e reduzir o número de usuários que têm uma exceção.

## <a name="how-to-create-an-exclusion-group-in-a-conditional-access-policy"></a>Como criar um grupo de exclusão em uma política de acesso condicional

Siga essas etapas para criar um novo grupo Azure AD e uma política de acesso condicional que não se aplica a esse grupo.

### <a name="create-an-exclusion-group"></a>Criar um grupo de exclusão

1. Entre no portal do Azure.

1. No painel de navegação esquerdo, clique em **Azure Active Directory** e clique em **Grupos**.

1. No menu superior, clique em **Novo Grupo** para abrir o painel do grupo.

1. Na lista **Tipo de grupo**, selecione **Segurança**. Especifique um nome e uma descrição.

1. Certifique-se de definir o tipo de **Associação** como **Atribuído**.

1. Selecione os usuários que devem fazer parte do grupo de exclusão e, em seguida, clique em **Criar**.

    ![Novo painel de grupo no Diretório Ativo do Azure](./media/conditional-access-exclusion/new-group.png)

### <a name="create-a-conditional-access-policy-that-excludes-the-group"></a>Crie uma política de acesso condicional que exclua o grupo

Agora você pode criar uma política de acesso condicional que usa esse grupo de exclusão.

1. Na navegação à esquerda, clique em **Azure Active Directory** e clique **em Acesso Condicional** para abrir a lâmina **Políticas.**

1. Clique em **Nova política** para abrir o painel **Novo**.

1. Especifique um nome.

1. Em Atribuições, clique em **Usuários e grupos**.

1. Na guia **Incluir**, selecione **Todos os Usuários**.

1. Na guia **Excluir**, adicione uma marca de seleção a **Usuários e grupos** e, em seguida, clique em **Selecionar usuários excluídos**.

1. Selecione o grupo de exclusão que você criou.

    > [!NOTE]
    > Como melhor prática, é recomendável excluir pelo menos uma conta de administrador da política ao fazer testes para garantir que você não esteja bloqueado de seu locatário.

1. Continue com a configuração da política de acesso condicional com base em seus requisitos organizacionais.

    ![Selecione o painel de usuários excluídos no Acesso Condicional](./media/conditional-access-exclusion/select-excluded-users.png)

Vamos cobrir dois exemplos onde você pode usar avaliações de acesso para gerenciar exclusões em políticas de acesso condicional.

## <a name="example-1-access-review-for-users-accessing-from-blocked-countriesregions"></a>Exemplo 1: Revisão de acesso para usuários que acessam de países/regiões bloqueados

Digamos que você tenha uma política de acesso condicional que bloqueia o acesso de certos países/regiões. Ela inclui um grupo que é excluído da política. Esta é uma revisão de acesso recomendada em que os membros do grupo são revisados.

> [!NOTE]
> Uma função de administrador global ou de administrador de usuário é necessária para criar avaliações de acesso.

1. A revisão se repetirá semanalmente.

2. Nunca terminará para ter certeza de que você está mantendo este grupo de exclusão o mais atualizado.

3. Todos os membros do grupo estarão no escopo da revisão.

4. Cada usuário terá que auto-atestar que ainda precisa ter acesso a esses países/regiões bloqueados, portanto, eles ainda precisam ser um membro do grupo.

5. Se o usuário não responder à solicitação de revisão, ele será automaticamente removido do grupo e, portanto, não poderá mais acessar o inquilino enquanto viaja para esses países/regiões.

6. Habilite as notificações por email para que os usuários sejam notificados do início e da conclusão da revisão de acesso.

    ![Crie um painel de revisão de acesso, por exemplo 1](./media/conditional-access-exclusion/create-access-review-1.png)

## <a name="example-2-access-review-for-users-accessing-with-legacy-authentication"></a>Exemplo 2: revisão de acesso para usuários que acessam usando autenticação herdada

Digamos que você tenha uma política de acesso condicional que bloqueia o acesso para usuários usando autenticação legada e versões de clientes mais antigas. Ela inclui um grupo que é excluído da política. Esta é uma revisão de acesso recomendada em que os membros do grupo são revisados.

1. Essa revisão precisaria ser recorrente.

2. Todos no grupo precisam passar por ela.

3. Ela poderia ser configurada para listar os proprietários das unidades de negócios como revisores selecionados.

4. Aplique os resultados automaticamente e remova usuários que não foram aprovados para continuar usando métodos de autenticação herdados.

5. Pode ser útil habilitar as recomendações para que os revisores de grandes grupos possam tomar decisões com facilidade.

6. Habilite as notificações por email para que os usuários sejam notificados do início e da conclusão da revisão de acesso.

    ![Criar um painel de revisão de acesso, por exemplo 2](./media/conditional-access-exclusion/create-access-review-2.png)

**Dica de PRO**: se você tem vários grupos de exclusão e, portanto, precisa criar várias revisões de acesso, agora temos uma API no ponto de extremidade beta do Microsoft Graph que lhe permite criar e gerenciá-las de forma programática. Para começar, consulte a [Referência da API de revisões de acesso do Azure AD](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/accessreviews_root) e o [Exemplo de recuperação de revisões de acesso do Azure AD por meio do Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/td-p/236096).

## <a name="access-review-results-and-audit-logs"></a>Resultados e logs de auditoria da revisão de acesso

Agora que você tem tudo no lugar, grupo, política de acesso condicional e revisões de acesso, é hora de monitorar e acompanhar os resultados dessas revisões.

1. No portal do Azure, abra a folha **Revisões de acesso**.

1. Abra o controle e o programa que você criou para gerenciar o grupo de exclusão.

1. Clique em **Resultados** para ver quem foi aprovado para permanecer na lista e quem foi removido.

    ![Os resultados das avaliações de acesso mostram quem foi aprovado](./media/conditional-access-exclusion/access-reviews-results.png)

1. Em seguida, clique em **Logs de auditoria** para ver as ações que foram executadas durante a revisão.

    ![Revisões de acesso registram ações de listagem](./media/conditional-access-exclusion/access-reviews-audit-logs.png)

Como administrador de TI, você sabe que o gerenciamento de grupos de exclusão de suas políticas, às vezes, é inevitável. No entanto, a manutenção desses grupos, sua revisão regular pelo proprietário da empresa ou pelos próprios usuários e a auditoria das alterações podem facilitadas com as revisões de acesso do Azure AD.

## <a name="next-steps"></a>Próximas etapas

- [Crie uma revisão de acesso de grupos ou aplicativos](create-access-review.md)
- [O que é o acesso condicional no Azure Active Directory?](../conditional-access/overview.md)
