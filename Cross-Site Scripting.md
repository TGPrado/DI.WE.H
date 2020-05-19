# Cross-Site Scripting

### O que é:

Para um melhor entendimento desta vulnerabilidade é interessante ter lido nosso artigo sobre HTML Injection, pois esta é uma particularidade dele.

Nos últimos anos houve um aumento nos casos de Cross-Site Scripting ou XSS, por isso tentarei explorar tudo sobre esta vulnerabilidade, logo este artigo tende a ficar maior que os anteriores.

Este problema ocorre quando o atacante consegue injetar código Javascript na página, logo o mesmo pode roubar cookies, redirecionar o usuário, criar elementos HTML e etc; isto se dá,assim como no HTML Injection, pela falta de sanitização dos inputs do usuário.

### Funcionamento Técnico:

Esta falha será divida em 3 tipos: Reflected XSS, Persistence XSS e o DOM-Based XSS.

Os dois primeiros exemplos são iguais ao Reflected e ao Stored HTML Injection,respectivamente.

#### Reflected XSS:

Este é o tipo mais simples, nele o valor de um parâmetro é refletido na página e quando não é sanitizado o atacante pode injetar código Javascript.

Imagine esta página html:

```html
<!DOCTYPE html>
<html lang="pt-br">
    <body>
        <form action="destino.php" method="get">
            <p>
                Pesquisa:  <input type="text" name="q" value="teste"/>
            </p>
                <input type="submit" value="Submit me!"/>
            </p>
        </form>
    </body>
</html>
```
E o seguinte código php:

```php
<?php
$var = $_GET['q'];
echo "Resultados para ".$var . ":";
?>
```

caso o usuário pesquise por: ```<script>alert('xss')</script>``` será aberta uma caixa de pop-up no qual estará escrito 'xss'.

O exemplo acima é o mais básico, isto também pode ocorrer com a tag img sendo o código : ```<img src="javascript:alert(1)">``` este payload funciona pois caso o usuário digite, em sua barra de pesquisa, "javascript:" tudo após os dois pontos é executado. Usando a tag img, também pode ser criado um payload: ```<img src="nnnnn" onerror="javascript:alert(1)">```, neste caso fornecemos um source inexistente, e informamos um atributo,auto explicativo, chamado onerror.

Em alguns raros casos o valor de um header é refletido na página, dentre eles o mais comum é o referer, quando isto ocorre o atacante pode substituir, também, o valor deste header por um código javascript.

Um payload muito utilizado também, é executar um alert(document.cookie) ou alert(document.domain), o primeiro mostra todos os cookies do usuário, já o segundo mostra o nome do site.

É interessante freezar que somente os que acessarem a URL com o código vulnerável serão atacados.

#### Persistence XSS:

Este tipo ocorre quando o atacante consegue injetar um código malicioso e o mesmo fica inserido na página permanentemente, exemplos destes podem ser: comentários de um site e upload de arquivos.

Usarei o mesmo exemplo do HTML Injection nele o usuário pode upar arquivos para um servidor e o nome do mesmo é refletido e armazenado na página, como na imagem:


![](https://i.imgur.com/GMPGeM9.png)

O código html da página é este:

```html
<!DOCTYPE html>
<html>
	<body>
		<table border="1">
			<tr>
				<td>Nome:</td>
				<td>Tamanho:</td>
				<td>Link:</td>
			</tr>
			<tr>
				<td>exemplo</td>
				<td>1.5MB</td>
				<td>exemplo.jpg</td>
			</tr>

		</table>
	</body>
</html>
```
Neste caso o atacante pode injetar uma tag img ou script, sendo elas: ```<img src=x onerror="alert(1)">```, ```<script>alert(1)</script>```.

E este é o resultado:
![](https://imgur.com/6dBr6Fg.png)


#### DOM-Based XSS:	

Este tipo é muito interessante, e o mais difícil de se entender entre os três, nos outros dois tipos o parâmetro infectado era enviado para ao servidor e o mesmo devolvia uma página HTML com o código injetado nela para o usuário, ou seja o servidor não realizava uma sanitização corretamente, mas neste caso não há nenhum parâmetro indo para o servidor, o erro ocorre totalmente no lado cliente.

Este é o exemplo do HTML de uma página vulnerável:

```html
<html>
	<head>
		<title>Custom Dashboard </title>
	</head>
	Main Dashboard for
	<script>
		var pos=document.URL.indexOf("context=")+8;
		document.write(decodeURI(document.URL.substring(pos,document.URL.length)));
	</script>
</html>
```

A parte vulnerável fica dentro da tag script, nela é criada uma variável chamada pos, e nela é contido tudo digitado após o "=",  depois tudo digitado é escrito na página usando o comando "document.write". A url com o ataque é parecida com esta:

https://example.com/teste.html#context=%3Cscript%3Ealert(1)%3C/script%3E


Bom, você deve estar pensando que esse é um caso de XSS reflected né? Mas a resposta é não. Para isto precisamos entender duas coisas, a primeira é que para que haja um xss refletido o código com o payload deve ser enviado para o servidor e o mesmo deve devolver uma página HTML vulnerável e a segunda é que se você prestar a atenção há um "#" na url ao invés de um "?", o primeiro carácter é usado em sites que só tem uma página(também chamados de single page), assim o conteúdo dela pode ser manipulado usando Javascript, sem precisar que sejam feitas requisições para o servidor.

### Exemplo:

Em 2015 o pesquisador Jouko Pynnönen descobriu uma vulnerabilidade no serviço de email do yahoo que permitia um stored XSS, isto foi causado pela recente adição do yahoo que permitia o envio de imagens nos emails por injeção de código HTML.

Jouko percebeu que ao tentar injetar algum atributo que tivesse um valor booleano, o site removeria apenas o valor deixando o atributo, como neste exemplo:

```html
<INPUT TYPE="checkbox" CHECKED="hello" NAME="check box">
```
viraria:
```html
<INPUT TYPE="checkbox" CHECKED= NAME="check box">
```

isto pode parecer inofensivo mas para o html o valor de CHECKED seria  ```NAME=”check``` pois ele permite um ou mais caracteres de espaço ao redor de um sinal de igual, quando o valor de um atributo não foi citado.

Então, Jouko inseriu o seguinte payload:

```html
<img ismap='xxx' itemtype='yyy style=width:100%;height:100%;position:fixed;left:0px;top:0px; onmouseover=alert(/XSS/)//'>
```  

Logo, o filtro transformou isto em:

```html
<img ismap=itemtype=yyy style=width:100%;height:100%;position:fixed;left:0px;top:0px; onmouseover=alert(/XSS/)//>
```

Assim, toda vez que alguém passasse o mouse na imagem o código xss seria executado.

O pesquisador recebeu $10000 de recompensa.

### Como explorar:

Como citado acima, esta é uma das vulnerabilidades mais encontradas e também uma das mais bem pagas, logo o pesquisador pode tentar atrelar outras vulnerabilidades para chegar a ela, por exemplo: ao encontrar uma Open Redirect Vulnerability e tentar levar o usuário à um XSS.

Também é interessante o pesquisador entender o funcionamento dos filtros do site, tem uma dica sobre isso [aqui.](https://github.com/T635/DI.WE.H/blob/master/HTML%20Injection.md#como-explorar) Não tenha medo de perder tempo entendendo como o filtro funciona para então, montar o payload.

Em caso de sites single page fique atento a como é utilizada a hash, ela pode te ajduar a encontrar um DOM based XSS, para isto não tenha medo de usar o modo desenvolvedor do navegador para analisar o javascript da página. Abaixo há uma lista de funções que se referem a hash, procure-as no source elas te ajudarão a montar o payload.

|**Javascript Functions**|
|--------------------|
|document.write()|
|document.writeln()|
|document.domain|
|someDOMElement.innerHTML|
|someDOMElement.outerHTML|
|someDOMElement.insertAdjacentHTML|
|someDOMElement.onevent|
|      **jQuery Functions**|
|add()|
|after()|
|append()|
|animate()|
|insertAfter()|
|insertBefore()|
|before()|
|html()|
|prepend()|
|replaceAll()|
|replaceWith()|
|wrap()|
|wrapInner()|
|wrapAll()|
|has()|
|constructor()|
|init()|
|index()|
|jQuery.parseHTML()|
|$.parseHTML()|

Há alguns casos em existem parâmetros que identificam o tipo de codificação da página, como utf-8, fique atento a eles pois podem te ajudar a "bypassar" o filtro ao substituí-lo.

Quem deseja se aprofundar nesta vulnerabilidade recomendo dar uma lida [neste artigo](https://owasp.org/www-community/xss-filter-evasion-cheatsheet), em inglês, da OWASP-Open Web Application Security Project.