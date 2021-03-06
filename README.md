# Como centralizar os logs do CloudTrail em uma unica conta 

## O objetivo é centralizar os logs de todas as contas da AWS em uma unica conta, sendo assim, os LOGs estarao disponiveis em um unico Bucket S3.

### **Requesitos:**

- *Acesso a todas as contas envolvidas*


### **PASSOS:**

* 1 Ativar o CloudTrail na conta princpal, onde sera centralizado os logs
* 2 Atualizar a política no bucket de destino para conceder ao CloudTrail permissões entre contas.
* 3 Ative o CloudTrail nas outras contas "Secundarias

*PASSO 1*


1 - Faça login em https://console.aws.amazon.com/cloudtrail/.

2 - Escolha a região em que você deseja que a trilha seja criada.

3 - Escolha Get Started Now.

![print01](/prints/01.png)

4 - Na página Create Trail, em Trail name, digite um nome para a sua trilha.

6 - Em Apply trail to all regions, escolha YES para receber os arquivos de log de todas as regiões. Essa é a configuração padrão recomendada. Se você escolher NO, a trilha registrará arquivos somente da região em que você a criou.

![print01.1](/prints/01.1.png)


Em Management event, Read/Write events, escolha se você deseja que sua trilha registre All, Read-only, Write-only ou None (print acima)
e, em seguida, escolha Create. Por padrão, as trilhas registram todos os eventos de gerenciamento, conforme o print abaixo.

![print01.2](/prints/01.2.png)
![print01.3](/prints/01.3.png)

*PASSO 2*

> Neste passo vamos atualizar a política no bucket para conceder ao CloudTrail permissões entre contas.

1 - No console da AWS abra o servico Amazon S3

![print02](/prints/02.png)

2 - Escolha o Bucket criado (neste exemplo: center-logs-full-accounts), Clique em Permissions depois em Bucket Police

![print02.1](/prints/2.1.png)

3 - Vamos modificar a política existente para adicionar uma linha para cada conta adicional cujos arquivos de log devem ser fornecidos a esse bucket. Veja o exemplo de política a seguir e observe a linha Resource onde foi especificando o ID 22222222222 de uma segunda conta.

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
> _Neste exemplo, em ( "Resource": ) foi adicionado a segunda conta:_ 

```bash
"arn:aws:s3:::center-logs-full-accounts/AWSLogs/222222222222/*"
```
> O numero ficticio da conta utilizado "222222222222", a cada inclusao de nova conta deve-se adicionar ser adicionada uma linha seguindo o exemplo acima.

Print de Exemplo:

![print02.1](/prints/2.2.png)


4 - Apos as configuracoes aplicadas, clique em SAVE.

*PASSO 3* 

> Neste passo vamos configurar as contas adicionais para enviar logs a conta principal,
como exemplo vou assumir que o ID da conta adicional é 222222222222.


1 - Faça login no console de gerenciamento da AWS usando as credenciais da conta 222222222222 e abra o console do AWS CloudTrail. Na barra de navegação, selecione a região em que você deseja ativar o CloudTrail.

2 - Escolha Get Started Now.

3- Na página seguinte, digite um nome para a sua trilha na caixa Trail name.

![print03](/prints/3.png)

4 - Em Create a new S3 bucket?, escolha NO. Use a caixa de texto para informar o nome do bucket que você criou anteriormente para o armazenamento de arquivos de log quando fez login usando as credenciais da conta PRINCIPAL. O CloudTrail exibe um aviso perguntando se você tem certeza de que deseja especificar um bucket do S3 em outra conta. Verifique o nome do bucket que você inseriu, ao final da configuracao clique em CREATE.

![print3.1](/prints/3.1.png)


5 - Escolha Advanced.

No campo Log file prefix digite o mesmo prefixo inserido para o armazenamento de arquivos de log ao ativar o CloudTrail usando as credenciais da conta PRINCIPAL. Se você optar por usar um prefixo diferente do informado quando ativou o CloudTrail na primeira conta, será necessário editar a política no seu bucket de destino para permitir que o CloudTrail grave arquivos de log em seu bucket usando esse novo prefixo.


Referencia:
[Documentacao Oficial AWS](https://docs.aws.amazon.com/pt_br/awscloudtrail/latest/userguide/cloudtrail-create-a-trail-using-the-console-first-time.html)

