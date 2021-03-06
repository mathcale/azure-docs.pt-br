---
title: Desenvolver módulos para o Azure IoT Edge | Microsoft Docs
description: Desenvolver módulos personalizados para o Azure IoT Edge que podem se comunicar com o runtime e o IoT Hub
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 07/22/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 96bd6b461a5374b5f5bc578c5f58dbcd09cd7087
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79271376"
---
# <a name="develop-your-own-iot-edge-modules"></a>Desenvolva seus próprios módulos do IoT Edge

Os módulos do Azure IoT Edge podem conectar outros serviços do Azure e contribuir para o pipeline de dados em nuvem maior. Este artigo descreve como é possível desenvolver módulos para comunicação com o runtime do IoT Edge e Hub IoT e, portanto, com o restante da nuvem do Azure.

## <a name="iot-edge-runtime-environment"></a>Ambiente de runtime do IoT Edge

O runtime do IoT Edge fornece a infraestrutura para integrar a funcionalidade de vários módulos do IoT Edge e implantá-los nos dispositivos IoT Edge. Qualquer programa pode ser embalado como um módulo IoT Edge. Para aproveitar ao máximo as funcionalidades de comunicação e gerenciamento do IoT Edge, um programa em execução em um módulo pode usar o Dispositivo IoT Azure SDK para se conectar ao hub ioT edge local.

## <a name="using-the-iot-edge-hub"></a>Usar o hub do IoT Edge

O hub do IoT Edge fornece duas funcionalidades principais: proxy para o Hub IoT e comunicações locais.

### <a name="iot-hub-primitives"></a>Primitivos do Hub IoT

O Hub IoT vê uma instância de módulo de forma análoga a um dispositivo, no sentido de que:

* possui um módulo gêmeo que é distinto e isolado do [dispositivo gêmeo](../iot-hub/iot-hub-devguide-device-twins.md) e do outro módulo gêmeos desse dispositivo;
* pode enviar [mensagens do dispositivo para a nuvem](../iot-hub/iot-hub-devguide-messaging.md);
* pode receber [métodos diretos](../iot-hub/iot-hub-devguide-direct-methods.md) destinados especificamente à sua identidade.

Atualmente, os módulos não podem receber mensagens de nuvem para dispositivo ou usar o recurso de upload de arquivos.

Ao escrever um módulo, você pode usar o [Dispositivo IoT Azure SDK](../iot-hub/iot-hub-devguide-sdks.md) para se conectar ao hub IoT Edge e usar a funcionalidade acima como você faria ao usar o IoT Hub com um aplicativo de dispositivo. A única diferença entre módulos IoT Edge e aplicativos de dispositivos IoT é que você tem que se referir à identidade do módulo em vez da identidade do dispositivo.

### <a name="device-to-cloud-messages"></a>Mensagens do dispositivo para a nuvem

Para habilitar o processamento complexo de mensagens de dispositivo para nuvem, o Hub do IoT Edge fornece roteamento declarativo de mensagens entre módulos e entre módulos e Hub IoT. O roteamento declarativo permite que os módulos interceptem e processem mensagens enviadas por outros módulos e os propaguem em pipelines complexos. Para obter mais informações, consulte [implantar módulos e estabelecer rotas no IoT Edge](module-composition.md).

Um módulo IoT Edge, em oposição a um aplicativo normal de dispositivo IoT Hub, pode receber mensagens dispositivo-nuvem que estão sendo fornecidas pelo seu hub local IoT Edge para processá-las.

O Hub do IoT Edge propaga as mensagens para o módulo com base nas rotas declarativas descritas no [manifesto de implantação](module-composition.md). Ao desenvolver um módulo IoT Edge, você pode receber essas mensagens definindo manipuladores de mensagens.

Para simplificar a criação de rotas, o IoT Edge inclui o conceito de pontos de extremidade de *entrada* e *saída*. Um módulo pode receber todas as mensagens de dispositivo para nuvem roteadas a ele sem especificar nenhuma entrada e pode enviar mensagens de dispositivo para nuvem sem especificar nenhuma saída. Usar entradas e saídas explícitas, porém, torna as regras de roteamento mais simples de entender.

Por fim, as mensagens de dispositivo para nuvem tratadas pelo hub do Edge são marcadas com as seguintes propriedades de sistema:

| Propriedade | Descrição |
| -------- | ----------- |
| $connectionDeviceId | A ID do dispositivo do cliente que enviou a mensagem |
| $connectionDeviceId | A ID do módulo que enviou a mensagem |
| $inputName | A entrada que recebeu esta mensagem. Pode ser um vazia. |
| $outputName | A saída usada para enviar a mensagem. Pode ser um vazia. |

### <a name="connecting-to-iot-edge-hub-from-a-module"></a>Conectar-se ao hub do IoT Edge a partir de um módulo

Conectar-se ao hub IoT Edge local a partir de um módulo envolve duas etapas:

1. Crie uma instância ModuleClient no aplicativo.
2. Verifique se que seu aplicativo aceita o certificado apresentado pelo hub do IoT Edge nesse dispositivo.

Crie uma instância ModuleClient para conectar o módulo ao Hub do IoT Edge em execução no dispositivo, de modo semelhante como as instâncias DeviceClient conectam dispositivos ao Hub IoT. Para obter mais informações sobre a classe ModuleClient e seus métodos de comunicação, consulte a referência da API para a sua linguagem SDK preferida: [C#](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.moduleclient?view=azure-dotnet), [C,](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothub-module-client-h) [Python,](https://docs.microsoft.com/python/api/azure-iot-device/azure.iot.device.iothubmoduleclient?view=azure-python) [Java](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.moduleclient?view=azure-java-stable)ou [Node.js](https://docs.microsoft.com/javascript/api/azure-iot-device/moduleclient?view=azure-node-latest).

## <a name="language-and-architecture-support"></a>Suporte à linguagem e arquitetura

O IoT Edge suporta vários sistemas operacionais, arquiteturas de dispositivos e linguagens de desenvolvimento para que você possa construir o cenário que corresponda às suas necessidades. Use esta seção para entender suas opções para desenvolver módulos de Borda IoT personalizados. Você pode aprender mais sobre suporte e requisitos de ferramentas para cada idioma em [Prepare seu ambiente de desenvolvimento e teste para IoT Edge](development-environment.md).

### <a name="linux"></a>Linux

Para todos os idiomas na tabela a seguir, o IoT Edge suporta o desenvolvimento de dispositivos Linux AMD64 e ARM32.

| Linguagem de desenvolvimento | Ferramentas de desenvolvimento |
| -------------------- | ----------------- |
| C | Visual Studio Code<br>Visual Studio 2017/2019 |
| C# | Visual Studio Code<br>Visual Studio 2017/2019 |
| Java | Visual Studio Code |
| Node.js | Visual Studio Code |
| Python | Visual Studio Code |

>[!NOTE]
>O suporte ao desenvolvimento e depuração para dispositivos ARM64 Linux está em [visualização pública](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para obter mais informações, confira [Desenvolver e depurar módulos do IoT Edge ARM64 no Visual Studio Code (versão prévia)](https://devblogs.microsoft.com/iotdev/develop-and-debug-arm64-iot-edge-modules-in-visual-studio-code-preview).

### <a name="windows"></a>Windows

Para todos os idiomas na tabela a seguir, o IoT Edge suporta o desenvolvimento de dispositivos AMD64 Windows.

| Linguagem de desenvolvimento | Ferramentas de desenvolvimento |
| -------------------- | ----------------- |
| C | Visual Studio 2017/2019 |
| C# | Visual Studio Code (sem recursos de depuração)<br>Visual Studio 2017/2019 |

## <a name="next-steps"></a>Próximas etapas

[Prepare o ambiente de desenvolvimento e teste para o IoT Edge](development-environment.md)

[Use o Visual Studio para desenvolver módulos C# para IoT Edge](how-to-visual-studio-develop-module.md)

[Use o Visual Studio Code para desenvolver módulos para IoT Edge](how-to-vs-code-develop-module.md)

[Entender e usar os SDKs de Hub IoT do Azure](../iot-hub/iot-hub-devguide-sdks.md)
