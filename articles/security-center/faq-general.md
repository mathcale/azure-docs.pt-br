---
title: Faq do Azure Security Center - Perguntas gerais
description: Perguntas gerais frequentes sobre o Azure Security Center, um produto que ajuda você a prevenir, detectar e responder a ameaças
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/25/2020
ms.author: memildin
ms.openlocfilehash: 54263a8be900ed381d8450b460e5cf9e6117ac54
ms.sourcegitcommit: ced98c83ed25ad2062cc95bab3a666b99b92db58
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/31/2020
ms.locfileid: "80436129"
---
# <a name="faq---general-questions-about-azure-security-center"></a>FAQ - Perguntas gerais sobre o Azure Security Center

## <a name="what-is-azure-security-center"></a>O que é a Central de Segurança do Azure?
O Azure Security Center ajuda você a prevenir, detectar e responder a ameaças com maior visibilidade e controle sobre a segurança de seus recursos. Ela permite o gerenciamento de políticas e o monitoramento da segurança integrada entre suas assinaturas, ajuda a detectar ameaças que poderiam passar despercebidas e funciona com uma enorme variedade de soluções de segurança.

O Security Center usa o agente Log Analytics para coletar e armazenar dados. Para obter detalhes aprofundados, consulte [coleta de dados no Azure Security Center](security-center-enable-data-collection.md).


## <a name="how-do-i-get-azure-security-center"></a>Como posso obter a Central de Segurança do Azure?
A Central de Segurança do Azure é habilitada com sua assinatura do Microsoft Azure e pode ser acessada no [portal do Azure](https://azure.microsoft.com/features/azure-portal/). Para acessá-lo, [entre no portal,](https://portal.azure.com) **selecione Procurar**e role até o Security **Center**.


## <a name="which-azure-resources-are-monitored-by-azure-security-center"></a>Quais recursos do Azure são monitorados pela Central de Segurança do Azure?
A Central de Segurança do Azure monitora os seguintes recursos do Azure:

* Máquinas virtuais (VMs) (incluindo [Serviços em Nuvem)](../cloud-services/cloud-services-choose-me.md)
* conjuntos de escala de máquina virtual
* Redes Virtuais do Azure
* Contêineres
* Serviço do SQL Azure
* Conta de Armazenamento do Azure
* Aplicativos Web do Azure (em um [Ambiente do Serviço de Aplicativo](../app-service/environment/intro.md))
* Soluções de parceiros integradas com sua assinatura do Azure, como um firewall de aplicativo Web em VMs e no Ambiente do Serviço de Aplicativo

Além disso, máquinas não-Azure (incluindo no local) também podem ser monitoradas pelo Azure Security Center. Tanto [as máquinas Windows](./quick-onboard-windows-computer.md) quanto as [máquinas Linux](./quick-onboard-linux-computer.md) são suportadas.


## <a name="how-can-i-see-the-current-security-state-of-my-azure-resources"></a>Como é possível ver o estado atual da segurança dos meus recursos do Azure?
A página **Visão Geral do Centro de Segurança** mostra a postura geral de segurança do ambiente discriminada por Dados de Computação, Rede, armazenamento & dados e Aplicativos. Cada tipo de recurso tem um indicador que mostra vulnerabilidades de segurança identificadas. Clicar em cada bloco exibe uma lista dos problemas de segurança identificados pela Central de Segurança, junto com um inventário dos recursos em sua assinatura.



## <a name="what-is-a-security-policy"></a>O que é uma política de segurança?
Uma política de segurança define o conjunto de controles recomendados para os recursos na assinatura ou grupo de recursos especificado. Na Central de Segurança do Azure, você pode definir as políticas para as assinaturas do Azure de acordo com os requisitos de segurança de sua empresa e os tipos de aplicativos ou a confidencialidade dos dados de cada assinatura.

As políticas de segurança habilitadas na Central de Segurança do Azure determinam as recomendações de segurança e o monitoramento. Para saber mais sobre as políticas de segurança, consulte a seção [Monitoramento de integridade de segurança na Central de segurança do Azure](security-center-monitoring.md).


## <a name="who-can-modify-a-security-policy"></a>Quem pode modificar uma política de segurança?
Para modificar uma política de segurança, você deve ser um Administrador de Segurança ou o Proprietário ou Colaborador dessa assinatura.

Para saber como configurar uma política de segurança, consulte a seção [Como configurar de políticas de segurança na Central de segurança do Azure](tutorial-security-policy.md).


## <a name="what-is-a-security-recommendation"></a>O que é são recomendações de segurança?
A Central de Segurança do Azure analisa o estado de segurança de seus recursos do Azure. Quando possíveis vulnerabilidades de segurança são identificadas, são criadas recomendações. As recomendações guiam você pelo processo de configuração do controle necessário. São exemplos:

* Provisionamento de antimalware para ajudar a identificar e remover software mal-intencionado
* [Grupos de segurança de rede](../virtual-network/security-overview.md) e regras para controlar o tráfego para máquinas virtuais
* Provisionamento de um Firewall do Aplicativo Web para ajudar a proteger contra ataques direcionados aos seus aplicativos Web
* Como implantar atualizações de sistema ausentes
* Endereçamento de configurações do sistema operacional que não coincidem com as linhas de base recomendadas

Somente as recomendações que são habilitadas nas Políticas de segurança são mostradas aqui.



## <a name="what-triggers-a-security-alert"></a>O que dispara um alerta de segurança?
A Central de Segurança do Azure automaticamente coleta, analisa e funde os dados de log de seus recursos do Azure, da rede e das soluções de parceiros como antimalware e firewalls. Quando forem detectadas ameaças, é criado um alerta de segurança. Os exemplos abrangem a detecção de:

* As máquinas virtuais comprometidas se comunicam com os endereços IP mal-intencionados conhecidos
* Malware avançado detectado com o relatório de erros do Windows
* Ataques por força bruta contra máquinas virtuais
* Alertas de segurança das soluções de segurança de parceiro integradas, como antimalware ou Firewalls de aplicativo Web


## <a name="why-did-secure-score-values-change"></a>Por que os valores do Secure Score mudaram? <a name="secure-score-faq"></a>
A partir de fevereiro de 2019, o Security Center ajustou a pontuação de algumas recomendações, a fim de melhor se adequar à sua gravidade. Como resultado desse ajuste, pode haver alterações nos valores gerais do Secure Score.  Para obter mais informações sobre o Secure Score, consulte [o cálculo Secure Score](security-center-secure-score.md).


## <a name="whats-the-difference-between-threats-detected-and-alerted-on-by-microsoft-security-response-center-versus-azure-security-center"></a>Qual é a diferença entre ameaças detectadas e alertadas pelo Microsoft Security Response Center versus pela Central de Segurança do Azure?
O MSRC (Microsoft Security Response Center) executa determinado monitoramento de segurança da rede e da infraestrutura do Azure e recebe reclamações de inteligência e abuso de ameaça de terceiros. Quando o MSRC fica ciente de que os dados do cliente foram acessados por uma pessoa não autorizada ou ilegal ou que o uso do cliente do Azure não está de acordo com os termos de Uso Aceitável, um gerente de incidentes de segurança notifica o cliente. Normalmente, a notificação ocorre com o envio de um email para os contatos de segurança especificados na Central de Segurança do Azure ou para o proprietário da assinatura do Azure, caso um contato de segurança não seja especificado.

A Central de Segurança é um serviço do Azure que monitora o ambiente do cliente do Azure continuamente e aplica a análise para detectar automaticamente uma ampla gama de atividades potencialmente mal-intencionadas. Essas detecções são exibidas como alertas de segurança no painel da Central de Segurança.