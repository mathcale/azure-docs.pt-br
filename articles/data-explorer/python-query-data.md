---
title: Consultar dados usando a biblioteca Python do Azure Data Explorer
description: Neste artigo, você aprende como consultar dados do Azure Data Explorer usando Python.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/05/2019
ms.openlocfilehash: ebd65f2dcbb0040b764290627bbfd2901aa9a7d3
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77443968"
---
# <a name="query-data-using-the-azure-data-explorer-python-library"></a>Consultar dados usando a biblioteca Python do Azure Data Explorer

Neste artigo, você consulta dados usando o Azure Data Explorer. O Azure Data Explorer é um serviço de exploração de dados rápido e altamente escalonável para dados de log e telemetria.

O Azure Data Explorer fornece uma biblioteca cliente de [dados para Python](https://github.com/Azure/azure-kusto-python/tree/master/azure-kusto-data). Esta biblioteca permite consultar dados do seu código. Conecte-se a uma tabela no *cluster de ajuda* que criamos para ajudar no aprendizado. Você pode consultar uma tabela nesse cluster e retornar os resultados.

Este artigo também está disponível como [um Notebook Azure](https://notebooks.azure.com/ManojRaheja/libraries/KustoPythonSamples/html/QueryKusto.ipynb).

## <a name="prerequisites"></a>Pré-requisitos

* [Python 3.4+](https://www.python.org/downloads/)

* Uma conta de email da organização que é membro do Azure Active Directory (AAD)

## <a name="install-the-data-library"></a>Instalar a biblioteca de dados

Instale * azure-kusto-data *.

```
pip install azure-kusto-data
```

## <a name="add-import-statements-and-constants"></a>Adicionar instruções de importação e constantes

Importe classes da biblioteca, bem como *pandas*, uma biblioteca de análise de dados.

```python
from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
import pandas as pd
```

Para autenticar um aplicativo, o Azure Data Explorer usa seu ID de locatário do AAD. Para encontrar seu ID de locatário, use a seguinte URL, substituindo seu domínio por *YourDomain*.

```
https://login.windows.net/<YourDomain>/.well-known/openid-configuration/
```

Por exemplo, se o seu domínio for *contoso.com*, a URL será: [https://login.windows.net/contoso.com/.well-known/openid-configuration/](https://login.windows.net/contoso.com/.well-known/openid-configuration/). Clique nesta URL para ver os resultados; a primeira linha é a seguinte.

```
"authorization_endpoint":"https://login.windows.net/6babcaad-604b-40ac-a9d7-9fd97c0b779f/oauth2/authorize"
```

A ID do locatário neste caso é `6babcaad-604b-40ac-a9d7-9fd97c0b779f`. Defina o valor para AAD_TENANT_ID antes de executar este código.

```python
AAD_TENANT_ID = "<TenantId>"
KUSTO_CLUSTER = "https://help.kusto.windows.net/"
KUSTO_DATABASE = "Samples"
```

Agora, construa a cadeia de caracteres de conexão. Este exemplo usa a autenticação do dispositivo para acessar o cluster. Você também pode usar [certificado de aplicativo AAD,](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L24) [chave de aplicativo AAD](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L20)e [usuário e senha AAD](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L34).

```python
KCSB = KustoConnectionStringBuilder.with_aad_device_authentication(
    KUSTO_CLUSTER)
KCSB.authority_id = AAD_TENANT_ID
```

## <a name="connect-to-azure-data-explorer-and-execute-a-query"></a>Conecte-se ao Azure Data Explorer e execute uma consulta

Execute uma consulta no cluster e armazene a saída em um quadro de dados. Quando esse código é executado, ele retorna uma mensagem como a seguinte: *Para entrar, use um navegador da web para abrir a páginahttps://microsoft.com/devicelogin e digite o código F3W4VWZDM para autenticar*. Siga as etapas para entrar e retorne para executar o próximo bloco de código.

```python
KUSTO_CLIENT = KustoClient(KCSB)
KUSTO_QUERY = "StormEvents | sort by StartTime desc | take 10"

RESPONSE = KUSTO_CLIENT.execute(KUSTO_DATABASE, KUSTO_QUERY)
```

## <a name="explore-data-in-dataframe"></a>Explorar dados no DataFrame

Depois que você insere um sinal, a consulta retorna resultados e eles são armazenados em um quadro de dados. Você pode trabalhar com os resultados como qualquer outro quadro de dados.

```python
df = dataframe_from_result_table(RESPONSE.primary_results[0])
df
```

Você deve ver os dez primeiros resultados da tabela StormEvents.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Ingerir dados usando a biblioteca Python do Azure Data Explorer](python-ingest-data.md)
