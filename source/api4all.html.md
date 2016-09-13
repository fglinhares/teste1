
---
title: API Conta 4all - 2.11

language_tabs:
 -  json: JSON

toc_footers:
  - <a href='http://4all.com'>4all.com</a>
  - <a href='https://4alltecnologia.github.io/Digital-Commerce/'>Home</a>

includes:

search: true
---

API Conta 4all
--------------

Versão 2.11 14
07/2016

#1. Introdução

##1.1. Mensageria
###1.1.1. Comunicação com TLS
Toda comunicação com o servidor 4all deve ser protegida por TLS 1.2 ou posterior. O certificado fornecido pelo servidor ***sempre*** deve ser inspecionado para confirmar que se trata de um certificado confiável.
###1.1.2. Mensagens HTTP
Os headers HTTP devem indicar `‘ContentType’` como `‘application/json’.` O header ‘`Accept’` deve ser usado e deve indicar `‘application/json’.`
###1.1.3. Respostas de sucesso
Respostas de sucesso podem possuir ou não conteúdo. Quando possuem conteúdo retornam status 200.
Quando não possuem conteúdo retornam status 204.
###1.1.4. Respostas de erro
Erros podem ser identificados pelo código de status HTTP 4xx (client error) ou 5xx (server error).

```json
{
	"error": {
	"code": "30.17",
	"message": "Parse error"
	}
}
```
O corpo da resposta de erro pode conter um JSON onde existe um objeto ‘error’, que possui os campos ‘code’
(com um string de erro para ser usado ‘internamente’ pela 4all) e um campo ‘message’ (com uma mensagem
que pode e deve ser exibida ao usuário final).

### 1.1.5. Evolução do formato de mensagens
O frontend deve estar preparado para receber campos extras/novos/não previstos nas mensagens sem que isto cause erros. Conforme o protocolo evoluir, campos podem ser adicionados nas mensagens, e o frontend
não deve parar de funcionar por causa disso. É aceitável e esperado, entretanto, que campos faltando gerem erros.

###1.1.6. Endereços dos servidores de testes e de produção
Os seguintes endpoints devem ser usados para executar as chamadas:

| Ambiente |URL
|-------------|---------
|Testes |conta.test.4all.com
|Produção |conta.api.4all.com

Os serviços sempre devem ser acessados usando TLS, na porta 443. Na prática, os endpoints acima sempre
serão precedidos de " **https://** ".

###1.1.7. Particularidades das mensagens
Alguns campos das requisições são padrão:
* ***geolocation*** é um objeto contendo informações de geolocalização do device. Quando não for possível determinar a posição do dispositivo (por exemplo: usuário negou a permissão), a mensagem deve omitir este objeto.
* ***device*** é um objeto contendo as informações do dispositivo que está acessando a conta 4all.
* ***itemIndex*** aparece em chamadas que retornam listas de itens, e indica o índice do primeiro item a ser retornado pelo servidor.
* ***itemCount*** também aparece em chamadas que retornam listas de itens, e indica quantos itens devem ser retornados pelo servidor.

A geolocalização não é obrigatória nas requisições, mas se estiver presente deve seguir o seguinte formato:

							GEOLOCATION
|Campo |Descrição |Formato |Tamanho |Obrigatório
|------|----------|--------|--------|-------
|**latitude**|Representa a latitude do dispositivo no momento da requisição, com 6 casas decimais; pode ser negativo |Number |90.0 Até 90.0 |sim
|**longitude** |Representa a longitude do dispositivo no momento da requisição, com 6 casas decimais, pode ser negativo|Number|180 até 180.0|sim
|**accuracy**|Representa a precisão estimada da posição, em metros|Number|1-4 bilhões |sim
|||||

Os dados do dispositivo devem seguir o seguinte formato:

							DEVICE
|Campo |Descrição |Formato |Tamanho |Obrigatório
|------|----------|--------|--------|-------
|**deviceType**|Tipo do dispositivo:  1 para iOS 2 para Android, 3 para Windows Phone, 4 para browser|Number|1|sim
|**deviceModel**|Modelo do dispositivo|String|50|sim
|**osVersion**|Versão do sistema operacional do dispositivo|String|50|sim
|**pushId**|Identificador que deve ser usado para o envio de mensagens push para este dispositivo.|String|240|não
|||||


Caso seja acessado via navegador, os campos "deviceModel" e "osVersion" devem ser preenchidos, respectivamente, com o nome do navegador e sua versão.

##1.2. Postback
O postback é um mecanismo de comunicação com o merchant para informar sobre mudanças ocorridas no status de um pagamento ou assinatura.
Quando um pagamento ou assinatura são criados pelo merchant, uma URL pode ser informada, para onde as mensagens de postback serão direcionadas. Esta URL deve estar num formato válido, apontar para um domínio autorizado no cadastro do merchant, e pode possuir até 250 caracteres.

> Exemplo
```json
{
	"type": 1,
	"id": "17999999999999999999",
	"status": 1,
}
```
Sempre que ocorrer uma alteração no status de um pagamento ou assinatura que possuam as informações de postback configuradas, uma requisição POST será feita para a URL informada, contendo um corpo JSON (Contenttype:application/json) como pode ser visto no exemplo a seguir:

O campo ***type*** pode assumir os valores 1 (para postback referente a pagamento) ou 2 (para assinatura).

Para que seja considerado que a comunicação ocorreu com sucesso, o retorno dado à mensagem de postback deve possuir status 200 ou 204. Do contrário, serão efetuadas retentativas intervaladas entre si em pelo menos um minuto até um  máximo de cinco vezes. Caso ainda não se obtenha sucesso, NÃO serão feitas tentativas adicionais de comunicação da mudança de status.

O recurso de postback é apenas uma conveniência para melhorar a integração com o sistema 4all, e está sujeito a falhas de comunicação, indisponibilidade de alguma das partes, etc. Desta forma, não elimina a necessidade de consultas periódicas ao status do pagamento ou da assinatura que se deseja monitorar.

Para garantir a autenticidade da mensagem, é enviado no seu cabeçalho um campo **Content-signature**. Este contém um *hash* em **HMAC-SHA-256** em hexadecimal calculado sobre o corpo da mensagem, usando como chave a *signatureKey* do estabelecimento.

##1.3. Datas e timestamps
Todas as datas e timestamps são armazenadas em UTC pelo servidor, e comunicadas em UTC nas mensagens dos webservices.

Para exibição, o frontend deve converter o horário UTC para o horário local.

De maneira geral, o formato usado em datas e *timestamps* nesta API é como segue:

|Informação|Formato |Exemplo|
|------|----------|--------|
|Ano|YYYY|1997|
|Ano e mês|YYYYMM|199707|
|Data completa|YYYYMMDD|19970716|
|Data completa com horas e minutos|YYYYMMDDThh:mmZ|19970716T19:20Z|
|Data completa com horas, minutos e segundos|YYYYMMDDThh:mm:ssZ|19970716T19:20:30Z|
||||

Algumas exceções são feitas, por exemplo nas datas de vencimento de cartões.

##1.4. Parâmetros das assinaturas (pagamentos recorrentes)
Descrição dos parâmetros que governam o funcionamento de uma assinatura:

 - ***upfrontAmount*** indica o valor que será pago imediatamente pelo cliente, no momento em que aceita a assinatura. Caso a data de início
   da cobrança periódica seja no mesmo dia, ambos os valores serão   
   cobrados do cliente (em uma única cobrança).
   
 - ***startDate*** indica a data em que os pagamentos periódicos terão início. Não afeta ***upfrontAmount*** de nenhuma forma, que continua sendo pago imediatamente mesmo que startDate esteja no futuro. Esta data não pode estar no passado.
   
  - Nem sempre as assinaturas são aceitas imediatamente, e em alguns
   casos - especialmente quando a assinatura estiver acontecendo próximo
   da meia-noite - uma assinatura acaba sendo aceita no dia seguinte à
   ***startDate***. Estes casos são permitidos pelo sistema e não alteram as
   datas de vencimento futuras.
   
  - Assinaturas cujo ***startDate*** esteja dois ou mais dias no passado não podem ser aceitas.
  
 - ***recurringAmount*** indica o valor das cobranças periódicas.
 -  ***recurrenceCount*** indica quantas cobranças periódicas serão feitas no total. O valor '0' indica infinitas cobranças (até o cancelamento). A cobrança inicial indicada por ***upfrontAmount*** não é contabilizada neste parâmetro.
 - ***intervalType*** e ***intervalValue*** indicam, em conjunto, o intervalo configurado para esta cobrança recorrente:
	- ***intervalType*** = 0: o campo ***intervalValue*** indica um intervalo de dias entre um pagamento e outro, e não pode ser 0. O valor 1 em ***intervalValue*** indica que o pagamento é diário.
	- ***intervalType*** = 1: o campo ***intervalValue*** indica um intervalo de semanas entre um pagamento e outro, e não pode ser 0. O valor 1 em ***intervalValue*** indica que o pagamento é semanal.
	- ***intervalType*** = 2: o campo ***intervalValue*** indica um intervalo de meses entre um pagamento e outro, e não pode ser 0. O valor 1 em ***intervalValue*** indica que o pagamento é mensal. Pagamentos que cairiam após o último dia de um mês (por exemplo pagamentos mensais no dia 31 em um mês com 30 dias) são efetuados no primeiro dia do mês seguinte, mas isto não muda a data de vencimento dos demais pagamentos.

A data indicada por ***startDate*** sempre é usada como referência para todos os pagamentos. Por exemplo, para ***intervalType*** = 2 com ***intervalValue*** = 1 (intervalo de um mês entre um pagamento e outro) e ***startDate*** = 25/01/2015, a data do primeiro pagamento recorrente ocorrerá no dia 25/01/2015,
o segundo em 25/02/2015, e assim por diante.

É necessário ter cuidado para não criar cobranças indevidas por engano; por  exemplo, criar no dia 24 um pagamento mensal de R\$ 10 para o dia 25 com um upfront de R\$ 10 significa que o usuário será cobrado tanto no dia 24 (momento da criação) quanto no dia 25 (próximo pagamento recorrente) do mesmo mês (mas apenas no primeiro mês, pois o pagamento upfront só é cobrado no momento de criação da assinatura). Neste tipo de situação, para assinaturas com datas de vencimento fixas, **upfrontAmount** poderá ser um valor prórata que depende do dia em que a assinatura foi criada, e ***recurringAmount*** será o valor cheio, independente do dia em que a assinatura for criada.

 - ***paymentTolerance*** indica quantos dias a assinatura pode permanecer sem ser paga antes de ser automaticamente cancelada por falta de pagamento. Caso este valor seja 0, a assinatura precisa ser paga no mesmo dia do vencimento; caso seja 1, pode ser paga até o dia seguinte; e assim por diante. Caso o pagamento seja feito com atraso, mas ainda dentro do período de tolerância, a assinatura continuará sendo cobrada periodicamente, sem que as futuras datas de cobrança sejam  alteradas de qualquer forma.


##1.5. Transações via cartão de débito
###1.5.1. Diferença na troca de mensagens
Na realização de pagamentos com cartão de débito, deve ser passado o atributo ***waitForTransaction*** como ***true***.
Na resposta, deve ser observado se o status indica que o pagamento está pendente, aguardando a conclusão do pagamento de débito (status ***9*** , de acordo com `Tabela 2 Anexo B` ).

###1.5.2. Início de transação de débito
Para transações utilizando cartão de débito, é necessário redirecionar o usuário para uma página onde o mesmo informará dados adicionais (como código de segurança e número de celular do portador) e se autenticará com o banco emissor do cartão.

Para isso, os serviços que realizam pagamento retornam o atributo ***debitTransactionURL*** contendo o exato endereço para o qual o usuário deve ser redirecionado. Este redirecionamento deve ocorrer dentro de um elemento de *webview/iframe* específico de cada plataforma.

###1.5.3. Conclusão de pagamento de débito
Ao final do pagamento via cartão de débito, no mesmo *webview/iframe*, uma página da própria 4all será exibida indicando o sucesso ou não do processo. A mesma não terá nenhum conteúdo. Seu propósito será indicar, através da URL, o desfecho da transação.

Em caso de sucesso/aprovação da transação, será carregada a página:
**`< endpoint>/debit/paymentOk`**

Já no caso de falha/negação da transação, a página a ser carregada será:
**`< endpoint>/debit/PaymentNotOk`**

###1.5.4. Integração com *postbacks*
Existem implementações que exigem a tomada de uma ação após a confirmação de um pagamento, como confirmar recarga de créditos, emitir ingresso, etc. Ações do gênero são de responsabilidade do estabelecimento comercial que se integrar à API de pagamentos 4all, mediante monitoramento dos pagamentos pendentes.

Para essas integrações, sugere-se o uso da funcionalidade de *postbacks*.

###1.5.5. Condições de disponibilidade e aceitação
O aceite de transações por débito está disponível para os estabelecimentos comerciais que possuem credenciamento com um dos adquirentes com suporte à referida modalidade.

Além disso, devido ao fluxo diferenciado, para aceitar transações por débito o estabelecimento comercial deve demonstrar à 4all que sua implementação se comporta de maneira adequada.

Por fim, só são aceitos pagamentos com cartões de débito cujos emissores estão cadastrados junto às bandeiras para utilizar suas implementações do protocolo 3D
Secure (Verified by Visa ou MasterCard SecureCode).


#2. API de Identificação
##2.1. Cadastro e Login
###2.1.1. Start Customer Creation
**Caminho**: `<endpoint>/customer/startCustomerCreation`
**Descrição**: Esta chamada é usada para iniciar o processo de cadastro de um novo customer.

> Exemplo
```json
{
	"emailAddress": "mail@example.com",
	"phoneNumber": "555199999999",
	"applicationId": "5366",
	"sendSms": true,
	"device": {
	"deviceType": 1,
	"deviceModel": "iPhone 5C",
	"osVersion": "iOS 8.1.1"
},
	"geolocation": {
	"latitude": 37.123123,
	"longitude": -127.123123,
	"accuracy": 10
	}
}
```
					REQUISIÇÃO
|Campo |Descrição |Formato |Tamanho |Obrigatório
|------|----------|--------|--------|-------
|**emailAddress**|Endereço de email.|String|200| sim|
|**phoneNumber**|Número do telefone celular. DDI + DD + 8/9 dígitos |String|20|sim|
|**applicationId**|Identifica o app/software que o customer está usando para se cadastrar.|String|15|não|
|**sendSms**|Quando presente e com valor true , a chamada com sucesso a esta rotina causa o envio de um SMS com o desafio para criação de conta.|Boolean||não|
|**requestVaultKey**|Quando presente e com valor true , o retorno desta chamada incluirá uma chave de acesso externo ao cofre de cartões (vault).|Boolean||não|
|**device**|Informações sobre o dispositivo.|Object|*|sim|
|**publicApiKey**|Chave de identificação de estabelecimento. Deve ser informada para permitir compras em estabelecimentos terceiros.|String|44|não
|**geolocation**|Geolocalização.|Object|*|não|
|||||

> Exemplo
```json
{
"creationToken": "JsFbKkyhM+q05nSKB...",
"pinRequired": false
}
```

					RESPOSTA

|Campo |Descrição |Formato |Tamanho |Presente
|------|----------|--------|--------|-------
|**creationToken**|Chave utilizada para concluir o processo de criação de um *customer*.|String|44|sempre|
|**accessKey**|Chave de acesso temporária ao vault para a realização do cadastro de um cartão. Presente na resposta somente se solicitado através do parâmetro ***requestVaultKey*** da requisição.|String|44|depende|
|**pinRequired**|Indica se é obrigatória a criação de um PIN durante a criação de uma nova conta.|Boolean||sempre|
|||||


**Observações**:
● Caso o email ou telefone indicados pelo cliente já estejam em uso por outra conta **ativa** , a criação de usuário deve falhar.
● Uma conta somente se torna **ativa** quando for concluído o seu processo de criação, através de uma chamada com sucesso a ***complete customer creation*.**
● O parâmetro da resposta ***creationToken*** expira em 48 horas; após este período, será necessário chamar esta funcionalidade novamente.
● Quando ***sendSms*** for **true** , a chamada com sucesso desta rotina provoca o envio de um SMS ao cliente, contendo um desafio de 6 dígitos, que deve ser informado em ***complete customer creation***. Mesmo quando o SMS não é enviado automaticamente ao cliente, ainda será necessário que um desafio de 6 dígitos seja usado em ***complete customer creation*** - a rotina send login sms pode ser usada para que o desafio seja enviado.
● O processo de criação da conta não pode ser concluído sem que se use um desafio de SMS. Não é permitida a existência de um cliente no sistema com um telefone não confirmado.
● O parâmetro ***accessKey*** da resposta contém uma chave de acesso temporário ao cofre de cartões, que deve ser utilizada pelo frontend para realizar o cadastro de um novo cartão diretamente no cofre, usando a chamada ***prepare card*** (consultar o  `Anexo C`).
● Quando estiver presente o parâmetro "publicApiKey", a sessão aberta deve ser vinculada ao referido estabelecimento.