---
title: Gerenciar DNS para serviços de domínio Azure AD | Microsoft Docs
description: Saiba como instalar as Ferramentas de Servidor DNS para gerenciar o DNS para um domínio gerenciado do Azure Active Directory Domain Services.
author: iainfoulds
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 10/31/2019
ms.author: iainfou
ms.openlocfilehash: 2694a5f250b746748a1b42ac4d211aa28ef1ebad
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77613699"
---
# <a name="administer-dns-in-an-azure-ad-domain-services-managed-domain"></a>Administrar DNS em um domínio gerenciado do Azure AD Domain Services

No Azure Active Directory Domain Services (Azure AD DS), um componente-chave é o DNS (Domain Name Resolution). O Azure AD DS inclui um servidor DNS que fornece resolução de nomes para o domínio gerenciado. Este servidor DNS inclui registros DNS incorporados e atualizações para os componentes-chave que permitem que o serviço seja executado.

À medida que você executa seus próprios aplicativos e serviços, você pode precisar criar registros de DNS para máquinas que não estão unidas ao domínio, configurar endereços IP virtuais para balanceadores de carga ou configurar reencaminhadores de DNS externos. Os usuários que pertencem ao grupo *Administradores AAD DC* recebem privilégios de administração de DNS no domínio gerenciado do Azure AD DS e podem criar e editar registros DNS personalizados.

Em um ambiente híbrido, as zonas e registros de DNS configurados em um ambiente AD DS no local não são sincronizados com o Azure AD DS. Para definir e usar suas próprias entradas de DNS, crie registros no servidor Azure AD DS DNS ou use encaminhadores condicionais que apontam para servidores DNS existentes em seu ambiente.

Este artigo mostra como instalar as ferramentas do DNS Server e usar o console DNS para gerenciar registros no Azure AD DS.

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="before-you-begin"></a>Antes de começar

Para concluir este artigo, você precisa dos seguintes recursos e privilégios:

* Uma assinatura ativa do Azure.
    * Se você não tiver uma assinatura do Azure, [crie uma conta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Um locatário do Azure Active Directory associado com a assinatura, sincronizado com um diretório local ou somente em nuvem.
    * Se necessário, [crie um locatário do Azure Active Directory][create-azure-ad-tenant] ou [associe uma assinatura do Azure à sua conta][associate-azure-ad-tenant].
* Um domínio gerenciado do Azure Active Directory Domain Services habilitado e configurado no locatário do Azure AD.
    * Se necessário, complete o tutorial para [criar e configurar uma instância de Serviços de Domínio do Diretório Ativo do Azure][create-azure-ad-ds-instance].
* Uma VM de gerenciamento do Windows Server que é acompanhada pelo domínio gerenciado pelo Azure AD DS.
    * Se necessário, complete o tutorial para [criar uma VM do Windows Server e junte-a a um domínio gerenciado][create-join-windows-vm].
* Uma conta de usuário que é membro do grupo de *administradores do Azure AD DC* no locatário do Azure AD.

## <a name="install-dns-server-tools"></a>Instale ferramentas do DNS Server

Para criar e modificar registros DNS no Azure AD DS, você precisa instalar as ferramentas do DNS Server. Essas ferramentas podem ser instaladas como um recurso no Windows Server. Para obter mais informações sobre como instalar as ferramentas administrativas em um cliente Windows, consulte instalar [o RSAT (Remote Server Administration Tools, ferramentas de administração de servidores remotos).][install-rsat]

1. Faça login na sua VM de gestão. Para obter etapas sobre como se conectar usando o portal Azure, consulte [Conecte-se a uma VM do Servidor Windows][connect-windows-server-vm].
1. Se **Gerenciador do Servidor** não abrir por padrão quando você entrar na VM, selecione o menu **Iniciar** e, em seguida, escolha **Gerenciador do Servidor**.
1. No painel *Dashboard* da janela **Gerenciador do Servidor**, selecione **Adicionar Funções e Recursos**.
1. Na página **Antes de Você Começar** do *Assistente de Adição de Funções e Recursos*, selecione **Avançar**.
1. Para o *Tipo de Instalação*, deixe a opção **Instalação baseada em função ou recurso** marcada e selecione **Avançar**.
1. Na página **Seleção do servidor,** escolha a VM atual no pool de servidores, como *myvm.aaddscontoso.com,* e selecione **Next**.
1. Na página **Funções do Servidor**, clique em **Avançar**.
1. Na página **Recursos**, expanda o nó **Ferramentas de Administração de Servidor Remoto** e, em seguida, expanda o nó **Ferramentas de Administração de Funções**. Selecione o recurso **Ferramentas do Servidor DNS** na lista de ferramentas de administração de funções.

    ![Escolha instalar as Ferramentas de Servidor DNS na lista de ferramentas de administração de função disponíveis](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-dns-tools.png)

1. Na página **Confirmação**, selecione **Instalar**. Pode levar um minuto ou dois para instalar as ferramentas de Gerenciamento de Políticas de Grupo.
1. Quando a instalação do recurso for concluída, selecione **Fechar** para sair do **Assistente de Adição de Funções e Recursos**.

## <a name="open-the-dns-management-console-to-administer-dns"></a>Abra o console de gerenciamento de DNS para administrar DNS

Com as ferramentas do DNS Server instaladas, você pode administrar registros DeD no domínio gerenciado do Azure AD DS.

> [!NOTE]
> Para administrar o DNS em um domínio gerenciado pelo Azure AD DS, você deve estar conectado a uma conta de usuário que seja membro do grupo *Administradores AAD DC.*

1. Na tela Iniciar, selecione **Ferramentas Administrativas**. Uma lista de ferramentas de gerenciamento disponíveis é mostrada, incluindo **DNS** instalado na seção anterior. Selecione **DNS** para iniciar o console DNS Management.
1. Na **caixa de diálogo Conectar ao servidor DNS,** selecione **O computador a seguir**e digite o nome de domínio DNS do domínio gerenciado, como *aaddscontoso.com*:

    ![Conecte-se ao domínio gerenciado do Azure AD DS no console DNS](./media/active-directory-domain-services-admin-guide/dns-console-connect-to-domain.png)

1. O Console DNS se conecta ao domínio gerenciado pelo Azure AD DS especificado. Expanda as **Zonas de análise para frente** ou zonas de análise **reversa** para criar as entradas de DNS necessárias ou editar os registros existentes conforme necessário.

    ![Console DNS - administrar domínio](./media/active-directory-domain-services-admin-guide/dns-console-managed-domain.png)

> [!WARNING]
> Ao gerenciar registros usando as ferramentas do DNS Server, certifique-se de que você não exclua ou modifique os registros DNS incorporados que são usados pelo Azure AD DS. Os registros DNS internos incluem registros DNS do domínio, registros de servidor de nome e outros registros usados para a localização do controlador de domínio. Se você modificar esses registros, os serviços de domínio serão interrompidos na rede virtual.

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre como gerenciar o DNS, veja o [artigo sobre ferramentas de DNS no TechNet](https://technet.microsoft.com/library/cc753579.aspx).

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[create-join-windows-vm]: join-windows-vm.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md
[connect-windows-server-vm]: join-windows-vm.md#connect-to-the-windows-server-vm

<!-- EXTERNAL LINKS -->
[install-rsat]: /windows-server/remote/remote-server-administration-tools#BKMK_Thresh
