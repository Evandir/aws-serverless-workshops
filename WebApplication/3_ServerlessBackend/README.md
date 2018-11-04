# Módulo 3: Serviço de Backend Serverless

Neste módulo, você usará o AWS Lambda e o Amazon DynamoDB para criar um processo de back-end para lidar com solicitações daa sua aplicação web. A aplicação web que você implantou no primeiro módulo permite que os usuários solicitem que um unicórnio seja enviado para um local de sua escolha. Para atender a essas solicitações, o JavaScript em execução no navegador precisará invocar um serviço em execução na nuvem.

Você implementará uma função do Lambda que será invocada toda vez que um usuário solicitar um unicórnio. A função selecionará um unicórnio da frota, registrará a solicitação em uma tabela do DynamoDB e responderá ao aplicativo front-end com detalhes sobre o unicórnio sendo enviado.

![Serverless backend architecture](../images/serverless-backend-architecture.png)

A função é chamada a partir do navegador usando o Amazon API Gateway. Você implementará essa conexão no próximo módulo. Para este módulo, você apenas testará sua função isoladamente.


## Instruções de Implementação

Cada uma das seções a seguir fornece uma visão geral da implementação e instruções detalhadas passo a passo. A visão geral deve fornecer um contexto suficiente para você concluir a implementação, caso já esteja familiarizado com o AWS Management Console ou deseje explorar os serviços sem seguir uma explicação passo a passo.

Se você estiver usando a versão mais recente dos navegadores Chrome, Firefox ou Safari, as instruções passo a passo não serão visíveis até você expandir a seção.

### 1. Crie uma tabela do Amazon DynamoDB

Use o console do Amazon DynamoDB para criar uma nova tabela do DynamoDB. Chame sua tabela `Rides` e dê a ela uma chave de partição chamada` RideId` com o tipo String. O nome da tabela e a chave de partição fazem distinção entre maiúsculas e minúsculas. Certifique-se de usar os IDs exatos fornecidos. Use os padrões para todas as outras configurações.

Depois de criar a tabela, observe o ARN para uso na próxima etapa.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> 
<p>

1. No AWS Management Console, escolha ** Serviços ** e selecione ** DynamoDB ** em Bancos de dados.

1. Escolha ** Criar tabela **.

1. Digite `Rides` para o ** nome da tabela **. Este campo diferencia maiúsculas de minúsculas.

1. Digite `RideId` para a  ** Partition key ** e selecione ** String ** para o tipo de chave. Este campo diferencia maiúsculas de minúsculas.

1. Marque a caixa ** Use default settings ** e escolha ** Create **.

    ![Create table screenshot](../images/ddb-create-table.png)

1. Role até a parte inferior da seção Visão geral de sua nova tabela e observe o ** ARN **. Você usará isso na próxima seção.

</p> </details>


### 2. Crie uma função IAM para sua função Lambda

#### Contexto

Cada função Lambda tem uma função do IAM associada a ela. Essa função define com quais outros serviços da AWS a função pode interagir. Para os propósitos deste workshop, você precisará criar uma função do IAM que conceda a permissão da função do Lambda para gravar logs no Amazon CloudWatch Logs e acessar itens de gravação na tabela do DynamoDB.

#### Instruções gerais

Use o console do IAM para criar um novo Role. Nomeie-o como "WildRydesLambda" e selecione o AWS Lambda para o tipo de função. Você precisará anexar políticas que concedam suas permissões de função para gravar nos logs do Amazon CloudWatch e colocar itens em sua tabela do DynamoDB.

Anexe a diretiva gerenciada chamada `AWSLambdaBasicExecutionRole` a essa função para conceder as permissões necessárias dos Logs do CloudWatch. Além disso, crie uma política embutida personalizada para sua função que permita a ação `ddb: PutItem` para a tabela criada na seção anterior.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. No AWS Management Console, clique em ** Serviços ** e selecione ** IAM ** na seção Segurança, identidade e conformidade.

1. Selecione ** Funções ** na barra de navegação à esquerda e, em seguida, escolha ** Criar nova função **.

1. Selecione ** Lambda ** para o tipo de função no grupo ** AWS service ** e clique em ** Next: Permissions **

1. Comece a digitar `AWSLambdaBasicExecutionRole` na caixa de texto ** Filter ** e marque a caixa ao lado dessa função.

1. Clique em ** Próximo: Revisar **.

1. Digite `WildRydesLambda` para o ** Nome da função **.

1. Escolha ** Criar papel **.

1. Digite `WildRydesLambda` na caixa de filtro na página Funções e escolha a função que você acabou de criar.

1. Na guia Permissões, escolha o link ** Adicionar política in-line ** no canto inferior direito para criar uma nova política in-line.
    ![Inline policies screenshot](../images/inline-policies.png)

1. Selecione ** Escolher um serviço **.

1. Comece a digitar `DynamoDB` na caixa de pesquisa chamada ** Encontrar um serviço ** e selecione ** DynamoDB ** quando ele aparecer.
    ![Select policy service](../images/select-policy-service.png)

1. Escolha ** Selecionar ações **.

1. Comece a digitar `PutItem` na caixa de pesquisa chamada ** Ações de filtro ** e marque a caixa ao lado de ** PutItem ** quando ele aparecer.

1. Selecione a seção ** Recursos **.

1. Com a opção ** Específico ** selecionada, escolha o link Adicionar ARN na seção ** tabela **.

1. Cole o ARN da tabela criada na seção anterior no campo ** Especificar ARN para a tabela ** e escolha ** Adicionar **.

1. Escolha ** Revisar política **.

1. Digite `DynamoDBWriteAccess` para o nome da política e escolha ** Create policy **.
    ![Review Policy](../images/review-policy.png)

</p> </details>

### 3. Criar uma função do Lambda para lidar com solicitações

#### Contexto

O AWS Lambda executará seu código em resposta a eventos como uma solicitação HTTP. Nesta etapa, você criará a função principal que processará solicitações de API do aplicativo da Web para despachar um unicórnio. No próximo módulo, você usará o Amazon API Gateway para criar uma API RESTful que exporá um endpoint HTTP que pode ser chamado a partir dos navegadores de seus usuários. Em seguida, você conectará a função Lambda criada nesta etapa a essa API para criar um back-end totalmente funcional para seu aplicativo da web.

#### Instruções gerais

Use o console do AWS Lambda para criar uma nova função do Lambda chamada `RequestUnicorn` que processará as solicitações da API. Use a implementação de exemplo fornecida [requestUnicorn.js] (requestUnicorn.js) para o seu código de função. Basta copiar e colar desse arquivo no editor do console do AWS Lambda.

Certifique-se de configurar sua função para usar a função IAM do `WildRydesLambda` criada na seção anterior.

<details>
<summary> <strong> Instruções passo a passo (expanda para detalhes) </strong> </summary> <p>

1. Escolha em ** Serviços ** e selecione ** Lambda ** na seção Computação.

1. Clique em ** Criar função **.

1. Mantenha o cartão padrão ** Criar do zero ** selecionado.

1. Digite `RequestUnicorn` no campo ** Nome **.

1. Selecione ** Node.js 6.10 ** para o ** Runtime **.

1. Certifique-se de que "Escolher uma função existente" esteja selecionado na lista suspensa ** Função **.

1. Selecione `WildRydesLambda` na lista suspensa ** Existing Role **.
    ![Create lambda function screenshot](../images/create-lambda-function.png)

1. Clique em ** Criar função **.

1. Role para baixo até a seção ** Código de função ** e substitua o código existente no editor de código ** index.js ** pelo conteúdo de [requestUnicorn.js] (requestUnicorn.js).
    ![Create lambda function screenshot](../images/create-lambda-function-code.png)

1. Clique em ** "Salvar" ** no canto superior direito da página.

</p> </details>

## Validação de Implementação

Para este módulo, você testará a função criada usando o console do AWS Lambda. No próximo módulo, você adicionará uma API REST com o API Gateway para poder invocar sua função a partir da aplicação web que você implantou no primeiro módulo.

1. Na tela de edição principal da sua função, selecione ** Configurar evento de teste ** no menu suspenso ** Selecionar um evento de teste ... **.
    ![Configure test event](../images/configure-test-event.png)

1. Mantenha ** Criar novo evento de teste ** selecionado.

1. Digite `TestRequestEvent` no campo ** Nome do evento **

1. Copie e cole o seguinte evento de teste no editor:

    ```JSON
    {
    "path": "/ride",
    "httpMethod": "POST",
    "headers": {
        "Accept": "*/*",
        "Authorization": "eyJraWQiOiJLTzRVMWZs",
        "content-type": "application/json; charset=UTF-8"
    },
    "queryStringParameters": null,
    "pathParameters": null,
    "requestContext": {
        "authorizer": {
            "claims": {
                "cognito:username": "the_username"
            }
        }
    },
    "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```

    ![Configure test event](../images/configure-test-event-2.png)

1. Clique em ** Criar **.

1. Na tela de edição da função principal, clique em ** Test ** com `TestRequestEvent` selecionado no menu suspenso.

1. Role para o topo da página e expanda a seção ** Detalhes ** da seção ** Resultado da execução **.

1. Verifique se a execução foi bem-sucedida e se o resultado da função é semelhante ao seguinte:
```JSON
{
    "statusCode": 201,
    "body": "{\"RideId\":\"SvLnijIAtg6inAFUBRT+Fg==\",\"Unicorn\":{\"Name\":\"Rocinante\",\"Color\":\"Yellow\",\"Gender\":\"Female\"},\"Eta\":\"30 seconds\"}",
    "headers": {
        "Access-Control-Allow-Origin": "*"
    }
}
```

Depois de ter testado com sucesso sua nova função usando o console do Lambda, você pode passar para o próximo módulo, ![RESTful APIs](../4_RESTfulAPIs).
