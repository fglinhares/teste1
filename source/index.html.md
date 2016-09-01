---
title: 4 Pay - Checkout 4all

language_tabs:
  - XML
  - Javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

**4 Pay - Checkout 4all**

Manual de Uso - Web/Javascript - Versão 0.1

# 1. Introdução

Com o Checkout 4all, você pode configurar seu website para aceitar pagamentos de cartões de crédito e débito de maneira fácil e rápida.

Para aceitar pagamentos de Cartão em seu site, você só precisa executar dois passos:

 1. Incluir o arquivo do Checkout em seu site;
 2. Capturar a transação no seu servidor.


# 2. Pagamentos
## 2.1 Janela de Checkout
A Janela de Checkout é apresentada quando o cliente confirma a compra (no submit do formulário de fechar pedido, por exemplo) ou seleciona a forma de pagamento “Cartão de Crédito ou Débito via 4all” (quando o seu site permite mais de uma forma de pagamento). 

O dialog permite que o usuário informe os dados do cartão ou se autentique com sua Conta 4all para buscar seus dados de pagamento já cadastrado. 

<TODO: inserir imagem do dialog sobre um site de compras>
<TODO: no site, inserir botão com demo do dialog>

Você pode inserir o Dialog no seu site de duas maneiras: via **Embedded Form**, ou via **Biblioteca Javascript**.

### 2.1.1 Embedded Form

> Inclua o código do checkout na sua página, como no exemplo:

```Javascript
<form action="pedido_concluido.php" method="POST">

  <!-- … demais inputs do formulário … -->
  
  <script
	src="endereco_4all_checkout" id="checkout-4all"
	data-token="pk_test_6pRNASCoBOKtIshFeQd4XMUh"
	data-items="[{descricao:"Camiseta Polo Tam P", valor: 5999}]"
	data-amount="5999"
	data-merchant="Loja do Alfredo"
	data-logo="http://urlDoSeuLogotipo/logo.png">
  </script>
</form>
```

**Como funciona:** Nesta modalidade de checkout, você adiciona à sua página um formulário que efetua, de forma segura, todo o processo de captura e validação dos dados de pagamento do cliente, assim, as informações sensíveis do seu cliente nunca entram em contato com ambientes não-seguros. Após a captura e validação dos dados, é adicionado ao formulário um token de pagamento (válido para apenas **uma** transação) que deverá ser utilizado para efetuar a confirmação na API de pagamentos 4all.

No evento de *submit* do seu formulário, a Janela de Checkout será apresentada para o cliente inserir os dados de pagamento.

Substitua o valor de cada atributo de acordo com o pagamento a ser efetuado. Os atributos aceitos são:


**Requisição:**

Atributo    |Descrição  |Formato    |Obrigatório 
------------|-----------|-----------|--------------
data-merchant-key|Token de identifição 4all.|String|Sim
data-amount|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim
data-merchant|Nome da loja que está efetuando a venda.|String|Sim
data-card-type|Tipos aceitos de cartão. (1 para apenas crédito, 2 para apenas débito e 0 para ambos).|String|Não
data-logo|URL completa do logotipo da sua empresa.|String|Não

### 2.1.2 Biblioteca Javascript

```XML
<script src="endereco_4all_checkout"></script>
```
Para fazer a integração via javascript, você deve primeiramente importar o arquivo da biblioteca na sua página, adicionando a seguinte linha ao seu HTML:

```Javascript
Checkout4all.startCheckout(amount, checkoutApiKey, options, function (paymentToken){
  
  //este código será executado após a confirmação do pagamento pelo  
  //usuário

});
```
Após isso, a biblioteca está disponível em seu escopo global como 'Checkout4all'. Para iniciar o processo de login/pagamento, basta chamar a função startCheckout da biblioteca.:

Os parâmetros dessa função são:

Atributo    |Descrição  |Formato    |Obrigatório 
------------|-----------|-----------|--------------
amount|Valor da transação em centavos. Ex: "1425" para R$ 14,25.|String|Sim
chckoutPublicApiKey|Chave de API pública do Checkout 4all.|String|Sim
options|Objeto com opções do pagamento (mais detalhes na tabela abaixo).|String|Sim
callbackAfterCheckout|Função que será chamada quando o checkout estiver finalizado. Recebe o paymentToken como parâmetro.|String|Sim
cardType|Tipos aceitos de cartão. (1 para apenas crédito, 2 para apenas débito e 0 para ambos). Quando não informado, serão aceitos ambos os tipos.|String|Não
logoUrl|URL completa do logotipo da sua empresa.|String|Não

## 2.2 Capturando a transação

Com o **payment_token**, você pode capturar a transação (efetuar a cobrança) através de uma chamada à API Conta 4all.

**Nota:** por motivos de segurança, você deve informar o valor total da transação novamente nesta chamada.

Exemplos:

`
curl -X POST "https://api.pagar.me/1/transactions/{TOKEN}/capture"
  -d 'amount=1000'
  -d 'api_key=ak_test_grXijQ4GicOa2BLGZrDRTR5qNQxJW0'
`


<aside class="success">
* O parâmetro a ccessKey da resposta contém uma chave de acesso temporário ao cofre de cartões, que deve ser utilizada pelo frontend para realizar o cadastro de um novo cartão diretamente no cofre, usando a chamada prepare card (consultar o Anexo C ).
</aside>

