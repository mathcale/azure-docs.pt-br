---
title: Serviços de aprendizado de máquina com R (visualização)
description: Este artigo descreve o Azure SQL Database Machine Learning Services (com R) e explica como ele funciona.
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: dphansen
ms.author: davidph
ms.reviewer: carlrab
manager: cgronlun
ms.date: 11/20/2019
ms.openlocfilehash: 2a2cd4bfc3d393543b41eea776f02723d94054b1
ms.sourcegitcommit: 8a9c54c82ab8f922be54fb2fcfd880815f25de77
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "80345820"
---
# <a name="azure-sql-database-machine-learning-services-with-r-preview"></a>Serviços de Machine Learning do Banco de Dados SQL do Azure com R (versão prévia)

Os Serviços do Machine Learning são um recurso do Banco de Dados SQL do Azure, usado para executar scripts do R no banco de dados. O recurso inclui pacotes R da Microsoft para análise preditiva de alto desempenho e machine learning. Os dados relacionais podem ser usados em scripts R por meio de procedimentos armazenados, do script T-SQL que contém instruções do R ou do código R que contém o T-SQL.

[!INCLUDE[ml-preview-note](../../includes/sql-database-ml-preview-note.md)]

> [!NOTE]
> A visualização está disponível para bancos de dados únicos e piscinas elásticas usando o modelo de compra baseado em vCore nos níveis de **serviços críticos** **de propósito geral** e de negócios. Nesta visualização inicial, o nível de serviço **de hiperescala** e a opção de implantação **de instância gerenciada** não são suportados. Atualmente, R é a única linguagem com suporte. No momento, não há suporte para Python.
>
> A pré-visualização está disponível atualmente nas seguintes regiões: Europa Ocidental, Europa Do Norte, Oeste dos EUA 2, Leste dos EUA, Centro-Sul dos EUA, Norte Central dos EUA, Canadá Central, Sudeste Asiático, Índia Do Sul e Austrália Sudeste.

## <a name="what-you-can-do-with-r"></a>O que você pode fazer com o R

Use o poder da linguagem R para fornecer análise avançada e machine learning no banco de dados. Essa capacidade leva os cálculos e o processamento para o local em que os dados residem, eliminando a necessidade de efetuar pull de dados na rede. Além disso, você pode aproveitar o poder dos pacotes R corporativos para fornecer análises avançadas em escala.

Os Serviços do Machine Learning incluem uma distribuição base do R, sobreposta com pacotes empresariais do R da Microsoft. As funções e os algoritmos do R da Microsoft foram desenvolvidos para escala e utilidade, fornecendo análise preditiva, modelagem estatística, visualizações de dados e algoritmos de machine learning de ponta.

### <a name="r-packages"></a>Pacotes R

Os pacotes R de código aberto mais comuns são pré-instalados em Serviços de Aprendizado de Máquina. Os seguintes pacotes do R da Microsoft também estão incluídos:

| Pacote R | Descrição|
|-|-|
| [Microsoft R Open](https://mran.microsoft.com/rro) | O Microsoft R Open é a distribuição aprimorada de R da Microsoft. É uma plataforma de software livre completa para análise estatística e ciência de dados. Ela é baseada e totalmente compatível com o R e inclui recursos adicionais para melhorar o desempenho e a capacidade de reprodução. |
| [RevoScaleR](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-revoscaler) | O RevoScaleR é a biblioteca principal para o R escalonável. As funções nessa biblioteca estão entre mais amplamente usadas. As transformações e a manipulação de dados, resumo estatístico, visualização e muitas formas de modelagem e de análises são encontrados nessas bibliotecas. Além disso, as funções nessas bibliotecas distribuem automaticamente as cargas de trabalho entre os núcleos disponíveis para processamento paralelo, com a capacidade de trabalhar em partes de dados coordenadas e gerenciadas pelo mecanismo de cálculo. |
| [MicrosoftML (R)](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-microsoftml) | O MicrosoftML adiciona algoritmos de machine learning para criar modelos personalizados para análise de texto, análise de imagem e de sentimento. |

Além dos pacotes pré-instalados, você pode [instalar pacotes adicionais](sql-database-machine-learning-services-add-r-packages.md).

<a name="signup"></a>

## <a name="sign-up-for-the-preview-closed"></a>Inscreva-se para a pré-visualização (fechada)

> [!IMPORTANT]
> Inscreva-se no Azure SQL Database Machine Learning Services (preview) está **atualmente fechado**.

## <a name="next-steps"></a>Próximas etapas

- Veja as [principais diferenças em relação aos Serviços de Aprendizagem de Máquina do Servidor SQL](sql-database-machine-learning-services-differences.md).
- Para saber como usar o R para consultar o Azure SQL Database Machine Learning Services (preview), consulte o [guia Quickstart](sql-database-connect-query-r.md).
- Para começar com alguns scripts R simples, consulte [Criar e executar scripts R simples no Azure SQL Database Machine Learning Services (visualização)](sql-database-quickstart-r-create-script.md).
