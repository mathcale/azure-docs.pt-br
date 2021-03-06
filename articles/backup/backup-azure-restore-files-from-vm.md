---
title: Recuperar arquivos e pastas do backup do Azure VM
description: Neste artigo, saiba como recuperar arquivos e pastas de um ponto de recuperação de máquina virtual do Azure.
ms.topic: conceptual
ms.date: 03/01/2019
ms.openlocfilehash: 0e3061ea8fc26adcf39fe415cd9a662de739543a
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "79273300"
---
# <a name="recover-files-from-azure-virtual-machine-backup"></a>Recuperar arquivos de um backup de máquina virtual do Azure

O Backup do Azure fornece a capacidade de restaurar [máquinas virtuais (VMs) do Azure e discos](./backup-azure-arm-restore-vms.md) de backups de máquina virtual do Azure, também conhecido como pontos de recuperação. Este artigo explica como recuperar arquivos e pastas de um backup de VM do Azure. A restauração de arquivos e pastas está disponível somente para VMs do Azure implantadas usando o modelo Gerenciador de Recursos e protegidas para um cofre de serviços de Recuperação.

> [!NOTE]
> Esse recurso está disponível para máquinas virtuais do Azure implantadas usando o modelo do Resource Manager e protegidas em um cofre dos Serviços de Recuperação.
> Não há suporte para a recuperação de arquivos de um backup criptografado de VM.
>

## <a name="mount-the-volume-and-copy-files"></a>Montar os volume e copiar arquivos

Para restaurar arquivos ou pastas no ponto de recuperação, vá para a máquina virtual e escolha o ponto de recuperação desejado.

1. Faça logon no [portal do Azure](https://portal.Azure.com) e no painel esquerdo, clique em **Máquinas Virtuais**. Na lista de máquinas virtuais, selecione a máquina virtual para abrir o painel da máquina virtual.

2. No menu da máquina virtual, clique em **Backup** para abrir o painel de Backup.

    ![Abra o item de backup do cofre de Serviços de Recuperação](./media/backup-azure-restore-files-from-vm/open-vault-for-vm.png)

3. No menu do painel de Backup, clique em **Recuperação de Arquivo**.

    ![Botão de recuperação de arquivo](./media/backup-azure-restore-files-from-vm/vm-backup-menu-file-recovery-button.png)

    O menu **Recuperação de Arquivo** é aberto.

    ![Menu de recuperação de arquivo](./media/backup-azure-restore-files-from-vm/file-recovery-blade.png)

4. No menu suspenso **Selecionar ponto de recuperação**, selecione o ponto de recuperação que contém os arquivos desejados. Por padrão, o ponto de recuperação mais recente já está selecionado.

5. Para baixar o software usado para copiar arquivos do ponto de recuperação, clique em **Baixar Executável** (para VMs do Windows Azure) ou **Baixar Script** (para VMs Linux Azure, um script python é gerado).

    ![Senha gerada](./media/backup-azure-restore-files-from-vm/download-executable.png)

    O Azure baixa o arquivo executável ou o script para o computador local.

    ![baixar mensagem para o arquivo executável ou o script](./media/backup-azure-restore-files-from-vm/run-the-script.png)

    Para executar o executável ou script como administrador, é sugerido que você salve o arquivo baixado no seu computador.

6. O arquivo executável ou o script é protegido por senha e requer uma senha. No menu **Recuperação de Arquivo**, clique no botão de cópia para carregar a senha na memória.

    ![Senha gerada](./media/backup-azure-restore-files-from-vm/generated-pswd.png)

7. No local do download (geralmente a pasta Downloads), clique com o botão direito do mouse no arquivo executável ou no script e execute-o com as credenciais de Administrador. Quando solicitado, digite a senha ou cole a senha da memória e **pressione Enter**. Quando a senha válida é inserida, o script conecta-se ao ponto de recuperação.

    ![Menu de recuperação de arquivo](./media/backup-azure-restore-files-from-vm/executable-output.png)

8. Para máquinas Linux, um script python é gerado. É preciso baixar o script e copiá-lo para o servidor Linux relevante/compatível. Você pode ter que modificar as ```chmod +x <python file name>```permissões para executá-lo com . Em seguida, execute ```./<python file name>```o arquivo python com .

Consulte a seção [Requisitos de acesso](#access-requirements) para certificar-se de que o script será executado com sucesso.

### <a name="identifying-volumes"></a>Identificação de volumes

#### <a name="for-windows"></a>Para Windows

Quando você executa o arquivo executável, o sistema operacional monta os novos volumes e atribui letras de unidade. Você pode usar o Windows Explorer ou o Explorador de arquivos para procurar essas unidades. As letras de unidade atribuídas aos volumes podem não ser as mesmas letras da máquina virtual original. No entanto, o nome do volume é preservado. Por exemplo, se o volume na máquina virtual original`\`foi "Data Disk (E: )", esse volume pode ser`\`anexado no computador local como "Disco de Dados ('Qualquer letra': ). Navegue por todos os volumes mencionados na saída do script até encontrar seus arquivos ou pasta.  

   ![Menu de recuperação de arquivo](./media/backup-azure-restore-files-from-vm/volumes-attached.png)

#### <a name="for-linux"></a>Para Linux

No Linux, os volumes do ponto de recuperação são montados para a pasta onde o script é executado. Os discos anexados, volumes e os caminhos de montagem correspondente são mostrados adequadamente. Esses caminhos de montagem ficam visíveis aos usuários que têm acesso ao nível raiz. Percorra os volumes mencionados na saída do script.

  ![Menu de recuperação de Arquivo do Linux](./media/backup-azure-restore-files-from-vm/linux-mount-paths.png)

## <a name="closing-the-connection"></a>Encerrando a conexão

Depois de identificar os arquivos e copiá-los para um local de armazenamento local, remova ou desmonte as unidades adicionais. Para desmontar as unidades no menu de **Recuperação de Arquivos** no Portal do Azure, clique em **Desmontar discos**.

![Desmontar discos](./media/backup-azure-restore-files-from-vm/unmount-disks3.png)

Quando os discos tiverem sido desmontados, você receberá uma mensagem. Pode levar alguns minutos até que a conexão seja atualizada para que você possa remover os discos.

No Linux, após a conexão com o ponto de recuperação ser interrompida, o sistema operacional não removerá automaticamente os caminhos de montagem correspondentes. Os caminhos de montagem existem como volumes "órfãos" e são visíveis, mas lançam um erro ao acessar/gravar os arquivos. Eles podem ser removidos manualmente. O script, quando executado, identifica esses volumes existentes em qualquer ponto de recuperação anterior e limpa mediante consentimento.

## <a name="special-configurations"></a>Configurações especiais

### <a name="dynamic-disks"></a>Discos dinâmicos

Se a VM do Azure protegida tiver volumes com uma ou ambas as seguintes características, não será possível executar o script executável na mesma VM.

- Volumes que abrangem vários discos (volumes distribuídos e estendidos)
- Volumes tolerantes a falhas (volumes RAID-5 e espelhados) em discos dinâmicos

Em vez disso, execute o script executável em qualquer outro computador com um sistema operacional compatível.

### <a name="windows-storage-spaces"></a>Espaços de Armazenamento do Windows

Os Espaços de Armazenamento do Windows é uma tecnologia Windows que permite virtualizar o armazenamento. Com os Espaços de Armazenamento do Windows é possível agrupar discos padrão do setor em pools de armazenamento. Dessa forma, é possível utilizar o espaço disponível nesses pools de armazenamento para criar discos virtuais, chamados espaços de armazenamento.

Se a VM do Azure protegida utilizar os Espaços de Armazenamento do Windows, não será possível executar o script executável na mesma VM. Em vez disso, execute o script executável em qualquer outra máquina com um sistema operacional compatível.

### <a name="lvmraid-arrays"></a>Matrizes LVM/RAID

No Linux, o LVM (Gerenciador de volumes lógicos) e/ou Matrizes RAID de software são usados para gerenciar volumes lógicos em vários discos. Se a VM do Linux protegida utilizar LVM e/ou Matrizes RAID, não será possível executar o script na mesma VM. Em vez disso, execute o script em qualquer outra máquina com um SO compatível e que forneça suporte ao sistema de arquivos da VM protegida.

A saída de script seguinte exibe os discos de Matrizes RAID e/ou LVM e os volumes com o tipo de partição.

   ![Menu de Saída de LVM do Linux](./media/backup-azure-restore-files-from-vm/linux-LVMOutput.png)

Para colocar essas partições online, execute os comandos nas seções a seguir.

#### <a name="for-lvm-partitions"></a>Para partições LVM

Para listar os nomes do grupo de volume em um volume físico:

```bash
#!/bin/bash
pvs <volume name as shown above in the script output>
```

Para listar todos os volumes lógicos, nomes e seus caminhos em um grupo de volumes:

```bash
#!/bin/bash
lvdisplay <volume-group-name from the pvs command's results>
```

Para montar os volumes lógicos no caminho de sua escolha:

```bash
#!/bin/bash
mount <LV path> </mountpath>
```

#### <a name="for-raid-arrays"></a>Para matrizes RAID

O comando a seguir exibe detalhes sobre todos os discos de ataque:

```bash
#!/bin/bash
mdadm –detail –scan
```

 O disco RAID relevante é exibido como `/dev/mdm/<RAID array name in the protected VM>`

Use o comando mount se o disco RAID tiver volumes físicos:

```bash
#!/bin/bash
mount [RAID Disk Path] [/mountpath]
```

Se o disco RAID tiver outro LVM configurado nele, use o procedimento anterior para partições LVM, mas use o nome de volume no lugar do nome do disco RAID.

## <a name="system-requirements"></a>Requisitos do sistema

### <a name="for-windows-os"></a>Para Windows 10

A tabela a seguir mostra a compatibilidade entre os sistemas operacionais de computador e servidor. Ao recuperar arquivos, você não poderá restaurar arquivos para uma versão anterior ou uma versão futura do sistema operacional. Por exemplo, você não pode restaurar um arquivo de uma VM do Windows Server 2016 para um computador Windows Server 2012 ou Windows 8. É possível restaurar arquivos de uma VM para o mesmo sistema operacional do servidor ou para o sistema operacional compatível do cliente.

|Sistema operacional de servidor | Sistema operacional de cliente compatível  |
| --------------- | ---- |
| Windows Server 2019    | Windows 10 |
| Windows Server 2016    | Windows 10 |
| Windows Server 2012 R2 | Windows 8.1 |
| Windows Server 2012    | Windows 8  |
| Windows Server 2008 R2 | Windows 7   |

### <a name="for-linux-os"></a>Para Sistema operacional Linux

No Linux, o SO do computador utilizado para restaurar arquivos deve fornecer suporte para o sistema de arquivos da máquina virtual protegida. Ao selecionar um computador para executar o script, certifique-se de que o computador possui um SO compatível e usa uma das versões identificadas na tabela a seguir:

|Sistema operacional Linux | Versões  |
| --------------- | ---- |
| Ubuntu | 12.04 e acima |
| CentOS | 6.5 e acima  |
| RHEL | 6.7 e acima |
| Debian | 7 e acima |
| Oracle Linux | 6.4 e acima |
| SLES | 12 e acima |
| openSUSE | 42.2 e acima |

> [!NOTE]
> Encontramos alguns problemas na execução do script de recuperação de arquivos em máquinas com sles 12 SP4 OS e estamos investigando com a equipe SLES.
> Atualmente, a execução do script de recuperação de arquivos está funcionando em máquinas com versões SLES 12 SP2 e SP3 OS.
>

O script também exige componentes Python e bash para executar e se conectar com segurança ao ponto de recuperação.

|Componente | Versão  |
| --------------- | ---- |
| bash | 4 e acima |
| python | 2.6.6 e acima  |
| TLS | 1.2 deve ser compatível  |

## <a name="access-requirements"></a>Requisitos de acesso

Se você executar o script em um computador com acesso restrito, verifique se há acesso a:

- `download.microsoft.com`
- URLs de serviço de recuperação (geo-name refere-se para a região onde reside o cofre dos serviços de recuperação)
  - `https://pod01-rec2.geo-name.backup.windowsazure.com` (Para as geografias públicas do Azure)
  - `https://pod01-rec2.geo-name.backup.windowsazure.cn`(Para Azure China 21Vianet)
  - `https://pod01-rec2.geo-name.backup.windowsazure.us` (Para Governo dos EUA para Azure)
  - `https://pod01-rec2.geo-name.backup.windowsazure.de` (Para Azure Alemanha)
- Portas de saída 53 (DNS), 443, 3260

> [!NOTE]
>
> - O nome do arquivo de script baixado terá o **nome geográfico** a ser preenchido na URL. Para examepple: O nome do \'script\'\_\'baixado\'\'começa\'com o geonome vMname _ GUID , como *ContosoVM_wcus_12345678*
> - A URL <https://pod01-rec2.wcus.backup.windowsazure.com>seria "
>

Para o Linux, o script exige os componentes 'open-iscsi' e 'lshw' para se conectar ao ponto de recuperação. Se os componentes não existirem no computador onde o script é executado, o script pedirá permissão para instalar os componentes. Conceda para instalar os componentes necessários.

O acesso `download.microsoft.com` é necessário para baixar componentes usados para construir um canal seguro entre a máquina onde o script é executado e os dados no ponto de recuperação.

Você pode executar o script em qualquer máquina que tenha o mesmo sistema operacional (ou compatível) que a VM do backup. Consulte a [tabela Sistema operacional compatível](backup-azure-restore-files-from-vm.md#system-requirements) para saber mais sobre os sistemas operacionais compatíveis. Se a máquina virtual do Azure protegida usar Espaços de Armazenamento do Windows (para VMs do Microsoft Azure) ou Matrizes LVM/RAID (para VMs Linux), você não poderá executar o arquivo executável nem o script na mesma máquina virtual. Em vez disso, execute o script ou o arquivo executável em qualquer outra máquina com um sistema operacional compatível.

## <a name="file-recovery-from-virtual-machine-backups-having-large-disks"></a>Recuperação de arquivos de backups de máquinas virtuais com grandes discos

Esta seção explica como executar a recuperação de arquivos de backups de máquinas Virtuais Azure com mais de 16 discos e cada tamanho de disco é maior que 32 TB.

Uma vez que o processo de recuperação de arquivos anexa todos os discos do backup, quando um grande número de discos (>16) ou discos grandes (> 32 TB cada) são usados, recomenda-se os seguintes pontos de ação:

- Mantenha um servidor de restauração separado (VM Azure VM D2v3) para recuperação de arquivos. Você pode usá-lo apenas para recuperação de arquivos e, em seguida, desligá-lo quando não for necessário. A restauração da máquina original não é recomendada, pois terá um impacto significativo na própria VM.
- Em seguida, execute o script uma vez para verificar se a operação de recuperação de arquivos é bem sucedida.
- Se o processo de recuperação de arquivos for travado (os discos nunca estão montados ou eles estão montados, mas os volumes não aparecerem), execute as seguintes etapas.
  - Se o servidor de restauração for um VM do Windows:
    - Certifique-se de que o Sistema Operacional é WS 2012 ou superior.
    - Certifique-se de que as chaves de registro estão definidas conforme sugerido abaixo no servidor de restauração e certifique-se de reiniciar o servidor. O número ao lado do GUID pode variar de 0001-0005. No exemplo a seguir, é 0004. Navegue pelo caminho de chave do registro até a seção parâmetros.

    ![iscsi-reg-key-changes.png](media/backup-azure-restore-files-from-vm/iscsi-reg-key-changes.png)

```registry
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Disk\TimeOutValue – change this from 60 to 1200
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\SrbTimeoutDelta – change this from 15 to 1200
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\EnableNOPOut – change this from 0 to 1
- HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e97b-e325-11ce-bfc1-08002be10318}\0003\Parameters\MaxRequestHoldTime - change this from 60 to 1200
```

- Se o servidor de restauração for um VM Linux:
  - No arquivo /etc/iscsi/iscsid.conf, altere a configuração de:
    - node.conn[0].timeo.noop_out_timeout = 5 to node.conn[0].timeo.noop_out_timeout = 30
- Depois de fazer a mudança acima, execute o script novamente. Com essas mudanças, é altamente provável que a recuperação do arquivo tenha sucesso.
- Cada vez que o usuário baixa um script, o Azure Backup inicia o processo de preparação do ponto de recuperação para download. Com discos grandes, esse processo levará um tempo considerável. Se houver sucessivas rajadas de solicitações, a preparação do alvo entrará em uma espiral de download. Portanto, recomenda-se baixar um script do Portal/Powershell/CLI, esperar de 20 a 30 minutos (um heurístico) e executá-lo. A essa altura, espera-se que o alvo esteja pronto para conexão a partir do script.
- Após a recuperação do arquivo, certifique-se de voltar ao portal e clicar **em Desmontar discos** para pontos de recuperação onde você não foi capaz de montar volumes. Essencialmente, essa etapa limpará quaisquer processos/sessões existentes e aumentará a chance de recuperação.

## <a name="troubleshooting"></a>Solução de problemas

Se você tiver problemas durante a recuperação de arquivos de máquinas virtuais, verifique a tabela a seguir para obter informações adicionais.

| Mensagem de erro/Cenário | Causas prováveis | Ação recomendada |
| ------------------------ | -------------- | ------------------ |
| Saída exe: *Exceção capturada ao se conectar ao alvo* | O script não é capaz de acessar o ponto de recuperação    | Verifique se a máquina preenche os [requisitos de acesso anteriores.](#access-requirements) |  
| Saída de exe: *o destino já foi acessado por meio de uma sessão iSCSI.* | O script já foi executado na mesma máquina e as unidades foram anexadas | Os volumes do ponto de recuperação já foram anexados. Eles NÃO podem ser montados com as mesmas letras de unidade da VM original. Navegue por todos os volumes disponíveis no explorador de arquivos para o seu arquivo. |
| Saída exe: *Este script é inválido porque os discos foram desmontados via portal/excedeu o limite de 12 horas. Baixe um novo script do portal.* |    Os discos foram desmontados do portal ou o limite de 12 horas foi excedido | Este ex em particular agora é inválido e não pode ser executado. Se você quiser acessar os arquivos desse ponto de recuperação no tempo, visite o portal para obter um novo exe.|
| Na máquina onde o exe é executado: Os novos volumes não são desmontados depois que o botão de desmontagem é clicado | O iniciador iSCSI na máquina não está respondendo/atualizando sua conexão com o alvo e mantendo o cache. |  Depois de clicar em **Desmontar**, aguarde alguns minutos. Se os novos volumes não forem desmontados, navegue por todos os volumes. A navegação em todos os volumes força o iniciador a atualizar a conexão, e o volume é desmontado com uma mensagem de erro de que o disco não está disponível.|
| Saída exe: O script é executado com sucesso, mas "Novos volumes conectados" não é exibido na saída do script |    Esse é um problema temporário    | Os volumes já foram anexados. Abra o Explorer para navegar. Se você estiver usando a mesma máquina para executar scripts todas as vezes, considere reiniciar a máquina e a lista deve ser exibida nas reprises posteriores do exe. |
| Específico do Linux: não é possível exibir os volumes desejados | O SO da máquina onde o script é executado pode não reconhecer o sistema de arquivos subjacente da VM protegida | Verifique se o ponto de recuperação é consistente com falhas ou com arquivo. Se for consistente com o arquivo, execute o script em outra máquina cujo sistema operacional reconhece o sistema de arquivos da VM protegida. |
| Específico do Windows: não é possível exibir os volumes desejados | Os discos podem ter sido anexados, mas os volumes não foram configurados | Na tela de gerenciamento de disco, identifique os discos adicionais relacionados ao ponto de recuperação. Se algum desses discos estiver em um estado offline, tente trazê-los on-line clicando com o botão direito do mouse no disco e clique **em Online**.|

## <a name="security"></a>Segurança

Esta seção discute as várias medidas de segurança tomadas para a implementação da recuperação de arquivos de backups do Azure VM.

### <a name="feature-flow"></a>Fluxo de recursos

Esse recurso foi criado para acessar os dados da VM sem a necessidade de restaurar todos os discos VM ou VM e com o número mínimo de etapas. O acesso aos dados da VM é fornecido por um script (que monta o volume de recuperação quando executado como mostrado abaixo) e forma a pedra angular de todas as implementações de segurança:

  ![Fluxo de recursos de segurança](./media/backup-azure-restore-files-from-vm/vm-security-feature-flow.png)

### <a name="security-implementations"></a>Implementações de segurança

#### <a name="select-recovery-point-who-can-generate-script"></a>Selecione ponto de recuperação (que pode gerar script)

O script fornece acesso aos dados da VM, por isso é importante regular quem pode gerá-lo em primeiro lugar. Você precisa entrar no portal do Azure e ser autorizado pelo [RBAC](backup-rbac-rs-vault.md#mapping-backup-built-in-roles-to-backup-management-actions) a gerar o script.

A recuperação de arquivos precisa do mesmo nível de autorização necessário para restauração de VM e restauração de discos. Em outras palavras, apenas usuários autorizados podem visualizar os dados da VM que podem gerar o script.

O script gerado é assinado com o certificado oficial da Microsoft para o serviço de backup do Azure. Qualquer adulteração no script significa que a assinatura está quebrada, e qualquer tentativa de executar o script é destacada como um risco potencial pelo SISTEMA OPERACIONAL.

#### <a name="mount-recovery-volume-who-can-run-script"></a>Volume de recuperação de montagem (que pode executar o script)

Apenas um admin pode executar o script e ele deve ser executado no modo elevado. O script executa apenas um conjunto de etapas pré-geradas e não aceita entrada de nenhuma fonte externa.

Para executar o script, é necessária uma senha que só é mostrada ao usuário autorizado no momento da geração do script no portal Azure ou PowerShell/CLI. Isso é para garantir que o usuário autorizado que baixa o script também seja responsável pela execução do script.

#### <a name="browse-files-and-folders"></a>Procurar arquivos e pastas

Para navegar por arquivos e pastas, o script usa o iniciador iSCSI na máquina e se conecta ao ponto de recuperação configurado como um alvo iSCSI. Aqui você pode imaginar cenários onde se está tentando imitar/falsificar todos os componentes.

Usamos um mecanismo de autenticação chap mútuo para que cada componente autentique o outro. Isso significa que é extremamente difícil para um iniciador falso se conectar ao alvo iSCSI e para que um alvo falso seja conectado à máquina onde o script é executado.

O fluxo de dados entre o serviço de recuperação e a máquina é protegido pela construção de um túnel TLS seguro sobre o TCP ([O TLS 1.2 deve ser suportado](#system-requirements) na máquina onde o script é executado).

Qualquer ACL (File Access Control List, lista de controle de acesso de acesso de arquivos) presente na VM pai/backup também é preservada no sistema de arquivos montado.

O script dá acesso somente à leitura a um ponto de recuperação e é válido por apenas 12 horas. Se desejar remover o acesso mais cedo, entre no Portal do Azure/PowerShell/CLI e execute **discos desmontagem** para esse ponto de recuperação em particular. O roteiro será invalidado imediatamente.

## <a name="next-steps"></a>Próximas etapas

- Para quaisquer problemas durante a restauração de arquivos, consulte a seção [Solução de problemas](#troubleshooting)
- Saiba como [restaurar arquivos via Powershell](https://docs.microsoft.com/azure/backup/backup-azure-vms-automation#restore-files-from-an-azure-vm-backup)
- Saiba como [restaurar arquivos via Azure CLI](https://docs.microsoft.com/azure/backup/tutorial-restore-files)
- Depois que a VM for restaurada, saiba como [gerenciar backups](https://docs.microsoft.com/azure/backup/backup-azure-manage-vms)
