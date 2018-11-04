# Módulo 1: Website estático com Amazon S3

Neste módulo, você configurará o Amazon Simple Storage Service (S3) para hospedar os recursos estáticos do seu aplicativo da web. Nos módulos subseqüentes, você adicionará funcionalidade dinâmica a essas páginas usando JavaScript para chamar APIs RESTful remotas criadas com o AWS Lambda e o Amazon API Gateway.


## Visão geral da arquitetura

A arquitetura deste módulo é muito simples. Todo o seu conteúdo da Web estático, incluindo HTML, CSS, JavaScript, imagens e outros arquivos, será armazenado no Amazon S3. Seus usuários finais acessarão seu site usando o URL do website público exposto pelo Amazon S3. Você não precisa executar nenhum servidor da Web ou usar outros serviços para disponibilizar seu site.
![Static website architecture](../images/static-website-architecture.png)

Para este módulo, você usará a URL do endpoint do site do Amazon S3 que nós fornecemos. Ele assume a forma `http: // {seu-nome-do-bucket} .s3-website- {region} .amazonaws.com` ou` bucket-name.s3-website.region.amazonaws.com` dependendo da região que você usar. Para a maioria das aplicações reais, você desejará usar um domínio personalizado para hospedar seu site. Se você estiver interessado em usar seu próprio domínio, siga as instruções para [configurar um site estático usando um domínio personalizado] (http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html) na documentação do Amazon S3.

## Instruções de implementação

Cada uma das seções a seguir fornece uma visão geral da implementação e instruções detalhadas passo a passo. A visão geral deve fornecer um contexto suficiente para você concluir a implementação, caso já esteja familiarizado com o AWS Management Console ou deseje explorar os serviços sem seguir uma explicação passo a passo.

Se você estiver usando a versão mais recente dos navegadores Chrome, Firefox ou Safari, as instruções passo a passo não ficarão visíveis até você expandir a seção.

### Seleção da região

Este workshop pode ser implantado em qualquer região da AWS que suporte os seguintes serviços:

- Amazon Cognito
- AWS Lambda
- Amazon API Gateway
- Amazon S3
- Amazon DynamoDB

Você pode consultar a [tabela de regiões] (https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) na documentação da AWS para ver quais regiões têm os serviços suportados. Entre as regiões com suporte que você pode escolher  estão N. Virginia ou Oregon.

Depois de escolher uma região, você deve implantar todos os recursos para este workshop lá. Certifique-se de selecionar sua região na lista suspensa no canto superior direito do AWS Console antes de começar.

![Region selection screenshot](../images/region-selection.png)

### 1. Criar um bucket S3

O Amazon S3 pode ser usado para hospedar sites estáticos sem ter que configurar ou gerenciar nenhum servidor web. Nesta etapa, você criará um novo bucket S3 que será usado para hospedar todos os ativos estáticos (por exemplo, HTML, CSS, JavaScript e arquivos de imagem) para sua aplicação Web.

#### Instruções gerais

Use o console ou o AWS CLI para criar um bucket do Amazon S3. Lembre-se de que o nome do seu bucket deve ser globalmente exclusivo em todas as regiões e clientes. Recomendamos usar um nome como `wildrydes-firstname-lastname`. Se você receber um erro informando que seu nome já existe, tente adicionar números ou caracteres adicionais até encontrar um nome não usado.

<details>
<summary><strong>instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. No AWS Management Console, selecione ** Serviços ** e selecione ** S3 ** em Armazenamento.

1. Escolha **+Criar Bucket**

1. Fornecer um nome globalmente exclusivo para o seu bucket, como `wildrydes-firstname-lastname`.

1. Selecione a região que você escolheu para usar neste workshop na lista suspensa.

1. Escolha ** Criar ** no canto inferior esquerdo da caixa de diálogo, sem selecionar um bucket para copiar as configurações.

    ![Create bucket screenshot](../images/create-bucket.png)

</p></details>

### 2. Upload de conteúdo

Faça o upload dos recursos do website para este módulo no seu bucket do S3. Você pode usar o AWS Management Console (requer o navegador Google Chrome) ou AWS CLI  para concluir esta etapa. Se você já tem o AWS CLI instalado e configurado em sua máquina local, recomendamos usar esse método. Caso contrário, use o console se você tiver a versão mais recente do Google Chrome instalada.

<details>
<summary><strong>CLI passo a passo (expand for details)</strong></summary><p>

Se você já tiver a CLI instalada e configurada, poderá usá-la para copiar os recursos da web necessários do `s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website` para o seu bucket.

Execute o seguinte comando, certificando-se de substituir "YOUR_BUCKET_NAME" pelo nome usado na seção anterior e "YOUR_BUCKET_REGION" pelo código de região (por exemplo, us-east-2) onde você criou seu bucket.

     aws s3 sync s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/site s3://YOUR_BUCKET_NAME-região SEU_BUCKET_REGION

Se o comando foi bem-sucedido, você verá uma lista de objetos que foram copiados para o seu bucket.
</p></details>


### 3. Adicionar uma política de bucket para permitir leituras públicas

Você pode definir quem pode acessar o conteúdo em seus buckets do S3 usando uma política de buckets. Políticas de bucket são documentos JSON que especificam quais recursos têm permissão para executar várias ações em relação aos objetos em seu bucket.

#### Instruções gerais

Você precisará adicionar uma política de buckets ao novo bucket do Amazon S3 para permitir que usuários anônimos visualizem seu site. Por padrão, seu bucket só será acessível por usuários autenticados com acesso à sua conta da AWS.

Veja [este exemplo] (http://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) de uma política que concederá apenas o acesso de leitura a usuários anônimos. Esta política de exemplo permite que qualquer pessoa na Internet visualize seu conteúdo. A maneira mais fácil de atualizar uma política de bucket é usar o console. Selecione o bucket, escolha a guia de permissão e selecione Bucket Policy.

<details>
<summary><strong>Instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. No console do S3, selecione o nome do bucket que você criou na seção 1.

1. Escolha a guia ** Permissões ** e, em seguida, escolha ** Política de bucket **.

1. Insira a seguinte política no editor de política de bucket substituindo `YOUR_BUCKET_NAME` pelo nome do bucket criado na seção 1:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
            }
        ]
    }
    ```

    ![Update bucket policy screenshot](../images/update-bucket-policy.png)

1. Escolha ** Salvar ** para aplicar a nova política.

</p></details>

### 4. Habilitando o Website Hosting

Por padrão, os objetos em um bucket do S3 estão disponíveis através de URLs com a estrutura `http: // <Prefixo Regional-S3> .amazonaws.com / <bucket-name> / <object-key>`. Para veicular recursos a partir da URL raiz (por exemplo, /index.html), você precisa ativar a hospedagem de sites no bucket. Para obter mais detalhes sobre como usar esse recurso, consulte a documentação [Site Endpoints] (https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html). Isso disponibilizará seus objetos no ponto final do site específico da região da AWS. Consulte os [Endpoints do site do Amazon Simple Storage Service] (https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_website_region_endpoints) para encontrar o domínio usado para sua região.

Você também pode usar um domínio personalizado para o seu site. Por exemplo, http://www.wildrydes.com está hospedado no S3. A configuração de um domínio personalizado não é abordada neste workshop, mas você pode encontrar instruções detalhadas em nossa [documentação] (http://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html).

#### Instruções Gerais

Usando o console, ative a hospedagem estática do site. Você pode fazer isso na guia Propriedades depois de selecionar o bucket. Defina `index.html` como o documento de índice e deixe o documento de erro em branco. Consulte a documentação sobre [configurando um repositório para hospedagem de sites estáticos] (https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html) para obter mais detalhes.

<details>
<summary><strong>Instruções passo a passo (expanda para detalhes)</strong></summary><p>

1. Na página de detalhes do bucket no console do S3, escolha a guia ** Properties **.

1. Escolha a opção **Static website hosting**.

1. Selecione **Use this bucket to host a website** e insira `index.html` como documento index. Deixe o resto em branco.

1. Tome nota da URL do ** Endpoint ** na parte superior da caixa de diálogo antes de escolher ** Salvar **. Você usará esse URL durante o restante do workshop para visualizar seu aplicativo da web. A partir daqui, este URL será referido como o URL base do seu Web site.

1. Clique em **Salvar**.

    ![Enable website hosting screenshot](../images/enable-website-hosting.png)

</p></details>


## Validação da implementação

Depois de concluir essas etapas de implementação, você poderá acessar seu site estático visitando o URL do endpoint do site para o seu bucket do S3.

Visite a URL base do seu site (este é a URL que você anotou na seção 4) no navegador de sua escolha. Você deve ver a página inicial do Wild Rydes exibida. Se você precisar pesquisar o URL base, visite o console do S3, selecione seu bucket e clique no cartão ** Static Web Hosting ** na guia ** Properties **.

Se a página renderizar corretamente (veja abaixo uma captura de tela de exemplo), você pode passar para o próximo módulo, ![Autenticação de usuários](../2_UserManagement).

![Wild Rydes homepage screenshot](../images/wildrydes-homepage.png)
