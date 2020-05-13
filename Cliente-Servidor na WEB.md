# Cliente-Servidor na WEB

#### O modelo Ciente-Servidor foi explicado no [Protocolo HTTP-HTTPS](Protocolo%20HTTP-HTTPS.md), agora explicarei o mesmo aplicado a WEB.

### Funcionamento técnico:

Para explicar esse modelo devemos separá-lo em dois lados[sides], o do cliente e do servidor.

![](https://feiteiraac.files.wordpress.com/2014/09/server-vs-client-time-zone-difference-problem.jpg)
###### Fonte: https://feiteiraac.files.wordpress.com/2014/09/server-vs-client-time-zone-difference-problem.jpg 

Usando como base a imagem acima, entende-se o funcionamento deste modelo, nele o cliente requisita por exemplo, uma página ao servidor, e o mesmo devolve uma cópia do HTML correspondente a ela, por fim o cliente renderiza o conteúdo em seu navegador.

#### Tecnologias Server-Side:

Este side, também chamado de back-end, é responsável por fazer a análise dos dados enviados de volta pro cliente, por gerenciar o banco de dados, gerenciar a regra de negócio da aplicação e etc.

O funcionamento disso se dá usando linguagens de programação server-side, como php, python, node e etc. Para este artigo, usarei php.

##### Php:

O php, atualmente conhecido como Hypertext Preprocessor, também já foi chamado de personal home page e isto tem um motivo: com ele o programador pode criar conteúdos personalizados para seus visitantes, para isso ele usa código que no final é interpretado, transformado em HTML e enviado ao usuário.

Abaixo mostrarei um código php simples.

Para que você replique este modelo deve primeiro criar um servidor web em sua máquina, não ensinarei isto aqui pois o artigo ficaria muito extenso.

Criado o servidor podemos criar uma página html simples como esta:

```html
<!DOCTYPE html>
<html lang="pt-br">
    <body>
        <form action="destino.php" method="post">
            <p>
                Name:  <input type="text" name="nome" value="thiago"/>
            </p>
                <input type="submit" value="Submit me!"/>
            </p>
        </form>
    </body>
</html>
```
Nela, ao usuário clicar em "Submit me!" é enviado uma request com o método POST e com o parâmetro "nome=thiago" no body.

Parâmetros podem ser definidos como métodos usados para passar informações para o servidor.

Agora precisamos pegar o valor do conteúdo enviado, para isto vamos criar uma página php chamada destino, e colocá-la na mesma pasta do código acima.

```php
<?php
	echo "seu nome eh " . $_POST["nome"] . "!";
?>
```
A página acima começa com uma identificação da linguagem usada, na segunda linha tem-se um echo que é basicamente um comando para mostrar dados em tela, ele concatena por meio de pontos três strings, sendo a segunda a mais interessante, ela é um array contendo todos os dados passados via post para destino.php.

Agora darei um exemplo de request enviada via GET.

```html
<!DOCTYPE html>
<html lang="pt-br">
    <body>
        <form action="destino.php" method="get">
            <p>
                Name:  <input type="text" name="idade" value="17"/>
            </p>
                <input type="submit" value="Submit me!"/>
            </p>
        </form>
    </body>
</html>
```
O código é basicamente o mesmo, diferenciando apenas no atributo method do form e o name e value do input.

```php
<?php
$var = $_GET['idade'];
$var = intval($var);
if ($var <= 18){
echo "Você é jovem";
}else{
echo "Você é adulto";
};
?>
```

Desta vez criei um código um pouco mais complexo, inicialmente peguei o valor do parâmetro 'idade' usando ```$_GET```, depois usei a funcão intval para transformar a variável ```$var``` em um valor inteiro; já na linha 4 criei uma estrutura de decisão para identificar, pela idade, se uma pessoa é jovem ou adulto.

Algo interessante de se perceber é que quando os parâmetros são passados via GET a url muda conforme o parâmetro muda, exemplo:

http://localhost/destino.php?idade=17

Aqui podemos ver que após o nome do arquivo é usado uma "?", ela indica o início dos parâmetros, tudo após é seguido do nome do parâmetro e de seu valor; caso seja necessário mais de um  parâmetro coloca-se um "&", exemplo:

http://localhost/destino.php?idade=17&nome=thiago&profissao=hacker


### Tecnologias Client-Side:

O client-side, também chamado de front-end, é basicamente o que roda em sua máquina; quando é feita uma request ao servidor, e o mesmo devolve um documento HTML, é no lado do client que esse arquivo é processado e renderizado.

No front-end não é possível, por exemplo, acessar arquivos da máquina do usuário, isso ocorre por medidas de segurança, essas linguagens podem somente manipular eventos no navegador.

Exemplos de linguagens client-side são, HTML, javascript e css.

Vou dar um exemplo de uma página simples e interativa com HTML + javascript.

```html
<!DOCTYPE html>
<html>
    <body>
        <h1 id="placar">Placar: 0</h1>
        <button id="button">Votar</button>
    <script>
        var button = document.getElementById("button");
        var placar = document.getElementById("placar");
        var value = 0;
        button.addEventListener("click", ()=>{
            value = value + 1;
            placar.innerHTML = "Placar : " + value; 
        })
    </script>
    </body>
</html>
```

Nesse código, estou criando uma página com um placar e um botão para voto, a mágica ocorre dentro das tags ```<script>```; na sétima linha eu defino uma variável chamada button, com ela é possível fazer manipulações no elemento de id igual a button, o mesmo ocorre com a variável placar, por fim defino uma variável chamada value e coloco seu valor igual a 0.

Agora, irei entrar na parte que de fato conta um voto, para isso usei a função ```addEventListener()```, ela aceita três argumentos, sendo dois obrigatórios, o primeiro define o evento ele pode ser desde um click até o pressionar de um botão, o segundo deve ser uma função que executará algo quando o evento for ativado, neste caso a função incrementa 1 em value e modifica o valor de placar.