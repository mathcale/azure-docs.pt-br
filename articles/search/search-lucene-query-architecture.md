---
title: Consulta completa de texto e arquitetura do mecanismo de indexação (Lucene)
titleSuffix: Azure Cognitive Search
description: Examina os conceitos de processamento de consultas de Lucene e recuperação de documentos para pesquisa completa de texto, conforme relacionado à Pesquisa Cognitiva do Azure.
manager: nitinme
author: yahnoosh
ms.author: jlembicz
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: d46d0309b3d2ffb638016e88ba022e49009eedf2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79282933"
---
# <a name="how-full-text-search-works-in-azure-cognitive-search"></a>Como funciona a pesquisa de texto completo no Azure Cognitive Search

Este artigo é para desenvolvedores que precisam de uma compreensão mais profunda de como a pesquisa de texto completa lucene funciona no Azure Cognitive Search. Para consultas de texto, o Azure Cognitive Search produzirá facilmente os resultados esperados na maioria dos cenários, mas, ocasionalmente, você poderá obter um resultado que pode parecer "estranho". Nessas situações, ter experiência nos quatro estágios da execução da consulta do Lucene (análise léxica, análise da consulta, correspondência de documentos e pontuação) pode ajudá-lo a identificar alterações específicas nos parâmetros de consulta ou na configuração de índice que proporcionarão o resultado desejado. 

> [!Note] 
> A Azure Cognitive Search usa Lucene para pesquisa completa de texto, mas a integração de Lucene não é exaustiva. Nós expomos e estendemos seletivamente a funcionalidade lucene para habilitar os cenários importantes para a Pesquisa Cognitiva do Azure. 

## <a name="architecture-overview-and-diagram"></a>Diagrama e visão geral da arquitetura

O processamento de uma consulta de pesquisa de texto completo começa com a análise do texto da consulta para extrair os termos de pesquisa. O mecanismo de pesquisa usa um índice para recuperar documentos com os termos correspondentes. Os termos de consulta individual, às vezes, são divididos e reconstituídos em novos formulários para obter uma rede mais ampla sobre o que poderia ser considerado como uma possível correspondência. Um conjunto de resultados é classificado por uma pontuação de relevância atribuída a cada documento correspondente individual. Aqueles no topo da lista com a classificação são retornados para o aplicativo de chamada.

A execução de consulta redefinida, tem quatro fases: 

1. Análise da consulta 
2. Análise léxica 
3. Recuperação de documentos 
4. Pontuação 

O diagrama a seguir ilustra os componentes usados para processar uma solicitação de pesquisa. 

 ![Diagrama de arquitetura de consulta lucene na Pesquisa Cognitiva do Azure][1]


| Principais componentes | Descrição funcional | 
|----------------|------------------------|
|**Analisadores de consulta** | Separam os termos de consulta de operadores de consulta e criam a estrutura da consulta (uma árvore de consulta) a ser enviada para o mecanismo de pesquisa. |
|**Analisadores** | Executam a análise léxica dos termos de consulta. Esse processo pode envolver a transformação, remoção ou expansão dos termos de consulta. |
|**Índice** | Uma estrutura de dados eficiente usada para armazenar e organizar termos pesquisáveis extraídos de documentos indexados. |
|**Mecanismo de pesquisa** | Recupera e atribui uma pontuação aos documentos correspondentes com base no conteúdo do índice invertido. |

## <a name="anatomy-of-a-search-request"></a>Anatomia de uma solicitação de pesquisa

Uma solicitação de pesquisa é uma especificação completa do que deve ser retornado em um conjunto de resultados. Na forma mais simples, é uma consulta vazia sem critérios de nenhum tipo. Um exemplo mais realista inclui parâmetros, vários termos de consulta, talvez com escopo para determinados campos, com possivelmente uma expressão de filtro e as regras de ordenação.  

O exemplo a seguir é uma solicitação de pesquisa que você pode enviar para a Pesquisa Cognitiva do Azure usando a [API REST](https://docs.microsoft.com/rest/api/searchservice/search-documents).  

~~~~
POST /indexes/hotels/docs/search?api-version=2019-05-06
{
    "search": "Spacious, air-condition* +\"Ocean view\"",
    "searchFields": "description, title",
    "searchMode": "any",
    "filter": "price ge 60 and price lt 300",
    "orderby": "geo.distance(location, geography'POINT(-159.476235 22.227659)')", 
    "queryType": "full" 
}
~~~~

Para essa solicitação, o mecanismo de pesquisa faz o seguinte:

1. Filtra os documentos em que o preço é pelo menos US $60 e menor que US $300.
2. Executa a consulta. Neste exemplo, a consulta de pesquisa consiste de frases e termos: `"Spacious, air-condition* +\"Ocean view\""` (os usuários normalmente não inserem pontuação, mas incluí-la no exemplo permite explicar como os analisadores tratam a pontuação). Para essa consulta, o mecanismo de pesquisa examina a descrição e os campos de título especificados em `searchFields` para documentos que contenham "Vista para o mar", além do termo "espaçoso" ou em termos que começam com o prefixo "ar-condicio". O parâmetro `searchMode` é usado para corresponder a qualquer termo (padrão) ou todos eles, para casos em que um termo não for explicitamente solicitado (`+`).
3. Ordena o conjunto resultante de hotéis por proximidade de uma localização geográfica indicada e retorna para o aplicativo de chamada. 

A maioria deste artigo é sobre o processamento `"Spacious, air-condition* +\"Ocean view\""`da consulta de *pesquisa*: . Filtragem e ordenação estão fora do escopo. Para obter mais informações, consulte as [documentação de referência da API de pesquisa](https://docs.microsoft.com/rest/api/searchservice/search-documents).

<a name="stage1"></a>
## <a name="stage-1-query-parsing"></a>Estágio 1: Análise da consulta 

Conforme observado, a cadeia de caracteres de consulta é a primeira linha da solicitação: 

~~~~
 "search": "Spacious, air-condition* +\"Ocean view\"", 
~~~~

O analisador de consulta separa os operadores (como `*` e `+` no exemplo) dos termos de pesquisa e desconstrói a consulta de pesquisa em *subconsultas* de um tipo com suporte: 

+ *consulta de termo* para termos independentes (espaçoso, por exemplo)
+ *consulta de frase* para termos entre aspas (vista para o mar, por exemplo)
+ *consulta de prefixo* por termos seguidos por um operador de prefixo `*` (ar-condicio, por exemplo)

Para obter uma lista completa dos tipos de consulta com suporte, veja [sintaxe da consulta do Lucene](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search)

Os operadores associados com uma subconsulta determinam se a consulta deve ser obrigatoriamente satisfeita ou não para um documento ser considerado uma correspondência. Por exemplo, `+"Ocean view"` é "obrigatória" devido ao operador `+`. 

O analisador de consulta reestrutura as subconsultas em uma *árvore de consulta* (uma estrutura interna que representa a consulta) passada para o mecanismo de pesquisa. No primeiro estágio da análise de consulta, a árvore de consulta se parece com isto.  

 ![Booliano consulta modo de pesquisa qualquer][2]

### <a name="supported-parsers-simple-and-full-lucene"></a>Analisadores com suporte: simples e Lucena completa 

 O Azure Cognitive Search expõe duas `simple` linguagens `full`de consulta diferentes(padrão) e . Ao definir o parâmetro `queryType` com sua solicitação de pesquisa, você informa ao analisador de consulta a linguagem de consulta que você escolheu para que ele saiba como interpretar os operadores e a sintaxe. A [linguagem de consulta simples](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) é intuitiva e robusta, geralmente adequada para interpretar a entrada do usuário conforme inserida, sem processamento no lado do cliente. Ela oferece suporte a operadores de consulta familiares de mecanismos de pesquisa. A [linguagem de consulta Lucene completa](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search), que você obtém definindo `queryType=full`, estende a linguagem de consulta simples padrão, adicionando suporte para mais operadores e tipos de consulta como caractere curinga, difusa, regex e consultas com escopo de campo. Por exemplo, uma expressão regular enviada na sintaxe de consulta simples será interpretada como uma cadeia de caracteres de consulta e não é uma expressão. A solicitação de exemplo neste artigo usa a linguagem de consulta Lucene completa.

### <a name="impact-of-searchmode-on-the-parser"></a>Impacto do modo de pesquisa no analisador 

Outro parâmetro de solicitação de pesquisa que afeta a análise é o parâmetro `searchMode`. Ele controla o operador padrão para consultas boolianas: qualquer (padrão) ou todos.  

Quando `searchMode=any`, que é o padrão, o delimitador de espaço entre espaçoso e ar-condicio é OR (`||`), tornando o texto da consulta de exemplo equivalente a: 

~~~~
Spacious,||air-condition*+"Ocean view" 
~~~~

Operadores explícitos, como `+` em `+"Ocean view"`, não são ambíguos na construção de consulta booliana (o termo *deve* corresponder). Menos óbvio é como interpretar os demais termos: espaçoso e ar-condicio. O mecanismo de pesquisa deve localizar correspondências para vista para o mar *e* espaçoso *e* ar-condicio? Ou deve encontrar vista para o mar mais *qualquer um* dos demais termos? 

Por padrão (`searchMode=any`), o mecanismo de pesquisa assume a interpretação mais ampla. Cada campo *deve* ter uma correspondência, refletindo a semântica de "ou". A árvore de consulta inicial ilustrada anteriormente, com as duas operações de “não obrigatório”, mostra o padrão.  

Suponha que agora definimos `searchMode=all`. Nesse caso, o espaço é interpretado como uma operação "e". Cada um dos demais termos deve estar presente no documento para ser qualificado como uma correspondência. O exemplo de consulta resultante será interpretado da seguinte maneira: 

~~~~
+Spacious,+air-condition*+"Ocean view"
~~~~

Uma árvore de consulta modificada para esta consulta seria a seguinte, onde um documento correspondente é a interseção de todas as três subconsultas: 

 ![Booliano consulta modo de pesquisa todos][3]

> [!Note] 
> Escolher `searchMode=any` em vez de `searchMode=all` é uma decisão melhor ao executar consultas representativas. Os usuários mais propensos a incluir operadores (comum ao pesquisar repositórios de documentos) pode encontrar resultados mais intuitivos se `searchMode=all` informa construções de consulta boolianas. Para obter mais informações sobre a interação entre `searchMode` e os operadores, consulte [sintaxe de consulta simples](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search).

<a name="stage2"></a>
## <a name="stage-2-lexical-analysis"></a>Estágio 2: Análise léxica 

Os analisadores léxicos processam *consultas de termo* e *consultas de frase* depois que a árvore de consulta é estruturada. Um analisador aceita as entradas de texto fornecidas pelo analisador, processa o texto e, em seguida, envia de volta os termos com token a serem incorporados na árvore de consulta. 

A forma mais comum de análise léxica é a *análise linguística* que transforma consultas baseadas em termos em regras específicas para um idioma específico: 

* Reduzindo um termo de consulta para a raiz de uma palavra 
* Removendo palavras não-essenciais (palavras irrelevantes, como "o/a" ou "e" em português) 
* Dividir uma palavra composta em componentes 
* Colocando letras minúsculas em uma palavra de letras maiúsculas 

Todas essas operações tendem a apagar as diferenças entre a entrada de texto fornecida pelo usuário e os termos armazenados no índice. Essas operações vão além do processamento de texto e exigem um conhecimento profundo do próprio idioma. Para adicionar essa camada de consciência linguística, o Azure Cognitive Search suporta uma longa lista de analisadores de [idiomas](https://docs.microsoft.com/rest/api/searchservice/language-support) tanto da Lucene quanto da Microsoft.

> [!Note]
> Os requisitos de análise podem variar de básicos a elaborados dependendo do seu cenário. Você pode controlar a complexidade da análise léxica selecionando um dos analisadores predefinidos ou criando seu próprio [analisador personalizado](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search). O escopo dos analisadores inclui campos pesquisáveis e são especificados como parte de uma definição do campo. Isso permite que você varie a análise léxica baseada no campo. Se não for especificado, o analisador *padrão* para Lucene é usado.

Em nosso exemplo, antes da análise, a árvore de consulta inicial tem o termo "Espaçoso," com um "E" maiúsculo e uma vírgula que o analisador de consulta interpreta como parte do termo de consulta (uma vírgula não é considerada um operador de linguagem de consulta).  

Quando o analisador padrão processa o termo, ele colocará "vista para o mar" e "espaçoso" em letras minúsculas e removerá o caractere de vírgula. A árvore de consulta modificada ficará da seguinte forma: 

 ![Consulta booliana com termos analisados][4]

### <a name="testing-analyzer-behaviors"></a>Testando os comportamentos do analisador 

O comportamento de um analisador pode ser testado usando a [API de análise](https://docs.microsoft.com/rest/api/searchservice/test-analyzer). Forneça o texto que você deseja analisar para ver quais termos o analisador irá gerar. Por exemplo, para ver como o analisador padrão processaria o texto "ar-condicio", você pode emitir a solicitação a seguir:

~~~~
{
    "text": "air-condition",
    "analyzer": "standard"
}
~~~~

O analisador padrão quebra o texto de entrada nos dois tokens a seguir, associando atributos como deslocamentos inicial e final (usados para realçar ocorrências), bem como sua posição (usada para correspondência de frase):

~~~~
{
  "tokens": [
    {
      "token": "air",
      "startOffset": 0,
      "endOffset": 3,
      "position": 0
    },
    {
      "token": "condition",
      "startOffset": 4,
      "endOffset": 13,
      "position": 1
    }
  ]
}
~~~~

<a name="exceptions"></a>

### <a name="exceptions-to-lexical-analysis"></a>Exceções para análise léxica 

A análise léxica só se aplica a tipos de consultas que exigem termos completos – uma consulta de termo ou uma consulta de frase. Ela não se aplica aos tipos de consulta com termos incompletos – consulta de prefixo, consulta de caractere curinga, consulta regex – ou a uma consulta difusa. Esses tipos de consulta, incluindo a consulta de prefixo com o termo `air-condition*` em nosso exemplo, são adicionados diretamente à árvore de consulta, ignorando o estágio de análise. A única transformação realizada em termos de consulta desses tipos é colocá-los em letras minúsculas.

<a name="stage3"></a>

## <a name="stage-3-document-retrieval"></a>Estágio 3: Recuperação de documentos 

A recuperação de documentos se refere à procura de documentos com correspondência de termos no índice. Este estágio é melhor compreendido por meio de um exemplo. Vamos começar com um índice de hotéis com o esquema simples a seguir: 

~~~~
{
    "name": "hotels",
    "fields": [
        { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
        { "name": "title", "type": "Edm.String", "searchable": true },
        { "name": "description", "type": "Edm.String", "searchable": true }
    ] 
} 
~~~~

Suponhamos ainda que esse índice contém os quatro documentos a seguir: 

~~~~
{
    "value": [
        {
            "id": "1",
            "title": "Hotel Atman",
            "description": "Spacious rooms, ocean view, walking distance to the beach."
        },
        {
            "id": "2",
            "title": "Beach Resort",
            "description": "Located on the north shore of the island of Kauaʻi. Ocean view."
        },
        {
            "id": "3",
            "title": "Playa Hotel",
            "description": "Comfortable, air-conditioned rooms with ocean view."
        },
        {
            "id": "4",
            "title": "Ocean Retreat",
            "description": "Quiet and secluded"
        }
    ]
}
~~~~

**Como os termos são indexados**

Para entender a recuperação, é útil conhecer algumas noções básicas sobre indexação. A unidade de armazenamento é um índice invertido, um para cada campo pesquisável. Dentro de um índice invertido está uma lista classificada de todos os termos de todos os documentos. Cada termo é mapeado para a lista de documentos nos quais ele ocorre, tão evidente no exemplo a seguir.

Para produzir os termos de um índice invertido, o mecanismo de pesquisa executa a análise léxica do o conteúdo dos documentos, de forma semelhante ao que acontece durante o processamento da consulta:

1. As *entradas de texto* são passadas para um analisador, em letras minúsculas, sem pontuação e assim por diante, dependendo da configuração do analisador. 
2. *Tokens* são a saída da análise de texto.
3. *Termos* são adicionados ao índice.

É comum, mas não obrigatório, usar os mesmo analisadores para operações de indexação para que os termos da consulta pareçam mais com os termos dentro do índice.

> [!Note]
> O Azure Cognitive Search permite especificar diferentes analisadores `indexAnalyzer` `searchAnalyzer` para indexação e pesquisa por meio de parâmetros adicionais e de campo. Se não forem especificados, o analisador definido com a propriedade `analyzer` é usado para indexação e pesquisa.  

**Índice invertido para documentos de exemplo**

Retornando ao nosso exemplo, para o campo **título**, o índice invertido tem esta aparência:

| Termo | Lista de documentos |
|------|---------------|
| atman | 1 |
| praia | 2 |
| hotel | 1, 3 |
| mar | 4  |
| playa | 3 |
| resort | 3 |
| retiro | 4 |

No campo título, apenas *hotel* aparece em dois documentos: 1, 3.

Para o campo **descrição**, o índice é o seguinte:

| Termo | Lista de documentos |
|------|---------------|
| aérea | 3
| e | 4
| praia | 1
| condicionado | 3
| confortável | 3
| distância | 1
| ilha | 2
| kauaʻi | 2
| local | 2
| norte | 2
| mar | 1, 2, 3
| de | 2
| on |2
| silencioso | 4
| quartos  | 1, 3
| reservado | 4
| beira-mar | 2
| espaçoso | 1
| o | 1, 2
| para | 1
| exibição | 1, 2, 3
| a pé | 1
| por | 3


**Correspondência de termos de consulta com os termos indexados**

Considerando os índices invertidos acima, vamos voltar para a consulta de exemplo e ver como documentos com correspondência são encontrados para a nossa consulta de exemplo. Lembre-se de que a árvore de consulta final tem esta aparência: 

 ![Consulta booliana com termos analisados][4]

Durante a execução de consulta, consultas individuais são executadas nos campos pesquisáveis de independente. 

+ A pesquisa do termo, "espaçoso", corresponde ao documento 1 (Hotel Atman). 

+ A consulta de prefixo, "ar-condicio *", não corresponde a nenhum documento. 

  Esse é um comportamento que às vezes confunde os desenvolvedores. Embora o termo com ar condicionado exista no documento, ele é dividido em dois termos pelo analisador padrão. Lembre-se de que as consultas de prefixo, que contêm termos parciais, não são analisadas. Portanto, os termos com o prefixo "ar-condicio" são pesquisados no índice invertido e não são encontrados.

+ A consulta de frase, "vista para o mar", procura os termos "mar" e "vista para o" e verifica a proximidade dos termos no documento original. Os documentos 1, 2 e 3 correspondem a essa consulta no campo descrição. Observe que o documento 4 possui o termo mar termo no título, mas não é considerado uma correspondência, pois estamos procurando a frase "vista para o mar" em vez de palavras individuais. 

> [!Note]
> Uma consulta de pesquisa é executada independentemente contra todos os campos pesquisáveis no índice `searchFields` de pesquisa cognitiva do Azure, a menos que você limite os campos definidos com o parâmetro, conforme ilustrado na solicitação de pesquisa por exemplo. Os documentos correspondentes em qualquer um dos campos selecionados são retornados. 

De modo geral, para a consulta em questão, os documentos que correspondem são 1, 2, 3. 

## <a name="stage-4-scoring"></a>Estágio 4: Pontuação  

Todos os documentos em um conjunto de resultados de pesquisa recebe uma pontuação de relevância. A função da pontuação de relevância é classificar com uma pontuação mais alta os documentos que melhor respondem a uma pergunta do usuário melhor conforme expressa pela consulta de pesquisa. A pontuação é calculada com base nas propriedades estatísticas dos termos com correspondência. A fórmula da pontuação é basicamente [TF/IDF (frequência do termo sobre frequência inversa do documento)](https://en.wikipedia.org/wiki/Tf%E2%80%93idf). Em consultas que contêm termos comuns e raros, TF/IDF fornece resultados que contêm o termo raro. Por exemplo, em um índice hipotético com todos os artigos da Wikipédia, de documentos que correspondem à consulta *o presidente*, os documentos com correspondência para *presidente* são considerados mais relevantes do que os documentos com correspondência para *o*.


### <a name="scoring-example"></a>Exemplo de pontuação

Lembre-se dos três documentos que correspondem à nossa consulta de exemplo:
~~~~
search=Spacious, air-condition* +"Ocean view"  
~~~~
~~~~
{
  "value": [
    {
      "@search.score": 0.25610128,
      "id": "1",
      "title": "Hotel Atman",
      "description": "Spacious rooms, ocean view, walking distance to the beach."
    },
    {
      "@search.score": 0.08951007,
      "id": "3",
      "title": "Playa Hotel",
      "description": "Comfortable, air-conditioned rooms with ocean view."
    },
    {
      "@search.score": 0.05967338,
      "id": "2",
      "title": "Ocean Resort",
      "description": "Located on a cliff on the north shore of the island of Kauai. Ocean view."
    }
  ]
}
~~~~

O documento 1 foi o que melhor correspondeu à consulta, pois tanto o termo *espaçoso* como a frase solicitada *vista para o mar* ocorrem no campo descrição. Os próximos dois documentos correspondem apenas à frase *vista para o mar*. Pode ser surpreendente que as pontuações de relevância para os documentos 2 e 3 sejam diferentes, mesmo que ambos tenham correspondido à consulta da mesma maneira. Isso ocorre porque a fórmula de pontuação tem mais componentes do que simplesmente TF/IDF. Nesse caso, o documento 3 recebeu uma pontuação ligeiramente mais alta porque sua descrição é mais curta. Saiba mais sobre a [Fórmula de pontuação prática do Lucene](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/search/similarities/TFIDFSimilarity.html) para entender como o tamanho do campo e outros fatores podem influenciar a pontuação de relevância.

Alguns tipos de consulta (caractere curinga, prefixo, regex) sempre contribuem com uma pontuação constante para a pontuação total do documento. Isso permite que as correspondências encontradas por meio da expansão de consulta sejam incluídas nos resultados, mas sem afetar a classificação. 

Um exemplo ilustra por que isso é importante. Pesquisas com caractere curinga, inclusive pesquisas de prefixo são ambíguas por definição, porque a entrada é uma cadeia de caracteres parcial com correspondências possíveis em um grande número de termos diferentes (considere uma entrada de "pass *", com correspondências encontradas em "passeios", "passagem" e "passarela"). Dada a natureza desses resultados, não é possível inferir de forma razoável quais termos são mais valiosos do que outros. Por esse motivo, podemos ignorar as frequências dos termos ao pontuar resultados em consultas dos tipos caractere curinga, prefixo e regex. Em uma solicitação de pesquisa de várias partes que inclui termos parciais e completos, os resultados da entrada parcial são incorporados com uma pontuação de constante para evitar desvios em relação às correspondências potencialmente inesperadas.

### <a name="score-tuning"></a>Ajuste da pontuação

Existem duas maneiras de ajustar as pontuações de relevância na Pesquisa Cognitiva do Azure:

1. **Perfis de pontuação** melhoram a classificação dos documentos na lista classificada de resultados com base em um conjunto de regras. Em nosso exemplo, podemos considerar a possibilidade de que os documentos correspondentes no campo de título são mais relevante do que os documentos correspondentes no campo descrição. Além disso, se o índice tiver um campo preço para cada hotel, poderíamos promover documentos com preços mais baixos. Saiba mais sobre como [adicionar perfis de pontuação a um índice de pesquisa.](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index)
2. **Incremento de termo** (disponível apenas na sintaxe da consulta Lucene completo) fornece um operador de incremento `^` que pode ser aplicado a qualquer parte da árvore de consulta. Em nosso exemplo, em vez de pesquisar sobre o prefixo\* *ar-condicionado, pode-se*procurar o termo *ar-condicionado* exato ou o prefixo, mas documentos que correspondem no termo exato são classificados mais alto, aplicando impulso ao termo consulta: *ar-condicionado^2|| ar-condicionado**. Saiba mais sobre [incremento do termo](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search#bkmk_termboost).


### <a name="scoring-in-a-distributed-index"></a>Pontuação em um índice distribuído

Todos os índices no Azure Cognitive Search são automaticamente divididos em vários fragmentos, permitindo-nos distribuir rapidamente o índice entre vários nós durante a escala de serviço para cima ou para baixo. Quando uma solicitação de pesquisa é emitida, ela é emitida em relação a cada fragmento de forma independente. Os resultados de cada fragmento são então mesclados e ordenados conforme a pontuação (se nenhuma outra ordem for definida). É importante saber que a função de pontuação faz a ponderação da frequência do termo de consulta em relação a sua frequência de documento inversa em todos os documentos dentro do fragmento, não em todos os fragmentos!

Isso significa que uma pontuação de relevância *pode* ser diferentes para documentos idênticos se estes estiverem em fragmentos diferentes. Felizmente, essas diferenças tendem a desaparecer conforme aumenta o número de documentos no índice devido a uma distribuição de termo mais uniforme. Não é possível supor em qual fragmento qualquer documento especificado será colocado. No entanto, supondo que uma chave de documento não é alterado, ela sempre será atribuída ao mesmo fragmento.

Em geral, a pontuação de documentos não é o melhor atributo para classificar documentos se a estabilidade da classificação for importante. Por exemplo, considerando dois documentos com uma pontuação idêntica, não há nenhuma garantia de qual aparece primeiro nas execuções posteriores da mesma consulta. A pontuação de documento deve dar somente uma noção geral de relevância do documento em relação a outros documentos no conjunto de resultados.

## <a name="conclusion"></a>Conclusão

O sucesso dos mecanismos de pesquisa da Internet gerou expectativas para a pesquisa de texto completo em dados particulares. Para quase qualquer tipo de experiência de pesquisa, esperamos que o mecanismo entenda a nossa intenção, mesmo quando os termos estão incorretos ou incompletos. Esperamos até correspondências com base em termos equivalentes ou sinônimos que nem especificamos.

Do ponto de vista técnico, a pesquisa de texto completo é altamente complexa, exigindo uma análise linguística sofisticada e uma abordagem sistemática para processamento de forma a extrair, expandir e transformar os termos da consulta para fornecer um resultado relevante. Devido às complexidades inerentes, há muitos fatores que podem afetar o resultado de uma consulta. Por esse motivo, investir tempo para entender os mecanismos de pesquisa de texto completo oferece benefícios tangíveis quando se tenta trabalhar com resultados inesperados.  

Este artigo explorou a pesquisa completa de texto no contexto da Pesquisa Cognitiva do Azure. Esperamos que todas essas informações sejam o suficiente para você reconhecer possíveis causas e resoluções para resolver problemas comuns de consulta. 

## <a name="next-steps"></a>Próximas etapas

+ Criar o índice de exemplo, experimentar consultas diferentes e examinar os resultados. Para obter instruções, consulte [Criar e consultar um índice no portal](search-get-started-portal.md#query-index).

+ Tente outras sintaxes de consulta a partir da seção de exemplo [Pesquisar documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents#bkmk_examples) da [sintaxe de consulta simples](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) no gerenciador de pesquisa no portal.

+ Analise os [perfis de pontuação](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index) para ajustar a classificação no seu aplicativo de pesquisa.

+ Saiba como aplicar [analisadores léxicos específico do idioma](https://docs.microsoft.com/rest/api/searchservice/language-support).

+ [Configurar analisadores personalizados](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search) para o mínimo de processamento ou processamento especializado em campos específicos.

## <a name="see-also"></a>Confira também

[API REST para pesquisar documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

[Sintaxe de consulta simples](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) 

[Sintaxe de consulta Lucene completa](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search) 

[Controlar os resultados da pesquisa](https://docs.microsoft.com/azure/search/search-pagination-page-layout)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
[2]: ./media/search-lucene-query-architecture/azSearch-queryparsing-should2.png
[3]: ./media/search-lucene-query-architecture/azSearch-queryparsing-must2.png
[4]: ./media/search-lucene-query-architecture/azSearch-queryparsing-spacious2.png
