# Template Injection

### O que é:

Esta falha pode acontecer tanto do lado servidor quando do lado cliente, em ambos os lados isto é causado por uma falta de sanitização dos inputs do usuário.

Como vocês devem ter percebido a maior partes das falhas se dão nas entradas do usuário, então fica mais que claro que o desenvolvedor deve sanitizá-las ao máximo, tanto no lado cliente como no lado servidor.

### Template Engine:

![](https://dkrn4sk0rn31v.cloudfront.net/2019/12/12151141/template-engine.png)
###### Fonte: https://dkrn4sk0rn31v.cloudfront.net/2019/12/12151141/template-engine.png
Não há como falar sobre template injection sem entender o que é um template, e seu engine então vamos lá!

Quando se faz a criação de sites um pouco mais complexos, há a necessidade de integrar dados há página HTML, sejam estes vindos de uma A.P.I ou do banco de dados, e organizar este conteúdo com regras de negócio como ifs e elses se torna muito complicado dificultando até a leitura do código; é neste contexo que entram os template engines ou view engines, eles são métodos que facilitam a criação destas páginas.

Este é um exemplo de um código que verifica se o usuário esta logado e gera uma página com o nome dele:

```php
<?php if ($usuario->logado()): ?> 
Bem vindo, <strong><?= $usuario->nome; ?></strong> 
<?php endif; ?>
```

Como é percebido este código está difícil de ser lido, então vou implementar o template engine Blade específico para php

```php
@if ($usuario->logado()) 
Bem vindo, <strong>{{ $usuario->nome }}</strong> 
@endif
```

percebe-se que o código ficou mais legível e fácil de manipular.

Cada linguagem de programação tem seus templates engines, por exemplo python tem o Jinja, nodejs tem o Jade e ruby tem o Haml.

### Funcionamento técnico:

#### Server-Side:

Esta injection, como as outras, funciona quando o atacante envia dados sensíveis e o servidor os anexa à página sem uma devida sanitização.

Vou dar um exemplo usando Flask/Jinja:

```python
@app.errorhandler(404)
def page_not_found(e):
	template = '''{%% extends "layout.html" %%}
	{%% block body %%}
	<div class="center-content error">
		<h1>Opps! That page doesn't exist.</h1>
		<h3>%s</h3>
	</div>
	{%% endblock %%}
	''' % (request.url)
return render_template_string(template), 404

```
###### source: [https://nvisium.com/blog/2016/03/09/exploring-ssti-in-flask-jinja2](https://nvisium.com/blog/2016/03/09/exploring-ssti-in-flask-jinja2)

Explicarei o máximo possível deste código sem me aprofundar muito à ele, inicialmente é criada uma página na função ```page_not_found``` ela é responsável por avisar quando, adivinhem, uma página não é encontrada; para isto é criada uma variável chamada template a qual tem o valor de um código jinja, ele usa uma combinação de chaves porcentos e hashs para diferenciar código python de código HTML.

Esta página contém em uma string chamada request.url ela contém o nome da página que não foi encontrada.


![](https://nvisium.com/articles/2016/2016-03-09-exploring-ssti-in-flask-jinja2/ssti_flask_1.png)
###### source:https://nvisium.com/articles/2016/2016-03-09-exploring-ssti-in-flask-jinja2/ssti_flask_1.png


Um atacante esperto tentaria de imediato injetar um código javascript no final da url para tentar gerar um XSS e de fato ele conseguiria, este será o foco do próximo tópico, primeiro vamos entender como este modelo pode causar problemas no lado servidor.

O atacante para verificar a se esta página é vulnerável a SSTI pode injetar um código simples como {{7+7}}, as chaves duplas indicam que o código deve ser analisado como python, caso o servidor retorne o resultado da soma entende-se que o mesmo está tratando os dados de maneira errada, logo ele está vulnerável.

Agora precisamos identificar até onde esta vulnerabilidade se extende e o que pode ser feito, com python use funcões como o ```dir()```,```locals()``` ou ```help``` para medir isto, elas nos dão informações sobre atributos e o que eles podem chamar, use e abuse da documentação para entendê-las.

Neste caso o atacante inseriu ```{{config.items()}}``` esta função mostra todos as configurações do servidor desde a conexão com o banco de dados até as SECRET_KEYS.	

![](https://nvisium.com/articles/2016/2016-03-09-exploring-ssti-in-flask-jinja2/ssti_flask_4.png)
###### source:https://nvisium.com/articles/2016/2016-03-09-exploring-ssti-in-flask-jinja2/ssti_flask_4.png
### Client-Side:

O template engine mais comum no client-side é o framework angularjS ele é usado para a criação e execução de aplicações single-page.


Este framework renderiza os modelos em tela e caso uma injeção, contendo conteúdo malicioso, for incorporada à página pode-se gerar, por exemplo, um XSS.

```html
<html>
	<body>
		<p>
			<?php
				$q = $_GET['q'];
				echo htmlspecialchars($q,ENT_QUOTES);
			?>
		</p>
	</body>
</html>
```

O exemplo acima é um código básico em php, ele não é vulnerável a xss
```html
<html ng-app>
	<head>
		<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.7/angular.js"></script>
	</head>
	<body>
		<p>
			<?php
				$q = $_GET['q'];
				echo htmlspecialchars($q,ENT_QUOTES);?>
		</p>  
	</body>
</html>
```

O código acima é um exemplo da implementação do angularJs, isto pode ser visto pela request do script e pelo ng-app na tag html.

O interessante é que ao renderizar conteúdo, podemos injetar código js, como no python, dentro de chaves duplas ```{{}}```.

Também deve-se freezar que o próprio angular não permite que o usuário faça este tipo de injeção mas a cada nova atualização pesquisadores descobrem novos payloads que podem "bypassar" essa permissão da linguagem.

Esta é uma tabela contendo todos os bypass das versões do angular.

| Versão | Bypass |
| ----			  			 | ----- |
| 1.0.1 - 1.1.5   			 | constructor.constructor('alert(1)')() |
| 1.2.0 - 1.2.18 			 | ```a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()``` 						 |
| 1.2.19 - 1.2.23 			 | ```toString.constructor.prototype.toString=toString.constructor.prototype.call;["a","alert(1)"].sort(toString.constructor);```    |
| 1.2.24-1.2.26   			 |  ```{}[['__proto__']]['x']=constructor.getOwnPropertyDescriptor;g={}[['__proto__']]['x'];{}[['__proto__']]['y']=g(''.sub[['__proto__']],'constructor');{}[['__proto__']]['z']=constructor.defineProperty;d={}[['__proto__']]['z'];d(''.sub[['__proto__']],'constructor',{value:false});{}[['__proto__']]['y'].value('alert(1)')()```|
| 1.2.27-1.2.29/1.3.0-1.3.20 | ```{}.")));alert(1)//";``` |
| 1.4.0-1.4.5 				 | ```o={};l=o[['__lookupGetter__']];(l=l)('event')().target.defaultView.location='javascript:alert(1)';``` |
| 1.4.5-1.5.8 				 | ```o={};l=o[['__lookupGetter__']];(l=l)('event')().target.defaultView.location='javascript:alert(1)';``` |
| >=1.6.0      				 | ```constructor.constructor('alert(1)')()``` |


### Exemplo:

Em 2016 o Uber abriu seu programa público de bug bounty, e com ele liberou um artigo explicando todas as tecnologias usadas na construção de seus sites, este pode ser lido [aqui.](https://eng.uber.com/bug-bounty-map/) Quando se trata de achar falhas é muito importante saber o que é usado tanto no lado servidor quanto no lado cliente, pois isto faz com que o pesquisador perca menos tempo tentando injetar payloads ineficientes.

Usando o artigo do Uber o pesquisador percebeu que o subdomínio http://riders.uber.com foi criado usando Node.js/Express + Backbone.js nenhum destes indicam uma potencial exploração de SSTI, mas os subdomínios http://vault.uber.com e http://partners.uber.com foram criados usando Flask/Jinja2.

Então o pesquisador pensou que mesmo que http://riders.uber.com não usasse Jinja ele poderia fornecer entrada para outros subdomínios. Então ele criou um perfil em riders com o nome {{1 + 1}} e percebeu que havia uma mudança no seu nome nos subdomínios vault e partners, os quais valiam 2, o mesmo também percebeu que ao receber um email informando a mudança de seu nome o email também o chamava de 2.

Orange, o hacker, tentou então, injetar um código python em seu nome igual a:

```python
{% for c in [1,2,3]%}
	{{c,c,c}}
{% endfor%}
```

Este código gerava um nome igual a: ```1,1,1,2,2,2,3,3,3```

Isto foi percebido em vault e partners, logo Orange poderia injetar códigos python no servidor, o mesmo parou seu pentest aqui e não tentou fazer injeções mais perigosas pois poderia ferir as diretrizes do Uber.

A empresa o recompensou em um merecido bounty de $10000. O report pode ser lido na íntegra [aqui.](https://hackerone.com/reports/125980)

### Como explorar:

Para usar o máximo que esta vulnerabilidade pode oferecer, tente descobrir as tecnologias usadas pelo seu alvo, para montar payloads eficientes.

Isto fica evidente no exemplo deste artigo, pois nele o pesquisador não viu nenhuma mudança no subdomínio que usava node.js/express/backbone.js

Uma injeção de ```{{7+7}}``` pode funcionar muito bem em um site criado com python, mas seria infeciente em um site usando ruby, por exemplo, pois o mesmo só funciona com injeções deste tipo: ```<%= 7 * 7 %>```, é interessante entender como a injeção esta sendo tratava, pois há casos em que o site pode retornar 14 para a soma anterior, e retonar 7777777 para uma multiplicação, isto ocorre pois o mesmo está tratando os dados como string.

[Este repositório do github](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#basic-injections) mostra todos os possíveis payloads para todas as tecnologias de back e de front-end.