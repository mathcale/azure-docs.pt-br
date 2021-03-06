---
title: Otimizar inserções em massa - Banco de dados Azure para PostgreSQL - Servidor Único
description: Este artigo descreve como você pode otimizar as operações de inserção em massa em um banco de dados Azure para PostgreSQL - Single Server.
author: dianaputnam
ms.author: dianas
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 4c4bac16917be0064ebb111328753d378d462a2a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "74770128"
---
# <a name="optimize-bulk-inserts-and-use-transient-data-on-an-azure-database-for-postgresql---single-server"></a>Otimizar inserções em massa e usar dados transitórios em um banco de dados Azure para PostgreSQL - Single Server 
Este artigo descreve como você pode otimizar operações de inserção em massa e usar dados temporários em um servidor do Banco de Dados do Azure para PostgreSQL.

## <a name="use-unlogged-tables"></a>Usar tabelas não registradas
Se você tem operações de carga de trabalho que envolvem dados temporários ou inserem grandes conjuntos de dados em massa, considere o uso de tabelas não registradas.

Tabelas sem registro é um recurso do PostgreSQL que pode ser usado efetivamente para otimizar inserções em massa. O PostgreSQL usa WAL (registro em log write-ahead). Por padrão, ele fornece atomicidade e durabilidade. Atomicidade, consistência, isolamento e durabilidade são as propriedades ACID. 

Inseri-las em uma tabela não registrada significa que o PostgreSQL faz inserções sem gravar no log de transações, que, por si só, é uma operação de E/S. Como resultado, essas tabelas são consideravelmente mais rápidas do que as tabelas comuns.

Use as seguintes opções para criar uma tabela não registrada:
- Crie uma nova tabela não registrada usando a sintaxe `CREATE UNLOGGED TABLE <tableName>`.
- Converta uma tabela registrada existente em uma tabela não registrada usando a sintaxe `ALTER TABLE <tableName> SET UNLOGGED`.  

Para reverter o processo, use a sintaxe `ALTER TABLE <tableName> SET LOGGED`.

## <a name="unlogged-table-tradeoff"></a>Troca de tabela sem registro
Tabelas não registradas não estão livres de falhas. Uma tabela não registrada é automaticamente truncada após uma falha ou sujeita a um desligamento não limpo. O conteúdo de uma tabela não registrada também não é replicado para servidores em espera. Quaisquer índices criados em uma tabela não registrada também são descompactados automaticamente. Após a conclusão da operação de inserção, converta a tabela para registrada para que a inserção seja durável.

Em algumas cargas de trabalho de clientes, observamos um aprimoramento no desempenho de aproximadamente 15% a 20% no uso de tabelas não registradas.

## <a name="next-steps"></a>Próximas etapas
Analise sua carga de trabalho para usos de dados transitórios e inserções em massa grandes. Confira a seguinte documentação do PostgreSQL:
 
- [Criar comandos SQL de tabela](https://www.postgresql.org/docs/current/static/sql-createtable.html)
