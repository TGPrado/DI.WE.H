
# Protocolo HTTP e HTTPS

### Começando do começo...
Para que possamos hackear alguma coisa devemos primeiro entender seu funcionamento.

Dito isto veremos o como funciona o protocolo HTTP (*Hypertext Transfer Protocol*), que é a base da *World Wide Web*.

### Conceito:
O HTTP é um protocolo da camada de aplicação do modelo TCP/IP que foi criado para possibilitar a transmissão de dados na internet.

### Funcionamento:
Este protocolo funciona em cima de um modelo chamado cliente-servidor, onde o cliente faz requisições a um servidor, que como resposta lhe envia informações. Uma analogia para este modelo pode ser feita com a relação entre as pizzarias (servidores) e seus respectivos clientes, onde estes solicitam suas pizzas e as aguardam como resposta.

<p align="center">
 <img src="https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png">
</p>
<h6 align="center">Fonte: https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png</h6>

 Sendo assim, quando um cliente acessa um site, ele está simplesmente solicitando ao servidor uma página web, que lhe é retornada (uma vez que exista) e renderizada pelo seu navegador.

### Funcionamento Técnico:
Entendido o sistema cliente-servidor, veremos agora exemplos de requisições (*requests*) e repostas (*responses*) aplicadas ao protocolo HTTP:
 
  Este é um exemplo de uma requisição:
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

  Tudo o que está escrito acima representa o cabeçalho (*header*) da requisição. Ele é usado para enviar todas as informações necessárias para que a conexão funcione adequadamente. Além do cabeçalho, podem ser anexados também um corpo (*body*), no qual são colocados os dados que o usuário deseja enviar para o servidor. Observe que o cabeçalho e o corpo da requisição são separados por uma quebra de linha.
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

#### A primeira linha de toda requisição é usada para definir três coisas:
1. O método da requisição, que pode ser:

> <h5>GET - O cliente requisita algum recurso, como uma página ou uma imagem.</h5>
> <h5>HEAD - Muito parecido com o GET, porém aqui o cliente requisita somente o cabeçalho (*header*) da página.</h5>
> <h5>POST - O cliente está enviando dados que estão contidos no corpo da requisição.</h5>
> <h5>PUT - Parecido com o POST, diferindo apenas em como o servidor irá lidar com os dados enviados. Por exemplo: Caso seja necessário atualizar os dados de um usúario, usa-se o método PUT, pois com ele o servidor irá sobrescrever os dados antigos com os novos, gerando somente um registro, com o POST o servidor cria vários registros, um para cada requisição feita.</h5>
> <h5>DELETE - O cliente requisita que algum recurso seja excluído do servidor.</h5>

2. A rota da aplicação para a qual a requisição está sendo feita.

3. A versão do protocolo HTTP.

#### O restante do header contém diversos items, sendo eles sempre definidos por `CHAVE: VALOR`. Os mais comuns são:
 
> <h5>Host - Nome de domínio do servidor. Ex: www.google.com; www.youtube.com</h5>
> <h5>User-Agent - Usado pelo servidor para identificar quem está fazendo a requisição. Ela contém dados como navegador e sistema operacional do cliente.</h5>
> <h5>Accept - Exprime quais tipos de dados o cliente é capaz de entender. Ex: text/plain; application/json</h5>
> <h5>Content-Type - Indica qual o tipo dos dados que o servidor ou o cliente está enviando. Ex: text/plain; application/json</h5>
> <h5>Accept-Language - Indica qual linguagem o cliente entende. Usado para definir se uma página estará em português ou inglês, por exemplo.</h5>
> <h5>Connection - Define se a conexão com o servidor deve ser mantida para futuras requisições, no primeiro caso seu valor é keep-alive, no segundo close.</h5>
> <h5>Cookie - Funciona como um identificador e mantenedor de sessão. Por definição o HTTP é um protocolo stateless, o que significa que ele não mantém estado, ou seja, se você fizer login na sua rede social e tentar fazer outra ação, como mandar uma mensagem, você teria que se autenticar novamente, pois nao há nada que defina você como logado, por isto foram definidos os cookies, com eles o usuário não precisa se autenticar toda vez. Entretanto, eles precisam ser enviados em todas as requisições subsequentes.</h5>
> <h5>Referer - Contém o endereço de onde a requisição foi originada. Isto é usado pelo servidor para saber de onde os visitantes de seu site se originam.</h5>
> <h5>Origin - Este header é muito parecido com o Referer, pois ambos indicam de onde a solicitação foi originada, no entando este indica só o nome do site, não o caminho todo. Exemplo: Referer: exemplo.com/artigos/protocolo%20HTTP%20HTTPS Origin: exemplo.com</h5>

#### Uma vez feita a requisição, agora analisaremos a resposta do servidor.

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
Essa é a resposta para a primeira requisição que fizemos, além deste conteúdo há também um corpo, porém não o anexarei por ser muito grande.

Vamos analisar os cabeçalhos dessa resposta, na primeira linha há duas coisas: A versão do protocolo HTTP usado e o status da resposta, que podem ser:

> <h5>1XX: Passa informações: Se a solicitação foi aceita, ou se o processo continua em desenvolvimento.</h5>
> <h5>2XX: A solicitação foi executada com sucesso.</h5>
> <h5>3XX: Indica que há a necessidade de redirecionamento para que a solicitação possa ser concluída.</h5>
> <h5>4XX: Mostra que houve um erro na solicitação por parte do cliente.</h5>
> <h5>5XX: Indica que o servidor não pôde responder a solicitação.</h5>

As seguintes linhas mostram cabeçalhos comuns das respostas, são eles:

> <h5>Date: Data em que a resposta foi originada.</h5>
> <h5>Expires: Indica quando o conteúdo deve ser considerado desatualizado, neste caso o valor -1 significa que o conteúdo expira imediatamente após ser enviado.</h5>
> <h5>Cache-Control: Define políticas de cache.</h5>
> <h5>Etag: Identifica uma versão específica de algum recurso. Permite assim que o servidor não envie a resposta completa, proporcionando uma maior velocidade.</h5>
> <h5>Server: Define informações acerca do servidor.</h5>
> <h5>Set-Cookie: Usado para o servidor enviar cookies para o cliente.</h5>
> <h5>X-Frame-Options: Indica se o navegador deve ou não renderizar uma página em ```<frame>```,```<iframe>```.</h5>

### HTTP vs. HTTPS:

A diferença entre o *Hypertext Transfer Protocol* e o *Hyper Text Transfer Protocol Secure*, como o próprio nome nos induz, está na questão da segurança.
O segundo é uma junção do protocolo HTTP e do protocolo SSL/TLS. O SSL/TLS permite que dados sejam transmitidos por meio de uma conexão criptografada e que o servidor e o cliente sejam autênticos. O HTTPS se faz necessário principalmente porque as conexões Wi-fi estão suscetíveis a ataques *Man-in-the-Middle*, no qual um atacante engana o servidor e o cliente para que as requisições e respostas passem por ele, vide a imagem:

<p align="center">
 <img src="https://std1.bebee.com/br/pb/83771/db1a1182/900">
</p>
<h6 align="center">Fonte: https://std1.bebee.com/br/pb/83771/db1a1182/900</h6>
