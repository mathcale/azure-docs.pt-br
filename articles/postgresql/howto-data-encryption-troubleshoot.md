---
title: Solucionar problemas de criptografia de dados - Banco de Dados Azure para PostgreSQL - Servidor Único
description: Saiba como solucionar problemas na criptografia de dados no banco de dados do Azure para PostgreSQL - Single Server
author: kummanish
ms.author: manishku
ms.service: postgresql
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 2902ff17ac14a48f1a11259339c2ab1bc4595980
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79299253"
---
# <a name="troubleshoot-data-encryption-in-azure-database-for-postgresql---single-server"></a>Solucionar problemas de criptografia de dados no Banco de Dados Do Azure para PostgreSQL - Servidor Único

Este artigo ajuda você a identificar e resolver problemas comuns que podem ocorrer na implantação de um servidor único do Banco de Dados Azure para PostgreSQL quando configurado com criptografia de dados usando uma chave gerenciada pelo cliente.

## <a name="introduction"></a>Introdução

Quando você configura a criptografia de dados para usar uma chave gerenciada pelo cliente no Azure Key Vault, o servidor requer acesso contínuo à chave. Se o servidor perder o acesso à chave gerenciada pelo cliente no Azure Key Vault, ele negará todas as conexões, devolverá a mensagem de erro apropriada e mudará seu estado para ***Inacessível*** no portal Azure.

Se você não precisar mais de um banco de dados Azure inacessível para servidor PostgreSQL, você pode excluí-lo para parar de incorrer em custos. Nenhuma outra ação no servidor é permitida até que o acesso ao cofre principal seja restaurado e o servidor esteja disponível. Também não é possível alterar a opção `Yes`de criptografia de `No` dados de (gerenciada pelo cliente) para (gerenciada por serviço) em um servidor inacessível quando ele é criptografado com uma chave gerenciada pelo cliente. Você terá que revalidar a chave manualmente antes que o servidor seja acessível novamente. Essa ação é necessária para proteger os dados contra acesso não autorizado, enquanto as permissões para a chave gerenciada pelo cliente são revogadas.

## <a name="common-errors-causing-server-to-become-inaccessible"></a>Erros comuns que causam o servidor ficam inacessíveis

As seguintes configurações erradas causam a maioria dos problemas com criptografia de dados que usam chaves do Azure Key Vault:

- O cofre-chave não está disponível ou não existe:
  - O cofre de chaves foi excluído por engano.
  - Um erro de rede intermitente faz com que o cofre de chaves fique indisponível.

- Você não tem permissões para acessar o cofre da chave ou a chave não existe:
  - A chave expirou ou foi acidentalmente excluída ou desativada.
- A identidade gerenciada do Banco de Dados Azure para a ocorrência PostgreSQL foi acidentalmente excluída.
  - A identidade gerenciada do Banco de Dados Azure para a instância PostgreSQL tem permissões-chave insuficientes. Por exemplo, as permissões não incluem Get, Wrap e Desembrulhar.
  - As permissões de identidade gerenciadas para o Banco de Dados Do Azure para a instância PostgreSQL foram revogadas ou excluídas.

## <a name="identify-and-resolve-common-errors"></a>Identificar e resolver erros comuns

### <a name="errors-on-the-key-vault"></a>Erros no cofre da chave

#### <a name="disabled-key-vault"></a>Cofre de chaves desativado

- `AzureKeyVaultKeyDisabledMessage`
- **Explicação**: A operação não pôde ser concluída no servidor porque a chave do Azure Key Vault está desativada.

#### <a name="missing-key-vault-permissions"></a>Permissões de cofre de chaves perdidas

- `AzureKeyVaultMissingPermissionsMessage`
- **Explicação**: O servidor não tem as permissões necessárias para obter, embrulhar e desembrulhar permissões para o Azure Key Vault. Conceda todas as permissões perdidas ao diretor de serviço com id.

### <a name="mitigation"></a>Atenuação

- Confirme se a chave gerenciada pelo cliente está presente no cofre da chave.
- Identifique o cofre de chaves e vá até ele no portal do Azure.
- Certifique-se de que a chave URI identifique uma chave presente.

## <a name="next-steps"></a>Próximas etapas

[Use o portal Azure para configurar criptografia de dados com uma chave gerenciada pelo cliente no Banco de Dados Do Azure para PostgreSQL](howto-data-encryption-portal.md)
