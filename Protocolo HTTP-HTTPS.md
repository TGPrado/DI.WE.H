# Protocolo HTTP
## Começando do começo...
Para que possamos hackear alguma coisa devemos primeiro, entender como ela funciona.

Dito isto explicarei o funcionamento do protocolo HTTP-Hypertext Transfer Protocol, o qual é a base da World Wide Web.
### O que é:
O HTTP é um protocolo da camada de aplicação do modelo TCP/IP, ele foi inventado para que houvesse uma transmissão de dados na World Wide Web.
### Funcionamento:
Este protocolo funciona em cima de um modelo chamado Cliente-Servidor, nele o cliente faz requisições de serviços ou recursos para um servidor. Uma analogia deste modelo no mundo real seriam as pizzarias[servidor] as quais, aguardam as requisições de pizzas, para assim, poderem produzi-las e enviá-las para o cliente.
![](https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png)
###### Fonte:https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png

O HTTP funciona de forma parecida, ao cliente[usuário] acessar um site, o mesmo cria uma conexão e envia para o servidor uma mensagem de *requisição HTTP* solicitando a página web, e o servidor o responde com a mesma, então o navegador interpreta e renderida a página HTML.
### Funcionamento técnico:
 Entendido o sistema Cliente-Servidor, mostrarei exemplos de requisições[requests] e repostas[responses] aplicadas no protocolo HTTP.
 
 Este é um exemplo de request:
```http
GET / HTTP/1.1
Host: www.google.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Connection: close
Cookie: 
```
A princípio isto pode parecer confuso, mas vamos simplificar. 

Tudo o que está escrito acima representa o cabeçalho/header da request ele é usado para enviar todas as informações necessárias para que a conexão ocorra corretamente, além do header podem ser anexados também um corpo/body, nele são colocados os dados que o usuário deseja enviar para o servidor, esses dados são separados por uma linha em branco do header,exemplo:
```http
POST / HTTP/1.1 
Host: localhost:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:76.0) Gecko/20100101 Firefox/76.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Content-Type: text/plain
Content-Lenght: 325
Accept-Language: pt-BR,pt;q=0.8,en-US;q=0.5,en;q=0.3
Connection: keep-alive
Cookie: 

body
```
A primeira linha de toda request define três coisas, o método da requisição, os quais podem ser:

###### GET - O cliente requisita algum dado, como uma página ou uma imagem, ele só deve ser usado para recuperar algum dado.
###### HEAD - Muito parecido com o GET, mas neste caso o cliente requisita somente o header da página.
###### POST - Neste caso, o usuário envia um dado para o servidor, o dado é colocado no corpo da request.
###### PUT - Parecido com o POST, diferindo apenas em como o servidor irá lidar com esse dado, por exemplo:
###### Caso seja necessário atualizar os dados de um usúario, usa-se o método PUT, pois com ele o servidor irá sobrescrever os dados antigos com os novos, gerando somente um registro, com o POST o servidor cria vários registros, um para cada request feita.
###### DELETE - Remoção de algo no servidor.
A segunda coisa definida pela primeira linha, é o caminho dentro do Host para onde os dados devem ir, ou ser solicidatos, a última é a versão do protocolo HTTP.

O restante do header conter diversos items, sendo eles sempre definidos por NOME: VALOR

Os mais comuns são:
 
###### Host - Nome de domínio do servidor. Ex: www.google.com; www.youtube.com
###### User-Agent - Usada para identificar, para o servidor, quem faz a request, ela contém dados como, Navegador e Sistema Operacional.
###### Accept - Exprime quais tipos de dados o cliente é capaz de entender. Ex: text/plain; application/json
###### Content-Type - Indica qual o tipo de dado que o servidor ou o cliente esta enviando. Ex: text/plain; application/json
###### Accept-Language - Indica qual linguagem o cliente entende, geralmente usado para definir se uma página estará em português ou inglês, por exemplo.
###### Connection - Define se a conexão com o servidor deve ser mantida para futuras requests ou não, no primeiro caso seu valor é keep-alive, no segundo close.
###### Cookie - Funcionam como um identificador e mantenedor de sessão; Por definição o HTTP é um protocolo stateless, isso significa que ele não mantém estado, ou seja se você fizer login na sua rede social e tentar fazer outra ação como, mandar uma mensagem, você teria que se autenticar de novo, pois nao há nada que defina você como logado, por isto foram definidos os cookies, com eles o usuário não precisa se autenticar toda a vez. Entretanto, eles precisam ser enviados em todas as requests subsequentes.
###### Referer - Contém o endereço de onde a requisição foi originada. Isto é usado pelo servidor para saber de onde os visitantes de seu site se originam.
###### Origin - Este header é muito parecido com o Referer, pois ambos indicam e onde a solicitação foi originada, no entando este indica só o nome do site, não o caminho todo. Exemplo: Referer: exemplo.com/artigos/protocolo%20HTTP%20HTTPS Origin: exemplo.com

Enviada uma request, agora analisaremos a response do servidor.

```http
HTTP/1.1 200 OK
Date: Mon, 11 May 2020 19:45:35 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=UTF-8
Strict-Transport-Security: max-age=31536000
Server: gws
X-Frame-Options: SAMEORIGIN
Set-Cookie: 
Connection: close
Content-Length: 194814
```
Essa é a response para a primeira request deste artigo, além deste conteúdo, há também um body o qual não anexarei por ser muito grande.

Vamos analisar os header desta response,na primeira linha há duas coisas, a versão do protocolo HTTP usado e o status da response, eles podem ser as seguintes:

###### 1XX: Passa informações, se a solicitação foi aceita, ou se o processo continua em desenvolvimento.
###### 2XX: A solicitação foi executada com sucesso.
###### 3XX: Indica que há a necessidade de redirecionamento para que a solicitação possa ser concluída.
###### 4XX: Mostra que houve um erro na solicitação por parte do cliente.
###### 5XX: Indica que o servidor não pôde responder a solicitação.

As seguintes linhas mostram headers comuns das responses, sendo eles:

###### Date: Data em que a response foi originada.
###### Expires: Indica quando o conteúdo deve ser considerado desatualizado, neste caso o valor -1 significa que o conteúdo expira imediatamente após ser enviado.
###### Cache-Control: Define políticas de cache.
###### Etag: Identifica uma versão específica de algum recurso, ele permite que um servidor não envie a resposta completa, o que permite uma maior velocidade.
###### Server: Define informações sobre o servidor.
###### Set-Cookie: Usado para o servidor, enviar cookies para o cliente.
###### X-Frame-Options: Indica se o navegador deve ou não renderizar uma página em ```<frame>```,```<iframe>```.




### HTTP vs. HTTPS:

A diferença entre o Hypertext Transfer Protocol e o Hyper Text Transfer Protocol Secure, como o próprio nome nos induz é na questão da segurança, o segundo é uma junção do HTTP e do protocolo SSL/TLS,o último permite que dados sejam transmitidos por meio de uma conexão criptografada e que o servidor e o cliente sejam autênticos. O HTTPS se fez necessário pois, principalmente as conexões Wi-fi estão sucetíveis a ataques Man-in-the-Middle na qual um atacante engana o servidor e o cliente para enviarem as request e responses para ele vide a imagem:

![](https://std1.bebee.com/br/pb/83771/db1a1182/900)
###### Fonte: https://std1.bebee.com/br/pb/83771/db1a1182/900