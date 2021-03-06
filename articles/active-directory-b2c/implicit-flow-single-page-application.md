---
title: Login de uma página usando fluxo implícito
titleSuffix: Azure AD B2C
description: Saiba como adicionar login de página única usando o fluxo implícito OAuth 2.0 com o Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 07/19/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 37b59c2a23a8f00e8376be2ac4a7b35a6d58aa28
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/28/2020
ms.locfileid: "78399007"
---
# <a name="single-page-sign-in-using-the-oauth-20-implicit-flow-in-azure-active-directory-b2c"></a>Login de página única usando o fluxo implícito OAuth 2.0 no Azure Active Directory B2C

Muitos aplicativos modernos têm um front-end de aplicativo de uma única página que é escrito principalmente em JavaScript. Muitas vezes, o aplicativo é escrito usando uma estrutura como React, Angular ou Vue.js. Os aplicativos de página única e outros aplicativos JavaScript que são executados principalmente em um navegador possuem alguns desafios adicionais para autenticação:

- As características de segurança desses aplicativos são diferentes dos aplicativos web tradicionais baseados em servidor.
- Muitos servidores de autorização e provedores de identidade não dão suporte para solicitações CORS (compartilhamento de recursos entre origens).
- Redirecionamentos de página inteira do aplicativo podem ser invasivos à experiência do usuário.

Para oferecer suporte a esses aplicativos, o Azure AD B2C (Azure Active Directory B2C) utiliza o fluxo implícito do OAuth 2.0. O fluxo de concessão implícita de autorização do OAuth 2.0 é descrito na [seção 4.2 da especificação do OAuth 2.0](https://tools.ietf.org/html/rfc6749). No fluxo implícito, o aplicativo recebe tokens diretamente do ponto de extremidade autorizado do Azure AD (Azure Active Directory) sem qualquer troca de servidor para servidor. Toda a lógica de autenticação e o manuseio de sessão são feitos inteiramente no cliente JavaScript com um redirecionamento de página ou uma caixa pop-up.

O Azure AD B2C estende o fluxo implícito do OAuth 2.0 padrão para mais que autenticação e autorização simples. O Azure AD B2C introduz o [parâmetro de política](user-flow-overview.md). Com o parâmetro de política, é possível usar o OAuth 2.0 para adicionar políticas ao seu aplicativo, como fluxos de usuários de inscrição, conexão e gerenciamento de perfil. No exemplo solicita http neste artigo, **{tenant}.onmicrosoft.com** é usado como exemplo. Substitua pelo `{tenant}` nome do seu inquilino se você tiver um e também tenha criado um fluxo de usuário.

O fluxo de entrada implícito parece ser semelhante à seguinte figura. Cada etapa é descrita detalhadamente mais adiante no artigo.

![Diagrama no estilo swimlane mostrando o fluxo implícito OpenID Connect](./media/implicit-flow-single-page-application/convergence_scenarios_implicit.png)

## <a name="send-authentication-requests"></a>Enviar solicitações de autenticação

Quando seu aplicativo web precisa autenticar o usuário e executar um fluxo `/authorize` de usuário, ele pode direcionar o usuário para o ponto final. O usuário toma medidas dependendo do fluxo do usuário.

Nesta solicitação, o cliente indica as permissões que precisa `scope` adquirir do usuário no parâmetro e o fluxo do usuário para executar. Para ter uma noção de como a solicitação funciona, tente colar a solicitação em um navegador e executá-la. Substitua `{tenant}` pelo nome de seu locatário do Azure AD B2C. Substitua pelo `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` ID do aplicativo que você já registrou anteriormente em seu inquilino. Substitua pelo `{policy}` nome de uma política que você criou `b2c_1_sign_in`em seu inquilino, por exemplo.

```HTTP
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=id_token+token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&response_mode=fragment
&scope=openid%20offline_access
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
```

| Parâmetro | Obrigatório | Descrição |
| --------- | -------- | ----------- |
|{inquilino}| Sim | Nome do seu inquilino Azure AD B2C|
|{política}| Sim| O fluxo de usuário a ser executado. Especifique o nome de um fluxo de usuário que você criou no seu inquilino Azure AD B2C. Por `b2c_1_sign_in`exemplo: `b2c_1_sign_up`, `b2c_1_edit_profile`, ou . |
| client_id | Sim | O ID de aplicação que o [portal Azure](https://portal.azure.com/) atribuiu ao seu aplicativo. |
| response_type | Sim | Deve incluir `id_token` para conexão do OpenID Connect. É possível também incluir o tipo de resposta `token`. Se utilizar `token`, seu aplicativo poderá receber imediatamente um token de acesso do ponto de extremidade autorizado, sem fazer uma segunda solicitação para o ponto de extremidade autorizado.  Se utilizar o tipo de resposta `token`, o `scope` parâmetro deverá conter um escopo indicando para quais recursos o token será emitido. |
| redirect_uri | Não | O URI de redirecionamento do seu aplicativo, onde as respostas de autenticação podem ser enviadas e recebidas pelo aplicativo. Ele deve corresponder exatamente a um dos URIs de redirecionamento registrados no portal, exceto que ele deve ser codificado como URL. |
| response_mode | Não | Especifica o método que deve ser usado para enviar o token resultante de volta ao aplicativo.  Para fluxos implícitos, utilize `fragment`. |
| scope | Sim | Uma lista de escopos separados por espaços. Um valor de escopo único indica ao Azure AD que ambas as permissões estão sendo solicitadas. O escopo `openid` indica uma permissão para entrar no usuário e obter dados sobre ele na forma de tokens de ID. O escopo `offline_access` é opcional para aplicativos Web. Isso indica que seu aplicativo precisa de um token de atualização para acesso de longa vida para recursos. |
| state | Não | Um valor incluído na solicitação que também é retornado na resposta de token. Pode ser uma cadeia de caracteres de qualquer conteúdo que você deseje usar. Geralmente, um valor exclusivo gerado aleatoriamente é utilizado para evitar ataques de solicitação intersite forjada. O estado também é utilizado para codificar informações sobre o estado do usuário no aplicativo antes da solicitação de autenticação ocorrida, como a página em que estava. |
| nonce | Sim | Um valor incluído na solicitação, gerado pelo aplicativo, incluído no token de ID resultante como uma declaração. O aplicativo pode verificar esse valor para reduzir os ataques de reprodução de token. Normalmente, o valor é uma cadeia de caracteres aleatória e exclusiva que pode ser usada para identificar a origem da solicitação. |
| prompt | Não | O tipo de interação do usuário que é necessária. Atualmente, o único valor válido é `login`. Este parâmetro força o usuário a inserir suas credenciais nessa solicitação. A única inscrição não faz efeito. |

Nesse momento, é solicitado que o usuário conclua o fluxo de trabalho da política. O usuário pode ter que digitar seu nome de usuário e senha, entrar com uma identidade social, se inscrever no diretório ou qualquer outro número de etapas. As ações do usuário dependem de como o fluxo de usuário é definido.

Depois que o usuário completar o fluxo de usuário, o Azure AD retornará uma resposta ao seu aplicativo no valor usado para `redirect_uri`. Ele usa o método especificado no parâmetro `response_mode`. A resposta é exatamente a mesma para cada um dos cenários de ação do usuário, independentemente de qual fluxo de usuário foi executado.

### <a name="successful-response"></a>Resposta bem-sucedida
Uma resposta bem sucedida que utiliza `response_mode=fragment` e `response_type=id_token+token` é semelhante à seguinte, com quebras de linha para legibilidade:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&token_type=Bearer
&expires_in=3599
&scope="90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6 offline_access",
&id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
```

| Parâmetro | Descrição |
| --------- | ----------- |
| access_token | O token de acesso que o aplicativo solicitou. |
| token_type | O valor do tipo de token. O único tipo com suporte do Azure AD é Portador |
| expires_in | O período de tempo pelo qual o token de acesso é válido (em segundos). |
| scope | Os escopos para os quais o token é válido. Você também pode usar os escopos para armazenar tokens em cache para uso posterior. |
| id_token | O token de ID que o aplicativo solicitou. Você pode usar o token de ID para verificar a identidade do usuário e iniciar uma sessão com o usuário. Para obter mais informações sobre tokens de identificação e seus conteúdos, consulte a [referência de token do Azure AD B2C](tokens-overview.md). |
| state | Se um parâmetro `state` estiver incluído na solicitação, o mesmo valor deverá aparecer na resposta. O aplicativo deve verificar se os valores `state` na solicitação e na resposta são idênticos. |

### <a name="error-response"></a>Resposta de erro
As respostas de erro também podem ser enviadas ao URI de redirecionamento, de modo que o aplicativo possa tratá-las adequadamente:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
error=access_denied
&error_description=the+user+canceled+the+authentication
&state=arbitrary_data_you_can_receive_in_the_response
```

| Parâmetro | Descrição |
| --------- | ----------- |
| error | Um código usado para classificar tipos de erros que ocorrem. |
| error_description | Uma mensagem de erro específica que pode ajudar você a identificar a causa raiz de um erro de autenticação. |
| state | Se um parâmetro `state` estiver incluído na solicitação, o mesmo valor deverá aparecer na resposta. O aplicativo deve verificar se os valores `state` na solicitação e na resposta são idênticos.|

## <a name="validate-the-id-token"></a>Validar o token de ID

Receber um token de ID não é suficiente para autenticar o usuário. Valide a assinatura do token de ID e verifique as reclamações no token de acordo com os requisitos do aplicativo. O Azure AD B2C usa [JWTs (Tokens Web JSON)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) e criptografia de chave pública para assinar tokens e verificar se eles são válidos.

Muitas bibliotecas de software livre estão disponíveis para validar JWTs, dependendo do idioma de sua preferência. Considere explorar bibliotecas de software livre disponíveis em vez de implementar sua própria lógica de validação. As informações contidas neste artigo podem ser utilizadas para ajudá-lo a aprender como utilizar essas bibliotecas corretamente.

O Azure AD B2C tem um ponto de extremidade de metadados OpenID Connect. Um aplicativo pode usar o ponto de extremidade para buscar informações sobre o Azure AD B2C em runtime. Essas informações incluem pontos de extremidade, conteúdos de token e chaves de assinatura de token. Há um documento de metadados JSON para cada fluxo de usuário no locatário do Azure AD B2C. Por exemplo, o documento de metadados para o fluxo de usuário b2c_1_sign_in no locatário fabrikamb2c.onmicrosoft.com está localizado em:

```HTTP
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/v2.0/.well-known/openid-configuration
```

Uma das propriedades deste documento de configuração é `jwks_uri`. O valor para o mesmo fluxo de usuário seria:

```HTTP
https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_sign_in/discovery/v2.0/keys
```

para determinar qual fluxo de usuário foi utilizado para assinar um token de ID (e onde buscar os metadados), há duas opções. Primeiro, o nome do fluxo de usuário está incluído na declaração `acr` em `id_token`. Para obter informações sobre como analisar as declarações de um token de ID, consulte a [referência de token do Azure AD B2C](tokens-overview.md). Sua outra opção é codificar o fluxo de usuário no valor do parâmetro `state` quando emitir a solicitação. Em seguida, decodifique o parâmetro `state` para determinar qual fluxo de usuário foi usado. Ambos os métodos são válidos.

Após ter adquirido o documento de metadados a partir do ponto de extremidade de metadados OpenID Connect, você poderá utilizar as chaves públicas RSA-256 (localizadas nesse ponto de extremidade) para validar a assinatura do token de ID. Poderá haver várias chaves listadas nesse ponto de extremidade em qualquer momento, cada uma identificada por um `kid`. O cabeçalho de `id_token` também contém uma declaração `kid`. Ele indica qual dessas chaves foi utilizada para assinar o token de ID. Para obter mais informações, incluindo aprender sobre [tokens de validação](tokens-overview.md), consulte a [referência de token do Azure AD B2C](tokens-overview.md).
<!--TODO: Improve the information on this-->

Após validar a assinatura do token de ID, várias declarações exigirão verificação. Por exemplo: 

* Valide a declaração `nonce` para evitar ataques de reprodução de token. Seu valor deve ser o que você especificou na solicitação de conexão.
* Valide `aud` para garantir que o token de ID foi emitido para seu aplicativo. Seu valor deve ser a ID do aplicativo do seu aplicativo.
* Valide as declarações `iat` e `exp` para garantir que o token de ID não expirou.

Várias outras validações que você deve executar são descritas detalhadamente na [Especificação do Núcleo de Conexão OpenID](https://openid.net/specs/openid-connect-core-1_0.html). Você também pode querer validar reivindicações adicionais, dependendo do seu cenário. Algumas validações comuns incluem:

* Garanta que o usuário ou a organização tenha se inscrito no aplicativo.
* Garanta que o usuário tenha autorização e privilégios adequados.
* Garanta que uma determina força de autenticação tenha ocorrido como, por exemplo, utilizando a autenticação multifator do Azure.

Para obter mais informações sobre as reivindicações em um token de ID, consulte a [referência de token do Azure AD B2C](tokens-overview.md).

Após validar o token de ID, você poderá iniciar uma sessão com o usuário. No seu aplicativo, use as declarações no token de ID para obter informações sobre o usuário. Essas informações podem ser usadas para exibição, registros, autorizações e outros.

## <a name="get-access-tokens"></a>Obter tokens de acesso
Se os seus aplicativos Web precisam apenas executar fluxos de usuários, você pode ignorar as próximas seções. As informações nas seções a seguir são aplicáveis apenas a aplicativos web que precisam fazer chamadas autenticadas para uma API web e que são protegidas pelo Azure AD B2C.

Agora que você assinou o usuário em seu aplicativo de página única, você pode obter tokens de acesso para chamar APIs da Web que são protegidas pelo Azure AD. Mesmo que já tenha recebido um token utilizando o tipo de resposta `token`, você poderá utilizar esse método para adquirir tokens para recursos adicionais sem redirecionar o usuário para conectar novamente.

Em um fluxo típico de aplicativo web, `/token` você faria uma solicitação para o ponto final. No entanto, o ponto final não suporta solicitações cors, portanto, fazer chamadas AJAX para obter um token de atualização não é uma opção. Em vez disso, você poderá utiliza o fluxo implícito em um elemento iframe HTML oculto para obter novos tokens para outras APIs Web. A seguir está um exemplo, com quebras de linha para legibilidade:

```HTTP
https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/authorize?
client_id=90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6
&response_type=token
&redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
&response_mode=fragment
&state=arbitrary_data_you_can_receive_in_the_response
&nonce=12345
&prompt=none
```

| Parâmetro | Obrigatório? | Descrição |
| --- | --- | --- |
|{inquilino}| Obrigatório | Nome do seu inquilino Azure AD B2C|
{política}| Obrigatório| O fluxo de usuário a ser executado. Especifique o nome de um fluxo de usuário que você criou no seu inquilino Azure AD B2C. Por `b2c_1_sign_in`exemplo: `b2c_1_sign_up`, `b2c_1_edit_profile`, ou . |
| client_id |Obrigatório |A ID do aplicativo atribuída ao seu aplicativo no [portal do Azure](https://portal.azure.com). |
| response_type |Obrigatório |Deve incluir `id_token` para conexão do OpenID Connect.  Também é possível incluir o tipo de resposta `token`. Se utilizar `token` aqui, seu aplicativo poderá receber imediatamente um token de acesso do ponto de extremidade autorizado, sem fazer uma segunda solicitação para o ponto de extremidade autorizado. Se utilizar o tipo de resposta `token`, o `scope` parâmetro deverá conter um escopo indicando para quais recursos o token será emitido. |
| redirect_uri |Recomendadas |O URI de redirecionamento do seu aplicativo, onde as respostas de autenticação podem ser enviadas e recebidas pelo aplicativo. Ele deve coincidir exatamente com um dos URIs de redirecionamento registrados no portal, exceto que deve ser codificado em URL. |
| scope |Obrigatório |Uma lista de escopos separados por espaços.  Para obter tokens, inclua todos os escopos que necessários para o recurso pretendido. |
| response_mode |Recomendadas |Especifica o método que deve ser usado para enviar o token resultante de volta ao aplicativo. Para fluxo implícito, use `fragment`. Dois outros modos podem `query` ser `form_post`especificados e, mas não funcionam no fluxo implícito. |
| state |Recomendadas |Um valor incluído na solicitação que retorna na resposta do token.  Pode ser uma cadeia de caracteres de qualquer conteúdo que você deseje usar.  Geralmente, um valor exclusivo gerado aleatoriamente é utilizado para evitar ataques de solicitação intersite forjada.  O estado também é usado para codificar informações sobre o estado do usuário no aplicativo, antes que a solicitação de autenticação tenha ocorrido. Por exemplo, a página ou a exibição do usuário estava ativada. |
| nonce |Obrigatório |Um valor incluído na solicitação, gerado pelo aplicativo, que está incluído no token de ID resultante como uma reivindicação.  O aplicativo pode verificar esse valor para reduzir os ataques de reprodução de token. Normalmente, o valor é uma cadeia de caracteres aleatória e exclusiva que identifica a origem da solicitação. |
| prompt |Obrigatório |Para atualizar e obter tokens em um iframe oculto, utilize `prompt=none` para garantir que o iframe não fique preso na página de entrada e retorna imediatamente. |
| login_hint |Obrigatório |Para atualizar e obter tokens em um iframe oculto, inclua o nome de usuário de usuário nesta dica para distinguir entre várias sessões que o usuário pode ter em um determinado momento. Você pode extrair o nome de usuário de `preferred_username` um `profile` login anterior usando a `preferred_username` reclamação (o escopo é necessário para receber a solicitação). |
| domain_hint |Obrigatório |Pode ser `consumers` ou `organizations`.  Para atualizar e obter tokens em um `domain_hint` iframe oculto, inclua o valor na solicitação.  Extrair `tid` a reclamação do token ID de um login anterior para `profile` determinar qual valor usar `tid` (o escopo é necessário para receber a reclamação). Se o valor da declaração `tid` for `9188040d-6c67-4c5b-b112-36a304b66dad`,  utilize `domain_hint=consumers`.  Caso contrário, use `domain_hint=organizations`. |

Ao configurar o parâmetro `prompt=none`, essa solicitação terá êxito ou falhará imediatamente e retornará ao seu aplicativo.  Uma resposta bem-sucedida será enviada para seu aplicativo no URI de redirecionamento indicado, utilizando o método especificado no parâmetro `response_mode`.

### <a name="successful-response"></a>Resposta bem-sucedida
Uma resposta bem `response_mode=fragment` sucedida usando parece este exemplo:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
access_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
&state=arbitrary_data_you_sent_earlier
&token_type=Bearer
&expires_in=3599
&scope=https%3A%2F%2Fapi.contoso.com%2Ftasks.read
```

| Parâmetro | Descrição |
| --- | --- |
| access_token |O token solicitado pelo aplicativo. |
| token_type |O tipo de token sempre será Portador. |
| state |Se um parâmetro `state` estiver incluído na solicitação, o mesmo valor deverá aparecer na resposta. O aplicativo deve verificar se os valores `state` na solicitação e na resposta são idênticos. |
| expires_in |Por quanto tempo o token de acesso é válido (em segundos). |
| scope |Os escopos para os quais o access_token é válido. |

### <a name="error-response"></a>Resposta de erro
As respostas de erro também podem ser enviadas para URI de redirecionamento, de modo que o aplicativo possa tratá-los adequadamente.  Para `prompt=none`, um erro esperado se parece com este exemplo:

```HTTP
GET https://aadb2cplayground.azurewebsites.net/#
error=user_authentication_required
&error_description=the+request+could+not+be+completed+silently
```

| Parâmetro | Descrição |
| --- | --- |
| error |Uma cadeia de caracteres de código de erro que pode ser utilizada para classificar os tipos de erros que ocorrem. Você também pode usar a cadeia de caracteres para reagir a erros. |
| error_description |Uma mensagem de erro específica que pode ajudar você a identificar a causa raiz de um erro de autenticação. |

Se você receber esse erro na solicitação do iframe, o usuário deverá entrar novamente de forma interativa para recuperar um novo token.

## <a name="refresh-tokens"></a>Tokens de atualização
Os tokens de ID e tokens de acesso expiram após um curto período de tempo. Seu aplicativo deverá estar preparado para atualizar esses tokens periodicamente.  Para atualizar qualquer tipo de token, execute a mesma solicitação de iframe oculto utilizada no exemplo anterior, utilizando o parâmetro `prompt=none` para controlar as etapas do AD Azure.  Para receber um novo valor `id_token`, certifique-se de usar `response_type=id_token` e `scope=openid` e um parâmetro `nonce`.

## <a name="send-a-sign-out-request"></a>Enviar uma solicitação de saída
Quando você quiser assinar o usuário fora do aplicativo, redirecione o usuário para o Azure AD para sair. Se você não redirecionar o usuário, ele pode ser capaz de reautenticar para o seu aplicativo sem inserir suas credenciais novamente porque eles têm uma sessão de login único válida com o Azure AD.

Você pode simplesmente redirecionar o usuário para `end_session_endpoint` que está listado no mesmo documento de metadados do OpenID Connect descrito em [Validar o ID de token](#validate-the-id-token). Por exemplo: 

```HTTP
GET https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/oauth2/v2.0/logout?post_logout_redirect_uri=https%3A%2F%2Faadb2cplayground.azurewebsites.net%2F
```

| Parâmetro | Obrigatório | Descrição |
| --------- | -------- | ----------- |
| {inquilino} | Sim | Nome do seu inquilino Azure AD B2C |
| {política} | Sim | O fluxo de usuário que você quer usar para desconectar o usuário do aplicativo. |
| post_logout_redirect_uri | Não | A URL para a a nossa ção que o usuário deve ser redirecionado após a saída bem sucedida. Se não estiver incluído, o Azure AD B2C mostra ao usuário uma mensagem genérica. |
| state | Não | Se um parâmetro `state` estiver incluído na solicitação, o mesmo valor deverá aparecer na resposta. O aplicativo deve `state` verificar se os valores na solicitação e resposta são idênticos. |


> [!NOTE]
> Direcionar o usuário para `end_session_endpoint` limpa alguns dos estados de logon único do usuário com o Azure AD B2C. No entanto, ele não desconecta o usuário da sessão do provedor de identidade social do usuário. Se o usuário selecionar o mesmo provedor de identidade durante um login subseqüente, o usuário será reautenticado, sem inserir suas credenciais. Se um usuário quiser sair do serviço de seu aplicativo do Azure AD B2C, isso não significa necessariamente que ele deseja se desconectar completamente de sua conta do Facebook, por exemplo. No entanto, para contas locais, a sessão do usuário será encerrada corretamente.
>

## <a name="next-steps"></a>Próximas etapas

### <a name="code-sample-azure-ad-b2c-with-microsoft-authentication-library-for-javascript"></a>Exemplo de código: Azure AD B2C com Biblioteca de Autenticação Microsoft para JavaScript

[Aplicativo de página única construído com msal.js para Azure AD B2C][github-msal-js-example] (GitHub)

Esta amostra no GitHub destina-se a ajudá-lo a começar a ad B2C azure em um aplicativo web simples construído com [msal.js][github-msal-js] e usando autenticação no estilo pop-up.

<!-- Links - EXTERNAL -->
[github-msal-js-example]: https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp
[github-msal-js]: https://github.com/AzureAD/microsoft-authentication-library-for-js
