---
title: Integração e implantação contínuas
description: Experiência de DevOps de banco de dados de classe corporativa para armazenamento de dados com suporte integrado para integração e implantação contínua usando pipelines Azure.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: ''
ms.date: 02/04/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: f73fbbc42085b3a0cb6ab7dd1c0e32838a166ccf
ms.sourcegitcommit: 8a9c54c82ab8f922be54fb2fcfd880815f25de77
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80350645"
---
# <a name="continuous-integration-and-deployment-for-data-warehousing"></a>Integração e implantação contínuas para armazenamento de dados

Este tutorial simples descreve como integrar seu projeto de banco de dados SSDT (SQL Server Data Tools) ao Azure DevOps e aproveitar o Azure Pipelines para configurar a integração e a implantação contínuas. Este tutorial é o segundo passo na construção do seu pipeline contínuo de integração e implantação para armazenamento de dados. 

## <a name="before-you-begin"></a>Antes de começar

- Percorra o [tutorial de integração de controle do código-fonte](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-source-control-integration)

- Configurar e conectar ao Azure DevOps


## <a name="continuous-integration-with-visual-studio-build"></a>Integração contínua ao build do Visual Studio

1. Navegue até o Azure Pipelines e crie um pipeline de build.

      ![Novo Gasoduto](./media/sql-data-warehouse-continuous-integration-and-deployment/1-new-build-pipeline.png "Novo Pipeline")

2. Selecione o repositório de código-fonte (Git do Azure Repos) e selecione o modelo de aplicativo da área de trabalho do .NET.

      ![Configuração do pipeline](./media/sql-data-warehouse-continuous-integration-and-deployment/2-pipeline-setup.png "Configuração do pipeline") 

3. Edite seu arquivo YAML para usar o pool adequado do seu agente. Seu arquivo YAML deve ter a seguinte aparência:

      ![YAML](./media/sql-data-warehouse-continuous-integration-and-deployment/3-yaml-file.png "YAML")

Neste momento, você tem um ambiente simples no qual qualquer check-in em seu branch mestre do repositório de controle do código-fonte deve disparar automaticamente um build do Visual Studio bem-sucedido do seu projeto de banco de dados. Confirme se a automação está funcionando de ponta a ponta fazendo uma alteração em seu projeto de banco de dados local e fazendo check-in dessa alteração no branch mestre.


## <a name="continuous-deployment-with-the-azure-sql-data-warehouse-or-database-deployment-task"></a>Implantação contínua com a tarefa de implantação do SQL Data Warehouse (ou Banco de Dados) do Azure

1. Adicione uma nova tarefa usando a [tarefa de implantação do Banco de Dados SQL do Azure](https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/sql-azure-dacpac-deployment?view=azure-devops) e preencha os campos obrigatórios para se conectar ao seu data warehouse de destino. Quando essa tarefa é executada, o DACPAC gerado com base no processo de build anterior é implantado no data warehouse de destino. Use também a [tarefa de implantação do SQL Data Warehouse do Azure](https://marketplace.visualstudio.com/items?itemName=ms-sql-dw.SQLDWDeployment). 

      ![Tarefa de implantação](./media/sql-data-warehouse-continuous-integration-and-deployment/4-deployment-task.png "Tarefa de implantação")

2. Se você estiver usando um agente auto-hospedado, verifique se você definiu sua variável de ambiente para usar o SqlPackage.exe correto para o SQL Data Warehouse. O caminho deve ter a seguinte aparência:

      ![Variável de ambiente](./media/sql-data-warehouse-continuous-integration-and-deployment/5-environment-variable-preview.png "Variável de ambiente")

   C:\Arquivos de Programas (x86)\Microsoft Visual Studio\2019\Preview\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\150  

   Execute e valide seu pipeline. Você pode fazer alterações localmente e check-in do controle do código-fonte que deve gerar um build e uma implantação automáticos.

## <a name="next-steps"></a>Próximas etapas

- Explore a [arquitetura MPP do SQL Analytics](massively-parallel-processing-mpp-architecture.md)
- [Criar rapidamente um pool de SQL](create-data-warehouse-portal.md)
- [Carregar dados da amostra](load-data-from-azure-blob-storage-using-polybase.md)
- Explore os [Vídeos](/azure/sql-data-warehouse/sql-data-warehouse-videos)
