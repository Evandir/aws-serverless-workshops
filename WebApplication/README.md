# Workshop Serverless Web Application 

Neste workshop, você implantará uma aplicação Web simples que permite aos usuários solicitar passeios de unicórnio [Wild Rydes] (http://www.wildrydes.com/). A aplicação apresentará aos usuários uma interface de usuário baseada em HTML para indicar o local onde eles gostariam de retirar e fará a interface no back-end com um serviço Web RESTful para enviar a solicitação e despachar um unicórnio próximo. A aplicação também fornecerá recursos para que os usuários se registrem no serviço e efetuem login antes de solicitar passeios.

A arquitetura da aaplicação usa o [AWS Lambda] (https://aws.amazon.com/lambda/), [Amazon API Gateway] (https://aws.amazon.com/api-gateway/), [Amazon S3] ( https://aws.amazon.com/s3/), [Amazon DynamoDB] e https://aws.amazon.com/dynamodb/ /). O S3 hospeda recursos da Web estáticos, incluindo HTML, CSS, JavaScript e arquivos de imagem que são carregados no navegador do usuário. O JavaScript executado no navegador envia e recebe dados de uma API pública de back-end construída usando o Lambda e o API Gateway. O Amazon Cognito fornece funções de gerenciamento e autenticação de usuários para proteger a API de back-end. Finalmente, o DynamoDB fornece uma camada de persistência onde os dados podem ser armazenados pela função Lambda da API.

Veja o diagrama abaixo para uma representação da arquitetura completa.

![Wild Rydes Web Application Architecture](images/wildrydes-complete-architecture.png)

Se você quiser entrar e começar, visite a página do módulo [Static Web hosting] (1_StaticWebHosting) para iniciar o workshop.

## Pré-requisitos

### Conta AWS

Para concluir este workshop, você precisará de uma conta da AWS com acesso para criar recursos do AWS IAM, S3, DynamoDB, Lambda, Gateway de API e Cognito. O código e as instruções deste workshop supõem que apenas um aluno esteja usando uma determinada conta da AWS por vez. Se você tentar compartilhar uma conta com outro aluno, encontrará conflitos de nomenclatura para determinados recursos. Você pode contornar isso anexando um único sufixo aos recursos que não foram criados devido a conflitos, mas as instruções não fornecem detalhes sobre as alterações necessárias para que isso funcione.

Todos os recursos que você lançará como parte deste workshop estarão qualificados para o nível gratuito da AWS se sua conta tiver menos de 12 meses. Consulte a [página do nível gratuito da AWS] (https://aws.amazon.com/free/) para obter mais detalhes.

### Browser

Recomendamos que você use a versão mais recente do Chrome para concluir este workshop.

### Editor de Texto

Você precisará de um editor de texto local para fazer pequenas atualizações nos arquivos de configuração.

## Módulos

Este workshop está dividido em vários módulos. Você deve concluir cada módulo antes de prosseguir para o próximo.

1. [Website estático](1_StaticWebHosting)
2. [Autenticação de usuários](2_UserManagement)
3. [Serverless Backend](3_ServerlessBackend)
4. [RESTful APIs](4_RESTfulAPIs)

Depois de concluir o workshop, você pode excluir todos os recursos criados seguindo o [cleanup guide] (9_CleanUp).
