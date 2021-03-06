---
title: Solucionar problemas de Proxy de Aplicativo | Microsoft Docs
description: Aborda como solucionar erros no Proxy de Aplicativo do Azure do AD.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/24/2019
ms.author: mimart
ms.reviewer: japere
ms.custom: H1Hack27Feb2017; it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7be9a17bed2a39d16f813332c2d6effc03393264
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79244219"
---
# <a name="troubleshoot-application-proxy-problems-and-error-messages"></a>Solucionar problemas e mensagens de erro do Proxy do Aplicativo

Ao solucionar problemas de proxy do aplicativo, recomendamos que você comece a revisar o fluxo de solução [de problemas, problemas do Conector proxy do aplicativo de depuração,](application-proxy-debug-connectors.md)para determinar se os conectores proxy do aplicativo estão configurados corretamente. Se você ainda estiver tendo problemas para se conectar ao aplicativo, siga o fluxo de solução de problemas em [problemas do aplicativo Debug Application Proxy](application-proxy-debug-apps.md).

Se ocorrerem erros ao acessar um aplicativo publicado ou em aplicativos de publicação, verifique as seguintes opções para ver se o Proxy de Aplicativo do AD do Microsoft Azure está funcionando corretamente:

* Abra o console do Windows Services. Verifique se o serviço **de proxy proxy do aplicativo AAD da Microsoft** está ativado e em execução. Também convém examinar a página de propriedades do serviço de Proxy de Aplicativo, conforme mostrado na imagem a seguir:   
  ![Captura de tela da janela Propriedades do Conector de Proxy de Aplicativo do Microsoft AAD](./media/application-proxy-troubleshoot/connectorproperties.png)
* Visualizador de eventos abertos e procure eventos de conector proxy de aplicativos em **aplicativos e serviços Logs** > **Microsoft** > **AadApplicationProxy** > **Connector** > **AdMin**.
* Se necessário, logs mais detalhados estão disponíveis via [ativação dos logs de sessão do conector do Proxy de Aplicativo](application-proxy-connectors.md#under-the-hood).

## <a name="the-page-is-not-rendered-correctly"></a>A página não é renderizada corretamente
Você pode ter problemas com a renderização do aplicativo ou de funcionamento incorreto sem receber mensagens específicas de erro. Isso pode ocorrer se você publicou o caminho do artigo, mas o aplicativo requer o conteúdo que existe fora desse caminho.

Por exemplo, se você publicar o caminho `https://yourapp/app`, mas o aplicativo chamar imagens em `https://yourapp/media`, elas não serão renderizadas. Publique o aplicativo usando o caminho de nível mais alto de que você precisa para incluir todo o conteúdo relevante. Neste exemplo, ele seria `http://yourapp/`.

Se você alterar o caminho para incluir o conteúdo referenciado, mas ainda precisar que os usuários cheguem a um link mais profundo, confira a postagem de blog [Setting the right link for Application Proxy applications in the Azure AD access panel and Office 365 app launcher (Definindo o link certo para aplicativos do Proxy de Aplicativo no painel de acesso do Azure AD e no inicializador de aplicativos do Office 365)](https://blogs.technet.microsoft.com/applicationproxyblog/2016/04/06/setting-the-right-link-for-application-proxy-applications-in-the-azure-ad-access-panel-and-office-365-app-launcher/).

## <a name="connector-errors"></a>Erros de conector

Se o registro falhar durante a instalação do assistente do Conector, há duas maneiras de exibir o motivo da falha. Procure no registro de eventos em **Registros e Serviços Logs\Microsoft\AadApplicationProxy\Connector\Admin**ou execute o seguinte comando Do Windows PowerShell:

    Get-EventLog application –source "Microsoft AAD Application Proxy Connector" –EntryType "Error" –Newest 1

Depois de encontrar o erro do Conector no log de eventos, use esta tabela de erros comuns para resolver o problema:

| Erro | Etapas recomendadas |
| ----- | ----------------- |
| Falha no registro de conector: verifique se você habilitou o Proxy de Aplicativo no Portal de Gerenciamento do Azure e se inseriu o nome de usuário e a senha do Active Directory corretamente. Erro: "ocorreram um ou mais erros”. | Se você fechou a janela de registro sem entrar no Azure AD, execute o assistente do Conector novamente e registrar o Conector. <br><br> Se a janela de registro abrir e fechar imediatamente sem permitir que você faça login, você provavelmente terá esse erro. Esse erro ocorre quando há algum erro de rede em seu sistema. Certifique-se de que é possível se conectar de um navegador a um site público e que as portas estão abertas conforme especificado nos [pré-requisitos](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment)do Proxy do aplicativo . |
| Apagar erro é apresentado na janela de registro. Não é possível continuar | Caso esse erro seja exibido e a janela fechar, você inseriu o nome de usuário e a senha incorretos. Tente novamente. |
| Falha no registro de conector: verifique se você habilitou o Proxy de Aplicativo no Portal de Gerenciamento do Azure e se inseriu o nome de usuário e a senha do Active Directory corretamente. Erro: ‘AADSTS50059: nenhuma informação de identificação de organização foi encontrada na solicitação nem está implícita em quaisquer credenciais fornecidas; a pesquisa por URI de entidade de serviço falhou. | Você está tentando entrar usando uma Conta Microsoft e não um domínio que faz parte do ID da organização do diretório que você está tentando acessar. Certifique-se de que o administrador faça parte do mesmo nome de domínio que o domínio do locatário; por exemplo, se o domínio do AD do Azure for contoso.com, o do administrador deverá ser admin@contoso.com. |
| Falha ao recuperar a política de execução atual para executar scripts do PowerShell. | Se a instalação do Conector falhar, verifique se a política de execução do PowerShell não está desativada. <br><br>1. Abra o Editor de Políticas do Grupo.<br>2. Vá para**modelos administrativos** > de **configuração** > de computador**Windows Components** > **Windows PowerShell** e clique duas vezes em Ativar **execução de script**.<br>3. A política de execução pode ser definida como **Não Configurada** ou **Habilitada**. Se estiver definido como **Habilitado**, verifique se a Política de Execução em Opções está definida como **Permitir scripts locais e scripts remotos assinados** ou como **Permitir todos os scripts**. |
| Falha ao baixar a configuração do Conector. | O certificado de cliente do Conector, que é usado para autenticação, expirou. Isso também pode ocorrer se você tiver o Conector instalado atrás de um proxy. Nesse caso, o Conector não poderá acessar a Internet e não será capaz de fornecer aplicativos a usuários remotos. Renove a confiança manualmente usando o cmdlet `Register-AppProxyConnector` do Windows PowerShell. Se seu Conector estiver atrás de um proxy, será necessário conceder acesso à Internet para as contas "serviços de rede" e "sistema local" do Conector. Isso pode ser feito concedendo acesso ao Proxy ou configurando-os para ignorar o proxy. |
| Falha no registro do conector: Certifique-se de que você é um administrador de aplicativos do seu Diretório Ativo para registrar o Conector. Erro: “a solicitação de registro foi negada”. | O alias com o qual você está tentando fazer logon não é um administrador neste domínio. O Conector sempre é instalado para o diretório que possui o domínio do usuário. Certifique-se de que a conta de administração com a qual você está tentando entrar tem pelo menos permissões de administrador de aplicativos para o inquilino azure AD. |
| O Conector não pôde se conectar ao serviço devido a problemas de rede. O Conector tentou acessar a SEGUINTE URL. | O conector não consegue se conectar ao serviço de nuvem proxy do aplicativo. Isso pode acontecer se você tiver uma regra de firewall bloqueando a conexão. Certifique-se de que você permitiu o acesso às portas e URLS corretos listados nos [pré-requisitos do Proxy do aplicativo](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment). |

## <a name="kerberos-errors"></a>Erros de Kerberos

Esta tabela cobre os erros mais comuns resultantes da instalação e configuração do Kerberos e inclui sugestões para resolução.

| Erro | Etapas recomendadas |
| ----- | ----------------- |
| Falha ao recuperar a política de execução atual para executar scripts do PowerShell. | Se a instalação do Conector falhar, verifique se a política de execução do PowerShell não está desabilitada.<br><br>1. Abra o Editor de Políticas do Grupo.<br>2. Vá para**modelos administrativos** > de **configuração** > de computador**Windows Components** > **Windows PowerShell** e clique duas vezes em Ativar **execução de script**.<br>3. A política de execução pode ser definida como **Não Configurada** ou **Habilitada**. Se estiver definido como **Habilitado**, verifique se a Política de Execução em Opções está definida como **Permitir scripts locais e scripts remotos assinados** ou como **Permitir todos os scripts**. |
| 12008 - O Azure AD excedeu o número máximo de tentativas de autenticação Kerberos permitidas para o servidor back-end. | Esse erro pode indicar uma configuração incorreta entre o Azure AD e o servidor de aplicativos back-end ou um problema na configuração de data e hora nos dois computadores. O servidor back-end recusou o tíquete Kerberos criado pelo AD do Azure. Verifique se o Azure AD e o servidor de aplicativos de back-end estão configurados corretamente. Verifique se a configuração de data e hora no AD do Azure e no servidor de aplicativos back-end estão sincronizadas. |
| 13016 - O AD do Azure não pode recuperar um tíquete Kerberos em nome do usuário porque não há nenhum UPN no token de borda ou no cookie de acesso. | Há um problema com a configuração de STS. Corrija a configuração de declaração UPN no STS. |
| 13019 - O AD do Azure não pode recuperar um tíquete Kerberos em nome do usuário devido ao seguinte erro geral da API. | Esse evento pode indicar uma configuração incorreta entre o AD do Azure e o servidor de controlador de domínio ou um problema na configuração de data e hora nos dois computadores. O controlador de domínio recusou o tíquete Kerberos criado pelo AD do Azure. Verifique se o Azure AD e do servidor de aplicativos de back-end estão configurados corretamente, especialmente a configuração de SPN. Verifique se que o AD do Azure é o domínio ingressado no mesmo domínio que o controlador de domínio para garantir que este estabeleça a relação de confiança com o AD do Azure. Verifique se a configuração de data e hora no AD do Azure e no controlador de domínio estão sincronizadas. |
| 13020 - O AD do Azure não pode recuperar um tíquete Kerberos em nome do usuário porque o SPN do servidor de back-end não está definido. | Esse evento pode indicar uma configuração incorreta entre o AD do Azure e o servidor de controlador de domínio ou um problema na configuração de data e hora nos dois computadores. O controlador de domínio recusou o tíquete Kerberos criado pelo AD do Azure. Verifique se o Azure AD e do servidor de aplicativos de back-end estão configurados corretamente, especialmente a configuração de SPN. Verifique se que o AD do Azure é o domínio ingressado no mesmo domínio que o controlador de domínio para garantir que este estabeleça a relação de confiança com o AD do Azure. Verifique se a configuração de data e hora no AD do Azure e no controlador de domínio estão sincronizadas. |
| 13022 - O AD do Azure não pode autenticar o usuário porque o servidor de back-end responde às tentativas de autenticação Kerberos com um erro HTTP 401. | Esse evento pode indicar uma configuração incorreta entre o AD do Azure e o servidor de aplicativos de back-end ou um problema na configuração de data e hora nos dois computadores. O servidor back-end recusou o tíquete Kerberos criado pelo AD do Azure. Verifique se o Azure AD e o servidor de aplicativos de back-end estão configurados corretamente. Verifique se a configuração de data e hora no AD do Azure e no servidor de aplicativos back-end estão sincronizadas. Para obter mais informações, consulte [Solucionar problemas de configurações de delegação restrita de Kerberos para proxy de aplicativo](application-proxy-back-end-kerberos-constrained-delegation-how-to.md).  |

## <a name="end-user-errors"></a>Erros de usuário final

Esta lista cobre os erros que os usuários finais podem encontrar quando tentam acessar o aplicativo e falham. 

| Erro | Etapas recomendadas |
| ----- | ----------------- |
| O site não pode exibir a página. | O usuário poderá receber esse erro ao tentar acessar o aplicativo publicado se o aplicativo for um aplicativo IWA. O SPN definido para esse aplicativo pode estar incorreto. Para aplicativos IWA, certifique-se de que o SPN configurado para este aplicativo esteja correto. |
| O site não pode exibir a página. | O usuário poderá ver esse erro ao tentar acessar o aplicativo publicado se o aplicativo for um aplicativo OWA. Isso pode ser causado por um dos seguintes motivos: <br><li>O SPN definido para este aplicativo está incorreto. Certifique-se de que o SPN configurado para este aplicativo esteja correto.</li><li>O usuário que tentou acessar o aplicativo está usando uma conta da Microsoft em vez da conta corporativa apropriada para entrar, ou o usuário é um usuário convidado. Verifique se o usuário faz logon usando sua conta corporativa correspondente ao domínio do aplicativo publicado. Convidados e usuários de Conta da Microsoft não podem acessar aplicativos IWA.</li><li>O usuário que tentou acessar o aplicativo não está devidamente definido para este aplicativo no lado das instalações. Certifique-se de que este usuário tenha as permissões adequadas definidas para este aplicativo back-end na máquina de instalações on. |
| Não foi possível acessar este aplicativo corporativo. Você não está autorizado a acessar este aplicativo. Falha na autorização. Certifique-se de atribuir o acesso a este aplicativo ao usuário. | Seu usuário pode ter esse erro ao tentar acessar o aplicativo que você publicou se eles usarem contas da Microsoft em vez de sua conta corporativa para fazer login. Os usuários convidados também podem receber esse erro. Convidados e usuários de Conta da Microsoft não podem acessar aplicativos IWA. Verifique se o usuário faz logon usando sua conta corporativa correspondente ao domínio do aplicativo publicado.<br><br>Você pode não ter atribuído o usuário para esse aplicativo. Vá para a guia **Aplicativo** e, em **Usuários e Grupos**, atribua esse usuário ou grupo de usuários a esse aplicativo. |
| Não foi possível acessar esse aplicativo corporativo no momento. Tente novamente mais tarde... O conector atingiu o tempo limite. | Seu usuário pode ter esse erro ao tentar acessar o aplicativo que você publicou se eles não estiverem devidamente definidos para este aplicativo no lado local. Certifique-se de que seus usuários tenham as permissões adequadas definidas para este aplicativo back-end na máquina de instalações on. |
| Não foi possível acessar este aplicativo corporativo. Você não está autorizado a acessar este aplicativo. Falha na autorização. Certifique-se de que o usuário tenha uma licença para o Azure Active Directory Premium. | Seu usuário pode ter esse erro ao tentar acessar o aplicativo que você publicou se eles não foram explicitamente atribuídos com uma licença Premium pelo administrador do assinante. Vá para a guia Active Directory **Licenses** do assinante e certifique-se de que este usuário ou grupo de usuários tenha uma licença Premium. |
| Um servidor com o nome de host especificado não foi encontrado. | Seu usuário pode ter esse erro ao tentar acessar o aplicativo que você publicou se o domínio personalizado do aplicativo não estiver configurado corretamente. Certifique-se de que você carregou um certificado para o domínio e configurou o registro DNS corretamente seguindo as etapas em [Trabalhar com domínios personalizados no Proxy de aplicativo Azure AD](application-proxy-configure-custom-domain.md) |

## <a name="my-error-wasnt-listed-here"></a>Meu erro não estava listado aqui

Se você encontrar um erro ou problema com o Proxy de Aplicativo do Azure AD que não está listado neste guia de solução de problemas, conte-nos. Envie um email para nossa [equipe de comentários](mailto:aadapfeedback@microsoft.com) com os detalhes do erro encontrado.

## <a name="see-also"></a>Confira também
* [Habilitar o Proxy de Aplicativo para o Azure Active Directory](application-proxy-add-on-premises-application.md)
* [Publique aplicativos com proxy de aplicativo](application-proxy-add-on-premises-application.md)
* [Habilitar logon único](application-proxy-configure-single-sign-on-with-kcd.md)
* [Habilitar acesso condicional](application-proxy-integrate-with-sharepoint-server.md)


<!--Image references-->
[1]: ./media/application-proxy-troubleshoot/connectorproperties.png
[2]: ./media/active-directory-application-proxy-troubleshoot/sessionlog.png
