# Módulo 4: APIs RESTful com o AWS Lambda e o Amazon API Gateway

Neste módulo, você usará o API Gateway para expor a função do Lambda construída no módulo anterior como uma API RESTful. Esta API estará acessível na Internet pública. Ele será protegido usando o pool de usuários do Amazon Cognito que você criou no módulo anterior. Usando essa configuração, você transformará seu site hospedado estaticamente em um aplicativo dinâmico da Web, adicionando o JavaScript do lado do cliente que faz chamadas AJAX para as APIs expostas.

![Dynamic web app architecture](../images/restful-api-architecture.png)

O diagrama acima mostra como o componente do API Gateway que você criará neste módulo se integra aos componentes existentes que você criou anteriormente. Os itens em cinza são peças que você já implementou nas etapas anteriores.

O site estático que você implantou no primeiro módulo já tem uma página configurada para interagir com a API que você criará neste módulo. A página em /ride.html possui uma interface simples baseada em mapas para solicitar um passeio de unicórnio. Depois de autenticar usando a página /signin.html, seus usuários poderão selecionar seu local de retirada clicando em um ponto no mapa e, em seguida, solicitando uma carona escolhendo o botão "Solicitar unicórnio" no canto superior direito.

Este módulo enfocará as etapas necessárias para criar os componentes de nuvem da API, mas se você estiver interessado em saber como funciona o código do navegador que chama essa API, inspecione o [ride.js](../1_StaticWebHosting/website/js/ride.js) arquivo do site. Nesse caso, o aplicativo usa o método [ajax ()] do jQuery (https://api.jquery.com/jQuery.ajax/) para fazer a solicitação remota.


## Instruções de Implementação

Cada uma das seções a seguir fornece uma visão geral da implementação e instruções detalhadas e passo a passo. A visão geral deve fornecer um contexto suficiente para você concluir a implementação, caso já esteja familiarizado com o AWS Management Console ou deseje explorar os serviços sem seguir uma explicação passo a passo.

Se você estiver usando a versão mais recente dos navegadores Chrome, Firefox ou Safari, as instruções passo a passo não serão visíveis até você expandir a seção.

### 1. Crie uma nova API REST
Use o console do Amazon API Gateway para criar uma nova API.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. No AWS Management Console, clique em ** Services ** e selecione ** API Gateway ** em Networking & Content Delivery.

1. Escolha ** Criar API **.

1. Selecione ** Nova API ** e insira "WildRydes" para o ** Nome da API **.

1. Mantenha 'Edge optimized' selecionado no menu suspenso ** Endpoint Type **.
    *** Nota ***: As margens otimizadas são melhores para serviços públicos acessados ​​pela Internet. Os terminais regionais geralmente são usados ​​para APIs acessadas principalmente na mesma região da AWS.

1. Escolha ** Criar API **

    ![Criar captura de tela da API](../images/create-api.png)

</p> </details>


### 2. Criar um autorizador de pools de usuários do Cognito

#### Contexto
O Amazon API Gateway pode usar os tokens JWT retornados pelos pools de usuários do Cognito para autenticar chamadas de API. Nesta etapa, você configurará um autorizador para sua API para usar o pool de usuários que você criou no [módulo 2](../2_UserManagement).

#### Instruções gerais
No console do Amazon API Gateway, crie um novo autorizador do pool de usuários do Cognito para sua API. Configure-o com os detalhes do pool de usuários que você criou no módulo anterior. Você pode testar a configuração no console copiando e colando o token de autenticação apresentado a você depois de efetuar login através da página /signin.html do seu site atual.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. Sob sua API recém-criada, escolha ** Autorizadores **.

1. Escolha ** Criar novo autorizador **.

1. Digite "WildRydes" para o nome do Autorizador.

1. Selecione ** Cognito ** para o tipo.

1. No menu suspenso Região, em ** Conjunto de usuários do Cognito **, selecione a Região na qual você criou seu conjunto de usuários do Cognito no módulo 2 (por padrão, a região atual deve ser selecionada).

1. Digite `WildRydes` (ou o nome que você forneceu ao seu pool de usuários) na entrada ** do Cognito User Pool **.

1. Digite "Authorization" para a ** Fonte do Token **.

1. Escolha ** Criar **.

    ![Criar captura de tela do autorizador do pool de usuários](../images/create-user-pool-authorizer.png)

#### Verifique a configuração do seu autorizador

1. Abra uma nova aba do navegador e visite `/ride.html` no domínio do seu site.

1. Se você for redirecionado para a página de entrada, entre com o usuário que você criou no último módulo. Você será redirecionado de volta para `/ride.html`.

1. Copie o token de autenticação da notificação no `/ride.html`,

1. Volte para a guia anterior onde você acabou de criar o Autorizador

1. Clique em ** Teste ** na parte inferior do cartão para o autorizador.

1. Cole o token de autenticação no campo ** Token de autorização ** na caixa de diálogo pop-up.

    ![Captura de tela do Authorizer](../images/apigateway-test-authorizer.png)

1. Clique no botão ** Test ** e verifique se o código de resposta é 200 e se você vê as declarações do usuário exibidas.

</p> </details>

### 3. Crie um novo resource e método
Crie um novo recurso chamado /ride na sua API. Em seguida, crie um método POST para esse recurso e configure-o para usar uma integração de proxy do Lambda respaldada pela função RequestUnicorn criada na primeira etapa deste módulo.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. Na nav esquerda, clique em ** Recursos ** sob sua API WildRydes.

1. Na lista suspensa ** Ações **, selecione ** Criar recurso **.

1. Digite `ride` como o ** Nome do Recurso **.

1. Assegure-se de que o ** Caminho do Recurso ** esteja configurado para `ride`.

1. Selecione ** Ativar o CORS do Gateway de API ** para o recurso.

1. Clique em ** Criar Recurso **.

    ![Criar captura de tela do recurso](../images/create-resource.png)

1. Com o recém-criado recurso `/ride` selecionado, no menu suspenso ** Ação **, selecione ** Criar Método **.

1. Selecione `POST` na nova lista suspensa que aparece, depois ** clique na marca de seleção **.

    ![Criar captura de tela do método](../images/create-method.png)

1. Selecione ** Função Lambda ** para o tipo de integração.

1. Marque a caixa para ** Usar integração do Lambda Proxy **.

1. Selecione a região que você está usando para a ** Região Lambda **.

1. Digite o nome da função que você criou no módulo anterior, `RequestUnicorn`, para ** Função Lambda **.

1. Escolha ** Salvar **. Por favor, note que se você encontrar um erro que sua função não existe, verifique se a região que você selecionou corresponde àquela que você usou no módulo anterior.

    ![Captura de tela de integração do método API](../images/api-integration-setup.png)

1. Quando solicitado a conceder ao Amazon API Gateway permissão para invocar sua função, escolha ** OK **.

1. Escolha no cartão ** Pedido de Método **.

1. Escolha o ícone de lápis ao lado de ** Autorização **.

1. Selecione o autorizador do pool de usuários do WildRydes Cognito na lista suspensa e clique no ícone de marca de seleção.

    ![Captura de tela da configuração do autorizador da API](../images/api-authorizer.png)

</p> </details>

### 4. Implemente sua API
No console do Amazon API Gateway, escolha Ações, Implantar API. Você será solicitado a criar um novo estágio. Você pode usar prod para o nome artístico.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. Na lista suspensa ** Ações **, selecione ** Implantar API **.

1. Selecione ** [New Stage] ** na lista suspensa ** Implantação **.

1. Digite `prod` para o ** Stage Name **.

1. Escolha ** Deploy **.

1. Observe o ** Invocar URL **. Você irá usá-lo na próxima seção.

</p> </details>

### 5. Atualize o Config do site
Atualize o arquivo /js/config.js na implantação do seu site para incluir o URL de invocação do estágio que você acabou de criar. Você deve copiar o URL de chamada diretamente da parte superior da página do editor de palco no console do Amazon API Gateway e colá-lo na chave \_config.api.invokeUrl do arquivo /js/config.js de seus sites. Certifique-se de que, ao atualizar o arquivo de configuração, ele ainda contenha as atualizações feitas no módulo anterior para o conjunto de usuários do Cognito.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

Se você completou o módulo 2 manualmente, você pode editar o arquivo `config.js` que você salvou localmente. Se você usou o modelo do AWS CloudFormation, primeiro baixe o arquivo `config.js` do seu bucket do S3. Para fazer isso, visite `/ js / config.js` sob o URL base do seu site, escolha ** Arquivo ** e escolha ** Salvar Página Como ** em seu navegador.

1. Abra o arquivo config.js em um editor de texto.

1. Atualize a configuração ** invokeUrl ** sob a chave ** api ** no arquivo config.js. Defina o valor para ** Invocar URL ** para o estágio de implantação criado na seção anterior.

    Um exemplo de um arquivo `config.js` completo está incluído abaixo. Note que os valores reais em seu arquivo serão diferentes.

    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // por exemplo us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // por exemplo 25ddkmj4v6hfsfvruhpfi7n4hv
            região: 'us-west-2' // por exemplo us-east-2
        }
        api: {
            invokeUrl: 'https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod' // por exemplo https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod,
        }
    };
    ```

1. Salve suas alterações localmente.

1. No AWS Management Console, escolha ** Serviços ** e selecione ** S3 ** em Armazenamento.

1. Escolha o intervalo do seu site e navegue até a pasta `js`.

1. Escolha ** Upload **.

1. Escolha ** Adicionar arquivos **, selecione a cópia local do `config.js` e clique em ** Avançar **.

1. Escolha ** Next ** sem alterar nenhum padrão através das seções `Set permissions` e` Set properties`.

1. Escolha ** Upload ** na seção `Review`.

</p> </details>

## Validação de Implementação

** Nota: ** É possível que você veja um atraso entre atualizar o arquivo config.js no seu bucket do S3 e quando o conteúdo atualizado estiver visível em seu navegador. Você também deve garantir que você limpe o cache do navegador antes de executar as etapas a seguir.

1. Visite `/ride.html` sob o domínio do seu site.

1. Se você for redirecionado para a página de login, faça login com o usuário que você criou no módulo anterior.

1. Depois que o mapa for carregado, clique em qualquer lugar no mapa para definir um local de coleta.

1. Escolha ** Solicitar Unicórnio **. Você deve ver uma notificação na barra lateral direita de que um unicórnio está a caminho e, em seguida, ver um ícone de unicórnio voando para o seu local de retirada.

Parabéns, você completou o Workshop de Aplicações Web Wild Rydes!

Veja o [guia de limpeza] deste workshop (../9_CleanUp) para obter instruções sobre como excluir os recursos que você criou.
