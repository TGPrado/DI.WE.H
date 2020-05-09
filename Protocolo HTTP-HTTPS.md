# Protocolo HTTP
## Começando do começo...
Para que possamos hackear alguma coisa devemos primeiro, entender como ela funciona.

Dito isto explicar o funcionamento do protocolo HTPP-Hypertext Transfer Protocol, o qual é a base da World Wide Web.
### O que é:
O HTTP é um protocolo da camada de aplicação do modelo TCP/IP, ele foi inventado para que houvesse uma transmissão de dados na World Wide Web.
### Funcionamento:
Este protocolo funciona em cima de um modelo chamado Cliente-Servidor, nele o cliente faz requisições de serviços ou recursos para um servidor. Uma analogia deste modelo no mundo real seriam as pizzarias[servidor] as quais, aguardam as requisições de pizzas, para assim, poderem produzi-las e enviá-las para o cliente.
![](https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png)
###### Fonte:https://upload.wikimedia.org/wikipedia/commons/1/1c/Cliente-Servidor.png

 O HTTP funciona de forma parecida, ao cliente[usuário] acessar um site, o mesmo cria uma conexão e envia para o servidor uma mensagem de *requisição HTTP* solicitando a página web, e o servidor o responde com a mesma.
### Funcionamento técnico:
 O método usado pelo HTTP para enviar as requisições e as respostas, a partir de agora as chamarei de resquests e responses, respectivamente.
 
 Este é um exemplo de request:
```
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
```
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
###### Cookie - Funcionam como um identificador de sessão; Por definição o HTTP é um protocolo stateless, isso significa que ele não mantém estado, ou seja se você fizer login na sua rede social e tentar fazer outra ação como, mandar uma mensagem, você teria que se autenticar de novo, pois nao há nada que defina você como logado, por isto foram definidos os cookies, com eles o usuário não precisa se autenticar toda a vez. Entretanto, eles precisam ser enviados em todas as requests subsequentes.