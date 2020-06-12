# Server Side Request Forgery

### O que é:

Como pode ser visto pelo nome esta falha é bem parecida com o CSRF, só que ela ocorre no outro lado.

Servidores geralmente necessitam de dados vindos de outros lugares, seja de banco de dados, api, ou até de aplicações externas, como o twitter para compartilhamento de links, entretanto temos um problema quando o mesmo permite que estes dados sejam solicitados sem nenhum controle.


### Funcionamento técnico:

Para abranger as coisas da melhor forma possível, dividirei esta falha em **Blind** e **Normal** SSRF.

#### Normal SSRF:

Começarei esta explicação com uma imagem

![](https://i.imgur.com/pHp0Eru.png)
###### source:https://www.netsparker.com/blog/web-security/server-side-request-forgery-vulnerability-ssrf/

nela é visível que as aplicações dentro do lado servidor, como o banco de dados são protegidos pelo firewall isto é feito para que ninguém de **fora da rede** acesse esses serviços, então o que o atacante faz? forja uma request, seja para o banco de dados ou para os próprios arquivos do sistema em nome do servidor.

Vou dar um exemplo, de um código php vulnerável.

```php
<?php

/**
*isset é usado para verificar se o parâmetro url foi passado
*caso tenha sido, o bloco if é executado
*/
if (isset($_GET['url'])){
$url = $_GET['url'];

/**
*aqui o arquivo cuja url foi passada no parâmetro é aberto e salvo em $image
*/
$image = fopen($url, 'rb');

/**
* esta função é usada para setar headers na response.
*/
header("Content-Type: image/png");

/**
* O valor é retornado para o usuário com fpassthru.
*/
fpassthru($image);}
```

Neste caso qualquer valor passado no parâmetro é aberto e mostrado para o usuário o atacante poderia então, acessar um arquivo como o ```/etc/passwd``` ele contém senhas criptografas usadas pelos usuários do sistema, o mesmo poderia também baixar arquivos no servidor e até upar arquivos usando o ```ftp://```.

Como pode ser visto no código php, o conteúdo do arquivo é retornado para o usuário, com isso o mesmo pode analisar quaisquer arquivos do servidor.

### Blind SSRF:

Não me aprofundarei neste tipo de SSRF pois ele é muito avançado deixarei para explicar mais sobre ele no artigo de Remote Code Execution

Bom, como devem imaginar este caso ocorre quando o atacante não tem o retorno dos dados do servidor, ele costuma ocorrer muito no cabeçalho Referer já que alguns servidores costumam fazer requests para o valor deste cabeçalho, para identificar esta falha crie um servidor Web e analise se ao mandar a request para o servidor alvo você receberá uma em seu servidor.

Mesmo que não consiga ter o retorno dos dados, fique atento a outras falhas que podem ocorrer; tente enviar códigos que analisem vulnerabilidades no servidor ou em sistemas do back-end.

### Exemplo:

Em 2016, Brett encontrou uma falha SSRF no servidor da ESEA, a E-Sports Entertainment Association, para achá-la ele pesquisou por ```site:https://play.esea.net/ ext:php``` no google, isto retorna todas as páginas de play.esea com a extensão php, ele então encontrou o seguinte link ```https://play.esea.net/global/media_preview.php?url=``` o mesmo percebeu que o site usava o parâmetro 'url' para renderizar dados vindos de outros locais.

O pesquisador tentou fornecer ```http://ziot.org``` como parâmetro mas o site só aceitava endereços com .png no nome, então Brett forneceu a seguinte url: ```http://ziot.org/?1.png``` assim, ele passaria o filtro usando 1.png como parâmetro para seu site, isto funcionou e o site renderizou o conteúdo da página "ziot.org".

O mesmo recebeu $1000 por seu bug, e o artigo pode ser lido [aqui](https://buer.haus/2016/04/18/esea-server-side-request-forgery-and-querying-aws-meta-data/).


### Como explorar:

Explorar esta falha não é tão difícil, analise quando o servidor esta renderizando, redirecionando, ou adquirindo dados de outros lugares, descoberto isto, tente acessar dados do servidor **mas tome cuidado, caso esteja fazendo bug boutny leia o escopo e veja se ele permite o acesso a arquivos do mesmo, não hesite em entrar em contato para perguntar isso**.

Caso não consiga executar comandos do lado servidor, tente atrelá-la a outras falhas client-side, por exemplo, caso um site faça uma request na url e a anexe a página tente substituir aquele valor por uma imagem contendo um XSS, ou por um site com XSS.

Não se esqueça de ficar atento a possíveis Blind SSRF.