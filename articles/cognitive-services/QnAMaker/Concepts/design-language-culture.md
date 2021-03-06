---
title: Design para linguagem - QnA Maker
description: O recurso QnA Maker e todas as bases de conhecimento dentro desse recurso suportam uma única língua. O idioma único é necessário para fornecer os melhores resultados de resposta para uma consulta.
ms.topic: conceptual
ms.date: 01/27/2020
ms.openlocfilehash: 5cb1dcd35649debbafd2e234606ad4c9d6906ea6
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "76843415"
---
# <a name="design-knowledge-base-for-content-language"></a>Base de conhecimento de design para linguagem de conteúdo

O recurso QnA Maker, e todas as bases de conhecimento dentro desse recurso, suportam uma única língua. O idioma único é necessário para fornecer os melhores resultados de resposta para uma consulta.

## <a name="single-language-per-resource"></a>Linguagem única por recurso

Considerações do QnA Maker para o suporte ao idioma:

* Um serviço QnA Maker, e todas as suas bases de conhecimento, suportam apenas uma língua.
* A linguagem é explicitamente definida quando a primeira base de conhecimento do serviço é criada
* A linguagem é determinada a partir dos arquivos e URLs adicionados quando a base de conhecimento é criada
* A linguagem não pode ser alterada para quaisquer outras bases de conhecimento no serviço
* A linguagem é usada pelo serviço de busca cognitiva (ranker #1) e pelo serviço QnA Maker (ranker #2) para gerar a melhor resposta para uma consulta

## <a name="supporting-multiple-languages"></a>Suporte a vários idiomas

Se você precisar apoiar um sistema de base de conhecimento, que inclui vários idiomas, você pode escolher um dos seguintes métodos:

* Use o [serviço Texto de Tradução](../../translator/translator-info-overview.md) para traduzir uma pergunta em um único idioma antes de enviar a pergunta para sua base de conhecimento. Isso permite que você se concentre na qualidade de uma única língua e na qualidade das perguntas e respostas alternativas.
* Crie um recurso QnA Maker, e uma base de conhecimento dentro desse recurso, para cada idioma. Isso permite que você gerencie perguntas alternativas separadas e responda texto que seja mais nuances para cada idioma. Isso lhe dá muito mais flexibilidade, mas requer um custo de manutenção muito maior quando as perguntas ou respostas mudam em todos os idiomas.

Revisar [idiomas suportados](../overview/language-support.md) para QnA Maker.

### <a name="support-each-language-with-a-qna-maker-resource"></a>Apoie cada idioma com um recurso QnA Maker

* Crie um recurso QnA Maker para cada idioma
* Apenas adicione arquivos e URLs para esse idioma
* Use uma convenção de nomeação para o recurso para identificar o idioma. Um exemplo `qna-maker-fr` é para todas as bases de conhecimento para documentos franceses

## <a name="next-steps"></a>Próximas etapas

Aprenda [conceitos](query-knowledge-base.md) sobre como consultar a base de conhecimento para obter uma resposta.