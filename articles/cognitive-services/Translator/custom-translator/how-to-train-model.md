---
title: Treinar um modelo - tradutor personalizado
titleSuffix: Azure Cognitive Services
description: Treinar um modelo é um passo importante na construção de um modelo de tradução. O treinamento acontece com base nos documentos selecionados para esses treinamentos.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 02/21/2019
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: a875cc8f5e69be8b940f9d177d3b915bfaa2c973
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "68595689"
---
# <a name="train-a-model"></a>Treinar um modelo

Treinar um modelo é o passo importante para construir um modelo de tradução, porque sem um treinamento, o modelo não pode ser construído. O treinamento acontece com base nos documentos selecionados para os treinamentos.

Para treinar um modelo:

1.  Selecione o projeto no qual você deseja construir um modelo.

2.  A guia Dados para o projeto mostrará todos os documentos relevantes para o par de idiomas do projeto. Selecione manualmente os documentos que você deseja usar para treinar seu modelo. Você pode selecionar documentos de treinamento, ajuste e teste nessa tela. Além disso, basta selecionar o conjunto de treinamento e fazer com que o Custom Translator crie os conjuntos de ajustes e testes para você.

    -  Nome do documento: nome do documento.

    -  Emparelhamento: se este documento for um documento paralelo ou monolíngue. Atualmente, documentos monolíngues não são suportados para treinamento.

    -  Tipo de documento: pode ser treinamento, ajuste, teste ou dicionário.

    -  Par de idiomas: mostra o idioma de origem e de destino do projeto.

    -  Frases de origem: Mostra o número de frases extraídas do arquivo de origem.

    -  Frases-alvo: Mostra o número de sentenças extraídas do arquivo de destino.

    ![Treinar um modelo](media/how-to/how-to-train-model.png)

3.  Clique o botão de treinamento.

4.  Na caixa de diálogo, especifique um nome para o seu modelo.

5.  Clique em modelo de treinamento.

    ![Caixa de diálogo de modelo de treinamento](media/how-to/how-to-train-model-2.png)

6.  Conversor personalizado enviar o treinamento e mostrar o status do treinamento na guia modelos.

    ![Página de modelo de treinamento](media/how-to/how-to-train-model-3.png)

>[!Note]
>O Personal Translator suporta 10 treinamentos simultâneos dentro de um espaço de trabalho em qualquer momento.


## <a name="edit-a-model"></a>Editar um modelo

Você pode editar um modelo usando o link de edição na página de detalhes do modelo.

1.  Clique no ícone do lápis.

    ![Editar modelo](media/how-to/how-to-edit-model.png)

2.  A alteração de caixa de diálogo,

    1.  Modelo de nome (obrigatório): dê um nome significativo ao seu modelo.

        ![Mais de caixa de diálogo Editar](media/how-to/how-to-edit-model-dialog.png)

3.  Clique em Salvar.


## <a name="next-steps"></a>Próximas etapas

- Saiba mais [como exibir detalhes do modelo](how-to-view-model-details.md).
