---
title: AWS de dev pra dev: Credentials e acesso programático - parte 1
published: true
description: Aprendar a usar credenciais na sua máquina e interagir com o AWS usando a AWS CLI.
tags: cli, aws, devops
---

Inicialmente, a idéia desse post era abordar conceitos sobre AWS IAM mas eu acabei desistindo porque essa informação tem na documentação. Além disso, o AWS IAM é tão vasto que o post ia ser infinito.

Dessa forma, a proposta aqui é diferente. Eu vou tentar responder algumas perguntas comuns que eu recebo/vejo sobre IAM. Como tem muita coisa, eu vou dividir esse post em partes. Nessa primeira parte eu irei focar em **credentials** de **IAM Users** e acesso programático usando a **AWS CLI**. Vamos lá?

## Pre-requisitos
* Instale a [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [Possuir uma conta na AWS](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)
* [Criar um ou mais **IAM User** no **AWS IAM**](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html)

## 1. Onde eu consigo minhas **credentials**?

Sempre que um **IAM user** é criado, é dada a opção para acesso programático. Se a opção for selecionado você vai receber uma **access_key** e uma **secret_key**.
Essas são as suas credentials para acesso programático (SDK, CLI).

## 2. Como eu posso configurar minhas credentials?

As credentials podem ser configuradas de diversas formas. Os usos mais comuns são:
* AWS CLI
* arquivo (manual ou gerado pela CLI)
* variável de ambiente

### AWS CLI

Você pode usar o comando `aws configure` para fornecer as suas credentials. O output seria algo do tipo:

```sh
$ aws configure
AWS Access Key ID [****************6TEK]: AIDAVGDKXECDYBS76YKXE
AWS Secret Access Key [****************eRkE]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-east-1
Default output format [None]: json
```

Note os arquivos que são gerado na máquina:

```sh
ls ~/.aws
config credentials
```

E se olharmos dentros desses arquivos:

```sh
$ cat ~/.aws/config
[default]
region = us-east-1
output = json

cat ~/.aws/credentials
[default]
aws_access_key_id = sdsada
aws_secret_access_key = sadsadasdsad
```

Perceba como o output segue os dados que fornecemos no comando `aws configure`.

### Arquivo

Os arquivos acima também pode ser criados e editados manualmente. Você pode simplesmente fazer:

```
$ vim ~/.aws/credentials
```

E sair editando o seu arquivo. Só tenha certeza que você sabe sair do VIM antes de fazer isso 😋.

### Variável de ambiente

Você pode dar override nos arquivos acima usando variáveis de ambiente:

```sh
$ export AWS_ACCESS_KEY_ID=AIDAVGDKXECDYBS76YKXE
$ export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
$ export AWS_DEFAULT_REGION=us-west-2 
```

Inclusive, um erro comum é a pessoa ter a variável de ambiente setada e por isso a CLI ou a SDK não lê a credential do arquivo. 

Note que se você cometer algum typo, a CLI vai continuar pegando as credentials do arquivo. Logo, se você setou a variável de ambiente e mesmo assim ela não tá funcionando, verifique se você não cometeu algum erro de **typo**.

## 3. Como eu sei qual **User** eu estou usando?

Uma forma rápida de verificar isso é fazer a chamada:

```
$ aws sts get-caller-identity
{
    "UserId": "AIDAVGDKXECDYBS76YKXE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/hugo-readonly-1"
}
```

## 4. Ok, agora que eu tenho credentials e agora? Como eu posso invocar as APIs do AWS?

Agora você pode executar chamadas ao AWS através da **AWS CLI** ou utilizando uma das várias SDKs que o AWS provê. 
Pra demonstrar isso, vamos utilizar uma das APIs do *DynamoDB*. O *AWS DynamoDB* é um banco de dados NoSQL do AWS. Que tal listarmos as tabelas do nosso banco de dados? Se tentarmos usar o comando  `aws dynamodb help` ele lista a documentação e as APIs que podemos usar. A CLI também lista as opções possível se fornecemos um comando errado. Por exemplo:

```sh 
$ aws dynamodb list

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

aws: error: argument operation: Invalid choice, valid choices are:

batch-execute-statement                  | batch-get-item
batch-write-item                         | create-backup
create-global-table                      | create-table
delete-backup                            | delete-item
delete-table                             | describe-backup
describe-continuous-backups              | describe-contributor-insights
describe-endpoints                       | describe-export
describe-global-table                    | describe-global-table-settings
describe-kinesis-streaming-destination   | describe-limits
describe-table                           | describe-table-replica-auto-scaling
describe-time-to-live                    | disable-kinesis-streaming-destination
enable-kinesis-streaming-destination     | execute-statement
execute-transaction                      | export-table-to-point-in-time
get-item                                 | list-backups
list-contributor-insights                | list-exports
list-global-tables                       | list-tables
list-tags-of-resource                    | put-item
query                                    | restore-table-from-backup
restore-table-to-point-in-time           | scan
tag-resource                             | transact-get-items
transact-write-items                     | untag-resource
update-continuous-backups                | update-contributor-insights
update-global-table                      | update-global-table-settings
update-item                              | update-table
update-table-replica-auto-scaling        | update-time-to-live
wizard                                   | wait
help
```

Observe como existem várias APIs `list-`. Beleza, agora que temos permissões para listar, vamos chamar a API `list-tables`? Eu criei uma tabela só pra demonstrar o uso da CLI:

```sh
$aws dynamodb list-tables

An error occurred (AccessDeniedException) when calling the ListTables operation: User: arn:aws:iam::356706099335:user/hugo-readonly-1 is not authorized to perform: dynamodb:ListTables on resource: arn:aws:dynamodb:us-east-1:123456789012:table/* because no identity-based policy allows the dynamodb:ListTables action
```

Opa, deu erro de permissão... e agora? 😥

## 5. Eu não tenho permissão pra invocar a API que eu quero. E agora?

Lembre-se que suas chamadas são restritas às permissões que seu **User** possui. Durante a criação do seu **User** é possível aplicar **policies** com permissões ou adicionar o **User** a um grupo que possui algumas permissões.

> Boa prática: Crie grupos para seus **IAM Users**. Adicione as policies para esses grupos e associe seus usuários aos grupos. Evite associar policies aos usuários diretamente.

Para detalhes de como adicionar uma policy ao usuário você pode seguir a [documentação oficial](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_manage-attach-detach.html).

Para simplificar, eu vou adicionar as permissões diretamente ao usuário mas como boa prática você deveria criar um grupo (devs, por exemplo) e associar seu usuário ao grupo.
Eu adicionei ao meu usuário a **policy** existente do AWS chamada **AmazonDynamoDBReadOnlyAccess**. Essa **policy** me dá acesso a várias APIs, incluindo as seguintes APIs do **DynamoDB**:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                ...
                "dynamodb:BatchGetItem",
                "dynamodb:Describe*",
                "dynamodb:List*",
                "dynamodb:GetItem",
                "dynamodb:Query",
                "dynamodb:Scan",
                "dynamodb:PartiQLSelect",
                ...
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        ...
    ]
}
```

Notem como a policy me dá direito a chamar a API `DynamoDB:List*`. Ou seja, todas as APIs do DynamoDB que começam com `List`. Agora que temos a permissão, podemos finalmente chamar a API:

```sh
 aws dynamodb list-tables
{
    "TableNames": [
        "hugo_teste"
    ]
}
```

Bacana né? Já que temos o nome da tabela e se quisermos mais informações sobre ela? Olhando a lista de APIs ali em cima, nós vemos que temos permissão para fazer um `describe-table`. 

```
$ aws dynamodb describe-table hugo_teste

usage: aws [options] <command> <subcommand> [<subcommand> ...] [parameters]
To see help text, you can run:

  aws help
  aws <command> help
  aws <command> <subcommand> help

aws: error: the following arguments are required: --table-name
```

Opa, deu erro! Faltou o argumento `--table-name`. 

```
$ aws dynamodb describe-table --table-name hugo_teste
{
    "Table": {
        "AttributeDefinitions": [
            {
                "AttributeName": "my_partition_key",
                "AttributeType": "S"
            }
        ],
        "TableName": "hugo_teste",
        "KeySchema": [
            {
                "AttributeName": "my_partition_key",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2022-08-09T16:27:24.046000-07:00",
        "ProvisionedThroughput": {
            "LastDecreaseDateTime": "2022-08-09T16:38:24.524000-07:00",
            "NumberOfDecreasesToday": 2,
            "ReadCapacityUnits": 1,
            "WriteCapacityUnits": 1
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:us-east-1:356706099335:table/hugo_teste",
        "TableId": "da75a055-61b0-479d-94e5-7c8dc2da806b"
    }
}
```

# 6. Mas eu tenho 2 Users, 1 pra Dev e outro pra QA. Como manter 2 usuários na mesma máquina?

Você pode usar as variáveis de ambiente pra fazer override como a gente discutiu antes ou você pode usar **profiles**.

> Boa prática: Use profiles para manter acesso a diversos **IAM Users** diferentes na sua máquina.

Por exemplo, eu vou conferir 2 perfis no meu arquivo `~/.aws/credentials` e `~/.aws/config`.

```
$ cat ~/.aws/credentials
[default]
aws_access_key_id = AIDAVGDKXECDYBS76YKXE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[qa]
aws_access_key_id = AKIAVGDKXECDVL33JTNZ
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYSAMPLEKEY

$ cat ~/.aws/config
[default]
region = us-east-1
output = json
cli_pager=

[profile qa]
region = us-east-1
output = json
cli_pager=

```

Note a diferença sútil na sintaxe do arquivo. O arquivo `config` usa o prefixo `profile` antes do nome `QA`.

Agora, eu tenho a opção de invocar a CLI usando o **profile default** (sem argumentos):

```
aws sts get-caller-identity
{
    "UserId": "AIDAVGDKXECDYBS76YKXE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/hugo-readonly-1"
}
```

Ou eu posso invocar a CLI passando o profile que eu desejo assumir:

```
$ aws sts get-caller-identity --profile qa
{
    "UserId": "AKIAVGDKXECDVL33JTNZ",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/hugo-readonly-2"
}
```

A AWS CLI é bem útil pra fazer scripts ou pra testar algo rápido. Por exemplo, eu usei bastante ela no passado para submeter **AWS Cloudformation** templates pro AWS criar todos os meus recursos durante desenvolvimento/testes.

## Conclusão

Vamos fechar a parte 1 por aqui. Recapitulando, até agora nós vimos: 
* Como configurar suas credenciais
* Como verificar com qual identity você está realizando suas chamadas
* Como as permissões te permitem executar chamadas ao AWS de forma programática
* (bônus) Como usar a AWS CLI para interagir com as APIs do AWS programaticamente
* Utilizar diferentes profiles para armazenar **credentials** para diferentes **IAM Users**.

Nos próximos posts nós vamos discutir mais sobre **IAM Roles** e acesso programático usando SDKs e **credential providers**.

Curtiu? Dá o seu like 💙 e deixa perguntas que eu vou respondendo. 
