---
title: Transparent Data Encryption
description: Uma visão geral da criptografia de dados transparente para o SQL Database e o Synapse SQL no Azure Synapse Analytics. O documento abrange os benefícios e as opções de configuração que incluem Transparent Data Encryption gerenciado pelo serviço e Bring Your Own Key.
services: sql-database
ms.service: sql-database
ms.subservice: security
titleSuffix: Azure SQL Database and Azure Synapse
ms.custom: seo-lt-2019
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 02/06/2020
ms.openlocfilehash: 982a59f1eb8717e2fe2d86728cdae731c919aaf0
ms.sourcegitcommit: efefce53f1b75e5d90e27d3fd3719e146983a780
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 04/01/2020
ms.locfileid: "80476957"
---
# <a name="transparent-data-encryption-for-sql-database-and-azure-synapse"></a>Criptografia de dados transparente para banco de dados SQL e Sinapse Azure

A criptografia de dados transparente (TDE) ajuda a proteger o Banco de Dados SQL do Azure, a instância gerenciada do Azure SQL e o Synapse SQL no Azure Synapse Analytics contra a ameaça de atividade off-line maliciosa criptografando dados em repouso. Ela realiza a criptografia e a descriptografia em tempo real do banco de dados, de backups associados e de arquivos de log de transações em repouso, sem a necessidade de alterações no aplicativo. Por padrão, a TDE está habilitada para todos os bancos de dados SQL do Azure recém-implantados. A TDE não pode ser utilizada para criptografar o banco de dados **mestre** lógico no Banco de Dados SQL.  O banco de dados **mestre** contém os objetos necessários para executar as operações de TDE nos bancos de dados do usuário.

O TDE precisa ser ativado manualmente para bancos de dados mais antigos do Banco de Dados SQL Do Azure, a Instância Gerenciada do Azure SQL ou do Azure Synapse.
Bancos de dados de instância gerenciada criados através da restauração herdam o status de criptografia do banco de dados de origem.

A Transparent Data Encryption criptografa o armazenamento de um banco de dados inteiro usando uma chave simétrica chamada de chave de criptografia de banco de dados. Esta chave de criptografia de banco de dados é protegida pelo protetor de Transparent Data Encryption. O protetor é um certificado de serviço gerenciado (Transparent Data Encryption de serviço gerenciado) ou uma chave assimétrica armazenada no Azure Key Vault (Bring Your Own Key). Você define o protetor de criptografia de dados transparente no nível do servidor para o Banco de Dados SQL do Azure e o Synapse do Azure e o nível de instância para a instância gerenciada do Azure SQL. O termo *servidor* refere-se ao servidor e instância ao longo deste documento, a menos que indicado de forma diferente.

Na inicialização do banco de dados, a chave de criptografia de banco de dados criptografada é descriptografada e, em seguida, usada para descriptografia e nova criptografia dos arquivos de banco de dados no processo do Mecanismo de Banco de Dados do SQL Server. A Transparent Data Encryption executa criptografia e descriptografia de E/S em tempo real dos dados no nível de página. Cada página é descriptografada quando é lida na memória e, em seguida, criptografada antes de ser gravada no disco. Para obter uma descrição geral da criptografia de dados transparente, consulte [criptografia de dados transparente](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption).

O SQL Server em execução em uma máquina virtual do Azure também pode usar uma chave assimétrica do Key Vault. As etapas de configuração são diferentes de usar uma chave assimétrica no Banco de Dados SQL e Instância Gerenciada do Banco de Dados SQL. Para obter mais informações, consulte [Gerenciamento de chave extensível usando Azure Key Vault (SQL Server)](https://docs.microsoft.com/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server).

## <a name="service-managed-transparent-data-encryption"></a>Transparent Data Encryption de serviço gerenciado

No Azure, a configuração padrão para Transparent Data Encryption é a chave de criptografia do banco de dados protegida por um certificado de servidor interno. O certificado de servidor incorporado é exclusivo para cada servidor e o algoritmo de criptografia usado é o AES 256. Se um banco de dados estiver em um relacionamento de replicação geográfica, tanto o banco de dados primário como o banco de dados geográfico secundário serão protegidos pela chave do servidor pai do banco de dados primário. Se dois bancos de dados estiverem conectados ao mesmo servidor, eles também compartilharão o mesmo certificado interno.  A Microsoft gira automaticamente esses certificados em conformidade com a política de segurança interna e a chave raiz é protegida por um armazenamento secreto interno da Microsoft.  Os clientes podem verificar a conformidade do Banco de Dados SQL com políticas de segurança interna em relatórios de auditoria independentes de terceiros disponíveis no [Microsoft Trust Center](https://servicetrust.microsoft.com/).

A Microsoft também move e gerencia as chaves conforme necessário para replicação geográfica e restaurações.

> [!IMPORTANT]
> Todos os bancos de dados SQL recém-criados e bancos de dados de instância gerenciada são criptografados por padrão usando criptografia de dados transparente gerenciada por serviço. Os bancos de dados SQL existentes criados antes de maio de 2017 e bancos de dados SQL criados por meio de restauração, georeplicação e cópia de banco de dados não são criptografados por padrão. Os bancos de dados de instância gerenciada existentes criados antes de fevereiro de 2019 não são criptografados por padrão. Bancos de dados de instância gerenciada criados através da restauração herdam o status de criptografia da fonte.

## <a name="customer-managed-transparent-data-encryption---bring-your-own-key"></a>Transparent Data Encryption gerenciada pelo cliente – Bring Your Own Key

O [TDE com chaves gerenciadas pelo cliente no Azure Key Vault](transparent-data-encryption-byok-azure-sql.md) permite criptografar a DEK (Chave de Criptografia do Banco de Dados) com uma chave assimétrica gerenciada pelo cliente chamada Protetor de TDE.  Isso também é geralmente chamado de suporte a BYOK (Bring Your Own Key) para Transparent Data Encryption. No cenário BYOK, o TDE Protector é armazenado em um [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault)de propriedade do cliente e gerenciado , o sistema de gerenciamento de chaves externas baseado em nuvem do Azure. O Protetor TDE pode ser [gerado pelo cofre de chaves ou transferido para o cofre de chaves](https://docs.microsoft.com/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys) a partir de um dispositivo HSM no local. A DEK de TDE, que é armazenada na página de inicialização de um banco de dados, é criptografada e descriptografada pelo Protetor de TDE, que é armazenado no Azure Key Vault e nunca deixa o cofre de chaves.  O Banco de Dados SQL precisa ter permissões concedidas para o cofre de chaves de propriedade do cliente para descriptografar e criptografar a DEK. Se as permissões do SQL Server lógico para o cofre de chaves forem revogadas, um banco de dados não poderá ser acessado e todos os dados serão criptografados. Para o Banco de Dados SQL do Azure, o protetor de TDE é definido no nível do SQL Server lógico e é herdado por todos os bancos de dados associados a esse servidor. Para a [Instância Gerenciada do SQL do Azure](https://docs.microsoft.com/azure/sql-database/sql-database-howto-managed-instance), o protetor de TDE é definido no nível de instância e é herdado por todos os bancos de dados *criptografados* nessa instância. O termo *servidor* refere-se ao servidor e instância ao longo deste documento, a menos que indicado de forma diferente.

Com a integração do TDE ao Azure Key Vault, os usuários podem controlar as principais tarefas de gerenciamento, incluindo rotações de chave, permissões de cofre de chaves, backups de chaves e habilitar auditoria/relatório em todos os protetores de TDE usando a funcionalidade do Azure Key Vault. O Key Vault fornece gerenciamento central de chaves, utiliza HSMs (Módulos de Segurança de Hardware) rigidamente monitorados e permite a separação de funções entre o gerenciamento de chaves e dados para ajudar a atender a conformidade com políticas de segurança.
Para saber mais sobre criptografia de dados transparente com a integração do Azure Key Vault (traga seu próprio suporte à chave) para o Banco de Dados SQL Do Azure, a Instância Gerenciada SQL e o Synapse do Azure, consulte [criptografia de dados transparente com integração do Azure Key Vault](transparent-data-encryption-byok-azure-sql.md).

Para começar a usar a Transparent Data Encryption com integração do Azure Key Vault (com suporte Bring Your Own Key), confira o guia de instruções [Ativar Transparent Data Encryption com sua própria chave no Key Vault usando PowerShell](transparent-data-encryption-byok-azure-sql-configure.md).

## <a name="move-a-transparent-data-encryption-protected-database"></a>Mover um banco de dados protegido por Transparent Data Encryption

Não é necessário descriptografar bancos de dados para operações no Azure. As configurações de Transparent Data Encryption no banco de dados de origem ou banco de dados primário são herdadas de forma transparente no destino. As operações incluídas envolvem:

- Restauração geográfica
- Restauração via autoatendimento point-in-time
- Restauração de um banco de dados excluído
- Replicação geográfica ativa
- Criação de uma cópia do banco de dados
- Restauração de arquivo de backup para a instância gerenciada do SQL do Azure

> [!IMPORTANT]
> Fazer backup somente cópia manual de um banco de dados criptografado por TDE gerenciada por serviço não é permitido na Instância Gerenciada do SQL do Azure, uma vez que o certificado usado para criptografia não está acessível. Use o recurso de ponto no tempo de restauração para mover esse tipo de banco de dados para outra Instância Gerenciada.

Ao exportar um banco de dados protegido por Transparent Data Encryption, o conteúdo exportado do banco de dados não é criptografado. Esse conteúdo exportado é armazenado em arquivos BACPAC não criptografados. Certifique-se de proteger os arquivos BACPAC adequadamente e habilite a Transparent Data Encryption após a conclusão da importação do novo banco de dados.

Por exemplo, se o arquivo BACPAC for exportado de uma instância do SQL Server local, o conteúdo importado do novo banco de dados não será automaticamente criptografado. Da mesma forma, se o arquivo BACPAC for exportado para uma instância do SQL Server local, o novo banco de dados também não será automaticamente criptografado.

A única exceção é quando você exporta para e de um banco de dados SQL. A Transparent Data Encryption está habilitada no novo banco de dados, mas o próprio arquivo BACPAC ainda não está criptografado.


## <a name="manage-transparent-data-encryption"></a>Gerenciar criptografia de dados transparente
# <a name="portal"></a>[Portal](#tab/azure-portal)
Gerencie criptografia de dados transparente no portal Azure.

Para configurar a Transparent Data Encryption por meio do portal do Azure, será necessário estar conectado como Proprietário do Azure, Colaborador ou Gerenciador de Segurança de SQL.

Você define a Transparent Data Encryption dentro e fora do nível do banco de dados. Para habilitar a Transparent Data Encryption em um banco de dados, vá para o [portal do Azure](https://portal.azure.com) e entre com a conta de Administrador ou Colaborador do Azure. Localize as configurações de Transparent Data Encryption no banco de dados do usuário. Por padrão, é utilizada a Transparent Data Encryption de serviço gerenciado. Um certificado de Transparent Data Encryption é gerado automaticamente para o servidor que contém o banco de dados. Para a instância gerenciada do SQL do Azure use T-SQL para ativar a transparent data encryption e desativar um banco de dados.

![Transparent Data Encryption de serviço gerenciado](./media/transparent-data-encryption-azure-sql/service-managed-transparent-data-encryption.png)

Você define a chave mestra de Transparent Data Encryption, também conhecida como protetor de Transparent Data Encryption, no nível do servidor. Para usar a Transparent Data Encryption com suporte Bring Your Own Key e proteger os bancos de dados com uma chave do Key Vault, consulte as configurações de Transparent Data Encryption no servidor.

![Transparent Data Encryption com suporte Bring Your Own Key](./media/transparent-data-encryption-azure-sql/tde-byok-support.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Gerencie criptografia de dados transparente usando o PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]
> [!IMPORTANT]
> O módulo PowerShell Azure Resource Manager ainda é suportado pelo Banco de Dados SQL do Azure, mas todo o desenvolvimento futuro é para o módulo Az.Sql. Para obter esses cmdlets, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/). Os argumentos para os comandos no módulo Az e nos módulos AzureRm são substancialmente idênticos.

Para configurar a Transparent Data Encryption por meio do PowerShell, é necessário estar conectado como Proprietário do Azure, Colaborador ou Gerenciador de Segurança de SQL.

### <a name="cmdlets-for-azure-sql-database-and-azure-synapse"></a>Cmdlets para Banco de Dados SQL Azure e Sinapse Azure

Use os seguintes cmdlets para O Banco de Dados SQL do Azure e para o Azure Synapse:

| Cmdlet | Descrição |
| --- | --- |
| [Set-AzSqlDatabaseTransparentDataEncryption](https://docs.microsoft.com/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) |Habilita ou desabilita a Transparent Data Encryption para um banco de dados|
| [Get-AzSqlDatabaseTransparentDataEncryption](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) |Obtém o estado de Transparent Data Encryption para um banco de dados |
| [Get-AzSqlDatabaseTransparentDataEncryptionActivity](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) |Verifica o progresso de Transparent Data Encryption de um banco de dados |
| [Add-AzSqlServerKeyVaultKey](https://docs.microsoft.com/powershell/module/az.sql/add-azsqlserverkeyvaultkey) |Adiciona uma chave do Key Vault a uma instância do SQL Server |
| [Get-AzSqlServerKeyVaultKey](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlserverkeyvaultkey) |Obtém as chaves do Key Vault para um servidor do Banco de Dados SQL do Azure  |
| [Set-AzSqlServerTransparentDataArmEncryptionProtector](https://docs.microsoft.com/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) |Define o protetor de Transparent Data Encryption para uma instância do SQL Server |
| [Get-AzSqlServerTransparentDataEncryptionProtector](https://docs.microsoft.com/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) |Obtém o protetor de Transparent Data Encryption |
| [Remove-AzSqlServerKeyKeyKey](https://docs.microsoft.com/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) |Remove uma chave do Key Vault de uma instância do SQL Server |
|  | |

> [!IMPORTANT]
> Para a instância gerenciada do SQL, use o comando T-SQL [ALTER DATABASE](https://docs.microsoft.com/sql/t-sql/statements/alter-database-azure-sql-database) para ativar a transparent data encryption e desativar um nível de banco de dados e verificar [exemplo de script PowerShell](transparent-data-encryption-byok-azure-sql-configure.md) para gerenciar a criptografia de dados transparente em um nível de instância.

# <a name="transact-sql"></a>[Transact-SQL](#tab/azure-TransactSQL)
Gerencie criptografia de dados transparente usando transact-SQL.

Conecte o banco de dados usando um logon que seja um administrador ou membro da função **dbmanager** no banco de dados mestre.

| Comando | Descrição |
| --- | --- |
| [ALTER DATABASE (Banco de Dados SQL do Azure)](https://docs.microsoft.com/sql/t-sql/statements/alter-database-azure-sql-database) | SET ENCRYPTION ON/OFF criptografa ou descriptografa um banco de dados |
| [sys.dm_database_encryption_keys](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-database-encryption-keys-transact-sql) |Retorna informações sobre o estado de criptografia de um banco de dados e as chaves de criptografia de banco de dados associadas |
| [sys.dm_pdw_nodes_database_encryption_keys](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql) |Retorna informações sobre o estado de criptografia de cada nó de data warehouse e as chaves de criptografia de banco de dados associadas |
|  | |

Não é possível alternar o protetor de Transparent Data Encryption para uma chave do Key Vault usando o Transact-SQL. Use PowerShell o portal do Azure.

# <a name="rest-api"></a>[API REST](#tab/azure-RESTAPI)
Gerencie a criptografia de dados transparente usando a API REST.

Para configurar a Transparent Data Encryption por meio da API REST, é necessário estar conectado como o Proprietário do Azure, Colaborador ou Gerenciador de Segurança do SQL.
Use o seguinte conjunto de comandos para O Banco de Dados SQL do Azure e sinapse do Azure:

| Comando | Descrição |
| --- | --- |
|[Criar ou atualizar o servidor](https://docs.microsoft.com/rest/api/sql/servers/createorupdate)|Adiciona uma identidade do Azure Active Directory a uma instância do SQL Server (usada para conceder acesso ao Key Vault)|
|[Criar ou atualizar a chave do servidor](https://docs.microsoft.com/rest/api/sql/serverkeys/createorupdate)|Adiciona uma chave do Key Vault a uma instância do SQL Server|
|[Excluir chave do servidor](https://docs.microsoft.com/rest/api/sql/serverkeys/delete)|Remove uma chave do Key Vault de uma instância do SQL Server|
|[Obter chaves do servidor](https://docs.microsoft.com/rest/api/sql/serverkeys/get)|Obtém uma chave específica do Key Vault de uma instância do SQL Server|
|[Listar chaves do servidor por servidor](https://docs.microsoft.com/rest/api/sql/serverkeys/listbyserver)|Obtém as chaves do Key Vault para uma instância do SQL Server |
|[Criar ou atualizar o protetor de criptografia](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/createorupdate)|Define o protetor de Transparent Data Encryption para uma instância do SQL Server|
|[Obter protetor de criptografia](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/get)|Obtém o protetor de Transparent Data Encryption para uma instância do SQL Server|
|[Listar protetores de criptografia por servidor](https://docs.microsoft.com/rest/api/sql/encryptionprotectors/listbyserver)|Obtém os protetores de Transparent Data Encryption para uma instância do SQL Server |
|[Criar ou atualizar a configuração de Transparent Data Encryption](https://docs.microsoft.com/rest/api/sql/transparentdataencryptions/createorupdate)|Habilita ou desabilita a Transparent Data Encryption para um banco de dados|
|[Obter configuração de Transparent Data Encryption](https://docs.microsoft.com/rest/api/sql/transparentdataencryptions/get)|Obtém a configuração de Transparent Data Encryption para um banco de dados|
|[Lista de resultados de configuração de Transparent Data Encryption](https://docs.microsoft.com/rest/api/sql/transparentdataencryptionactivities/listbyconfiguration)|Obtém o resultado da criptografia para um banco de dados|

## <a name="next-steps"></a>Próximas etapas

- Para obter uma descrição geral da criptografia de dados transparente, consulte [criptografia de dados transparente](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption).
- Para saber mais sobre criptografia de dados transparente com o suporte ao Azure SQL Database, Azure SQL Managed Instance e Azure Synapse, consulte [criptografia de dados transparente com suporte ao Bring Your Own Key](transparent-data-encryption-byok-azure-sql.md).
- Para começar a usar a Transparent Data Encryption com o suporte Bring Your Own Key, consulte o guia de instruções [Ativar Transparent Data Encryption com sua própria chave no Key Vault usando PowerShell](transparent-data-encryption-byok-azure-sql-configure.md).
- Para obter mais informações sobre Key Vault, consulte a [página de documentação do Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-secure-your-key-vault).
