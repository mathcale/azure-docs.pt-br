---
title: 'Início Rápido: Traduzir uma fala em fala, C++ (Windows) – Serviço de Fala'
titleSuffix: Azure Cognitive Services
description: TBD
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 12/09/2019
ms.author: erhopf
ms.openlocfilehash: c06aa9a91fdf80b3c0b143eb6feb7dcfb3097ef0
ms.sourcegitcommit: 9ee0cbaf3a67f9c7442b79f5ae2e97a4dfc8227b
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "78925560"
---
## <a name="prerequisites"></a>Pré-requisitos

Antes de começar, é preciso:

> [!div class="checklist"]
> * [Criar um Recurso de Fala do Azure](../../../../get-started.md)
> * [Configurar seu ambiente de desenvolvimento e criar um projeto vazio](../../../../quickstarts/setup-platform.md?tabs=windows)

## <a name="add-sample-code"></a>Adicionar código de exemplo

1. Abra o arquivo de origem **helloworld.cpp**.

1. Substitua todo o código pelo seguinte snippet de código:

   ```C++
   #include <iostream>
   #include <vector>
   #include <speechapi_cxx.h>

   using namespace std;
   using namespace Microsoft::CognitiveServices::Speech;
   using namespace Microsoft::CognitiveServices::Speech::Translation;

   void TranslateSpeechToSpeech()
   {
       // Creates an instance of a speech translation config with specified subscription key and service region.
       // Replace with your own subscription key and region identifier from here: https://aka.ms/speech/sdkregion
       auto config = SpeechTranslationConfig::FromSubscription("YourSubscriptionKey", "YourServiceRegion");

       // Sets source and target languages.
       // Replace with the languages of your choice, from list found here: https://aka.ms/speech/sttt-languages
       auto fromLanguage = "en-US";
       auto toLanguage = "de";
       config->SetSpeechRecognitionLanguage(fromLanguage);
       config->AddTargetLanguage(toLanguage);

       // Sets the synthesis output voice name.
       // Replace with the languages of your choice, from list found here: https://aka.ms/speech/tts-languages
       config->SetVoiceName("de-DE-Hedda");

       // Creates a translation recognizer using the default microphone audio input device.
       auto recognizer = TranslationRecognizer::FromConfig(config);

       // Prepare to handle the synthesized audio data.
       recognizer->Synthesizing.Connect([](const TranslationSynthesisEventArgs& e)
       {
           auto size = e.Result->Audio.size();
           cout << "AUDIO SYNTHESIZED: " << size << " byte(s)" << (size == 0 ? "(COMPLETE)" : "") << std::endl;
       });

       // Starts translation, and returns after a single utterance is recognized. The end of a
       // single utterance is determined by listening for silence at the end or until a maximum of 15
       // seconds of audio is processed. The task returns the recognized text as well as the translation.
       // Note: Since RecognizeOnceAsync() returns only a single utterance, it is suitable only for single
       // shot recognition like command or query.
       // For long-running multi-utterance recognition, use StartContinuousRecognitionAsync() instead.
       cout << "Say something...\n";
       auto result = recognizer->RecognizeOnceAsync().get();

       // Checks result.
       if (result->Reason == ResultReason::TranslatedSpeech)
       {
           cout << "RECOGNIZED '" << fromLanguage << "': " << result->Text << std::endl;
           cout << "TRANSLATED into '" << toLanguage << "': " << result->Translations.at(toLanguage) << std::endl;
       }
       else if (result->Reason == ResultReason::RecognizedSpeech)
       {
           cout << "RECOGNIZED '" << fromLanguage << "' " << result->Text << " (text could not be translated)" << std::endl;
       }
       else if (result->Reason == ResultReason::NoMatch)
       {
           cout << "NOMATCH: Speech could not be recognized." << std::endl;
       }
       else if (result->Reason == ResultReason::Canceled)
       {
           auto cancellation = CancellationDetails::FromResult(result);
           cout << "CANCELED: Reason=" << (int)cancellation->Reason << std::endl;

           if (cancellation->Reason == CancellationReason::Error)
           {
               cout << "CANCELED: ErrorCode=" << (int)cancellation->ErrorCode << std::endl;
               cout << "CANCELED: ErrorDetails=" << cancellation->ErrorDetails << std::endl;
               cout << "CANCELED: Did you update the subscription info?" << std::endl;
           }
       }
   }

   int wmain()
   {
       TranslateSpeechToSpeech();
       return 0;
   }
   ```

1. No mesmo arquivo, substitua a cadeia de caracteres `YourSubscriptionKey` por sua chave de assinatura.

1. Substitua a cadeia de caracteres `YourServiceRegion` pela [região](~/articles/cognitive-services/Speech-Service/regions.md) associada à assinatura (por exemplo, `westus` para a assinatura de avaliação gratuita).

1. Na barra de menus, escolha **Arquivo** > **Salvar tudo**.

## <a name="build-and-run-the-application"></a>Compile e execute o aplicativo

1. Na barra de menus, selecione **Compilar** > **Compilar solução** para compilar o aplicativo. Agora, o código deve compilar sem erros.

1. Escolha **Depurar** > **Iniciar depuração** (ou pressione **F5**) para iniciar o aplicativo **Olá Mundo**.

1. Fale uma frase ou expressão em inglês. O aplicativo transmite sua fala para os serviço de Fala, que a traduz e transcreve em texto (neste caso, para alemão). O serviço de Fala retorna o áudio sintetizado e o texto para o aplicativo para exibição.

````
Say something...
AUDIO SYNTHESIZED: 76784 byte(s)
AUDIO SYNTHESIZED: 0 byte(s)(COMPLETE)
RECOGNIZED 'en-US': What's the weather in Seattle?
TRANSLATED into 'de': Wie ist das Wetter in Seattle?
````

## <a name="next-steps"></a>Próximas etapas

[!INCLUDE [footer](./footer.md)]
