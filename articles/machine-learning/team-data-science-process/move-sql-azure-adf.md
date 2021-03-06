---
title: Dados do SQL Server para SQL Azure com o Azure Data Factory - Processo de Ciência de Dados de Equipe
description: Configure um pipeline do ADF que compõe duas atividades de migração de dados que movem os dados juntos diariamente entre bancos de dados locais e na nuvem.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 8f696f1c6c414cd9db082e79e0f34c56156e1ee0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "76722485"
---
# <a name="move-data-from-an-on-premises-sql-server-to-sql-azure-with-azure-data-factory"></a>Mover dados de um SQL Server local para o SQL Azure com o Azure Data Factory

Este artigo mostra como mover dados de um banco de dados sql server local para um banco de dados SQL Azure via Azure Blob Storage usando a Fábrica de Dados Azure (ADF): este método é uma abordagem herdada suportada que tem as vantagens de uma cópia de encenação replicada, embora [sugerimos olhar nossa página de migração de dados para as opções mais recentes](https://datamigration.microsoft.com/scenario/sql-to-azuresqldb?step=1).

Para conferir uma tabela que resume diversas opções de movimentação de dados para um Banco de Dados SQL do Azure, consulte [Mover dados para um Banco de Dados SQL do Azure para o Azure Machine Learning](move-sql-azure.md).

## <a name="introduction-what-is-adf-and-when-should-it-be-used-to-migrate-data"></a><a name="intro"></a>Introdução: O que é o ADF e quando ele deve ser usado para migrar dados?
O Azure Data Factory é um serviço de integração de dados baseado em nuvem que automatiza a movimentação e a transformação dos dados. O conceito fundamental no modelo do ADF é o pipeline. Um pipeline é um agrupamento lógico de Atividades, e cada uma delas define as ações a serem executadas nos dados contidos em Conjuntos de dados. Serviços vinculados definem as informações necessárias para o Data Factory conectar-se a recursos de dados.

Com o ADF, serviços de processamento de dados existentes podem ser compostos em pipelines de dados que são altamente disponíveis e gerenciados na nuvem. Esses pipelines de dados podem ser agendados para ingerir, preparar, transformar, analisar e publicar os dados, e o ADF gerencia e orquestra os dados complexos e dependências de processamento. As soluções podem ser criadas e implantadas rapidamente na nuvem, conectando um número crescente de fontes de dados em nuvem e locais.

Considere usar o ADF:

* quando dados precisarem ser migrados continuamente em um cenário híbrido que acessa recursos locais e de nuvem
* quando os dados precisam de transformações ou têm a lógica de negócios adicionada a ele quando são migrados.

O ADF permite o planejamento e monitoramento de trabalhos usando scripts simples de JSON que gerenciam a movimentação de dados em intervalos periódicos. O ADF também possui outros recursos, como suporte para operações complexas. Para obter mais informações sobre o ADF, consulte a documentação em [Azure Data Factory (ADF)](https://azure.microsoft.com/services/data-factory/).

## <a name="the-scenario"></a><a name="scenario"></a>O cenário
Criamos um pipeline do ADF que compõe duas atividades de migração de dados. Juntos, eles movem dados diariamente entre um banco de dados SQL no local e um Banco de Dados SQL do Azure na nuvem. As duas atividades são:

* copiar os dados de um banco de dados de SQL Server local para uma conta de Armazenamento de Blobs do Azure
* copiar dados da conta de armazenamento de blob do Azure para um banco de dados SQL do Azure.

> [!NOTE]
> As etapas aqui mostradas foram adaptadas a partir do tutorial mais detalhado fornecido pela equipe do ADF: [Copiar dados de um banco de dados SQL Server no local para o armazenamento Do Azure Blob](https://docs.microsoft.com/azure/data-factory/tutorial-hybrid-copy-portal/) As referências às seções relevantes desse tópico são fornecidas quando apropriado.
>
>

## <a name="prerequisites"></a><a name="prereqs"></a>Pré-requisitos
Este tutorial presume que você tenha:

* Uma **assinatura do Azure.** Se você não tiver uma assinatura, você pode se inscrever em uma [avaliação gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Uma **conta de armazenamento Azure.** Você usará uma conta de armazenamento do Azure para armazenar os dados neste tutorial. Se você não tiver uma conta de armazenamento do Azure, consulte o artigo [Criar uma conta de armazenamento](../../storage/common/storage-account-create.md) . Depois de criar a conta de armazenamento, você precisa obter a chave de conta usada para acessar o armazenamento. Consulte [Gerenciar as chaves de acesso da conta de armazenamento](../../storage/common/storage-account-keys-manage.md).
* Acesso a um **Banco de dados SQL do Azure**. Se você deve configurar um Banco de Dados SQL do Azure, o tópico [Getting Started with Microsoft Azure SQL Database](../../sql-database/sql-database-get-started.md) fornece informações sobre como fornecer uma nova instância de um Banco de Dados SQL do Azure.
* **Azure PowerShell** instalado e configurado localmente. Para saber mais, confira [Como instalar e configurar o PowerShell do Azure](/powershell/azure/overview).

> [!NOTE]
> Este procedimento usa o [Portal do Azure](https://portal.azure.com/).
>
>

## <a name="upload-the-data-to-your-on-premises-sql-server"></a><a name="upload-data"></a> Carregar os dados para o SQL Server local
Usamos o [conjunto de dados de Táxi de NYC](https://chriswhong.com/open-data/foil_nyc_taxi/) para demonstrar o processo de migração. O conjunto de dados de Táxi de NYC está disponível, como observado nessa postagem, nos [Dados de Táxi de NYC](https://www.andresmh.com/nyctaxitrips/)do armazenamento de blobs do Azure. Os dados têm dois arquivos, o arquivo trip_data.csv que contém detalhes da viagem e o arquivo trip_far.csv que contém detalhes das tarifas pagas para cada viagem. Um exemplo e uma descrição desses arquivos são fornecidos na [Descrição do Conjunto de Dados de Viagens de Táxi de NYC](sql-walkthrough.md#dataset).

Você pode adaptar o procedimento fornecido aqui para um conjunto de seus próprios dados ou seguir as etapas conforme descrito usando o conjunto de dados de Táxi de NYC. Para carregar o conjunto de dados de Táxi de NYC em seu banco de dados do SQL Server local, siga o procedimento descrito em [Importação de dados em massa para o Banco de Dados do SQL Server](sql-walkthrough.md#dbload). Essas instruções são para um SQL Server em uma máquina virtual do Azure, mas o procedimento para carregar o SQL Server local é o mesmo.

## <a name="create-an-azure-data-factory"></a><a name="create-adf"></a>Criar uma fábrica de dados do Azure
As instruções para criar um novo Azure Data Factory e um grupo de recursos no [Portal do Azure](https://portal.azure.com/) são fornecidas em [Create an Azure Data Factory (Criar um Azure Data Factory)](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-data-factory). Nomeie a nova instância ADF como *adfdsp* e nomeie o grupo de recursos criado como *adfdsprg*.

## <a name="install-and-configure-azure-data-factory-integration-runtime"></a>Instalar e configurar o Microsoft Integration Runtime do Azure Data Factory
O Integration Runtime é uma infra-estrutura de integração de dados gerenciada pelo cliente usada pela Fábrica de Dados Do Azure para fornecer recursos de integração de dados em diferentes ambientes de rede. Esse runtime foi denominado "Gateway de Gerenciamento de Dados".

Para configurar, [siga as instruções para criar um gasoduto](https://docs.microsoft.com/azure/data-factory/tutorial-hybrid-copy-portal#create-a-pipeline)

## <a name="create-linked-services-to-connect-to-the-data-resources"></a><a name="adflinkedservices"></a>Criar serviços vinculados para conectar-se aos recursos de dados
Um serviço vinculado define as informações necessárias para o Azure Data Factory conectar-se a um recurso de dados. Temos três recursos neste cenário para os quais os serviços vinculados são necessários:

1. SQL Server local
2. Armazenamento do Blobs do Azure
3. Banco de Dados SQL do Azure

O procedimento passo a passo para criar serviços vinculados é fornecido em [Create linked services (Criar serviços vinculados)](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).


## <a name="define-and-create-tables-to-specify-how-to-access-the-datasets"></a><a name="adf-tables"></a>Definir e criar tabelas para especificar como acessar os conjuntos de dados
Crie tabelas que especificam a estrutura, a localização e a disponibilidade dos conjuntos de dados com os procedimentos a seguir baseados em script. Os arquivos JSON são usados para definir as tabelas. Para obter mais informações sobre a estrutura desses arquivos, consulte [Conjuntos de dados](../../data-factory/concepts-datasets-linked-services.md).

> [!NOTE]
> Você deve executar o cmdlet `Add-AzureAccount` antes de executar o cmdlet [New-AzureDataFactoryTable](https://msdn.microsoft.com/library/azure/dn835096.aspx) para confirmar que a assinatura correta do Azure esteja selecionada para a execução do comando. Para obter a documentação desse cmdlet, consulte [Add-AzureAccount](/powershell/module/servicemanagement/azure/add-azureaccount?view=azuresmps-3.7.0).
>
>

As definições baseadas em JSON nas tabelas usam os seguintes nomes:

* o **nome da tabela** no SQL Server local é *nyctaxi_data*
* o **nome do contêiner** na conta de armazenamento de Blob do Azure é *containername\\\\\\\\\\*

Três definições de tabela são necessárias para este pipeline do ADF:

1. [Tabela do SQL local](#adf-table-onprem-sql)
2. [Mesa Blob](#adf-table-blob-store)
3. [Tabela SQL Azure](#adf-table-azure-sql)

> [!NOTE]
> Estes procedimentos usam o Azure PowerShell para definir e criar as atividades do ADF. Mas essas tarefas também podem ser realizadas pelo Portal do Azure. Para obter detalhes, consulte [Criar conjuntos de dados](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

### <a name="sql-on-premises-table"></a><a name="adf-table-onprem-sql"></a>Tabela do SQL local
A definição da tabela do SQL Server local é especificada no seguinte arquivo JSON:

```json
{
    "name": "OnPremSQLTable",
    "properties":
    {
        "location":
        {
            "type": "OnPremisesSqlServerTableLocation",
            "tableName": "nyctaxi_data",
            "linkedServiceName": "adfonpremsql"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1,
            "waitOnExternal":
            {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```

Os nomes de coluna não foram incluídos aqui. Você pode subselecionar os nomes da coluna incluindo-os aqui (para obter detalhes, verifique o tópico de documentação do [ADF.](../../data-factory/copy-activity-overview.md)

Copie a definição de JSON da tabela em um arquivo chamado *onpremtabledef.json* e salve-o em um local conhecido (neste caso deve ser *C:\temp\onpremtabledef.json*). Crie a tabela no ADF com o seguinte cmdlet do Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp –File C:\temp\onpremtabledef.json


### <a name="blob-table"></a><a name="adf-table-blob-store"></a>Tabela de blob 
A definição da tabela para o local do blob de saída está a seguir (isso mapeia os dados ingeridos localmente para o blob do Azure):

```json
{
    "name": "OutputBlobTable",
    "properties":
    {
        "location":
        {
            "type": "AzureBlobLocation",
            "folderPath": "containername",
            "format":
            {
                "type": "TextFormat",
                "columnDelimiter": "\t"
            },
            "linkedServiceName": "adfds"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Copie a definição de JSON da tabela para um arquivo chamado *bloboutputtabledef.json* e salve-o em um local conhecido (neste caso deve ser *C:\temp\bloboutputtabledef.json*). Crie a tabela no ADF com o seguinte cmdlet do Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\bloboutputtabledef.json

### <a name="sql-azure-table"></a><a name="adf-table-azure-sql"></a>Tabela do SQL Azure
A definição da tabela para a saída do SQL Azure está a seguir (esse esquema mapeia os dados provenientes do blob):

```json
{
    "name": "OutputSQLAzureTable",
    "properties":
    {
        "structure":
        [
            { "name": "column1", "type": "String"},
            { "name": "column2", "type": "String"}
        ],
        "location":
        {
            "type": "AzureSqlTableLocation",
            "tableName": "your_db_name",
            "linkedServiceName": "adfdssqlazure_linked_servicename"
        },
        "availability":
        {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

Copie a definição de JSON da tabela em um arquivo chamado *AzureSqlTable.json* e salve-o em um local conhecido (neste caso deve ser *C:\temp\AzureSqlTable.json*). Crie a tabela no ADF com o seguinte cmdlet do Azure PowerShell:

    New-AzureDataFactoryTable -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\AzureSqlTable.json


## <a name="define-and-create-the-pipeline"></a><a name="adf-pipeline"></a>Definir e criar o pipeline
Especifique as atividades que pertencem ao pipeline e crie o pipeline com os procedimentos a seguir baseados em script. Um arquivo JSON é usado para definir as propriedades de pipeline.

* O script assume que o **nome do pipeline** é *AMLDSProcessPipeline*.
* Observe também que definimos a periodicidade do pipeline para ser executado diariamente e usar o tempo de execução padrão para o trabalho (12:00 UTC).

> [!NOTE]
> Os procedimentos a seguir usam o Azure PowerShell para definir e criar o pipeline do ADF. Mas essa tarefa também pode ser realizada pelo Portal do Azure. Para obter detalhes, consulte [Criar pipeline](../../data-factory/tutorial-hybrid-copy-portal.md#create-a-pipeline).
>
>

Usando as definições de tabela fornecidas anteriormente, a definição de pipeline para o ADF é especificada da seguinte maneira:

```json
{
    "name": "AMLDSProcessPipeline",
    "properties":
    {
        "description" : "This pipeline has one Copy activity that copies data from an on-premises SQL to Azure blob",
        "activities":
        [
            {
                "name": "CopyFromSQLtoBlob",
                "description": "Copy data from on-premises SQL server to blob",
                "type": "CopyActivity",
                "inputs": [ {"name": "OnPremSQLTable"} ],
                "outputs": [ {"name": "OutputBlobTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "SqlSource",
                        "sqlReaderQuery": "select * from nyctaxi_data"
                    },
                    "sink":
                    {
                        "type": "BlobSink"
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            },
            {
                "name": "CopyFromBlobtoSQLAzure",
                "description": "Push data to Sql Azure",
                "type": "CopyActivity",
                "inputs": [ {"name": "OutputBlobTable"} ],
                "outputs": [ {"name": "OutputSQLAzureTable"} ],
                "transformation":
                {
                    "source":
                    {
                        "type": "BlobSource"
                    },
                    "sink":
                    {
                        "type": "SqlSink",
                        "WriteBatchTimeout": "00:5:00",
                    }
                },
                "Policy":
                {
                    "concurrency": 3,
                    "executionPriorityOrder": "NewestFirst",
                    "style": "StartOfInterval",
                    "retry": 2,
                    "timeout": "02:00:00"
                }
            }
        ]
    }
}
```

Copie a definição de JSON da tabela em um arquivo chamado *pipelinedef.json* e salve-o em um local conhecido (neste caso deve ser *C:\temp\pipelinedef.json*). Crie o pipeline no ADF com o seguinte cmdlet do Azure PowerShell:

    New-AzureDataFactoryPipeline  -ResourceGroupName adfdsprg -DataFactoryName adfdsp -File C:\temp\pipelinedef.json


## <a name="start-the-pipeline"></a><a name="adf-pipeline-start"></a>Iniciar o Pipeline
O pipeline agora pode ser executado usando o seguinte comando:

    Set-AzureDataFactoryPipelineActivePeriod -ResourceGroupName ADFdsprg -DataFactoryName ADFdsp -StartDateTime startdateZ –EndDateTime enddateZ –Name AMLDSProcessPipeline

Os valores de parâmetro *startdate* e *enddate* precisam ser substituídos pelas datas reais entre os quais você deseja executar o pipeline.

Depois que o pipeline é executado, você poderá ver os dados aparecerem no contêiner selecionado para o blob, um arquivo por dia.

Não aproveitamos a funcionalidade fornecida pela ADF para canalizar dados incrementalmente. Para obter mais informações sobre como fazer isso e outros recursos fornecidos pelo ADF, consulte a [documentação do ADF](https://azure.microsoft.com/services/data-factory/).
