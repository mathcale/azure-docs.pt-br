---
title: 'Tutorial: Configure 4me para provisionamento automático do usuário com o Azure Active Directory | Microsoft Docs'
description: Saiba como configurar o Azure Active Directory para provisionar e desprovisionar automaticamente contas de usuário para o 4me.
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: na
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/3/2019
ms.author: jeedes
ms.openlocfilehash: 423ba8c7aea9659a4c91f68a01392954c2ba6db2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77059134"
---
# <a name="tutorial-configure-4me-for-automatic-user-provisioning"></a>Tutorial: Configure 4me para provisionamento automático do usuário

O objetivo deste tutorial é demonstrar as etapas a serem executadas no 4me e no Azure Active Directory (Azure AD) para configurar o Azure AD para provisão e desprovisionamento automático de usuários e/ou grupos para o 4me.

> [!NOTE]
> Este tutorial descreve um conector compilado na parte superior do Serviço de Provisionamento de Usuário do Microsoft Azure AD. Para detalhes importantes sobre o que esse serviço faz, como funciona e as perguntas frequentes, consulte [Automatizar o provisionamento e desprovisionamento de usuários para aplicativos SaaS com o Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Atualmente, esse conector está em versão prévia pública. Para obter mais informações sobre os Termos de uso gerais do Microsoft Azure para a versão prévia de recursos, confira [Termos de uso adicionais para versões prévias do Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Pré-requisitos

O cenário descrito neste tutorial pressupõe que você já tem os seguintes pré-requisitos:

* Um locatário do Azure AD
* [Um inquilino 4me](https://www.4me.com/trial/)
* Uma conta de usuário no 4me com permissões de administração.

## <a name="add-4me-from-the-gallery"></a>Adicione 4me da galeria

Antes de configurar o 4me para provisionamento automático do usuário com o Azure AD, você precisa adicionar 4me da galeria de aplicativos Azure AD à sua lista de aplicativos SaaS gerenciados.

**Para adicionar 4me na galeria de aplicativos do Azure AD, execute as seguintes etapas:**

1. No **[portal Azure](https://portal.azure.com)**, no painel de navegação à esquerda, selecione **Azure Active Directory**.

    ![O botão Azure Active Directory](common/select-azuread.png)

2. Vá para **aplicativos Enterprise**e selecione Todos **os aplicativos**.

    ![A folha Aplicativos empresariais](common/enterprise-applications.png)

3. Para adicionar um novo aplicativo, selecione o botão **Novo aplicativo** na parte superior do painel.

    ![O botão Novo aplicativo](common/add-new-app.png)

4. Na caixa de pesquisa, **digite 4me**, selecione **4me** no painel de resultados e clique no botão **Adicionar** para adicionar o aplicativo.

    ![4me na lista de resultados](common/search-new-app.png)

## <a name="assigning-users-to-4me"></a>Atribuindo usuários ao 4me

O Azure Active Directory usa um conceito chamado *atribuições* para determinar quais usuários devem receber acesso a aplicativos selecionados. No contexto do provisionamento automático do usuário, apenas os usuários e/ou grupos que foram atribuídos a um aplicativo no Azure AD são sincronizados.

Antes de configurar e habilitar o provisionamento automático do usuário, você deve decidir quais usuários e/ou grupos no Azure AD precisam ter acesso ao 4me. Uma vez decidido, você pode atribuir esses usuários e/ou grupos ao 4me seguindo as instruções aqui:

* [Atribuir um usuário ou um grupo a um aplicativo empresarial](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-4me"></a>Dicas importantes para atribuir usuários ao 4me

* Recomenda-se que um único usuário azure AD seja designado ao 4me para testar a configuração automática de provisionamento do usuário. Outros usuários e/ou grupos podem ser atribuídos mais tarde.

* Ao atribuir um usuário ao 4me, você deve selecionar qualquer função específica de aplicativo (se disponível) na caixa de diálogo de atribuição. Os usuários com a **função Default Access** são excluídos do provisionamento.

## <a name="configuring-automatic-user-provisioning-to-4me"></a>Configuração do provisionamento automático do usuário para o 4me 

Esta seção orienta você através das etapas para configurar o serviço de provisionamento Azure AD para criar, atualizar e desativar usuários e/ou grupos no 4me com base em atribuições de usuário e/ou grupo no Azure AD.

> [!TIP]
> Você também pode optar por ativar o single sign-on baseado em SAML para o 4me, seguindo as instruções fornecidas no [tutorial de signon-on único do 4me](4me-tutorial.md). O logon único pode ser configurado independentemente do provisionamento automático de usuário, embora esses dois recursos sejam complementares.

### <a name="to-configure-automatic-user-provisioning-for-4me-in-azure-ad"></a>Para configurar o provisionamento automático do usuário para o 4me no Azure AD:

1. Faça login no [portal Azure](https://portal.azure.com). Selecione **Aplicativos Corporativos**e selecione **Todos os aplicativos**.

    ![Folha de aplicativos empresariais](common/enterprise-applications.png)

2. Na lista de aplicativos, escolha **4me**.

    ![O link do 4me na lista de Aplicativos](common/all-applications.png)

3. Selecione a guia **Provisionamento**.

    ![Guia de provisionamento](common/provisioning.png)

4. Defina o **modo de provisionamento** como **automático**.

    ![Guia de provisionamento](common/provisioning-automatic.png)

5. Para recuperar a **URL do inquilino** e o **token secreto** da sua conta 4me, siga o passo a passo conforme descrito no Passo 6.

6. Faça login no seu console 4me Admin. Navegue até **Configurações**.

    ![Configurações de 4me](media/4me-provisioning-tutorial/4me01.png)

    Digite **aplicativos** na barra de pesquisa.

    ![Aplicativos 4me](media/4me-provisioning-tutorial/4me02.png)

    Abra a gota do **SCIM** para recuperar o Token Secreto e o ponto final do SCIM.

    ![4me SCIM](media/4me-provisioning-tutorial/4me03.png)

7. Ao preencher os campos mostrados no Passo 5, clique em **Conexão de teste** para garantir que o Azure AD possa se conectar ao 4me. Se a conexão falhar, certifique-se de que sua conta 4me tenha permissões de administração e tente novamente.

    ![Token](common/provisioning-testconnection-tenanturltoken.png)

8. No campo **Notificação por Email**, insira o endereço de email de uma pessoa ou grupo que deverá receber as notificações de erro de provisionamento e selecione a caixa de seleção - **Enviar uma notificação por email quando ocorrer uma falha**.

    ![Email de notificação](common/provisioning-notification-email.png)

9. Clique em **Salvar**.

10. Na seção **Mapeamentos,** selecione **Sincronizar usuários do diretório ativo do Azure para 4me**.

    ![Mapeamentos de usuários do 4me](media/4me-provisioning-tutorial/4me-user-mapping.png)
    
11. Revise os atributos do usuário sincronizados do Azure AD para o 4me na seção **Mapeamento de atributos.** Os atributos selecionados como **propriedades de correspondência** são usados para corresponder as contas de usuário no 4me para operações de atualização. Selecione o botão **Salvar** para confirmar as alterações.

    ![Mapeamentos de usuários do 4me](media/4me-provisioning-tutorial/4me-user-attributes.png)
    
12. Na seção **Mapeamentos,** selecione **Sincronizar grupos de diretórios ativos do Azure para 4me**.

    ![Mapeamentos de usuários do 4me](media/4me-provisioning-tutorial/4me-group-mapping.png)
    
13. Revise os atributos de grupo sincronizados do Azure AD para o 4me na seção **Mapeamento de atributos.** Os atributos selecionados como **propriedades correspondentes** são usados para corresponder os grupos no 4me para operações de atualização. Selecione o botão **Salvar** para confirmar as alterações.

    ![Mapeamentos de grupos 4me](media/4me-provisioning-tutorial/4me-group-attribute.png)

14. Para configurar filtros de escopo, consulte as seguintes instruções fornecidas no [tutorial do Filtro de Escopo](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Para habilitar o serviço de provisionamento Azure AD para 4me, altere o **Status de Provisionamento** para **Ativado** na seção **Configurações.**

    ![Status do provisionamento ativado](common/provisioning-toggle-on.png)

16. Defina os usuários e/ou grupos que você gostaria de provisionar para o 4me escolhendo os valores desejados no **Escopo** na seção **Configurações.**

    ![Escopo de provisionamento](common/provisioning-scope.png)

17. Quando estiver pronto para provisionar, clique em **Salvar**.

    ![Salvando a configuração de provisionamento](common/provisioning-configuration-save.png)

Essa operação inicia a sincronização inicial de todos os usuários e/ou grupos definidos no **Escopo** na seção **Configurações**. Observe que a sincronização inicial levará mais tempo do que as sincronizações subsequentes, que ocorrem aproximadamente a cada 40 minutos, desde que o serviço de provisionamento do Microsoft Azure Active Directory esteja em execução. Você pode usar a seção **Detalhes de Sincronização** para monitorar o progresso e seguir links para o relatório de atividade de provisionamento, que descreve todas as ações executadas pelo serviço de provisionamento Azure AD no 4me.

Para saber mais sobre como ler os logs de provisionamento do Azure AD, consulte [Relatórios sobre o provisionamento automático de contas de usuário](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitações do conector

* O 4me possui diferentes URLs de ponto final SCIM para ambientes de teste e produção. O primeiro termina com **.qa,** enquanto o segundo termina com **.com**
* Os Tokens Secretos gerados 4me têm uma data de validade de um mês a partir da geração.
* 4me não suporta operações **DELETE**

## <a name="additional-resources"></a>Recursos adicionais

* [Gerenciamento do provisionamento de contas de usuário para Aplicativos Corporativos](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [O que é acesso ao aplicativo e logon único com o Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Próximas etapas

* [Saiba como fazer revisão de logs e obter relatórios sobre atividade de provisionamento](../app-provisioning/check-status-user-account-provisioning.md)