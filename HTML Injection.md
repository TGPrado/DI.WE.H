# HTML Injection

### O que é:

Esta vulnerabilidade ocorre quando o usuário consegue injetar tags html no site e modificar a página, imagine um usuário conseguir injetar formulários, solicitando senhas e quando a vítima preenche-lo ela é hackeada. Esse problema ocorre graças a falta de sanitização do site, o mesmo confia nos dados injetados pelo usuário e os usa indiscriminadamente.


### Funcionamento técnico:

Para um melhor entendimento, irei dividir essa vulnerabilidade em dois tipos, o **stored** e o **reflected**.

#### Stored:

Neste caso o html injection fica salvo no site vulnerável, seja por upload de arquivo, envio de comentários ou até mesmo por envio de mensagens em um chat.

Imagine que ao enviar um arquivo para um site, o mesmo fique em uma tabela, tendo seu nome, tamanho e link para acesso, algo parecido com isto:

![](https://i.imgur.com/GMPGeM9.png)

acessando o código html da página o atacante veria isto:
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

Bom, o atacante percebe que o nome do arquivo upado fica dentro da tag ```<td></td>```, ele poderia então tentar upar um arquivo o qual o nome seria um código html, exemplo: ```<img src="https://media1.tenor.com/images/186241d89ba256a3f3c1e105b0668ca2/tenor.gif">``` injetado o código , pode-se ver isto:

![](https://i.imgur.com/FbhDNvn.jpg)

Assim, todos os clientes que acessassem a tabela sofreriam o ataque.

Injetar um código de imagem pode parecer inofensivo, mas não se engane, esta vulnerabilidade é perigosa pois o atacante pode literalmente modificar tudo da página, um caso interessante é injetar códigos de redirecionamento ou de formulários.

#### Reflected:

Este caso é menos perigoso que o primeiro, mas ainda assim pode causar muitos problemas, pois aqui somente os clientes que acessarem a url vulnerável serão atingidos, diferentemente do primeiro, que atinge qualquer usuário.

O caso mais comum em que esta vulnerabilidade é encontrada é em campos de busca, vamos supor que um site contenha um parâmetro GET usado para pesquisar livros em seu servidor, seria comum que a página de resultados contenha um texto "Resultados de: suapesquisa", aqui que mora o perigo, o invasor pode injetar códigos html como o do exemplo acima modificando apenas o valor do parâmetro da url, assim todos que acessaram a url com o código html podem ser atacados.

Este exemplo de código não faz pesquisa em um banco de dados, mas pode ser usado para exemplo.

Página home:
```html
<!DOCTYPE html>
<html>
	<body>
		<form action="destino.php" method="get">
			<div>
				<label >Pesquise seu livro:</label>
				<input type="text" name="q" />
			</div>
			<div >
				<button type="submit">Enviar sua mensagem</button>
			</div>
		</form>
	</body>
</html>
```

O código acima já deve ser facilmente entendido, criou-se um formulário que fazia uma request do tipo GET com o parâmetro "q" para "destino.php", e um botão que tem a função de enviá-lo.

```php
<?php
$new = $_GET["q"];
echo "Resultados para: ".$new
?>
```

O código php acima, apenas mostra o resultado da pesquisa, caso o atacante injete o mesmo código usado anteriormente ele teria uma url parecida com esta:
http://127.0.0.1/destino.php?q=%3Cimg+src%3D%22https%3A%2F%2Fmedia1.tenor.com%2Fimages%2F186241d89ba256a3f3c1e105b0668ca2%2Ftenor.gif%22%3E
assim somente os que acessarem esta url serão atingidos pelo ataque.

#### HTML Entities:

Não há como falar sobre HTML Injection sem ao menos mencionar HTML Entities eles são, assim como o url enconde, codificações para alguns caracteres específicos que poderiam causar problemas, por exemplo quando um usuário injetar um > em algum parâmetro, o servidor deve codificá-lo para ```&gt;``` pois isto tem visualmente o mesmo efeito de >, mas para o código html o significado não é igual. Logo não é possível criar tags HTML com os entities, inviabilizando assim, na maioria dos casos, o HTML injection.

Segue tabela com as principais entities:

Caracter | Codificação
---------|------------
```<```|```&lt;```
```>```|```&gt;```
```&```|```&amp;```
```'```|```&apos;```
```"```|```&quot;```


### Exemplo:

Um pesquisador encontrou, em 2015, uma vulnerabilidade do tipo html injection na página login error da empresa Within Security, o mesmo percebeu que ao errar o login em sua conta, a página gerava um parâmetro error, deixando a url assim: https://withinsecurity.com/wp-login.php?error=access_denied
 ele tentou substituir este parâmetro por um valor arbritário e, para sua surpresa, o valor foi refletido na tela. O mesmo enviou a seguinte PoC(prova de conceito) para a companhia esta:

![](https://hackerone-us-west-2-production-attachments.s3.us-west-2.amazonaws.com/000/069/550/9d3c0814dd1f8fcfdbaf311b1b6dc84df82a1a51/Text_Injection_OR_Content_Spoofing_Withinsecurity.jpg?response-content-disposition=attachment%3B%20filename%3D%22Text_Injection_OR_Content_Spoofing_Withinsecurity.jpg%22%3B%20filename%2A%3DUTF-8%27%27Text_Injection_OR_Content_Spoofing_Withinsecurity.jpg&response-content-type=image%2Fjpeg&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIAQGK6FURQ3VFKEHBW%2F20200515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20200515T020322Z&X-Amz-Expires=3600&X-Amz-SignedHeaders=host&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEDgaCXVzLXdlc3QtMiJHMEUCIBN8dq6g%2FRcMTxJW%2BTFRtubEFlkiRw35NGe3NYmjR20mAiEAy%2FevEOL49Izbtmiu2tNrOKiMrCIle9BmN6HXovNagskqvQMIgf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgwwMTM2MTkyNzQ4NDkiDIiA1x1hV50Y4OJoiiqRA6dZojzQRP2jRyAQKsJvAR4AfYLVEyZGrX%2BdxdDiUWbj0aEFXEU2AOELKzjk%2BrNcD7%2FEgXyroXsidGBliTeyflgGTL1qkgMFDaCdSe0X85gLESs8IYmcjeuQm4b1VCxp7jDB92SB8NbJ87UuY5g9M4VwP%2FOzQpLeURtbB5mgk%2Bqvti7Sb2LebhlJINDSf%2BwIxfSBzbrQq5sqe9j5hvrVBvnk3DYp6%2BmM%2BD5tBqYT1734ubRVp7nZAsw0%2FqRImkt%2FpW3Kr3qY7sd%2Bc7%2BeKigh2RaHgp2vJ17IZmEUmw8Eon79X2j7oXkoRb4cNoh7FyppvB%2BNk7hefY9SFclNa5j1H6BGjQT%2BVsAj6R6vPP7EpD15Ef%2FUUJQJEOfHMHJXM0VIhphUkWIvfI60sUQ6Vyj6PIP3LeOLIv9rYgyAb%2BOtSzxgAe6HXRAwu8ieNwhePqHqdhoYu2qDqGUWOdAionve7LlUW090ica07y%2FZe8marvhZueaUFzx401FNzK7j3%2FvSsIuEl77OZrM1lPz%2Fz4j5eT4GMOm89%2FUFOusBLIr2DcsI%2FpR7gbfrzDx5tDFLWg3bBC5U6gGtG45vjwK8dmZCkp75bmO2GJ5OVOD03RFAa1zvWKLrBj5dPUtdB6d1PBBIhKjFrT6N0CxhkPfZIgcdu%2BwlnDm382TJIZfB6au5jQwPMjJ7qQ9yK0%2B3kCydT8lTDLRFXxe7tVYfCNA6qxw%2Fp5b2qXYS8zvjdH1RipS7oqHAkwXusePzhp6QbEeKBy%2BJjtkBpQGHPB8gkFHYhooqtA54j3RhuileFZ%2Bn6SEQeJrpoStDUI3xp4M0zKtFgT55HtU34uBEbUyxyezrhGnHQpMqLkBnPw%3D%3D&X-Amz-Signature=d03b6f242c4405d332f80d8b0e804ca847bd389d26e5caaca18df02c5885610d)


### Como explorar:

Para explorar esta vulnerabilidade o pesquisador deve estar atento ao parâmetros e como eles são adicionados à página, caso eles sejam adicionados no atributo de uma tag, tente fecha-la seja com aspas simples ou aspas duplas, para decidir qual usar analise o html da página usando o inspetor de elementos.

Também é interessante notar como o filtro esta reagindo aos parâmetros com carateres especiais, alguns simplesmente os excluem para estes, tente adicionar um URL encode dentro de outro, por exemplo, sabendo que ```%3C``` corresponde a ```<``` tente então injetar ```%3%3CC```.

Acima, dei um exemplo de filtros server side, agora quando eles estiverem no client side tente usar um proxy para interceptar e modificar a request depois que ela já passou por pelo filtro.
	