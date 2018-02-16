# Como centralizar os logs do CloudTrail em unica conta 

## O Objetivo é centralizar os logs de todas as contas da AWS em uma unica conta, sendo assim, os LOGs estarao disponiveis em um unico Bucket S3.

###  Requisitos 

- Acesso a todas as contas envolvidas


### Passos

* 1 Ativar o CloudTrail na conta princpal, onde sera centralizado os logs
* 2 Atualizar a política no bucket de destino para conceder ao CloudTrail permissões entre contas.
* 3 Ative o CloudTrail nas outras contas.

Passo 1 


Faça login em https://console.aws.amazon.com/cloudtrail/.

Escolha a região em que você deseja que a trilha seja criada.

Escolha Get Started Now.

Na página Create Trail, em Trail name, digite um nome para a sua trilha.

PRINT 01

Em Apply trail to all regions, escolha Yes para receber os arquivos de log de todas as regiões. Essa é a configuração padrão recomendada. Se você escolher No, a trilha registrará arquivos somente da região em que você a criou.

PRINT 01.1


Em Management event, Read/Write events, escolha se você deseja que sua trilha registre All, Read-only, Write-only ou None e, em seguida, escolha Salve. Por padrão, as trilhas registram todos os eventos de gerenciamento.

PRINT 01.2
PRINT 01.3

PASSO 2

Neste passo vamos atualizar a política no bucket para conceder ao CloudTrail permissões entre contas.

No console da AWS abra o servico Amazon S3

PRINT 02

Escolha o Bucket criado (Neste Exemplo: center-logs-full-accounts), Clique em Permissions depois em  Bucket Police

PRINT 02.2

Vamos modificar a política existente para adicionar uma linha para cada conta adicional cujos arquivos de log devem ser fornecidos a esse bucket. Veja o exemplo de política a seguir e observe a linha Resource onde foi especificando o ID 22222222222 de uma segunda conta.

```yaml 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AWSCloudTrailAclCheck20131101",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:GetBucketAcl",
      "Resource": "arn:aws:s3:::center-logs-full-accounts"
    },
    {
      "Sid": "AWSCloudTrailWrite20131101",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudtrail.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": [
        "arn:aws:s3:::center-logs-full-accounts/AWSLogs/401188835562/*",
        "arn:aws:s3:::center-logs-full-accounts/AWSLogs/222222222222/*"
      ],
      "Condition": { 
        "StringEquals": { 
          "s3:x-amz-acl": "bucket-owner-full-control" 
        }
      }
    }
  ]

```
_Neste exemplo, em ( "Resource": ) foi adicionado a segunda conta:_ 

"arn:aws:s3:::center-logs-full-accounts/AWSLogs/222222222222/*", com o numero ficticio da conta "222222222222", a cada nova conta deve-se adicionar uma abaixdo da outra conforme o exemplo acima.

Print de Exemplo:

PRINT 02.3


Apos as configuracoes aplicadas, clique em SAVE.

PASSO 3 

Neste passo vamos configurar as contas adicionais para enviar logs a conta principal.

como exemplo vou assumir que o ID da conta adicional é 222222222222:


Faça login no console de gerenciamento da AWS usando as credenciais da conta 222222222222 e abra o console do AWS CloudTrail. Na barra de navegação, selecione a região em que você deseja ativar o CloudTrail.

Escolha Get Started Now.

Na página seguinte, digite um nome para a sua trilha na caixa Trail name.

Em Create a new S3 bucket?, escolha NO. Use a caixa de texto para informar o nome do bucket que você criou anteriormente para o armazenamento de arquivos de log quando fez login usando as credenciais da conta 111111111111. O CloudTrail exibe um aviso perguntando se você tem certeza de que deseja especificar um bucket do S3 em outra conta. Verifique o nome do bucket que você inseriu, ao final da configuracao clique em CREATE.

PRINT 3.0

Escolha Advanced.

No campo Log file prefix digite o mesmo prefixo inserido para o armazenamento de arquivos de log ao ativar o CloudTrail usando as credenciais da conta PRINCIPAL. Se você optar por usar um prefixo diferente do informado quando ativou o CloudTrail na primeira conta, será necessário editar a política no seu bucket de destino para permitir que o CloudTrail grave arquivos de log em seu bucket usando esse novo prefixo.

PRINT 3.1


