---
title: Restaurar VMs do VMware com o Servidor de Backup do Azure
description: Use o MABS (Azure Backup Server, servidor de backup do Azure) para restaurar vMware VMware em execução em um servidor VMware vCenter/ESXi.
ms.topic: conceptual
ms.date: 08/18/2019
ms.openlocfilehash: ab2fb4f8f79fa5a664f5cb0ba1bb537c1df658c2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "77212335"
---
# <a name="restore-vmware-virtual-machines"></a>Restaurar máquinas virtuais VMware

Este artigo explica como usar o Microsoft Azure Backup Server (MABS) para restaurar os pontos de recuperação do VMware VM. Para obter uma visão geral sobre o uso do MABS para recuperar dados, consulte [Recuperar dados protegidos](https://docs.microsoft.com/azure/backup/backup-azure-alternate-dpm-server). No Console administrador do MABS, existem duas maneiras de encontrar dados recuperáveis - pesquisar ou navegar. Ao recuperar dados, você pode ou não desejar restaurar os dados ou uma VM no mesmo local. Por essa razão, o MABS suporta três opções de recuperação para backups VMware VMware:

* **OLR (Recuperação no local original)**: use OLR para restaurar uma máquina virtual protegida no seu local original. Você pode restaurar uma VM em sua localização original somente se nenhum disco tiver sido adicionado ou excluído, uma vez que o backup ocorreu. Se discos foram adicionados ou excluídos, você deve usar a recuperação em local alternativo.

* **ALR (Recuperação em local alternativo)**: quando a VM original está ausente ou você não quer interferir na VM original, recupere a VM em um local alternativo. Para recuperar uma VM em um local alternativo, você deve fornecer o local de um host ESXi, um pool de recursos, uma pasta, o repositório de dados de armazenamento e o caminho. Para ajudar a diferenciar a VM restaurada da VM original, o MABS anexa "-Recuperado" ao nome da VM.

* **Recuperação individual de localização de arquivos (ILR)** - Se a VM protegida for uma VM do Windows Server, arquivos/pastas individuais dentro da VM podem ser recuperados usando o recurso ILR do MABS. Para recuperar arquivos individuais, confira o procedimento neste artigo.

## <a name="restore-a-recovery-point"></a>Restaurar um ponto de recuperação

1. No console de administrador do MABS, clique em 'Recuperação'.

2. Usando o painel Procurar, procure ou filtre para localizar a VM que você deseja recuperar. Quando você seleciona uma VM ou uma pasta, os pontos de recuperação do painel exibem os pontos de recuperação disponíveis.

    ![Pontos de recuperação disponíveis](./media/restore-azure-backup-server-vmware/recovery-points.png)

3. No campo **Pontos de recuperação para**, use o calendário e os menus suspensos para selecionar uma data em que um ponto de recuperação foi criado. Datas do calendário em negrito têm pontos de recuperação disponíveis.

4. Na faixa de opções de ferramenta, clique em **Recuperar** para iniciar o **Assistente de Recuperação**.

    ![Assistente de recuperação, revisão da seleção de recuperação](./media/restore-azure-backup-server-vmware/recovery-wizard.png)

5. Clique em **Avançar** para ir para a tela **Especificar Opções de Recuperação**.

6. Na tela **Especificar Opções de Recuperação**, se você deseja habilitar a limitação da largura de banda da rede, clique em **Modificar**. Para deixar a limitação da rede desabilitada, clique em **Avançar**. Nenhuma outra opção na tela do assistente está disponível para VMs VMware. Se você optar por modificar a limitação da largura de banda da rede, no diálogo Limitação, selecione **Habilitar limitação do uso da largura de banda da rede** para ativá-lo. Uma vez habilitada, defina as **Configurações** e a **Agenda de Trabalho**.

7. Na tela **Selecionar Tipo de Recuperação**, escolha se deseja recuperar na instância original ou em um novo local e clique em **Avançar**.

     * Se você escolher **Recuperar na instância original**, não precisará escolher outras opções no assistente. Os dados da instância original são usados.

     * Se você escolher **Recuperar como máquina virtual em qualquer host,** então na tela **Especificar destino,** forneça as informações para **host ESXi, pool de recursos, pasta** e **caminho**.

      ![Selecionar Tipo de Recuperação](./media/restore-azure-backup-server-vmware/recovery-type.png)

8. Na tela **Resumo**, verifique as configurações e clique em **Recuperar** para iniciar o processo de recuperação. A tela **Status de recuperação** mostra o progresso da operação de recuperação.

## <a name="restore-an-individual-file-from-a-vm"></a>Restaurar um arquivo individual de uma VM

Você pode restaurar arquivos individuais de um ponto de recuperação de VM protegida. Esse recurso só está disponível para VMs do Windows Server. A restauração de arquivos individuais é semelhante à restauração de toda a VM, exceto que você navega até o VMDK e localiza os arquivos que deseja antes de iniciar o processo de recuperação. Para recuperar um arquivo individual ou selecionar arquivos de uma VM do Windows Server:

>[!NOTE]
>Restaurar um arquivo individual de uma VM está disponível apenas para pontos de recuperação de vm e disco do Windows.

1. No console de administrador do MABS, clique em **'Recuperação'.**

2. Usando o painel **Procurar**, procure ou filtre para localizar a VM que você deseja recuperar. Quando você seleciona uma VM ou uma pasta, os pontos de recuperação do painel exibem os pontos de recuperação disponíveis.

    ![Pontos de recuperação disponíveis](./media/restore-azure-backup-server-vmware/vmware-rp-disk.png)

3. No painel **Pontos de Recuperação para:**, use o calendário para selecionar a data que contém os pontos de recuperação desejados. Dependendo de como a política de backup foi configurada, as datas podem ter mais de um ponto de recuperação. Depois de selecionar o dia em que o ponto de recuperação foi tomado, certifique-se de que escolheu o **tempo de recuperação**correto . Se a data selecionada tiver vários pontos de recuperação, escolha o ponto de recuperação selecionando-o no menu suspenso Tempo de recuperação. Depois de escolher o ponto de recuperação, a lista de itens recuperáveis aparece no painel **Caminho:**.

4. Para localizar os arquivos que você deseja recuperar, no painel **Caminho**, clique duas vezes no item na coluna **Item recuperável** para abri-lo. Selecione os arquivos ou as pastas que você deseja recuperar. Para selecionar vários itens, pressione a tecla **Ctrl** ao selecionar cada item. Use o painel **Caminho** para pesquisar a lista de arquivos ou pastas que aparecem na coluna **Item recuperável**. A **lista de pesquisa abaixo** não pesquisa em subpastas. Para pesquisar em subpastas, clique duas vezes na pasta. Use o botão **Para cima** a fim de sair de uma pasta filho para a pasta pai. Você pode selecionar vários itens (arquivos e pastas), mas eles devem estar na mesma pasta pai. Não é possível recuperar itens de várias pastas no mesmo trabalho de recuperação.

    ![Rever Seleção de Recuperação](./media/restore-azure-backup-server-vmware/vmware-rp-disk-ilr-2.png)

5. Quando você tiver selecionado os itens para recuperação, na faixa de opções de ferramenta do Console do Administrador, clique em **Recuperar** para abrir o **Assistente de Recuperação**. No Assistente de Recuperação, a tela **Examinar Seleção de Recuperação** mostra os itens selecionados a serem recuperados.

6. Na tela **Especificar Opções de Recuperação**, se você deseja habilitar a limitação da largura de banda da rede, clique em **Modificar**. Para deixar a limitação da rede desabilitada, clique em **Avançar**. Nenhuma outra opção na tela do assistente está disponível para VMs VMware. Se você optar por modificar a limitação da largura de banda da rede, no diálogo Limitação, selecione **Habilitar limitação do uso da largura de banda da rede** para ativá-lo. Uma vez habilitada, defina as **Configurações** e a **Agenda de Trabalho**.
7. Na tela **Selecionar Tipo de Recuperação**, clique em **Avançar**. Você só pode recuperar seus arquivos ou pastas em uma pasta de rede.
8. Na tela **Especificar Destino**, clique em **Procurar** para encontrar um local de rede para seus arquivos ou pastas. O MABS cria uma pasta onde todos os itens recuperados são copiados. O nome da pasta tem o prefixo, MABS_day-mês. Quando você seleciona um local para os arquivos ou pasta recuperados, os detalhes desse local (destino, caminho de destino e espaço disponível) são fornecidos.

    ![Especificar localização para recuperar arquivos](./media/restore-azure-backup-server-vmware/specify-destination.png)

9. Na tela **Especificar Opções de Recuperação**, escolha quais configurações de segurança aplicar. Você pode optar por modificar a limitação do uso de largura de banda de rede, mas a limitação é desabilitada por padrão. Além disso, as opções **Recuperação SAN** e **Notificação** não estão habilitadas.
10. Na tela **Resumo**, verifique as configurações e clique em **Recuperar** para iniciar o processo de recuperação. A tela **Status de recuperação** mostra o progresso da operação de recuperação.

## <a name="next-steps"></a>Próximas etapas

Para solucionar problemas ao usar o Azure Backup Server, revise o [guia de solução de problemas do Azure Backup Server](./backup-azure-mabs-troubleshoot.md).
