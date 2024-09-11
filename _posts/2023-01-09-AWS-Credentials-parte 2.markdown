---
title: "AWS de Dev pra Dev: Credentials e acesso programático - parte 2"
published: true
description: Entendendo como AWS IAM Roles funcionam.
tags: AWS, IAM, Role 
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
---

Essa é a parte 2 dessa série de posts sobre credenciais e acesso programático. [Na parte 1](https://dev.to/hugaomarques/aws-de-dev-pra-dev-credentials-e-acesso-programatico-parte-1-9mg) nós discutimos sobre *IAM Users*, credenciais e como utilizar essas a CLI para invocar APIs do AWS.

Nesse post, vamos explorar sobre **IAM Roles**.

## Glossário

* **IAM**: *IAM (Identity and Access Management)* é um serviço do AWS que permite que os usuários gerenciem as permissões de acesso aos seus recursos AWS. Com o IAM, os usuários podem criar e gerenciar usuários e grupos, e definir políticas de acesso para esses usuários e grupos.
* **IAM User**: *IAM User* é um usuário individual criado e gerenciado no AWS IAM. Um IAM user pode ser associado a uma conta de usuário, ou seja, uma conta de usuário pode ter vários *IAM users*.
* **IAM Group**: *IAM Group* é um conjunto de usuários IAM que compartilham as mesmas permissões de acesso. Ao adicionar um usuário a um grupo, esse usuário herda as permissões do grupo.
* **SDK**: *SDK (Software Development Kit)* é uma coleção de ferramentas de desenvolvimento de software fornecida por fornecedores de serviços e plataformas, como o AWS, para permitir que os desenvolvedores interajam com os serviços de uma forma programática. Existem *SDKs* disponíveis para várias linguagens de programação, como Java, Python e C#.
* **IaC**: IaC (Infrastructure as Code) é uma técnica para gerenciar a infraestrutura de TI de uma organização como código, usando ferramentas de automação como o *AWS CloudFormation* ou *Terraform*. Isso permite que os usuários provisionem, gerenciem e documentem sua infraestrutura de forma automatizada e consistente, e facilita a implantação de mudanças e a reprodução de configurações.
* **ARN**: *AWS ARN (Amazon Resource Name)* é uma string única que identifica de forma segura e consistente recursos dentro do Amazon Web Services (AWS). Ele é composto por vários elementos, como o serviço, a região, a conta e o nome do recurso, e é usado para permitir acesso a recursos específicos na sua conta AWS. Por exemplo, um ARN de função AWS Lambda pode ser usado para permitir que outro recurso da AWS, como uma tabela do Amazon DynamoDB, acesse a função.

## 1. O que é um *IAM Role*?

O *IAM Role* (vou chamar apenas de **Role**) é um tipo de identidade fornecida pelo serviço *AWS IAM*. A principal diferença entre o *Role* e o *User* é que o primeiro pode ser utilizado por qualquer usuário, aplicação ou recurso que precise utilizá-lo, já o segundo é associado a um usuário ou aplicação.

## 2. Quando eu devo usar um **IAM Role**?

**Roles** são bem úteis em situações específicas: 
* Dar acesso a recursos rodando DENTRO do AWS para invocar serviços do AWS.
* Dar acesso a recursos rodando FORA do AWS para invocar serviços do AWS.
* Garantir acesso a serviços do AWS para executar ações em sua conta.
* Dar acesso a outra conta do AWS para executar ações em sua conta.

## 3. Como eu crio um *Role*?

Primeiro, você tem que criar um *Role* no *IAM*. 
Normalmente, como uma pessoa dev, você vai utilizar a *SDK* ou utilizar *IaC* para criar o seu *Role*, por exemplo, utilizando o *AWS CloudFormation*:

```YAML
Resources:

  LambdaRole:
    Type: "AWS::IAM::Role"
    Properties:
      RoleName: LambdaRole
      Description: "Role used to grant permissions to an AWS Lambda function"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: "lambda.amazonaws.com"
            Action: "sts:AssumeRole"
          - Effect: Allow
            Principal: 
              AWS: 
                Ref: AWS::AccountId
            Action: "sts:AssumeRole"
      Policies:
        - PolicyName: WriteToS3Policy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action: 
                  - "s3:PutObject"
                  - "s3:ListAllMyBuckets"
                Resource: "arn:aws:s3:::*"
```

O *Role* acima tem duas partes principais:
* **AssumeRolePolicyDocument**: Esse bloco define quais principals do AWS podem assumir esse role.
* **Policies**: Nesse bloco nós definimos quais ações o *Role* tem permissão para executar. No nosso caso, o role tem permissão para executar as ações "s3:putObject" e "s3:ListBucket" em qualquer *bucket* do AWS.

Quer fazer um teste? Copie o código acima para um arquivo `.yaml` e execute o seguinte comando:

> Nota: O comando abaixo requer que sua CLI esteja configurada. Dê uma olhada [na parte 1](https://dev.to/hugaomarques/aws-de-dev-pra-dev-credentials-e-acesso-programatico-parte-1-9mg) sobre como usar a CLI.

```
aws cloudformation create-stack --stack-name example --template-body file://<path para o seu aquivo.yaml> --capabilities CAPABILITY_NAMED_IAM
```

Você pode verificar que o recurso foi criado utilizando o seguinte comando do *CloudFormation*

```
 aws cloudformation list-stack-resources --stack-name example
```

Você deve conseguir ver algo com o `"ResourceStatus": "CREATE_COMPLETE"`.

E finalmente, você pode conferir o resultado da execução do template acima usando a CLI pra pegar detalhes sobre o **Role**:

```
aws iam get-role --role-name LambdaRole
```

Observe como o trecho `Principal: AWS: Ref: AWS::AccountId` foi substituído por sua conta. Isso significa que qualquer usuário daquela conta pode assumir aquele **Role**.

## 4. Como eu uso um *Role*?

Com o **Role** criado agora nós podemos utilizá-lo. As formas mais comuns são:
* **Assumir um Role**: Você pode usar a função *AssumeRole* da *AWS STS (Security Token Service)* para assumir um *Role* em outra conta de usuário ou na sua própria conta. 
* **Atribuir um Role a uma instância**: você pode atribuir um papel a uma instância do *EC2 (Elastic Compute Cloud)*. Isso permite que a instância acesse recursos usando as permissões do *Role*.
* **Atribuir um *Role* a um serviço**: alguns serviços do AWS, como o Lambda, suportam a atribuição de um *Role* ao serviço. Isso permite que o serviço acesse recursos usando as permissões do *Role*.
* Usar um Role com o *SDK*: Você pode usar um papel com o *SDK* do AWS para acessar recursos usando as permissões do *Role*. Isso geralmente envolve a configuração de credenciais de acesso temporárias usando a função *AssumeRole*.

## 5. Assumindo um *Role* com a CLI

Vamos fazer um experimento, usando um usuário que não tenha permissões para o s3, execute o seguinte comando:

```
aws s3 ls
```

Observe como a saída do comando foi o seguinte erro:
```
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```

Agora, vamos assumir o *Role* usando um *User* ou *Role* que você tenha criado que possua permissão pra ação *AssumeRole*. Qualquer *User* ou **Role** funciona, desde que tenha a seguinte permissão:

```
AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Condition:
          - Effect: Allow
            Principal:
              AWS:
                Ref: AWS::AccountId
            Action: "sts:AssumeRole"
            Resource:
              "arn:aws:iam::ACCOUNT_ID:role/LambdaRole"
```

Calma, não se assuste. Vamos ver o que essa permissão faz. Ela é muito similar às permissões que demos quando criamos o *LambdaRole*. Nós estamos dando permissão a qualquer usuário da nossa conta (*Ref: AWS::AccountId*) à ação *sts:AssumeRole*, ao recurso representado pelo *AWS ARN* "arn:aws:iam::ACCOUNT_ID:role/LambdaRole".

Lembre-se que a *policy* precisa ser associado a um usuário ou outro Role (sim, *Roles* podem assumir Roles). Fica o dever de casa de você criar esse usuário (a parte 1 pode te ajudar com isso).

Antes de continuar, execute o seguinte comando e anote a identidade que você está utilizando no momento.
```
aws sts get-caller-identity
```

Agora que você tem um usuário com permissão de *AssumeRole* vamos usar esse usuário para executar o seguinte comando:
```
aws sts assume-role --role-arn arn:aws:iam::<YOUR_ACCOUNT_ID>:role/LambdaRole --role-session-name my-session
```

Você vai receber um retorno com as seguintes variáveis:
```
AccessKeyId
SecretAccessKey
SessionToken
```

Copie esses valores e execute o comando abaixo:
```
export AWS_ACCESS_KEY_ID=<AccessKeyId> && 
export AWS_SECRET_ACCESS_KEY=<SecretAccessKey> && 
export AWS_SESSION_TOKEN=<SessionToken>
```

Com essas variáveis setadas, vamos verificar que nós agora estamos usando a identidade do nosso role. Execute novamente o comando:
```
aws sts get-caller-identity
```

Observe como o seu *ARN* mudou. Ele agora deve ser algo do tipo:
```
arn:aws:sts::<YOUR_ACCOUNT_ID>:assumed-role/LambdaRole/my-session
```

Agora que nós estamos usando a identidade do nosso **Lambda Role** nós podemos finalmente re-executar nosso comando:
```
aws s3 ls
```

E o resultado, deve ser uma lista com todos os *buckets* que você tem na sua conta. Se não listou nada, vá lá na sua conta e crie uns *buckets* só pra brincar.

## 6. Usando *IAM Roles* no acesso entre contas (*cross-account*)

Outra coisa legal que podemos fazer com *Roles* é dar acesso a outras contas. O princípio é o mesmo, no seu *Role* você precisa declarar a *TrustedPolicy* apontando as identidades na outra conta que podem assumir o seu *Role*. 
Na sua outra conta, lembre-se que você precisa dar permissão à ação *assumeRole*.

Um exemplo usando *CloudFormation*. Na sua conta "origem" você pode declarar:
```YAML
Resources:
  TargetRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              AWS:
                - "arn:aws:iam::DESTINO_ACCOUNT_ID:root"
            Action: "sts:AssumeRole"
      Policies:
        - <QUALQUER POLICY QUE VOCÊ DESEJAR, POR EXEMPLO, A NOSSA POLICY DE ACESSO AO S3>
```

E finalmente, na outra conta, você pode fazer:
```YAML
Resources:
  SourceRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Condition:
          - Effect: Allow
            Principal:
              AWS:
                - "arn:aws:iam::DESTINO_ACCOUNT_ID:root"
            Action: "sts:AssumeRole"
            Resource: "arn:aws:iam::ORIGEM_ACCOUNT_ID:role/TARGET_ROLE_NAME"
```

Preste atenção nessa última parte. Note o trecho:
```
Principal:
    AWS:
        - "arn:aws:iam::DESTINO_ACCOUNT_ID:root"
```
Veja como ali usamos a conta "Destino", ou seja, a conta que vai assumir o **Role**. Já nesse outro trecho:
```
Resource: "arn:aws:iam::ORIGEM_ACCOUNT_ID:role/TARGET_ROLE_NAME"
```
Nós usamos a conta "origem", ou seja, nós estamos dizendo que a conta "Destino" pode assumir o role na conta "Origem". 

Importante, lembre-se que isso só funciona se na conta "origem" nós demos permissão para a conta "destino" também. Um pouco confuso eu sei. Eu gosto de pensar que é um aperto de mãos, como o estabelecimento de uma conexão. A conta origem precisa aceitar a conta destino, e a conta destino precisa ter permissão para executar a ação na conta origem. Sem a permissão nos dois lados, o nosso "aperto de mãos" não funciona.

## Conclusão

Existem muitos detalhes sobre o uso de *Roles*. Eu passei longe de cobrir todos eles. Os aspectos mais importantes e que eu quero que fiquem com você são:
* **Roles são bem mais flexíveis** do que *Users* e na MINHA experiência, eu só trabalho com *Roles*.
* Existem diversos casos de uso pra *Roles*, os mais comuns são dar **permissões a serviços do AWS, acesso a outras contas e acesso a usuários**.
* Quando você assume o *Role* você **tem as mesmas permissões** que o *Role* tem.
* Finalmente, pra assumir um *Role*, o *Role* origem precisa ter uma *TrustedPolicy* que dá permissão a identidade destino e a identidade destino precisa ter permissão de executar a ação *assumeRole*.

Eu sei que foi muita informação, mas eu espero que esse post dê uma facilitada na sua jornada de entender o *AWS*. Se ficou alguma dúvida, me pergunta aí nos comentários.

Muito obrigado pela leitura 💙
Gostou? Me segue aqui no dev.to e no (BlueSky)[https://bsky.app/profile/hugomarques.dev]

## Quer aprender mais?

Dá uma assistida nessa talk da fantástica Becky Weiss:

<iframe width="560" height="315" src="https://www.youtube.com/embed/Zvz-qYYhvMk?si=eo6-rQ8s8Rrdq0YQ&amp;controls=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

"Happy Coding!" 🥰