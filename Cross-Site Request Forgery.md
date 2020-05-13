# Cross-Site Request Forgery

### O que é:

Cross-Site Request Forgery-CSRF, ou falsificação de requisição entre sites, é um dos ataques mais antigos, ele existe praticamente desde a fundação da WEB. Ele ocorre quando, um site malicioso força o navegador do usuário a realizar requests arbritárias em um aplicativo WEB que ele esteja autenticado. Para que este ataque ocorra é esperado que a vítima esteja logada em sua conta e acesse o link malicioso.

![](https://i.imgur.com/kq6Tq54.png)
###### Fonte: https://www.infosec.com.br/wp-content/uploads/2017/07/cross-site-request-forgery.png
### Funcionamento técnico

Para um melhor entendimento desta vulnerabilidade, vou dar um exemplo. 

1. Você faz login em seu banco.
2. Recebe um link no email e o acessa.
3. Percebe que sumiram $1000 da sua conta.

Como isso funcionou? Simples, ao fazer login no  banco, o usuário recebe cookies do servidor, os mesmos são anexados em todas as requests feitas ao banco, independente da origem delas. Portanto, se o atacante analisar como funciona o mecanismo de transferência de valores o mesmo pode reproduzí-lo em seu site malicioso, assim quando um cliente, autenticado, acessá-lo sofrerá o ataque.

Na prática o ataque funcionaria da seguinte forma:

Vamos supor que para fazer uma transferência o seu banco envie uma request do tipo GET, nada seguro, nesta url: http://banco.com/transfer?para=nome&montante=valor o atacante poderia usar uma página web maliciosa simples como esta para fazer o ataque:

```html
<!DOCTYPE html>
<html>
	<body>
		<img src="http://www.banco.com/transfer?para=thiago&montante=1000">
	</body>
</html>
```

Vamos entender o que esta página maliciosa está fazendo, o atributo src da tag html ```img``` funciona fazendo um GET no link da source e exibindo o conteúdo, logo no carregamento da página é feito um GET no link de transferência e o cookie é indexado, gerando assim, uma solicitação autêntica.

O mesmo tipo de ataque pode ser feito via post, neste caso a request feita pelo banco seria parecida com esta:

```
POST / HTTP/1.1 
Host: banco.com/transfer
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Content-Type: text/plain
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Connection: keep-alive
Cookie: SID=abcdefg

para=nome&montante=valor
```
O atacante então, poderia criar uma página parecida com esta:

```html
<!DOCTYPE html>
<html>
	<body>
		<form method="POST" 
		action="http://www.banco.com/transfer"
		id="form"
		enctype="text/plain">

		<input type="hidden" name="para" value="thiago">
		<input type="hidden" name="montante" value="1000">

		</form>
		<script>document.getElementById("form").submit()</script>
	</body>
</html>
```

Esta página funciona com um [formulário](https://github.com/T635/DI.WE.H/blob/master/Hypertext%20Markup%20Language%20-%20HTML.md), nele inserimos os atributos method, lá inserimos o tipo de request, definimos o action que é basicamente o link para onde a request deve ser enviada, definimos o id para identificá-lo em nosso código javascript, e por último definimos o enctype que é o Content-Type da request.Nas tags input, definimos o nome e o valor, isso será indexado no body da nossa request, da seguinte forma:```para=thiago&montante=1000``` o "&" é usado para separar dois inputs.Ao acessar esta página a vitíma envia uma request post para o banco, e indexa seu cookie, sofrendo assim, o ataque.

#### Nem tudo são rosas...

Com a disseminação desse tipo de ataque foram criadas diversas políticas de proteção contra ele, sendo a mais conhecida o same origin policy, nele o navegador impede que requests ou recursos sejam solcitados entre um site, e por exemplo, um iframe contido nele.Para que uma request, seja de fato feita, o site e o iframe devem ter a mesma url, o mesmo protocolo-http/https, e a mesma [porta](encurtador.com.br/vCQW1). 

Essa proteção foi afrouxada com a chegada a política de CORS, ou compartilhamento de recursos de origem cruzada, nela sites com urls, protocolos e até portas diferentes podem se comunicar e trocar recursos, para isso o servidor define um cabeçalho chamado ```Access-Control-Allow-Origin```, ele define se os recursos podem ser compartilhados com a Origin fornecida,mesmo não sendo a maneira mais segura de se proteger contra CSRF, ainda assim é melhor do que aceitar qualquer request.

No entanto, não são todas as requests que "ativam" o CORS, as mais simples como as citadas acima passam sem problemas por ele.

### Exemplo:

Em 2016, o pesquisador Akhil Reni encontrou uma falha no Shopify que permitia desconectar um usuário de seu Twitter, para isso Akhil analisou como funcionava o processo de logout shopify, e percebeu que o mesmo realizava uma request do tipo GET para a seguinte url: https://twitter-commerce.shopifyapps.com/auth/twitter/disconnect segue um exemplo da request:

```
GET /auth/twitter/disconnect HTTP/1.1
Host: twitter-commerce.shopifyapps.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:43.0) Gecko/20100101 Firefox/43.0
Accept: text/html, application/xhtml+xml, application/xml
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://twitter-commerce.shopifyapps.com/account
Cookie: _twitter-commerce_session=bmpuTE5EdnUvYUU0eGxJRk1kMWo5WkI3Wmh1clJkempOTDcya2R3eFNIMG8zWGdpenMvTXY4eFczTWUrNGRQeXV4ZGVycEVtTDZWcFZVbEg1eEtFQjhzSEJVbkM5K05VUVJaeHVtNXBnNTJCNTdwZ2hLL0x0Kyt4eUVlSjRIOWdYTkcwd1NQWWJnbjRNaTF5UXlwa1ZIUlAwR1JmZ1Y5WmRvN2ZHWFY5REZSUmlsR0lnMHZlSjR1OTlTMW5xWDdZRnVGSnBSeEhqbWpNS3lYZmxBNjZoVE00L3pQT2NMd1NONkdwb2pkMXhDS1E2M2RXYlovZjYwaUZnV0JQKzQySlN0MTNKNG55Zlg2azFDdVJJL3RidmJMM0VJNmRVejhZbjVDTnFZNmxFN0k9LS1lY1Y2dnpBZTJCalZzS014SldFUllBPT0%3D--77463ef21e4c8ef530f466db49f78b8e1c2e1129; _ga=GA1.2.469272249.1453024796; _gat=1
Connection: keep-alive
```
O pesquisador montou uma página com o seguinte código:

```html
<html>
<body>
 <img src="https://twitter-commerce.shopifyapps.com/auth/twitter/disconnect">
  </body>
</html>
```

Feito isso, conseguiu gerar uma prova de conceito(PoC), e a enviou para a empresa, ele recebeu $500 por esta falha.

O report todo pode ser lido [aqui](https://hackerone.com/reports/111216).

### Como explorar:

Para explorar esta falha o pesquisador deve estar atento aos funcionamento dos processos mais perigosos como, remoção de conta, troca de email e troca de senha, analisar como funcionam e tentar reproduzi-los em seu site malicioso. 

Alguns problemas podem ser encontrados ao tentar explorar esta vulnerabilidade, os principais são, Content-Type diferentes ou CSRF token, o segundo é um token aleatório e único gerado pelo servidor e que fica salvo na sessão do usuário, ele geralmente fica em um input hidden e é enviado e verificado em toda a requisicão; infelizmente não há muitas formas de burlá-lo, o pesquisador deve apenas conferir se ele está implementado corretamente ou seja, se o servidor está realmente verificando o valor, para isto, basta troca-lo por um valor parecido. 

O problema do Content-Type ocorre quando ele é do tipo application/json, os formulários HTTP atualmente não permitem este cabeçalho aceitando apenas, text/plain, multipart/form-data e application/x-www-form-urlencoded, outro método seria usar requests javascript para mudar o Content-Type, mas isso ativa o CORS e impede a request de ser feita.Para tentar burlá-lo o pesquisador deve verificar se o servidor valida o Content-Type, ou não, caso valide o mesmo pode tentar usar um flash + 307 redirection, mas não irei comentá-lo aqui pois flash applications não funcionaram mais após dezembro de 2020; caso não haja validação, o pesquisador pode usar um Content-Type text/plain e reproduzir um json no body da request. Exemplo:


```html
<!DOCTYPE html>
<html>
	<body>
		<script>
			var url = http://banco.com/transfer
			var headers ={method: ‘POST’, credentials: ‘include’, headers: {‘Content-Type’: ‘text/plain’}, body: ‘{“para”:”thiago”,”montante”:”1000”}’}
			fetch(url,headers)
		</script>
	</body>
</html>
```
Vamos analisar o que fiz acima, primeiro criei uma página html normal com um código javascript nela, esse código usa uma função para requisições http chama [fetch](https://developer.mozilla.org/pt-BR/docs/Web/API/Fetch_API/Using_Fetch), ela aceita dois argumentos, sendo o primeiro nossa url e o segundo um array contendo o método da request, informações sobre cookies, content-type e o body dela.