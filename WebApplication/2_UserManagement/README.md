# Módulo 2: Autenticação do usuário e registro com Amazon Cognito User Pools

Neste módulo, você criará um pool de usuários do Amazon Cognito para gerenciar as contas de seus usuários. Você implantará páginas que permitem que os clientes se registrem como novos usuários, verifiquem seus endereços de e-mail e façam login no site.

## Visão geral de arquitetura

Quando os usuários visitam seu site, eles primeiro registram uma nova conta de usuário. Para este workshop, apenas exigiremos que eles forneçam um endereço de e-mail e uma senha para se registrarem. No entanto, você pode configurar o Amazon Cognito para exigir atributos adicionais em seus próprios aplicativos.

Depois que os usuários enviarem seu registro, o Amazon Cognito enviará um email de confirmação com um código de verificação para o endereço fornecido. Para confirmar a conta, os usuários retornarão ao seu site e digitarão o endereço de e-mail e o código de verificação que receberam. Você também pode confirmar as contas de usuário usando o console do Amazon Cognito se desejar usar endereços de e-mail falsos para testes.

Depois que os usuários tiverem uma conta confirmada (usando o processo de verificação por e-mail ou uma confirmação manual por meio do console), eles poderão entrar. Quando os usuários entrarem, eles digitarão seu nome de usuário (ou e-mail) e senha. Uma função JavaScript, em seguida, se comunica com o Amazon Cognito, autentica usando o protocolo Secure Remote Password (SRP) e recebe de volta um conjunto de JSON Web Tokens (JWT). Os JWTs contêm declarações sobre a identidade do usuário e serão usados ​​no próximo módulo para autenticação em relação à API RESTful que você cria com o Amazon API Gateway

![Authentication architecture](../images/authentication-architecture.png)

## Instruções gerais

Cada uma das seções a seguir fornece uma visão geral da implementação e instruções detalhadas e passo a passo. A visão geral deve fornecer um contexto suficiente para você concluir a implementação, caso já esteja familiarizado com o AWS Management Console ou deseje explorar os serviços sem seguir uma explicação passo a passo.

Se você estiver usando a versão mais recente dos navegadores Chrome, Firefox ou Safari, as instruções passo a passo não ficarão visíveis até você expandir a seção.

### 1. Crie uma Amazon Cognito User Pool

#### Contexto

O Amazon Cognito fornece dois mecanismos diferentes para autenticar usuários. Você pode usar os Pools de usuários do Cognito para adicionar a funcionalidade de inscrição e login ao seu aplicativo ou usar os pools de identidade do Cognito para autenticar usuários por meio de provedores de identidade social como Facebook, Twitter ou Amazon, com soluções de identidade SAML ou usando seus próprios sistema de identidade. Para este módulo, você usará um pool de usuários como o back-end para as páginas de registro e de login fornecidas.

#### Instruções gerais

Use o console do Amazon Cognito para criar um novo pool de usuários usando as configurações padrão. Depois que seu pool for criado, observe o ID do Pool. Você usará esse valor em uma seção posterior.

<details>
<summary><strong>Instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. No AWS Console, clique em ** Serviços ** e selecione ** Cognito ** em Segurança, identidade e conformidade.

1. Selecione **Manage your User Pools**.

1. Selecione **Create a User Pool**

1. Insira um nome, como `WildRydes`, e depois selecione **Review Defaults**

    ![Create a user pool screenshot](../images/create-a-user-pool.png)

1. Na página de review, clique em **Create pool**.

1. Tome nota do ** ID do pool ** na página de detalhes do pool de seu pool de usuários recém-criado.

</p></details>

### 2. Adicione um App Client ao seu User Pool
No console do Amazon Cognito, selecione seu pool de usuários e, em seguida, selecione a seção ** App clients **. Adicione um novo aplicativo e verifique se a opção Gerar segredo do cliente está desmarcada. Os segredos do cliente não são compatíveis com o JavaScript SDK. Se você criar um aplicativo com um segredo gerado, exclua-o e crie um novo com a configuração correta.

<details>
<summary><strong>Instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. Na página Detalhes do pool do seu pool de usuários, selecione ** App Clients ** na seção ** Configurações gerais ** na barra de navegação à esquerda.

1. Selecione **Add an app client**.

1. Dê um nome, assim como `WildRydesWebApp`.

1. **Uncheck** a opção Gerar segredo do cliente. Client secrets não são suportados para uso com aplicativos baseados em navegador.

1. Selecione **Create app client**.

   <kbd>![Create app client screenshot](../images/add-app.png)</kbd>

1. Tome nota do **App client id** para a nova aplicação.

</p></details>

### 3. Atualize o config.js no seu bucket

O arquivo [/js/config.js](../1_StaticWebHosting/website/js/config.js) contém configurações para o ID do conjunto de usuários, o ID do cliente do aplicativo e a Região. Atualize este arquivo com as configurações do pool de usuários e do aplicativo que você criou nas etapas anteriores e faça o upload do arquivo de volta ao seu bucket.

<details>
<summary><strong>Instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. Faça o download do arquivo [config.js] (../1_StaticWebHosting/website/js/config.js) do diretório de sites do primeiro módulo neste repositório para sua máquina local.

1. Abra o arquivo baixado usando o editor de texto de sua escolha.

1. Atualize a seção `cognito` com os valores corretos para o conjunto de usuários e o aplicativo que você acabou de criar.

    Você pode encontrar o valor para `userPoolId` na página Detalhes do pool do console do Amazon Cognito depois de selecionar o pool de usuários que você criou.

    ![Pool ID](../images/pool-id.png)

    Você pode encontrar o valor para `userPoolClientId` selecionando ** App clients ** na barra de navegação à esquerda. Use o valor do campo ** ID do cliente do aplicativo ** para o aplicativo que você criou na seção anterior.

    ![Pool ID](../images/client-id.png)

    O valor de `region` deve ser o código da região da AWS em que você criou seu pool de usuários. Por exemplo. `us-east-1` para a região de N. Virginia, ou` us-west-2` para a região de Oregon. Se você não tiver certeza de qual código usar, poderá ver o valor do Pool ARN na página de detalhes do Pool. O código da região é a parte do ARN imediatamente após `arn: aws: cognito-idp:`.

    O arquivo config.js atualizado deve se parecer com isso. Observe que os valores reais do seu arquivo serão diferentes:
    ```JavaScript
    window._config = {
        cognito: {
            userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
            userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
            region: 'us-west-2' // e.g. us-east-2
        },
        api: {
            invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
        }
    };
    ```

1. Salve o arquivo modificado, certificando-se de que o nome do arquivo ainda esteja `config.js`.

1. Abra o console do Amazon S3 visitando [https://console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/).

1. Selecione o bucket do site do Wild Rydes que você criou no módulo anterior.

1. Abra a pasta `js`.

1. Escolha **Upload**, então **Add Files**.

1. Navegue até o diretório em que você salvou a versão modificada localmente do arquivo config.js, selecione-a e escolha **Open**.

    ![s3-upload.png](../images/s3-upload.png)

1. Escolha **Upload** na parte esquerda do diálogo.

</p></details>

## Validação da implementação

1. Visite `/register.html` no domínio do seu website ou escolha o botão ** Giddy Up! ** na página inicial do seu site.

1. Preencha o formulário de inscrição e escolha ** Let's Ryde **. Você pode usar seu próprio e-mail. Certifique-se de escolher uma senha que contenha pelo menos uma letra maiúscula, um número e um caractere especial. Não esqueça a senha que você digitou para mais tarde. Você deve ver um alerta que confirme que seu usuário foi criado.

  1. Se você usou um endereço de e-mail controlado por você, pode concluir o processo de verificação da conta acessando `/verify.html` no domínio do seu website e inserindo o código de verificação que lhe foi enviado por e-mail. Por favor, note que o email de verificação pode acabar na sua pasta de spam. Para implantações reais, recomendamos [configurar seu pool de usuários para usar o Amazon Simple Email Service] (http://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pool-settings-message-customizations.html# cognito-user-pool-settings-ses-autorização-para-enviar-e-mail) para enviar e-mails de um domínio que você possui.

1. Depois de confirmar o novo usuário usando a página `/verify.html` ou o console do Cognito, visite o arquivo` /signin.html` e efetue login usando o endereço de e-mail e a senha digitados durante a etapa de registro.

1. Se tiver sucesso, você deve ser redirecionado para `/ride.html`. Você deve ver uma notificação de que a API não está configurada.

    ![Successful login screenshot](../images/successful-login.png)

Depois de ter logado com sucesso em sua aplicação web, você pode prosseguir para o próximo módulo, ![Serverless Backend(../3_ServerlessBackend).

### Extra

* Tente copiar o ** auth_token ** que você recebeu e cole em um [Decodificador JWT on-line] (https://jwt.io/) para entender o significado desse token para seu aplicativo

