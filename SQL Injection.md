# SQL Injection

### O que é:
Esta vulnerabilidade ocorre quando, advinhem, o desenvolvedor não sanitiza os dados enviados; quando encontrada o atacante pode injetar códigos de modo que o mesmo pode, retornar toda a base de dados, modificar o banco de dados e excluí-los. Imaginem o problema que um atacante pode gerar ao retornar todo o banco de dados de uma rede social, de uma folha de pagamento, ou até modificar o valor que receberá.

### SQL:

Structured Query Language, ou Linguagem de Consulta Estruturada é uma linguagem usada para ser fazer consultas em bancos de dados relacionais. 

Estes são formados por tabelas, as quais possuem linhas e colunas, as colunas representam um tipo de dado e as linhas os dados como no exemplo abaixo.

![](https://lh3.ggpht.com/franciscogpneto/SMVjY4tWShI/AAAAAAAAGpw/Uqm0mEtEt3o/image_thumb%5B3%5D.png)
###### source: https://lh3.ggpht.com/franciscogpneto/SMVjY4tWShI/AAAAAAAAGpw/Uqm0mEtEt3o/image_thumb%5B3%5D.png

Agora darei alguns exemplos de comandos sql para o banco de dados acima.

Caso queira retornar todos os dados da primeira tabela podemos usar o comando
```sql
DESCRIBE STUDENT;
```
ou
```sql
SELECT * FROM STUDENT;
```
para se retornar uma coluna específica da tabela, usa-se
```sql
SELECT Name from STUDENT;
```
para retornar um valor específico da tabela usa-se
```sql
SELECT Name from STUDENT where Name='Smith';
```
agora caso se queira retornar valores com base em dois argumentos usa-se
```sql
SELECT Name, StudentNumber from STUDENT where Name='Smith' and StudentNumber=17;
```
para retornar toda uma linha com base em um valor
```sql
SELECT * from STUDENT where Name='Smith';
```
### Funcionamento técnico:

Existem dois tipos de SQL Injection, normal e o blind.

#### Normal SQL Injection:

Este é o tipo mais simples, ele ocorre quando os dados são passados seja via get ou post e o servidor os anexa a query sql sem a devida sanitização, retornando, os resultados e até mesmo erros para o cliente.

Estes dados podem ser vistos em urls deste tipo:

```
http://teste.com?id=1
```
```
http://exemplo.com?user=thiago
```
ou em um formulário passado via post.

Isto é um exemplo de um código php vulnerável, onde o desenvolvedor simplesmente anexou os dados passado na url:

```php
<? php
$sqlQuery = "SELECT * FROM Products WHERE ID = " . $_GET["id"];
?>
```
caso o atacante feche as aspas duplas, e injete um or seguido de uma verdade absoluta ele pode ter acesso a todos os produtos.

Esta é uma id maliciosa: ```" or '1'='1'"```, neste caso o valor do id não existe na tabela, isto retorna falso, como injetamos um or e uma verdade, tem-se falso ou verdadeiro = verdadeiro, toda a base de dados é retornada.

O atacante também poderia injetar ```" or '1'='1';--``` isto fecharia a query e comentaria tudo após os dois traços.


#### Blind SQL Injection:

Este tipo é muito complexo e difícil de identificar pois o atacante não tem acesso aos erros e nem aos dados.Logo ele deve então, tentar enviar querys em que ele consiga identificar o SQL Injection mesmo sem os dados.

Vamos supor que haja um código igual ao do exemplo acima, mas que, desta vez os dados não sejam retornados ao usuário.

```php
<? php
$sqlQuery = "SELECT * FROM Products WHERE ID = " . $_GET["id"];
?>
```

Que tipo de pesquisa poderia ser feita, para identificar o injection? Simples, vamos colocar um sleep no final da query, assim o servidor demoraria o tempo setado para responder, e com base nisso o cliente pode identificá-la.

Para isto o atacante deve injetar: ```" and SLEEP(15)"``` caso o servidor demore 15 segundos para responder, o mesmo é vulnerável a blind SQL Injection.

### Exemplo:

Em 2014 um pesquisador descobriu uma falha do tipo Blind SQL Injection no yahoo sports ela se dava em um campo, passado via GET, que fazia pesquisas sobre jogares no banco de dados.

A url vulnerável era esta:  http://sports.yahoo.com/nfl/draft?year=2010&type=20&round=2 ela retornava essa pesquisa:

![](https://imgur.com/EApdO8p.png)

O pesquisador resolveu então, alterar o valor do campo year para ```2010--```, e recebeu isto:
![](https://imgur.com/6qdVFXP.png)

Quando percebeu que o resultado foi alterado, o mesmo entendeu que havia ali uma falha,então fez a seguinte pesquisa: http://sports.yahoo.com/nfl/draft?year=(2010)and(if(mid(version(),1,1)='5',true,false))&type=20&round=2

Esta consulta fazia uma pesquisa pela versão do banco de dados, e caso o primeiro numero fosse igual a 5, a consulta retornaria resultado, neste caso ela não retornou nada.

![](https://imgur.com/2vFLCtN.png)

Isto se trata de um blind SQL Injection já que o atacante não consegue retornar o que quiser do servidor, pois o mesmo está configurado para retornar somente jogadores, mas o pesquisador consegue analisar várias coisas usando a resposta correta como exemplo.

O hacker recebeu um bounty de $3705 pela falha.

### Como explorar:

Para explorar tente substituir o valor dos parâmetros por valores maliciosos, as vezes uma simples ```'``` pode te dar a indicação da falha; use e abuse dos caracteres de comentários, eles devem ser usados para tentar gerar uma consulta correta excluindo o restante da query, também é interessante freezar que o pesquisador pode simplesmente fechar o que foi passado, finalizar a consulta com um ponto e vírgula e criar outra exemplo: ```";select * from teste;--```.

Fique atento a possíveis falhas do tipo Blind, não desista caso não consiga retornar valores, elas também são bastante perigosas; a função SLEEP,é sua amiga neste caso.

Por último mas não menos importante, use também outras palavras reservadas do sql, como UNION, para juntar mais de uma query.