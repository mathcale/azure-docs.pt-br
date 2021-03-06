---
title: Melhorar o desempenho de aplicativos do Azure com o Assistente do Azure
description: Use o Assistente para otimizar o desempenho das implantações do Azure.
ms.topic: article
ms.date: 01/29/2019
ms.openlocfilehash: 405ec395feeb33b8511b9b915151b2ed9503c371
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75443050"
---
# <a name="improve-performance-of-azure-applications-with-azure-advisor"></a>Melhorar o desempenho de aplicativos do Azure com o Assistente do Azure

As recomendações de desempenho do Assistente do Azure ajudam a melhorar a velocidade e a capacidade de resposta dos aplicativos críticos para os negócios. Você pode obter recomendações de desempenho do Assistente na guia **Desempenho** do painel do Assistente.

## <a name="reduce-dns-time-to-live-on-your-traffic-manager-profile-to-fail-over-to-healthy-endpoints-faster"></a>Reduza o tempo de DNS para viver no seu perfil do Gerenciador de Tráfego para fazer o failover para endpoints saudáveis mais rapidamente

As [configurações de tempo de vida (TTL)](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-performance-considerations) em seu perfil do Gerenciador de Tráfego permitem que você especifique a velocidade de alternar os terminais se um determinado ponto de extremidade parar de responder a consultas. Reduzir os valores de TTL significa que os clientes serão roteados para os endpoints em funcionamento mais rapidamente.

O Azure Advisor identifica os perfis do Gerenciador de Tráfego com um TTL mais longo configurado e recomenda configurar o TTL para 20 segundos ou 60 segundos, dependendo de o perfil estar configurado para [Fast Failover](https://azure.microsoft.com/roadmap/fast-failover-and-tcp-probing-in-azure-traffic-manager/).

## <a name="improve-database-performance-with-sql-db-advisor"></a>Melhorar o desempenho do banco de dados com o Assistente do BD SQL

O Advisor fornece uma exibição consistente e consolidada de recomendações para todos os seus recursos do Azure. Ele se integra ao Advisor do Banco de Dados SQL para fornecer recomendações de melhoria de desempenho para o banco de dados do SQL Azure.O Advisor do Banco de Dados SQL avalia o desempenho dos bancos de dados do SQL Azure analisando o histórico de uso. Em seguida, ele oferece recomendações que são mais adequadas para execução da carga de trabalho típica do banco de dados.

> [!NOTE]
> Para obter recomendações, um banco de dados deve ter aproximadamente uma semana de uso e dentro dessa semana deve haver atividades consistentes. O Assistente do Banco de Dados SQL pode ser otimizado com mais facilidade para padrões de consulta consistentes do que para intermitências irregulares de atividade.

Para obter mais informações sobre o SQL Database Advisor, consulte [SQL Database Advisor](https://azure.microsoft.com/documentation/articles/sql-database-advisor/).

## <a name="improve-app-service-performance-and-reliability"></a>Melhorar o desempenho e a confiabilidade do Serviço de Aplicativo

O Azure Advisor integra as práticas recomendadas para melhorar sua experiência com os Serviços de Aplicativos e descobrir recursos relevantes de plataforma. Os exemplos de recomendações dos Serviços de Aplicativos são:
* Detecção de instâncias nas quais os recursos de memória ou de CPU são esgotados por runtimes de aplicativo com opções de mitigação.
* Detecção de instâncias nas quais a disposição de recursos como aplicativos Web e bancos de dados pode melhorar o desempenho e reduzir custos.

Para saber mais sobre recomendações de Serviços de Aplicativos, veja [Práticas recomendadas para o Serviço de Aplicativo do Azure](https://azure.microsoft.com/documentation/articles/app-service-best-practices/).

## <a name="use-managed-disks-to-prevent-disk-io-throttling"></a>Usar o Managed Disks para evitar a limitação de E/S do disco

O Assistente identificará as máquinas virtuais que pertencem a uma conta de armazenamento que está atingindo sua meta de escalabilidade. Essa condição torna essas VMs suscetíveis à limitação de E/S. O Assistente recomendará que elas usem o Managed Disks para evitar a degradação do desempenho.

## <a name="improve-the-performance-and-reliability-of-virtual-machine-disks-by-using-premium-storage"></a>Melhorar o desempenho e a confiabilidade dos discos da máquina virtual usando o Armazenamento Premium

O Assistente identifica máquinas virtuais com discos padrão que têm um alto volume de transações em sua conta de armazenamento e recomenda a atualização para discos premium. 

O Armazenamento Premium do Azure dá suporte de disco de alto desempenho e baixa latência para máquinas virtuais executando cargas de trabalho com uso intensivo de E/S. Os discos da máquina virtual que usam contas de Armazenamento Premium armazenam dados em SSDs (unidades de estado sólido). Para o melhor desempenho em seu aplicativo, recomendamos migrar todos os discos de máquinas virtuais que exijam IOPS altos para o armazenamento premium.

## <a name="remove-data-skew-on-your-sql-data-warehouse-table-to-increase-query-performance"></a>Remova a distorção de dados em sua tabela do armazém de dados SQL para aumentar o desempenho da consulta

A distorção de dados pode causar movimentação desnecessária de dados ou gargalos de recursos ao executar sua carga de trabalho. O Advisor detectará uma distorção de dados de distribuição maior que 15% e recomendará que você redistribua seus dados e revisite suas seleções de chave de distribuição de tabela. Para saber mais sobre como identificar e remover distorções, consulte [solução de problemas skew](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-distribute#how-to-tell-if-your-distribution-column-is-a-good-choice).

## <a name="create-or-update-outdated-table-statistics-on-your-sql-data-warehouse-table-to-increase-query-performance"></a>Criar ou atualizar as estatísticas de tabela desatualizados em sua tabela do SQL data warehouse para aumentar o desempenho de consulta

O assistente identifica tabelas que não têm atualizadas [estatísticas da tabela](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-tables-statistics) e recomenda a criação ou atualização de estatísticas da tabela. O otimizador de consulta do data warehouse SQL usa estática atualizada para estimar a cardinalidade ou o número de linhas no resultado da consulta, o que permite que o otimizador de consulta crie um plano de consulta de alta qualidade para desempenho mais rápido.

## <a name="scale-up-to-optimize-cache-utilization-on-your-sql-data-warehouse-tables-to-increase-query-performance"></a>Escalar verticalmente para otimizar a utilização do cache na tabelas do SQL Data Warehouse a fim de aumentar o desempenho de consultas

O Assistente do Azure detecta se o SQL Data Warehouse tem um percentual alto de cache usado e uma baixa porcentagem de acessos. Essa condição indica alta remoção de cache o que pode afetar o desempenho do SQL Data Warehouse. O Assistente sugere escalar verticalmente o SQL Data Warehouse para garantir a alocação de capacidade de cache suficiente para sua carga de trabalho.

## <a name="convert-sql-data-warehouse-tables-to-replicated-tables-to-increase-query-performance"></a>Converter tabelas do SQL Data Warehouse em tabelas replicadas para aumentar o desempenho de consultas

O Assistente identifica tabelas que não são replicadas, mas que se beneficiariam da conversão e sugere que você as converta. As recomendações baseiam-se no tamanho da tabela replicada, no número de colunas, no tipo de distribuição da tabela e no número de partições da tabela do SQL Data Warehouse. Outras heurísticas podem ser fornecidas na recomendação para o contexto. Para saber mais sobre como essa recomendação é determinada, consulte [Recomendações do SQL Data Warehouse](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-concept-recommendations#replicate-tables). 

## <a name="migrate-your-storage-account-to-azure-resource-manager-to-get-all-of-the-latest-azure-features"></a>Migre sua Conta de Armazenamento para o Azure Resource Manager para obter todos os recursos mais recentes do Azure

Migre seu modelo de implantação da Conta de Armazenamento para o Azure Resource Manager (Resource Manager) e usufrua das implantações de modelo, de mais opções de segurança e da capacidade de atualizar para uma conta GPv2 a fim de poder usar os recursos mais recentes do Armazenamento do Azure. O Assistente identificará quaisquer contas de armazenamento autônomo que estão usando o modelo de implantação clássico e recomenda a migração para o modelo de implantação do Resource Manager.

> [!NOTE]
> Alertas clássicos no Azure Monitor foram retirados em agosto de 2019. Recomendamos que você atualize sua conta de armazenamento clássica para usar o Resource Manager e manter a funcionalidade de alerta com a nova plataforma. Para saber mais, confira [Desativação dos alertas clássicos](https://docs.microsoft.com/azure/azure-monitor/platform/monitoring-classic-retirement#retirement-of-classic-monitoring-and-alerting-platform).

## <a name="design-your-storage-accounts-to-prevent-hitting-the-maximum-subscription-limit"></a>Projete suas contas de armazenamento para evitar atingir o limite máximo de assinatura

Uma região do Azure pode suportar um máximo de 250 contas de armazenamento por assinatura. Uma vez que o limite seja atingido, você não poderá criar mais contas de armazenamento nessa combinação região/assinatura. O Advisor verificará suas assinaturas e recomendações de superfície para que você projete para menos contas de armazenamento para qualquer uma que esteja perto de atingir o limite máximo.

## <a name="optimize-the-performance-of-your-azure-mysql-azure-postgresql-and-azure-mariadb-servers"></a>Otimize o desempenho dos servidores Azure MySQL, Azure PostgreSQL e Azure MariaDB 

### <a name="fix-the-cpu-pressure-of-your-azure-mysql-azure-postgresql-and-azure-mariadb-servers-with-cpu-bottlenecks"></a>Corrija a pressão da CPU dos servidores Azure MySQL, Azure PostgreSQL e Azure MariaDB com gargalos de CPU
A utilização muito alta da CPU durante um período prolongado pode causar um desempenho lento de consulta para sua carga de trabalho. Aumentar o tamanho da CPU ajudará a otimizar o tempo de execução das consultas do banco de dados e melhorar o desempenho geral. O Azure Advisor identificará servidores com uma alta utilização de CPU que provavelmente estão executando cargas de trabalho restritas da CPU e recomendará o dimensionamento do seu cálculo.

### <a name="reduce-memory-constraints-on-your-azure-mysql-azure-postgresql-and-azure-mariadb-servers-or-move-to-a-memory-optimized-sku"></a>Reduza as restrições de memória em seus servidores Azure MySQL, Azure PostgreSQL e Azure MariaDB ou mude para um SKU otimizado para memória
Uma taxa de acerto de cache baixa pode resultar em um desempenho de consulta mais lento e aumento do IOPS. Isso pode ser devido a um plano de consulta ruim ou a execução de uma carga de trabalho intensiva de memória. Corrigir o plano de consulta ou [aumentar a memória](https://docs.microsoft.com/azure/postgresql/concepts-pricing-tiers) do banco de dados Azure para servidor de banco de dados PostgreSQL, servidor de banco de dados Azure MySQL ou servidor Azure MariaDB ajudará a otimizar a execução da carga de trabalho do banco de dados. O Azure Advisor identifica servidores afetados devido a esse alto churn de pool de buffer e recomenda a fixação do plano de consulta, a mudança para um SKU mais alto com mais memória ou o aumento do tamanho do armazenamento para obter mais IOPS.

### <a name="use-a-azure-mysql-or-azure-postgresql-read-replica-to-scale-out-reads-for-read-intensive-workloads"></a>Use uma réplica de leitura do Azure MySQL ou do Azure PostgreSQL para dimensionar leituras para leitura de cargas de trabalho intensivas
O Azure Advisor aproveita a heurística baseada em carga de trabalho, como a proporção de leituras para gravações no servidor nos últimos sete dias para identificar cargas de trabalho com uso intensivo de leitura. O banco de dados Do Zure para recursos PostgreSQL ou banco de dados Azure para recursos MySQL com uma relação de leitura/gravações muito alta pode resultar em disputas de CPU e/ou memória, levando a um desempenho lento de consulta. A [adição](https://docs.microsoft.com/azure/postgresql/howto-read-replicas-portal) de uma réplica ajudará a dimensionar leituras para o servidor de réplicas, impedindo restrições de CPU e/ou memória no servidor principal. O Advisor identificará servidores com cargas de trabalho tão altas e intensivas em leitura e recomendará adicionar uma [réplica](https://docs.microsoft.com/azure/postgresql/concepts-read-replicas) de leitura para descarregar algumas das cargas de trabalho lidas.


### <a name="scale-your-azure-mysql-azure-postgresql-or-azure-mariadb-server-to-a-higher-sku-to-prevent-connection-constraints"></a>Dimensione seu servidor Azure MySQL, Azure PostgreSQL ou Azure MariaDB para um SKU mais alto para evitar restrições de conexão
Cada nova conexão com seu servidor de banco de dados ocupa alguma memória. O desempenho do servidor de banco de dados se degrada se as conexões ao servidor estiverem falhando devido a um [limite superior](https://docs.microsoft.com/azure/postgresql/concepts-limits) na memória. O Azure Advisor identificará servidores em execução com muitas falhas de conexão e recomendará atualizar os limites de conexões do servidor para fornecer mais memória ao seu servidor, aumentando a computação ou usando SKUs otimizados de memória, que têm mais computação por núcleo.

## <a name="scale-your-cache-to-a-different-size-or-sku-to-improve-cache-and-application-performance"></a>Dimensione seu cache para um tamanho ou SKU diferente para melhorar o cache e o desempenho do aplicativo

As instâncias de cache funcionam melhor quando não são realizadas sob alta pressão de memória, alta carga do servidor ou alta largura de banda de rede, o que pode fazer com que elas não respondam, experimentem perda de dados ou se tornem indisponíveis. O Advisor identificará instâncias de cache nessas condições e recomendará a aplicação de práticas recomendadas para reduzir a pressão de memória, a carga do servidor ou a largura de banda da rede ou o dimensionamento para um tamanho diferente ou SKU com mais capacidade.

## <a name="add-regions-with-traffic-to-your-azure-cosmos-db-account"></a>Adicione regiões com tráfego à sua conta Azure Cosmos DB

O Advisor detectará contas do Azure Cosmos DB que tenham tráfego de uma região que não está configurada no momento e recomendará adicionar essa região. Isso melhorará a latência para solicitações provenientes daquela região e garantirá a disponibilidade em caso de paralisação da região. [Saiba mais sobre a distribuição global de dados com o Azure Cosmos DB](https://aka.ms/cosmos/globaldistribution)

## <a name="configure-your-azure-cosmos-db-indexing-policy-with-customer-included-or-excluded-paths"></a>Configure sua política de indexação Azure Cosmos DB com caminhos incluídos ou excluídos pelo cliente

O Azure Advisor identificará os contêineres Cosmos DB que estão usando a política de indexação padrão, mas podem se beneficiar de uma política de indexação personalizada com base no padrão de carga de trabalho. A política de indexação padrão indexa todas as propriedades, mas usar uma política de indexação personalizada com caminhos explícitos incluídos ou excluídos usados em filtros de consulta pode reduzir as RUs e o armazenamento consumidos para indexação. [Saiba mais sobre como modificar políticas de índice](https://aka.ms/cosmosdb/modify-index-policy)

## <a name="configure-your-azure-cosmos-db-query-page-size-maxitemcount-to--1"></a>Configurar o tamanho da página de consulta do Azure Cosmos DB (MaxItemCount) como -1 

O Azure Advisor identificará os contêineres Azure Cosmos DB que estão usando o tamanho da página de consulta de 100 e recomendará o uso de um tamanho de página de -1 para varreduras mais rápidas. [Saiba mais sobre a Contagem de Itens Max](https://aka.ms/cosmosdb/sql-api-query-metrics-max-item-count)

## <a name="how-to-access-performance-recommendations-in-advisor"></a>Como acessar as recomendações de desempenho no Advisor

1. Entre no [Portal do Azure](https://portal.azure.com) e, em seguida, abra o [Assistente](https://aka.ms/azureadvisordashboard).

2.  No painel do Assistente, clique na guia **Desempenho**.

## <a name="next-steps"></a>Próximas etapas

Para saber mais sobre as recomendações do Assistente, consulte:

* [Introdução ao Advisor](advisor-overview.md)
* [Introdução ao Advisor](advisor-get-started.md)
* [Recomendações de custo do Advisor](advisor-cost-recommendations.md)
* [Recomendações de alta disponibilidade do Advisor](advisor-high-availability-recommendations.md)
* [Recomendações de segurança do Advisor](advisor-security-recommendations.md)
* [Recomendações de Excelência Operacional do Orientador](advisor-operational-excellence-recommendations.md)
