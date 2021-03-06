---
title: Desenvolver no Azure Kubernetes Service (AKS) com rascunho
description: Usar o Draft com AKS e Registro de Contêiner do Azure
services: container-service
author: zr-msft
ms.topic: article
ms.date: 06/20/2019
ms.author: zarhoads
ms.openlocfilehash: b03256ee65a3c40d8a64d70b877c49e44e68f822
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "77595214"
---
# <a name="quickstart-develop-on-azure-kubernetes-service-aks-with-draft"></a>Quickstart: Desenvolva no Azure Kubernetes Service (AKS) com draft

Draft é uma ferramenta de código aberto que ajuda a empacotar e executar contêineres de aplicativos em um cluster Kubernetes. Com o Draft, você pode reimplantar rapidamente um aplicativo no Kubernetes à medida que as alterações de código ocorrem sem ter que comprometer suas alterações no controle de versão. Para obter mais informações sobre o Draft, consulte a [Documentação do Draft no GitHub][draft-documentation].

Este artigo mostra como usar o Draft para empacotar e executar um aplicativo no AKS.


## <a name="prerequisites"></a>Pré-requisitos

* Uma assinatura do Azure. Caso não tenha uma assinatura do Azure, é possível criar uma [conta gratuita](https://azure.microsoft.com/free).
* A [CLI do Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) instalada.
* Docker instalado e configurado. O Docker fornece pacotes que o configuram facilmente em qualquer sistema [Mac][docker-for-mac], [Windows][docker-for-windows] ou [Linux][docker-for-linux].
* [Helm v2 instalado.][helm-install]
* [Rascunho instalado][draft-documentation].

## <a name="create-an-azure-kubernetes-service-cluster"></a>Criar um cluster do Serviço de Kubernetes do Azure

Criar um cluster do AKS. Os comandos a seguir criam um grupo de recursos chamado MyResourceGroup e um cluster do AKS chamado MyAKS.

```azurecli
az group create --name MyResourceGroup --location eastus
az aks create -g MyResourceGroup -n MyAKS --location eastus --node-vm-size Standard_DS2_v2 --node-count 1 --generate-ssh-keys
```

## <a name="create-an-azure-container-registry"></a>Criar um Registro de Contêiner do Azure
Para usar o Draft para executar seu aplicativo no cluster AKS, você precisa de um Registro de Contêiner do Azure para armazenar suas imagens de contêiner. O exemplo abaixo usa [aaz acr create][az-acr-create] para criar um ACR chamado *MyDraftACR* no grupo de recursos *MyResourceGroup* com o *SKU básico.* Você deve fornecer seu próprio nome de registro exclusivo. O nome do registro deve ser exclusivo no Azure e conter de 5 a 50 caracteres alfanuméricos. A SKU *Basic* é um ponto de entrada de otimização de custo para fins de desenvolvimento que fornece um equilíbrio entre o armazenamento e taxa de transferência.

```azurecli
az acr create --resource-group MyResourceGroup --name MyDraftACR --sku Basic
```

A saída deverá ser semelhante ao seguinte exemplo: Anote o valor do *loginServer* para o seu ACR, pois ele será usado em uma etapa posterior. No exemplo abaixo, *mydraftacr.azurecr.io* é o *loginServer* para *MyDraftACR*.

```console
{
  "adminUserEnabled": false,
  "creationDate": "2019-06-11T13:35:17.998425+00:00",
  "id": "/subscriptions/<ID>/resourceGroups/MyResourceGroup/providers/Microsoft.ContainerRegistry/registries/MyDraftACR",
  "location": "eastus",
  "loginServer": "mydraftacr.azurecr.io",
  "name": "MyDraftACR",
  "networkRuleSet": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```


Para que o Draft use a instância ACR, você deve primeiro fazer login. Use o [comando de login az acr][az-acr-login] para fazer login. O exemplo abaixo entrará em um ACR chamado *MyDraftACR*.

```azurecli
az acr login --name MyDraftACR
```

O comando retorna uma mensagem *de login bem-sucedida* uma vez concluída.

## <a name="create-trust-between-aks-cluster-and-acr"></a>Criar confiança entre o cluster do AKS e o ACR

Seu cluster AKS também precisa ter acesso ao seu ACR para puxar as imagens do contêiner e executá-las. Você permite o acesso ao ACR da AKS estabelecendo uma confiança. Para estabelecer confiança entre um cluster do AKS e um registro do ACR, conceda permissões à entidade de serviço do Azure Active Directory usado pelo cluster do AKS para acessar o registro do ACR. Os seguintes comandos concedem permissões ao diretor de serviço do cluster *MyAKS* no *MyResourceGroup* para o *MyDraftACR* ACR no *MyResourceGroup*.

```azurecli
# Get the service principal ID of your AKS cluster
AKS_SP_ID=$(az aks show --resource-group MyResourceGroup --name MyAKS --query "servicePrincipalProfile.clientId" -o tsv)

# Get the resource ID of your ACR instance
ACR_RESOURCE_ID=$(az acr show --resource-group MyResourceGroup --name MyDraftACR --query "id" -o tsv)

# Create a role assignment for your AKS cluster to access the ACR instance
az role assignment create --assignee $AKS_SP_ID --scope $ACR_RESOURCE_ID --role contributor
```

## <a name="connect-to-your-aks-cluster"></a>Conecte-se ao seu cluster AKS

Para se conectar ao cluster Kubernetes no computador local, use o [kubectl][kubectl], o cliente de linha de comando do Kubernetes.

Se você usa o Azure Cloud Shell, o `kubectl` já estará instalado. Se você quiser instalá-lo localmente, use o comando [az aks install-cli][]:

```azurecli
az aks install-cli
```

Para configurar o `kubectl` para se conectar ao cluster do Kubernetes, use o comando [az aks get-credentials][]. O exemplo a seguir obtém credenciais para o cluster AKS chamado *MyAKS* no *MyResourceGroup*:

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
```

## <a name="create-a-service-account-for-helm"></a>Crie uma conta de serviço para helm

Antes de implantar o Helm em um cluster AKS habilitado para RBAC, você precisa de uma conta de serviço e ligação de função para o serviço do Tiller. Para obter mais informações sobre como proteger o Helm / cluster habilitado para o Tiller em um RBAC, consulte [Tiller, Namespaces e RBAC][tiller-rbac]. Se o cluster AKS não estiver habilitado para RBAC, pule este passo.

Crie um arquivo chamado `helm-rbac.yaml` e o copie no YAML a seguir:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Crie a conta de serviço e a associação de funções com o comando `kubectl apply`:

```console
kubectl apply -f helm-rbac.yaml
```

## <a name="configure-helm"></a>Configurar o Helm
Para implantar um Tiller básico em um cluster do AKS, use o comando [helm init][helm-init]. Se o cluster não estiver habilitado `--service-account` para RBAC, remova o argumento e o valor.

```console
helm init --service-account tiller --node-selectors "beta.kubernetes.io/os"="linux"
```

## <a name="configure-draft"></a>Configurar o Draft

Se você não tiver configurado o Rascunho `draft init`na máquina local, execute:

```console
$ draft init
Installing default plugins...
Installation of default plugins complete
Installing default pack repositories...
...
Happy Sailing!
```

Você também precisa configurar o Draft para usar o servidor de *login* do seu ACR. O comando `draft config set` a `mydraftacr.azurecr.io` seguir é usado como registro.

```console
draft config set registry mydraftacr.azurecr.io
```

Você configurou o Draft para usar seu ACR, e o Draft pode empurrar imagens de contêiner para o seu ACR. Quando o Draft executa seu aplicativo no cluster AKS, não são necessárias senhas ou segredos para empurrar ou puxar do registro ACR. Uma vez que um trust foi criado entre o cluster AKS e o Seu ACR, a autenticação acontece no nível do Azure Resource Manager, usando o Azure Active Directory.

## <a name="download-the-sample-application"></a>Baixar o aplicativo de exemplo

Este quickstart usa [um aplicativo Java de exemplo do repositório Draft GitHub][example-java]. Clone o aplicativo do GitHub `draft/examples/example-java/` e navegue até o diretório.

```console
git clone https://github.com/Azure/draft
cd draft/examples/example-java/
```

## <a name="run-the-sample-application-with-draft"></a>Execute o aplicativo de exemplo com draft

Use `draft create` o comando para preparar a aplicação.

```console
draft create
```

Este comando cria os artefatos que são usados para executar o aplicativo em um cluster do Kubernetes. Esses itens incluem um Dockerfile, um gráfico de Helm e um arquivo *draft.toml*, que é o arquivo de configuração do Draft.

```console
$ draft create

--> Draft detected Java (92.205567%)
--> Ready to sail
```

Para executar o aplicativo de exemplo no cluster do AKS, use o comando `draft up`.

```console
draft up
```

Este comando constrói o Arquivo Docker para criar uma imagem de contêiner, empurra a imagem para o seu ACR e instala o gráfico Helm para iniciar o aplicativo em AKS. A primeira vez que você executar este comando, empurrando e puxando a imagem do contêiner pode levar algum tempo. Depois que as camadas de base são armazenadas em cache, o tempo gasto para implantar o aplicativo é drasticamente reduzido.

```
$ draft up

Draft Up Started: 'example-java': 01CMZAR1F4T1TJZ8SWJQ70HCNH
example-java: Building Docker Image: SUCCESS ⚓  (73.0720s)
example-java: Pushing Docker Image: SUCCESS ⚓  (19.5727s)
example-java: Releasing Application: SUCCESS ⚓  (4.6979s)
Inspect the logs with `draft logs 01CMZAR1F4T1TJZ8SWJQ70HCNH`
```

## <a name="connect-to-the-running-sample-application-from-your-local-machine"></a>Conecte-se ao aplicativo de amostra em execução da sua máquina local

Para testar o aplicativo, use o comando `draft connect`.

```console
draft connect
```

Esse comando gera proxy de uma conexão segura com o pod do Kubernetes. Ao concluir, o aplicativo pode ser acessado na URL fornecida.

```console
$ draft connect

Connect to java:4567 on localhost:49804
[java]: SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
[java]: SLF4J: Defaulting to no-operation (NOP) logger implementation
[java]: SLF4J: See https://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
[java]: == Spark has ignited ...
[java]: >> Listening on 0.0.0.0:4567
```

Navegue até o aplicativo `localhost` em um navegador usando a URL para ver o aplicativo de exemplo. No exemplo acima, a `http://localhost:49804`URL é . Pare a `Ctrl+c`conexão usando .

## <a name="access-the-application-on-the-internet"></a>Acessar o aplicativo na internet

A etapa anterior criou uma conexão proxy para o pod do aplicativo no cluster AKS. Na medida em que você desenvolve e testa o aplicativo, convém disponibilizar o aplicativo na Internet. Para expor um aplicativo na internet, você pode criar um serviço Kubernetes com um tipo de [LoadBalancer][kubernetes-service-loadbalancer].

Atualize `charts/example-java/values.yaml` para criar um serviço *LoadBalancer.* Alterar o valor do *service.type* de *ClusterIP* para *LoadBalancer*.

```yaml
...
service:
  name: java
  type: LoadBalancer
  externalPort: 80
  internalPort: 4567
...
```

Salve suas alterações, feche `draft up` o arquivo e execute o aplicativo.

```console
draft up
```

Demora alguns minutos para o serviço retornar um endereço IP público. Para monitorar o andamento, use o comando `kubectl get service` com o parâmetro *watch*:

```console
$ kubectl get service --watch

NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
example-java-java   LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
...
example-java-java   LoadBalancer   10.0.141.72   52.175.224.118  80:32150/TCP   7m
```

Navegue até o balanceador de carga do seu aplicativo em um navegador usando o *EXTERNAL-IP* para ver o aplicativo de exemplo. No exemplo acima, o `52.175.224.118`IP é .

## <a name="iterate-on-the-application"></a>Iterar no aplicativo

Você pode iterar seu aplicativo fazendo alterações localmente e reexecutando `draft up`.

Atualize a mensagem retornada na [linha 7 de src/main/java/helloworld/Hello.java][example-java-hello-l7]

```java
    public static void main(String[] args) {
        get("/", (req, res) -> "Hello World, I'm Java in AKS!");
    }
```

Execute o comando `draft up` para reimplementar o aplicativo:

```console
$ draft up

Draft Up Started: 'example-java': 01CMZC9RF0TZT7XPWGFCJE15X4
example-java: Building Docker Image: SUCCESS ⚓  (25.0202s)
example-java: Pushing Docker Image: SUCCESS ⚓  (7.1457s)
example-java: Releasing Application: SUCCESS ⚓  (3.5773s)
Inspect the logs with `draft logs 01CMZC9RF0TZT7XPWGFCJE15X4`
```

Para ver o aplicativo atualizado, navegue até o endereço IP do balanceador de carga novamente e verifique se suas alterações aparecerão.

## <a name="delete-the-cluster"></a>Excluir o cluster

Quando o cluster não for mais necessário, use o comando [az group delete][az-group-delete] para remover o grupo de recursos, o cluster AKS, o registro de contêiner, as imagens de contêiner armazenadas lá e todos os recursos relacionados.

```azurecli-interactive
az group delete --name MyResourceGroup --yes --no-wait
```

> [!NOTE]
> Quando você excluir o cluster, a entidade de serviço do Azure Active Directory usada pelo cluster do AKS não será removida. Para obter etapas sobre como remover a entidade de serviço, veja [Considerações sobre a entidade de segurança e a exclusão de serviço AKS][sp-delete].

## <a name="next-steps"></a>Próximas etapas

Para obter mais informações sobre o uso do Draft, consulte a documentação do Draft no GitHub.

> [!div class="nextstepaction"]
> [Documentação do Draft][draft-documentation]


[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-create]: /cli/azure/acr#az-acr-login
[az-group-delete]: /cli/azure/group#az-group-delete
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[kubernetes-ingress]: ./ingress-basic.md

[docker-for-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-for-mac]: https://docs.docker.com/docker-for-mac/
[docker-for-windows]: https://docs.docker.com/docker-for-windows/
[draft-documentation]: https://github.com/Azure/draft/tree/master/docs
[example-java]: https://github.com/Azure/draft/tree/master/examples/example-java
[example-java-hello-l7]: https://github.com/Azure/draft/blob/master/examples/example-java/src/main/java/helloworld/Hello.java#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubernetes-service-loadbalancer]: https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer
[helm-init]: https://v2.helm.sh/docs/helm/#helm-init
[helm-install]: https://v2.helm.sh/docs/using_helm/#installing-helm
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[tiller-rbac]: https://v2.helm.sh/docs/using_helm/#tiller-namespaces-and-rbac
