---
title: Monte azure Arquivos volume para grupo de contêineres
description: Saiba como montar um volume de Arquivos do Azure para persistir o estado com Instâncias de Contêiner do Azure
ms.topic: article
ms.date: 12/30/2019
ms.custom: mvc
ms.openlocfilehash: f66890c503de8de9160f11fb28795012ae57daeb
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/27/2020
ms.locfileid: "75561330"
---
# <a name="mount-an-azure-file-share-in-azure-container-instances"></a>Montar um compartilhamento de arquivos do Azure em Instâncias de Contêiner do Azure

Por padrão, as Instâncias de Contêiner do Azure são sem monitoração de estado. Se o contêiner parar ou falhar, todo o seu estado será perdido. Para persistir o estado além do tempo de vida do contêiner, você deve montar um volume de um repositório externo. Como mostrado neste artigo, o Azure Container Instances pode montar um compartilhamento de arquivos Azure criado com [o Azure Files](../storage/files/storage-files-introduction.md). O Azure Files oferece compartilhamentos de arquivos totalmente gerenciados hospedados no Azure Storage que são acessíveis através do protocolo SMB (Server Message Block, bloco de mensagens padrão do servidor) padrão do setor. Usar um compartilhamento de arquivos do Azure com Instâncias de Contêiner do Azure fornece recursos de compartilhamento de arquivos semelhantes ao uso de um compartilhamento de arquivos do Azure com máquinas virtuais do Azure.

> [!NOTE]
> Montar um Compartilhamento de Arquivos do Azure está atualmente restrito a contêineres do Linux. Encontre as diferenças atuais da plataforma na [visão geral.](container-instances-overview.md#linux-and-windows-containers)
>
> A montagem de um compartilhamento de arquivos Azure em uma instância de contêiner é semelhante a uma [montagem de vinculação](https://docs.docker.com/storage/bind-mounts/)do Docker . Esteja ciente de que se você montar uma parte em um diretório de contêineres no qual existem arquivos ou diretórios, esses arquivos ou diretórios são obscurecidos pela montagem e não são acessíveis enquanto o contêiner é executado.
>

## <a name="create-an-azure-file-share"></a>Criar um compartilhamento de arquivos do Azure

Antes de usar um compartilhamento de arquivos do Azure com Instâncias de Contêiner do Azure, você deve criá-lo. Execute o seguinte script para criar uma conta de armazenamento para hospedar o compartilhamento de arquivos e o próprio compartilhamento. O nome da conta de armazenamento deve ser globalmente exclusivo para que o script adicione um valor aleatório à cadeia de caracteres de base.

```azurecli-interactive
# Change these four parameters as needed
ACI_PERS_RESOURCE_GROUP=myResourceGroup
ACI_PERS_STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM
ACI_PERS_LOCATION=eastus
ACI_PERS_SHARE_NAME=acishare

# Create the storage account with the parameters
az storage account create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --location $ACI_PERS_LOCATION \
    --sku Standard_LRS

# Create the file share
az storage share create \
  --name $ACI_PERS_SHARE_NAME \
  --account-name $ACI_PERS_STORAGE_ACCOUNT_NAME
```

## <a name="get-storage-credentials"></a>Obter credenciais de armazenamento

Para montar um compartilhamento de arquivos do Azure como um volume nas Instâncias de Contêiner do Azure, você precisa de três valores: o nome da conta de armazenamento, o nome do compartilhamento e a chave de acesso de armazenamento.

* **Nome da conta de armazenamento** - Se você usou o `$ACI_PERS_STORAGE_ACCOUNT_NAME` script anterior, o nome da conta de armazenamento seria armazenado na variável. Para ver o nome da conta, digite:

  ```console
  echo $ACI_PERS_STORAGE_ACCOUNT_NAME
  ```

* **Nome de ação** - Este valor `acishare` já é conhecido (definido como no script anterior)

* **Chave da conta de armazenamento** - Este valor pode ser encontrado usando o seguinte comando:

  ```azurecli-interactive
  STORAGE_KEY=$(az storage account keys list --resource-group $ACI_PERS_RESOURCE_GROUP --account-name $ACI_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" --output tsv)
  echo $STORAGE_KEY
  ```

## <a name="deploy-container-and-mount-volume---cli"></a>Implantar o volume de contêiner e montagem - CLI

Para montar um compartilhamento de arquivo Azure como um volume em um contêiner usando o Azure CLI, especifique o ponto de montagem de compartilhamento e volume ao criar o contêiner com [a criação de contêiner az][az-container-create]. Se você seguiu as etapas anteriores, poderá montar o compartilhamento criado anteriormente usando o seguinte comando para criar um contêiner:

```azurecli-interactive
az container create \
    --resource-group $ACI_PERS_RESOURCE_GROUP \
    --name hellofiles \
    --image mcr.microsoft.com/azuredocs/aci-hellofiles \
    --dns-name-label aci-demo \
    --ports 80 \
    --azure-file-volume-account-name $ACI_PERS_STORAGE_ACCOUNT_NAME \
    --azure-file-volume-account-key $STORAGE_KEY \
    --azure-file-volume-share-name $ACI_PERS_SHARE_NAME \
    --azure-file-volume-mount-path /aci/logs/
```

O `--dns-name-label` valor deve ser único dentro da região do Azure onde você cria a instância do contêiner. Atualize o valor no comando anterior se você receber uma mensagem de erro do **rótulo do nome DNS** ao executar o comando.

## <a name="manage-files-in-mounted-volume"></a>Gerenciar arquivos no volume montado

Uma vez iniciado o contêiner, você pode usar o aplicativo web simples implantado através da imagem [Aci-Hellofiles][aci-hellofiles] da Microsoft para criar pequenos arquivos de texto no compartilhamento de arquivos do Azure no caminho de montagem especificado. Obtenha o FQDN (nome de domínio totalmente qualificado) do aplicativo Web com o comando [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group $ACI_PERS_RESOURCE_GROUP \
  --name hellofiles --query ipAddress.fqdn --output tsv
```

Depois de salvar o texto usando o aplicativo, você pode usar o [portal Azure][portal] ou uma ferramenta como o [Microsoft Azure Storage Explorer][storage-explorer] para recuperar e inspecionar o arquivo ou arquivos gravados no compartilhamento de arquivos.

## <a name="deploy-container-and-mount-volume---yaml"></a>Implantar o volume de contêiner e montagem - YAML

Você também pode implantar um grupo de contêineres e montar um volume em um contêiner com o Azure CLI e um [modelo YAML](container-instances-multi-container-yaml.md). A implantação pelo modelo YAML é um método preferido ao implantar grupos de contêineres que consistem em vários contêineres.

O modelo YAML a seguir define um grupo `aci-hellofiles` de contêineres com um contêiner criado com a imagem. O contêiner monta o compartilhamento de arquivos Azure *acishare* criado anteriormente como um volume. Quando indicado, digite a chave de nome e armazenamento da conta de armazenamento que hospeda o compartilhamento de arquivos. 

Como no exemplo da `dnsNameLabel` CLI, o valor deve ser único dentro da região do Azure onde você cria a instância do contêiner. Atualize o valor no arquivo YAML, se necessário.

```yaml
apiVersion: '2018-10-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: hellofiles
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-hellofiles
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /aci/logs/
        name: filesharevolume
  osType: Linux
  restartPolicy: Always
  ipAddress:
    type: Public
    ports:
      - port: 80
    dnsNameLabel: aci-demo
  volumes:
  - name: filesharevolume
    azureFile:
      sharename: acishare
      storageAccountName: <Storage account name>
      storageAccountKey: <Storage account key>
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

Para implantar com o modelo YAML, salve o YAML anterior em um arquivo nomeado `deploy-aci.yaml` e, em seguida, execute o comando [az container create][az-container-create] com o parâmetro `--file`:

```azurecli
# Deploy with YAML template
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```
## <a name="deploy-container-and-mount-volume---resource-manager"></a>Implantar volume de contêiner e montagem - Gerenciador de recursos

Além da implantação da CLI e da YAML, você pode implantar um grupo de contêineres e montar um volume em um contêiner usando um modelo do Azure [Resource Manager](/azure/templates/microsoft.containerinstance/containergroups).

Primeiro, popule a matriz `volumes` na seção `properties` do grupo de contêineres do modelo. 

Em seguida, para cada recipiente no qual você gostaria `volumeMounts` de `properties` montar o volume, preencha a matriz na seção da definição do contêiner.

O modelo do Gerenciador de recursos a seguir `aci-hellofiles` define um grupo de contêineres com um contêiner criado com a imagem. O contêiner monta o compartilhamento de arquivos Azure *acishare* criado anteriormente como um volume. Quando indicado, digite a chave de nome e armazenamento da conta de armazenamento que hospeda o compartilhamento de arquivos. 

Como nos exemplos anteriores, o `dnsNameLabel` valor deve ser único dentro da região do Azure onde você cria a instância do contêiner. Atualize o valor no modelo, se necessário.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "variables": {
    "container1name": "hellofiles",
    "container1image": "mcr.microsoft.com/azuredocs/aci-hellofiles"
  },
  "resources": [
    {
      "name": "file-share-demo",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2018-10-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "containers": [
          {
            "name": "[variables('container1name')]",
            "properties": {
              "image": "[variables('container1image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "port": 80
                }
              ],
              "volumeMounts": [
                {
                  "name": "filesharevolume",
                  "mountPath": "/aci/logs"
                }
              ]
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "tcp",
              "port": "80"
            }
          ],
          "dnsNameLabel": "aci-demo"
        },
        "volumes": [
          {
            "name": "filesharevolume",
            "azureFile": {
                "shareName": "acishare",
                "storageAccountName": "<Storage account name>",
                "storageAccountKey": "<Storage account key>"
            }
          }
        ]
      }
    }
  ]
}
```

Para implantar com o modelo do Resource Manager, salve o JSON anterior em um arquivo nomeado `deploy-aci.json` e, em seguida, execute o comando [az group deployment create][az-group-deployment-create] com o parâmetro `--template-file`:

```azurecli
# Deploy with Resource Manager template
az group deployment create --resource-group myResourceGroup --template-file deploy-aci.json
```


## <a name="mount-multiple-volumes"></a>Montar vários volumes

Para montar vários volumes em uma instância de contêiner, você deve ser implantado usando um [modelo do Azure Resource Manager,](/azure/templates/microsoft.containerinstance/containergroups)um arquivo YAML ou outro método programático. Para usar um modelo ou arquivo YAML, forneça os detalhes `volumes` do compartilhamento `properties` e defina os volumes povoando a matriz na seção do arquivo. 

Por exemplo, se você criou dois compartilhamentos de arquivos do Azure chamados `volumes` *share1* e *share2* na conta de armazenamento *myStorageAccount,* o array em um modelo do Gerenciador de recursos apareceria semelhante ao seguinte:

```JSON
"volumes": [{
  "name": "myvolume1",
  "azureFile": {
    "shareName": "share1",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
},
{
  "name": "myvolume2",
  "azureFile": {
    "shareName": "share2",
    "storageAccountName": "myStorageAccount",
    "storageAccountKey": "<storage-account-key>"
  }
}]
```

Em seguida, para cada contêiner do grupo de contêineres no qual você deseja montar os volumes, popule a matriz `volumeMounts` na seção `properties` da definição de contêiner. Por exemplo, isso monta os dois volumes, *myvolume1* e *myvolume2*, definidos anteriormente:

```JSON
"volumeMounts": [{
  "name": "myvolume1",
  "mountPath": "/mnt/share1/"
},
{
  "name": "myvolume2",
  "mountPath": "/mnt/share2/"
}]
```

## <a name="next-steps"></a>Próximas etapas

Saiba como montar outros tipos de volume em Instâncias de Contêiner do Azure:

* [Montar um volume emptyDir em Instâncias de Contêiner do Azure](container-instances-volume-emptydir.md)
* [Montar um volume gitRepo em Instâncias de Contêiner do Azure](container-instances-volume-gitrepo.md)
* [Montar um volume secreto em Instâncias de Contêiner do Azure](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-hellofiles]: https://hub.docker.com/_/microsoft-azuredocs-aci-hellofiles 
[portal]: https://portal.azure.com
[storage-explorer]: https://storageexplorer.com

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create