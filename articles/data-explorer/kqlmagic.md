---
title: Use um Notebook Jupyter para analisar dados no Azure Data Explorer
description: Este tópico mostra como analisar dados no Azure Data Explorer usando um Notebook Jupyter e a extensão Kqlmagic.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 07/10/2019
ms.openlocfilehash: 83902ea5a3e73603311a0c469126ed603d0ebd16
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77064862"
---
# <a name="use-a-jupyter-notebook-and-kqlmagic-extension-to-analyze-data-in-azure-data-explorer"></a>Use um Jupyter Notebook e uma extensão Kqlmagic para analisar dados no Azure Data Explorer

O Jupyter Notebook é um aplicativo Web de código-fonte aberto que permite que criar e compartilhar documentos que contêm código ao vivo, equações, visualizações e texto narrativo. O uso inclui limpeza de dados e transformação, simulação numérica, modelagem estatística, visualização de dados e aprendizado de máquina.
O [Jupyter Notebook](https://jupyter.org/) dá suporte a funções mágicas que ampliam os recursos do kernel, oferecendo suporte a comandos adicionais. O KQL magic é um comando que estende as funcionalidades do kernel do Python no Jupyter Notebook, de modo que você possa executar consultas na linguagem Kusto nativamente. Você pode facilmente combinar a linguagem de consulta Python e Kusto para consultar e visualizar dados usando a rica biblioteca Plot.ly integrada com comandos `render`. Há suporte para fontes de dados para execução de consultas. Essas fontes de dados incluem o Azure Data Explorer, um serviço de exploração de dados rápido e altamente escalável para dados de log e telemetria, bem como registros do Monitor do Azure e insights de aplicativos. O KQL magic também funciona com o Azure Notebooks, o Jupyter Lab e a extensão do Jupyter do Visual Studio Code.

## <a name="prerequisites"></a>Pré-requisitos

- Uma conta de e-mail da organização que é membro do Azure Active Directory (AAD).
- Jupyter Notebook instalado no computador local ou use o Azure Notebooks e clone o exemplo [Azure Notebooks](https://kustomagicsamples-manojraheja.notebooks.azure.com/j/notebooks/Getting%20Started%20with%20kqlmagic%20on%20Azure%20Data%20Explorer.ipynb)

## <a name="install-kql-magic-library"></a>Instalar a biblioteca do KQL magic

1. Instalar o KQL magic:

    ```python
    !pip install Kqlmagic --no-cache-dir  --upgrade
    ```
    > [!NOTE]
    > Quando o Azure Notebooks é usado, esta etapa não é necessária.

1. Carregar o KQL magic:

    ```python
    %reload_ext Kqlmagic
    ```
    > [!NOTE]
    > Altere a versão do Kernel para Python 3.6 clicando no Kernel > Change Kernel > Python 3.6
    
## <a name="connect-to-the-azure-data-explorer-help-cluster"></a>Conecte-se ao cluster de ajuda do Azure Data Explorer

Use o seguinte comando para se conectar ao banco de dados *Amostras* hospedado no cluster *Ajuda*. Para usuários do AAD não Microsoft, substitua o nome do locatário `Microsoft.com` com seu locatário do AAD.

```python
%kql AzureDataExplorer://tenant="Microsoft.com";code;cluster='help';database='Samples'
```

## <a name="query-and-visualize"></a>Consultar e visualizar

Consulte de dados usando o [operador renderizar](/azure/kusto/query/renderoperator) e visualize dados usando a biblioteca ploy.ly. Essa consulta e visualização fornece uma experiência integrada que usa um KQL nativo. O Kqlmagic dá suporte a maioria dos gráficos, exceto `timepivot`, `pivotchart`, e `ladderchart`. A renderização é compatível com todos os atributos, exceto `kind`, `ysplit`, e `accumulate`. 

### <a name="query-and-render-piechart"></a>Consultar e renderizar o gráfico de pizza

```python
%%kql
StormEvents
| summarize statecount=count() by State
| sort by statecount 
| limit 10
| render piechart title="My Pie Chart by State"
```

### <a name="query-and-render-timechart"></a>Consultar e renderizar o gráfico de tempo

```python
%%kql
StormEvents
| summarize count() by bin(StartTime,7d)
| render timechart
```

> [!NOTE]
> Os gráficos são interativos. Selecione um intervalo de tempo para ampliar um horário específico.

### <a name="customize-the-chart-colors"></a>Personalizar a legenda de cores

Se não gostar da paleta de cores padrão, personalize os gráficos usando as opções de paleta. As paletas disponíveis podem ser encontradas aqui: [Escolha a paleta de cores para o resultado do gráfico de consulta mágica KQL](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb)

1. Para uma lista de paletas:

    ```python
    %kql --palettes -popup_window
    ```

1. Selecione a paleta de cores`cool` e renderize a consulta novamente:

    ```python
    %%kql -palette_name "cool"
    StormEvents
    | summarize statecount=count() by State
    | sort by statecount
    | limit 10
    | render piechart title="My Pie Chart by State"
    ```

## <a name="parameterize-a-query-with-python"></a>Parametrizar uma consulta com Python

O KQL magic permite um intercâmbio simples entre a linguagem de consulta Kusto e o Python. Para saber mais: [Parametrize sua consulta mágica KQL com python](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb)

### <a name="use-a-python-variable-in-your-kql-query"></a>Usar uma variável de Python em sua consulta KQL

Use o valor de uma variável de Python em sua consulta para filtrar os dados:

```python
statefilter = ["TEXAS", "KANSAS"]
```

```python
%%kql
let _state = statefilter;
StormEvents 
| where State in (_state) 
| summarize statecount=count() by bin(StartTime,1d), State
| render timechart title = "Trend"
```

### <a name="convert-query-results-to-pandas-dataframe"></a>Converter os resultados da consulta em DataFrame do Pandas

Você pode acessar os resultados de uma consulta KQL no DataFrame Pandas. Acesse os últimos resultados da consulta executada pela variável `_kql_raw_result_` e converta facilmente os resultados em DataFrame Pandas da seguinte maneira:

```python
df = _kql_raw_result_.to_dataframe()
df.head(10)
```

### <a name="example"></a>Exemplo

Em muitos cenários de análise, você talvez queira criar blocos de anotações reutilizáveis que contêm muitas consultas e alimentam os resultados de uma consulta em consultas subsequentes. O exemplo a seguir usa a variável de Python `statefilter` para filtrar os dados.

1. Executar uma consulta para exibir os estados de 10 principais com máximo `DamageProperty`:

    ```python
    %%kql
    StormEvents
    | summarize max(DamageProperty) by State
    | order by max_DamageProperty desc
    | limit 10
    ```

1. Execute uma consulta para extrair o estado superior e defina-o em uma variável de Python:

    ```python
    df = _kql_raw_result_.to_dataframe()
    statefilter =df.loc[0].State
    statefilter
    ```

1. Executar uma consulta usando a instrução `let` e a variável de Python:

    ```python
    %%kql
    let _state = statefilter;
    StormEvents 
    | where State in (_state)
    | summarize statecount=count() by bin(StartTime,1d), State
    | render timechart title = "Trend"
    ```

1. Execute o comando de Ajuda:

    ```python
    %kql --help "help"
    ```

> [!TIP]
> Para receber informações sobre todas `%config Kqlmagic`as configurações disponíveis use . Para solucionar e capturar erros do Kusto, como problemas de conexão e consultas incorretas, use`%config Kqlmagic.short_errors=False`

## <a name="next-steps"></a>Próximas etapas

Execute o comando de ajuda para explorar os seguintes notebooks de amostra que contêm todos os recursos com suporte:
- [Introdução ao KQL magic para o Azure Data Explorer](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStart.ipynb) 
- [Introdução ao KQL magic para o Application Insights](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStartAI.ipynb) 
- [Comece com a magia KQL para logs do Monitor Do Azure](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FQuickStartLA.ipynb) 
- [Parametrizar a consulta KQL magic com o Python](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FParametrizeYourQuery.ipynb) 
- [Escolher a paleta de cores para o resultado do gráfico da consulta KQL magic](https://mybinder.org/v2/gh/Microsoft/jupyter-Kqlmagic/master?filepath=notebooks%2FColorYourCharts.ipynb)
