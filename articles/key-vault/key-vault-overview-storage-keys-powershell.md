---
title: Conta de armazenamento gerenciado do Azure Key Vault - versão PowerShell
description: O recurso de conta de armazenamento gerenciado fornece uma integração perfeita entre o Azure Key Vault e uma conta de armazenamento Azure.
ms.topic: conceptual
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/10/2019
ms.openlocfilehash: 833f78d89a1a9033e62c10c3b16c5adfc65e1da4
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78195083"
---
# <a name="manage-storage-account-keys-with-key-vault-and-azure-powershell"></a>Gerenciar chaves de conta de armazenamento com Key Vault e Azure PowerShell

Uma conta de armazenamento Azure usa credenciais que incluem um nome de conta e uma chave. A chave é gerada automaticamente e serve como uma senha, em vez de uma chave criptográfica. O Key Vault gerencia as chaves da conta de armazenamento armazenando-as como [segredos do Key Vault](/azure/key-vault/about-keys-secrets-and-certificates#key-vault-secrets). 

Você pode usar o recurso de chave de armazenamento gerenciado do Key Vault para listar (sincronizar) chaves com uma conta de armazenamento Azure e regenerar (girar) as chaves periodicamente. Você pode gerenciar chaves para contas de armazenamento e contas de armazenamento Classic.

Ao usar o recurso de chave da conta de armazenamento gerenciado, considere os seguintes pontos:

- Os valores-chave nunca são devolvidos em resposta a um chamador.
- Apenas o Key Vault deve gerenciar as chaves da sua conta de armazenamento. Não gerencie as chaves você mesmo e evite interferir nos processos do Key Vault.
- Apenas um único objeto do Key Vault deve gerenciar as chaves da conta de armazenamento. Não permita o gerenciamento de chaves de vários objetos.
- Você pode solicitar o Key Vault para gerenciar sua conta de armazenamento com um diretor de usuário, mas não com um diretor de serviço.
- Regenerar chaves usando apenas o Key Vault. Não regenere manualmente as chaves da conta de armazenamento.

Recomendamos o uso da integração do Azure Storage com o Azure Active Directory (Azure AD), o serviço de gerenciamento de identidade e acesso baseado na nuvem da Microsoft. A integração do Azure AD está disponível para [blobs e filas do Azure](../storage/common/storage-auth-aad.md)e fornece acesso baseado em token OAuth2 ao Azure Storage (assim como o Azure Key Vault).

O Azure AD permite que você autentique seu aplicativo cliente usando um aplicativo ou identidade de usuário, em vez de credenciais de conta de armazenamento. Você pode usar [uma identidade gerenciada do Azure AD](/azure/active-directory/managed-identities-azure-resources/) quando você executar no Azure. As identidades gerenciadas removem a necessidade de autenticação do cliente e armazenamento de credenciais dentro ou com seu aplicativo.

O Azure AD usa o RBAC (Role-Based Access Control, controle de acesso baseado em função) para gerenciar a autorização, que também é suportada pelo Key Vault.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="service-principal-application-id"></a>ID do aplicativo principal do serviço

Um inquilino Azure AD fornece cada aplicativo registrado com um [diretor de serviço](/azure/active-directory/develop/developer-glossary#service-principal-object). O principal serviço serve como iD de aplicativo, que é usado durante a configuração de autorização para acesso a outros recursos do Azure via RBAC.

Key Vault é um aplicativo da Microsoft pré-registrado em todos os inquilinos do Azure AD. O Key Vault está registrado o mesmo ID de aplicativo em cada nuvem do Azure.

| Locatários | Nuvem | ID do aplicativo |
| --- | --- | --- |
| AD do Azure | Azure Government | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| AD do Azure | Público do Azure | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Outros  | Qualquer | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Pré-requisitos

Para completar este guia, você deve primeiro fazer o seguinte:

- [Instale o módulo Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.6.0).
- [Criar um cofre de chaves](quick-create-powershell.md)
- [Crie uma conta de armazenamento Azure.](../storage/common/storage-account-create.md?tabs=azure-powershell) O nome da conta de armazenamento deve usar apenas letras e números minúsculos. O comprimento do nome deve ser entre 3 e 24 caracteres.
      

## <a name="manage-storage-account-keys"></a>Gerenciar chaves da conta de armazenamento

### <a name="connect-to-your-azure-account"></a>Conectar-se à sua conta do Azure

Autue sua sessão PowerShell usando o cmdlet [Connect-AzAccount.](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0) 

```azurepowershell-interactive
Connect-AzAccount
```
Se você tiver várias assinaturas do Azure, você pode listá-las usando o cmdlet [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription?view=azps-2.5.0) e especificar a assinatura que deseja usar com o cmdlet [Set-AzContext.](/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0) 

```azurepowershell-interactive
Set-AzContext -SubscriptionId <subscriptionId>
```

### <a name="set-variables"></a>Definir variáveis

Primeiro, defina as variáveis a serem usadas pelos cmdlets PowerShell nas etapas seguintes. Certifique-se de <YourResourceGroupName> <YourStorageAccountName>atualizar <YourKeyVaultName> o , e os `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` espaços reservados, e definir $keyVaultSpAppId para (conforme especificado no [ID de aplicativo principal do serviço](#service-principal-application-id), acima).

Também usaremos os cmdlets Azure PowerShell [Get-AzContext](/powershell/module/az.accounts/get-azcontext?view=azps-2.6.0) e [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount?view=azps-2.6.0) para obter seu ID de usuário e o contexto de sua conta de armazenamento Azure.

```azurepowershell-interactive
$resourceGroupName = <YourResourceGroupName>
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>
$keyVaultSpAppId = "cfa8b339-82a2-471a-a3c9-0fc0be7a4093"
$storageAccountKey = "key1"

# Get your User Id
$userId = (Get-AzContext).Account.Id

# Get a reference to your Azure storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName
```

### <a name="give-key-vault-access-to-your-storage-account"></a>Dê acesso ao Key Vault à sua conta de armazenamento

Antes que o Key Vault possa acessar e gerenciar suas chaves de conta de armazenamento, você deve autorizar o acesso a sua conta de armazenamento. O aplicativo do Key Vault requer permissões para *lista* e *regenerar* chaves para sua conta de armazenamento. Essas permissões são habilitadas por meio da função RBAC interna [Função do Serviço do Operador de Chaves da Conta de Armazenamento](/azure/role-based-access-control/built-in-roles#storage-account-key-operator-service-role). 

Atribua essa função ao principal do serviço Key Vault, limitando o escopo à sua conta de armazenamento, usando o cmdlet Azure PowerShell [New-AzRoleAssignment.](/powershell/module/az.resources/new-azroleassignment?view=azps-2.6.0)

```azurepowershell-interactive
# Assign RBAC role "Storage Account Key Operator Service Role" to Key Vault, limiting the access scope to your storage account. For a classic storage account, use "Classic Storage Account Key Operator Service Role." 
New-AzRoleAssignment -ApplicationId $keyVaultSpAppId -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storageAccount.Id
```

Após a atribuição de função bem-sucedida, você verá uma saída semelhante ao exemplo a seguir:

```console
RoleAssignmentId   : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso/providers/Microsoft.Authorization/roleAssignments/189cblll-12fb-406e-8699-4eef8b2b9ecz
Scope              : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
DisplayName        : Azure Key Vault
SignInName         :
RoleDefinitionName : storage account Key Operator Service Role
RoleDefinitionId   : 81a9662b-bebf-436f-a333-f67b29880f12
ObjectId           : 93c27d83-f79b-4cb2-8dd4-4aa716542e74
ObjectType         : ServicePrincipal
CanDelegate        : False
```

Se o Key Vault já foi adicionado à função na sua conta de armazenamento, você receberá a mensagem *"A atribuição de função já existe".* erro. Você também pode verificar a atribuição de função usando a página de "Controle de acesso (IAM)" da conta de armazenamento no portal do Azure.  

### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Dê a sua permissão de conta de usuário para as contas de armazenamento gerenciadas

Use o cmdlet Azure PowerShell [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy?view=azps-2.6.0) para atualizar a política de acesso do Key Vault e conceder permissões de conta de armazenamento à sua conta de usuário.

```azurepowershell-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $userId -PermissionsToStorage get, list, delete, set, update, regeneratekey, getsas, listsas, deletesas, setsas, recover, backup, restore, purge
```

Observe que as permissões para contas de armazenamento não estão disponíveis na página de "Políticas de acesso" da conta de armazenamento no portal do Azure.

### <a name="add-a-managed-storage-account-to-your-key-vault-instance"></a>Adicione uma conta de armazenamento gerenciada para a sua instância do Key Vault

Use o cmdlet [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount?view=azps-2.6.0) do Azure PowerShell para criar uma conta de armazenamento gerenciada na instância do Cofre-chave. O switch `-DisableAutoRegenerateKey` especifica NÃO regenerar as chaves da conta de armazenamento.

```azurepowershell-interactive
# Add your storage account to your Key Vault's managed storage accounts

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -DisableAutoRegenerateKey
```

Após a adição bem sucedida da conta de armazenamento sem regeneração de chave, você verá saída semelhante para o exemplo a seguir:

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : False
Regeneration Period : 90.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                : 
```

### <a name="enable-key-regeneration"></a>Habilitar regeneração de chave

Se você quiser que o Key Vault regenere as chaves da conta de armazenamento periodicamente, você pode usar o cmdlet Azure PowerShell [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount?view=azps-2.6.0) para definir um período de regeneração. Neste exemplo, estabelecemos um período de regeneração de três dias. Após três dias, o Key Vault regenerará a 'key2' e trocará a tecla ativa de 'key2' para 'key1'.

```azurepowershell-interactive
$regenPeriod = [System.Timespan]::FromDays(3)

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -RegenerationPeriod $regenPeriod
```

Após a adição bem-sucedida da conta de armazenamento com a regeneração de chave, você verá uma saída semelhante ao exemplo a seguir:

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : True
Regeneration Period : 3.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                : 
```

## <a name="shared-access-signature-tokens"></a>Tokens de assinatura de acesso compartilhado

Você também pode pedir ao Key Vault para gerar tokens de assinatura de acesso compartilhado. Uma assinatura de acesso compartilhado fornece acesso delegado aos recursos da sua conta de armazenamento. Você pode conceder aos clientes acesso a recursos em sua conta de armazenamento sem compartilhar as chaves de sua conta. Uma assinatura de acesso compartilhado fornece uma maneira segura de compartilhar seus recursos de armazenamento sem comprometer as chaves de sua conta.

Os comandos desta seção completam as seguintes ações:

- Defina uma definição de assinatura de acesso compartilhado de conta. 
- Crie um token de assinatura de acesso compartilhado de conta para os serviços Blob, File, Table e Queue. O token é criado para os tipos de recursos Serviço, Contêiner e Objeto. O token é criado com todas as permissões, sobre https e com as datas de início e término especificadas.
- Defina uma definição de assinatura de acesso compartilhada do Key Vault no cofre. A definição tem o modelo URI do token de assinatura de acesso compartilhado que foi criado. A definição tem o tipo `account` de assinatura de acesso compartilhado e é válida por N dias.
- Verifique se a assinatura de acesso compartilhado foi salva no cofre da chave como um segredo.
- 
### <a name="set-variables"></a>Definir variáveis

Primeiro, defina as variáveis a serem usadas pelos cmdlets PowerShell nas etapas seguintes. Certifique-se de <YourStorageAccountName> <YourKeyVaultName> atualizar os espaços reservados e espaços reservados.

Também usaremos os cmdlets Azure PowerShell [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext?view=azps-2.6.0) para obter o contexto da sua conta de armazenamento Azure.

```azurepowershell-interactive
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>

$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -Protocol Https -StorageAccountKey Key1
```

### <a name="create-a-shared-access-signature-token"></a>Crie um token de assinatura de acesso compartilhado

Crie uma definição de assinatura de acesso compartilhado usando os cmdlets do Azure PowerShell [New-AzStorageAccountSASToken.](/powershell/module/az.storage/new-azstorageaccountsastoken?view=azps-2.6.0)
 
```azurepowershell-interactive
$start = [System.DateTime]::Now.AddDays(-1)
$end = [System.DateTime]::Now.AddMonths(1)

$sasToken = New-AzStorageAccountSasToken -Service blob,file,Table,Queue -ResourceType Service,Container,Object -Permission "racwdlup" -Protocol HttpsOnly -StartTime $start -ExpiryTime $end -Context $storageContext
```
O valor de $sasToken será semelhante a isso.

```console
?sv=2018-11-09&sig=5GWqHFkEOtM7W9alOgoXSCOJO%2B55qJr4J7tHQjCId9S%3D&spr=https&st=2019-09-18T18%3A25%3A00Z&se=2019-10-19T18%3A25%3A00Z&srt=sco&ss=bfqt&sp=racupwdl
```

### <a name="generate-a-shared-access-signature-definition"></a>Gerar uma definição de assinatura de acesso compartilhado

Use o cmdlet Azure PowerShell [Set-AzKeyVaultManagedStorageSasDefinition](/powershell/module/az.keyvault/set-azkeyvaultmanagedstoragesasdefinition?view=azps-2.6.0) para criar uma definição de assinatura de acesso compartilhado.  Você pode fornecer o nome `-Name` de sua escolha para o parâmetro.

```azurepowershell-interactive
Set-AzKeyVaultManagedStorageSasDefinition -AccountName $storageAccountName -VaultName $keyVaultName -Name <YourSASDefinitionName> -TemplateUri $sasToken -SasType 'account' -ValidityPeriod ([System.Timespan]::FromDays(30))
```

### <a name="verify-the-shared-access-signature-definition"></a>Verifique a definição de assinatura de acesso compartilhado

Você pode verificar se a definição de assinatura de acesso compartilhado foi armazenada no cofre principal usando o cmdlet Azure PowerShell [Get-AzKeyVaultSecret.](/powershell/module/az.keyvault/get-azkeyvaultsecret?view=azps-2.6.0)

Primeiro, encontre a definição de assinatura de acesso compartilhado em seu cofre de chaves.

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName <YourKeyVaultName>
```

O segredo correspondente à sua definição de SAS terá essas propriedades:

```console
Vault Name   : <YourKeyVaultName>
Name         : <SecretName>
...
Content Type : application/vnd.ms-sastoken-storage
Tags         :
```

Agora você pode usar o [cmdlet Get-AzKeyVaultSecret](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) e a propriedade secreta `Name` para visualizar o conteúdo desse segredo.

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName <YourKeyVaultName> -Name <SecretName>

Write-Host $secret.SecretValueText
```

A saída deste comando mostrará sua seqüência de definição SAS.


## <a name="next-steps"></a>Próximas etapas

- [Amostras da conta de armazenamento gerenciada](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
- [Sobre chaves, segredos e certificados](about-keys-secrets-and-certificates.md)
- [Referência do PowerShell do Key Vault](/powershell/module/az.keyvault/?view=azps-1.2.0#key_vault)
