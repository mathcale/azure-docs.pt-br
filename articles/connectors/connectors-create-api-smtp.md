---
title: Conecte-se ao SMTP a partir de aplicativos azure logic
description: Automatizar tarefas e fluxos de trabalho que enviam email por meio de sua conta de protocolo SMTP (SMTP) usando os Aplicativos Lógicos do Azure
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
tags: connectors
ms.openlocfilehash: 60acd128495176cd0a90418c61edf53bdcd88e5a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77647572"
---
# <a name="send-email-from-your-smtp-account-with-azure-logic-apps"></a>Enviar um email de sua conta SMTP com Aplicativos Lógicos do Azure

Com os Aplicativos Lógicos do Azure e o conector de protocolo SMTP (SMTP), você pode criar tarefas automatizadas e fluxos de trabalho que enviam email usando sua conta SMTP. Você também pode fazer com que outras ações usem a saída das ações de SMTP. Por exemplo, depois que o SMTP envia um email, você pode notificar sua equipe no Slack com o conector do Slack. Se você é novo em aplicativos lógicos, [revise o que é o Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

## <a name="prerequisites"></a>Pré-requisitos

* Uma assinatura do Azure. Se você não tiver uma assinatura do Azure, [inscreva-se em uma conta gratuita do Azure](https://azure.microsoft.com/free/). 

* Suas credenciais de usuário e conta SMTP

  Suas credenciais autorizam o aplicativo lógico a criar uma conexão e acessar sua conta SMTP.

* Conhecimento básico sobre [como criar aplicativos lógicos](../logic-apps/quickstart-create-first-logic-app-workflow.md)

* O aplicativo lógico no qual você deseja acessar a conta SMTP. Para usar uma ação de SMTP, inicie o aplicativo lógico com um gatilho, como um gatilho do Salesforce se você tiver uma conta do Salesforce.

  Por exemplo, você pode iniciar o aplicativo lógico com o gatilho **Quando um registro é criado** do Salesforce. 
  Esse gatilho é acionado sempre que um novo registro, como um lead, for criado no Salesforce. 
  Então, você pode dar sequência neste gatilho com a ação de SMTP **Enviar Email**. Dessa forma, quando o novo registro for criado, seu aplicativo lógico enviará um email a respeito do novo registro usando sua conta SMTP.

## <a name="connect-to-smtp"></a>Conectar-se ao SMTP

[!INCLUDE [Create connection general intro](../../includes/connectors-create-connection-general-intro.md)]

1. Entre no [portal do Azure](https://portal.azure.com) e abra seu aplicativo lógico no Designer de Aplicativo Lógico, se ele ainda não estiver aberto.

1. Na última etapa em que você deseja adicionar uma ação de SMTP, escolha **Nova etapa**. 

   Para adicionar uma ação entre as etapas, mova o ponteiro sobre a seta entre as etapas. 
   Escolha o sinal**+** de adição () que aparece e, em seguida, **selecione Adicionar uma ação**.

1. Na caixa de pesquisa, insira "smtp" como filtro. Na lista de ações, selecione a ação desejada.

1. Quando solicitado, forneça essas informações de conexão:

   | Propriedade | Obrigatório | Descrição |
   |----------|----------|-------------|
   | **Nome da conexão** | Sim | Um nome para a conexão com seu servidor SMTP | 
   | **Endereço do Servidor SMTP** | Sim | O endereço do seu servidor SMTP | 
   | **Nome do usuário** | Sim | O nome de usuário da sua conta SMTP | 
   | **Senha** | Sim | A senha da sua conta SMTP | 
   | **Porta do Servidor SMTP** | Não | Uma porta específica no servidor SMTP que você deseja usar | 
   | **Habilitar SSL?** | Não | Ligar ou desligar a criptografia SSL. | 
   |||| 

1. Forneça os detalhes necessários para a ação selecionada. 

1. Salve seu aplicativo lógico ou continuar criando o fluxo de trabalho do aplicativo lógico.

## <a name="connector-reference"></a>Referência de conector

Para obter mais detalhes técnicos sobre este conector, como gatilhos, ações e limites descritos pelo arquivo Swagger do conector, consulte a [página de referência do conector](https://docs.microsoft.com/connectors/smtpconnector/).

> [!NOTE]
> Para aplicativos lógicos em um [ambiente de serviço de integração (ISE),](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md)a versão rotulada pelo conector ISE usa os limites de [mensagem ISE.](../logic-apps/logic-apps-limits-and-config.md#message-size-limits)

## <a name="next-steps"></a>Próximas etapas

* Saiba mais sobre outros [conectores de Aplicativos Lógicos](../connectors/apis-list.md)