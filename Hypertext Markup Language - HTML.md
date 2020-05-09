# HyperText Markup Language - HTML
### O que é:

HTML é, como o próprio nome diz, uma linguagem de marcação ela é atualmente usada na internet para a criação de páginas WEB.

Para se criar páginas interativas e visualmente interessantes costuma-se usar junto ao HTML, CSS-Cascading Style Sheets e JS-Javascript, formando assim a santíssima trindade do front-end.

### Criando páginas web com HTML
No HTML, usa-se tags ou marcações para demarcar como o navegador deve interpretar aquele conteúdo.

Um código html geralmente começa assim:

```html
<!DOCTYPE html>
<html>
	<head>
	</head>
	<body>
	</body>
</html>
```
Na primeira linha do nosso código HTML, definimos para o navegador o tipo do documento, neste caso HTML.

Na segunda linha, abrimos a tag HTML, isso diz para o navegador que tudo que estiver ali dentro deve ser interpretado como HTML, na última linha fechamos o nossa tag HTML, indicando ao navegador que o código terminou.

Na terceira linha com a tag ```<head>```,definimos o cabeçalho do nosso HTML, ele é usado para inserir-se informações sobre nossa página, como o título ,codificação dos caracteres, scripts e etc.

Na quinta linha definimos o corpo do nosso código, tudo escrito lá será exibido ao usuário.

```html
<!DOCTYPE html>
<html>
	<head>
	</head>
	<body>
		<h1> Ola mundo!</h1>
	</body>
</html>
```

No código acima, inserimos uma tag de ```<h1>``` tudo dentro dela será interpretado como um título.

No html existem diversas tags, todas elas podem ser encontradas em https://www.w3schools.com/

Dentre as tags html, existem algumas que usaremos muito daqui para frente, sendo as principais:

```<script></script>``` : Como dito anteriormente, html nao é uma linguagem de programação, mas isso nao significa que não possamos inserir uma linguagem de programação nela, com essa tag podemos injetar códigos Javascript para tornar a página mais dinâmica.

```<img>``` : Com esta tag podemos inserir imagens na nossa página, para isso usamos ```<img src="localização da imagem">``` onde, src é o source ou código/localizacao da imagem.

```<svg></svg> ``` : Com isto podemos inserir Scalable Vector Graphics ou ,gráficos vetoriais escalonáveis, svg possui diversos métodos para desenhar retângulos, círculos ou até imagens.

```<form></form>``` : Com esta tag, iniciamos um formulário, com ele podemos enviar dados para outros locais usando os protocolos HTTP, caso você não tenha lido meu artigo sobre o HTTP [LEIA](Protocolo%20HTTP-HTTPS) antes de continuar.

```<input></input>``` : O input nos permite inserir um local para que o usuário possa digitar algo, seja seu email, sua senha, ou até mesmo um input para pesquisa em um site.

Dentro das tags html, existem também o conceito de elementos e atributos, os primeiros são basicamente um bloco de construção HTML, eles são compostos por tags de início e fim e seu conteúdo, por exemplo isto é um elemento: ```<h1>Ola mundo!</h1>```. Os atributos são informações sobre os elementos, o "src" usado na tag ```<img>``` é um atributo.