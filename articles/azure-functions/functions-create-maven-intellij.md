---
title: Crie uma função Azure com Java e IntelliJ
description: Saiba como criar e publicar um aplicativo sem servidor simples disparado por HTTP no Azure com Java e IntelliJ.
author: jeffhollan
ms.topic: conceptual
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter
ms.openlocfilehash: 7003dc19a7bfc405809de91534028aba8e0416c5
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79136843"
---
# <a name="create-your-first-azure-function-with-java-and-intellij"></a>Crie sua primeira função Azure com Java e IntelliJ

Este artigo mostra:
- Como criar um projeto de função [sem servidor](https://azure.microsoft.com/overview/serverless-computing/) com IntelliJ IDEA e Apache Maven
- Etapas para testar e depurar a função no IDE (ambiente de desenvolvimento integrado) em seu próprio computador
- Instruções para implantar o projeto de função para o Azure Functions

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurar seu ambiente de desenvolvimento

Para desenvolver uma função com Java e IntelliJ, instale o software a seguir:

- [Java Developer Kit](https://www.azul.com/downloads/zulu/) (JDK), versão 8
- [Apache Maven](https://maven.apache.org), versão 3.0 ou superior
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download), versões Community ou Ultimate com Maven
- [Azure CLI](https://docs.microsoft.com/cli/azure)

> [!IMPORTANT]
> A variável de ambiente JAVA_HOME deve ser definida como o local de instalação do JDK para concluir as etapas neste artigo.

 É recomendável que você instale o [Azure Functions Core Tools, versão 2](functions-run-local.md#v2). Ele fornece um ambiente de desenvolvimento local para escrever, executar e depurar o Azure Functions.

## <a name="create-a-functions-project"></a>Criar um projeto do Functions

1. No IntelliJ IDEA, selecione **Criar Novo Projeto**.  
1. Na janela **Novo Projeto**, selecione **Maven** no painel esquerdo.
1. Marque a caixa de seleção **Criar do arquétipo** e, em seguida, selecione **Adicionar Arquétipo** para [azure-functions-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype).
1. Na janela **Adicionar Arquétipo**, preencha os campos da seguinte maneira:
    - _GroupId_: com.microsoft.azure
    - _ArtefatoId_: azure-funções-arquétipo
    - _Versão_: Verifique e use a versão mais recente [do repositório](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype)
    ![central Crie um projeto Maven a partir do arquétipo na IntelliJ IDEA](media/functions-create-first-java-intellij/functions-create-intellij.png)  
1. Selecione **OK**e selecione **Next**.
1. Insira os detalhes do projeto atual e selecione **Concluir**.

Maven cria os arquivos de projeto em uma nova pasta com o mesmo nome que o valor _ArtifactId_. O código gerado no projeto é uma função [Disparada por HTTP](/azure/azure-functions/functions-bindings-http-webhook) simples que ecoa o corpo da solicitação HTTP que está sendo disparada.

## <a name="run-functions-locally-in-the-ide"></a>Executar funções localmente no IDE

> [!NOTE]
> Para executar e depurar funções localmente, verifique se você instalou o [Azure Functions Core Tools, versão 2](functions-run-local.md#v2).

1. Importe as alterações manualmente ou habilite a [importação automática](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html).
1. Abra a barra de ferramentas da **Maven Projects.**
1. Expanda **Ciclo de Vida** e, em seguida, abra o **pacote**. A solução é compilada e empacotada em um diretório de destino criado recentemente.
1. Expanda as**funções azure do** **Plugine** > e abra **funções azure:execute** para iniciar o tempo de execução local do Azure Functions.  
  ![Barra de ferramentas Maven para Azure Functions](media/functions-create-first-java-intellij/functions-intellij-java-maven-toolbar.png)  

1. Feche a caixa de diálogo de execução quando terminar de testar a função. Apenas um host de função pode estar ativo e em execução localmente de cada vez.

## <a name="debug-the-function-in-intellij"></a>Depurar a função no IntelliJ

1. Para iniciar o host de função no modo de depuração, adicione **-DenableDebug** como argumento quando ao executar sua função. Você pode alterar a configuração em [metas do maven](https://www.jetbrains.com/help/idea/maven-support.html#run_goal) ou execute o seguinte comando em uma janela de terminal:  

   ```
   mvn azure-functions:run -DenableDebug
   ```

   Esse comando faz com que o host de função abra uma porta de depuração em 5005.

1. No menu **Executar**, selecione **Editar Configurações**.
1. Selecione **(+)** para adicionar um **Remoto**.
1. Preencha os campos _Nome_ e _Configurações_ e, em seguida, selecione **OK** para salvar a configuração.
1. Após a instalação, selecione **Depurar < Nome da Configuração Remota >** ou pressione Shift+F9 em seu teclado para iniciar a depuração.

1. Ao terminar, pare o depurador e o processo em execução. Apenas um host de função pode estar ativo e em execução localmente de cada vez.

## <a name="deploy-the-function-to-azure"></a>Implantar a função no Azure

1. Antes de implantar sua função no Azure, você deve [fazer login usando o Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

   ``` azurecli
   az login
   ```

1. Implante seu código em uma nova função usando o destino do Maven `azure-functions:deploy`. Você também pode selecionar a opção **azure-functions:deploy** na janela de projetos do Maven.

   ```
   mvn azure-functions:deploy
   ```

1. Encontre a URL para sua função na saída da CLI do Azure depois que a função tiver sido implantada com êxito.

   ``` output
   [INFO] Successfully deployed Function App with package.
   [INFO] Deleting deployment package from Azure Storage...
   [INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
   [INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
   [INFO] ------------------------------------------------------------------------
   ```

## <a name="next-steps"></a>Próximas etapas

- Examine o [Guia do desenvolvedor de funções Java](functions-reference-java.md) para saber mais sobre o desenvolvimento de funções Java.
- Adicione outras funções com gatilhos diferentes ao seu projeto usando o destino do Maven `azure-functions:add`.
