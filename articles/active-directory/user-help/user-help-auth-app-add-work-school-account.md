---
title: Adicione uma conta de trabalho ou escola ao aplicativo Microsoft Authenticator - Azure AD
description: Adicione sua conta de trabalho ou escola ao aplicativo Microsoft Authenticator para verificar sua identidade enquanto usa a verificação de dois fatores.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: conceptual
ms.date: 01/24/2019
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: f0cc14a53f7ead7f0a496728d477d7d30857a0fb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77063910"
---
# <a name="add-your-work-or-school-account-to-the-microsoft-authenticator-app"></a>Adicione sua conta de trabalho ou escola ao aplicativo Microsoft Authenticator

Se a organização usa a verificação de dois fatores, você pode configurar sua conta corporativa ou de estudante para usar o aplicativo Microsoft Authenticator como um dos métodos de verificação.

>[!Important]
>Antes de poder adicionar sua conta, você deverá baixar e instalar o aplicativo Microsoft Authenticator. Se você ainda não tiver feito isso, siga as etapas no artigo [Baixar e instalar o aplicativo](user-help-auth-app-download-install.md).

## <a name="add-your-work-or-school-account"></a>Adicionar a conta corporativa ou de estudante

1. Em seu computador, vá para a página [Verificação de segurança adicional](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

    >[!Note]
    >Caso não veja a página **Verificação de segurança adicional**, é possível que o administrador tenha ativado a experiência de informações de segurança (versão prévia). Se esse for o caso, você deverá seguir as instruções da seção [Configurar informações de segurança para usar um aplicativo Authenticator](security-info-setup-auth-app.md). Se não for o caso, você precisará entrar em contato com o suporte técnico da sua organização para obter assistência. Para obter mais informações sobre informações de segurança, consulte [visão geral das informações de segurança (visualização).](user-help-security-info-overview.md)

2. Marque a caixa ao lado de **aplicativo do autenticador** e selecione **Configurar**.

    A página **Configurar aplicativo móvel** é exibida.

    ![Tela que fornece o código QR](./media/user-help-auth-app-download-install/auth-app-barcode.png)

3. Abra o aplicativo Microsoft Authenticator, selecione **adicionar conta** da **personalizar e controle** ícone no canto superior direito e, em seguida, selecione **trabalho ou conta de escola**.

    >[!Note]
    >Se esta for a primeira vez que está configurando o aplicativo Microsoft Authenticator, você poderá receber um prompt perguntando se deseja permitir que o aplicativo acesse sua câmera (iOS) ou que o aplicativo tire fotos e grave vídeo (Android). Você precisa selecionar **Permitir** para que o aplicativo autenticador possa acessar sua câmera para tirar uma foto do código QR na próxima etapa. Se não permitir acesso da câmera, você ainda poderá configurar o aplicativo autenticador, mas precisará adicionar as informações de código manualmente. Para obter informações sobre como adicionar o código manualmente, consulte [Adicionar manualmente uma conta ao aplicativo](user-help-auth-app-add-account-manual.md).

4. Use a câmera do dispositivo para digitalizar o código QR da tela **Configurar aplicativo móvel** em seu computador e, em seguida, escolha **Concluído**.

    >[!Note]
    >Se a câmera não conseguir capturar o código QR, você poderá adicionar manualmente as informações da conta ao aplicativo Microsoft Authenticator para verificação de dois fatores. Confira [Adicionar sua conta manualmente](user-help-auth-app-add-account-manual.md) para obter mais informações e saber como fazê-lo.

5. Examine a tela **Contas** do aplicativo em seu dispositivo, para garantir que sua conta está correta e que há um código de verificação de seis dígitos associado. Para segurança adicional, o código de verificação muda a cada 30 segundos, evitando que alguém use o mesmo código várias vezes.

    ![Tela de contas](./media/user-help-auth-app-download-install/auth-app-accounts.png)

## <a name="next-steps"></a>Próximas etapas

- Depois de adicionar as contas ao aplicativo, você poderá entrar usando o aplicativo Authenticator em seu dispositivo. Para obter mais informações, confira [Entrar usando o aplicativo](user-help-auth-app-sign-in.md).

- Para dispositivos que executam o iOS, você também pode fazer backup das credenciais da sua conta e das configurações de aplicativos relacionadas, tais como a ordem de suas contas, na nuvem. Para obter mais informações, confira [Fazer backup e recuperar informações com o aplicativo Microsoft Authenticator](user-help-auth-app-backup-recovery.md).
