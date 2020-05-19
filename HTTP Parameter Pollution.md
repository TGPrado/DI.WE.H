# HTTP Parameter Pollution

### O que é:
O HTTP Parameter Pollution, ou HPP, ocorre quando um atacante manipula os parâmetros enviados, seja injetando parâmetros extras ou manipulando os existentes, o que pode causar uma reposta inesperada do servidor. Esta falha pode ocorrer tanto no Client-side quanto no Server-side, no primeiro a falha pode gerar efeitos inesperados, como redirecionamento do usuário e injeção de dados, já do outro lado, esta falha pode ser considera grave, pois pode fazer alterações no banco de dados e adquirir informações sensíveis do servidor.

Esta falha já existe a bastante tempo, entretanto nos últimos anos, devido ao aumento da complexidade dos sites, houve também um aumento no número de parâmetros o que trouxe esta vulnerabilidade de volta à tona.

### Funcionamento técnico:

No artigo sobre Client e Server side expliquei como o servidor faz para pegar os parâmetros enviados à ele, mas o que acontece se, por exemplo, duplicarmos um parâmetro? Como a aplicação reagirá? Cada aplicação reage de uma forma diferente a esse problema, se usarmos php + apache será usado o último parâmetro, se for usado ASP.NET + IIS será usado uma concatenação de todos os parâmetros repetidos. Segue imagem da tabela de tratamento das linguagens e servidores:

![](https://www.acunetix.com/wp-content/uploads/2012/02/photo-12.jpg)
###### Fonte: https://www.acunetix.com/wp-content/uploads/2012/02/photo-12.jpg


#### Server-Side:
Imagine que ao fazer uma transferência no seu banco você envie uma request do tipo GET para: http://banco.com/transfer?para=thiago&valor=1000 e que o banco faça uma request, interna, a qual você não tem acesso, para http://pagamentos.com:7979/?de=carlos&para=thiago&valor=1000 . Tendo isto em vista, o atacante pode simplesmente adicionar, ao fim de sua request "&de=henrique", assim quando a request interna for feita, ela ficará assim: http://pagamentos.com:7979/?de=carlos&para=thiago&valor=1000&de=henrique , caso o sevidor seja um php + apache o parâmetro "de" usado será o último, logo a transferência será feita no nome de henrique. 

Caso o atacante esteja usando uma proxy para modificar as requests é necessário fazer a codificação de alguns caracteres nas urls, caso isto não seja feito o servidor pode não entender o conteúdo enviado, a tabela abaixo mostra essa codificação.

Caracter | Codificação 	|	Caracter | Codificação|
-------- | -------------|   -------- | -----------|
espaço   |     %20		|	```#```  |     %23    |
$        |     %24      |	   %     |	   %25	  |
&        |     %26      |      @     |	   %40	  |
`        |     %60		|	   /     |     %2F    |
:        |     %3A      |      ;     |     %3B	  |
<        |     %3C      |	   =     |     %3D	  |
```>```  |     %3E      |      ?     |     %3F    |
[        |     %5B      |      \     |     %5C	  |
]        |     %5D      |      ^     |     %5E	  |
{        |     %7B      |   ```|```  |     %7C	  |
}        |     %7D      |      ~     |     %7E	  |
+        |     %2B		|	   ,     |     %2C	  |

	
Esta vulnerabilidade fica mais interessante quando ela é atrelada a outras,por exemplo, o atacante pode manipular os parâmetros para "bypassar" filtros e injetar códigos SQL, que é uma linguagem de programação usada para lidar com banco de dados, então imagine o atacante retornar toda a base de usuarios e senhas de uma rede social, esta vulnerabilidade tem um nome específico: SQL injection, ela será explicada mais para frente.

#### Client-Side:

O HPP neste lado ocorre quando o atacante manipula os parâmetros de modo que haja a injeção deles em links ou atributos src, como o da imagem ou do svg.

Vamos levar em conta o seguinte código para este exemplo:

```php
<? $val=htmlspecialchars($_GET['par'],ENT_QUOTES); ?>
<a href="/page.php?action=view&par='.<?=$val?>.'">View Me!</a>
```

Na primeira linha é requerido o parâmetro 'par' e o mesmo é transformado em sua respectiva entities html, explarei sobre ela no próximo artigo, entretanto é necessário saber que elas são usadas para transformar caracteres especiais do html em caracteres com o mesmo valor visual mas com significados diferentes.
Enfim, na segunda linha é criado um link, o qual tem dois parâmetros o action com o valor de view, e o  'par' com o valor do parâmetro enviado, podemos então substituir 'par' por algo como par=123%26action%3Dedit, assim quando o valor for adquirido pelo ```$_GET``` e o link for criado ele ficará assim: 
```<a href="/page.php?action=view&par=123&amp;action=edit">View Me!</a>```.

Como o server-side, esta vunerabilidade fica mais interessante atreladas a outras, por exemplo, podemos juntá-la com um HTML injection, com um Cross-Site Scripting, ou até mesmo com um Open Redirect Vulnerability.

### Exemplo:

Em 2015, foi reportada uma vulnerabilidade para a empresa de bug bounty, Hackerone, neste caso a organização dispunha de uma funcionalidade que permitia o compartilhamento de páginas ou reports em redes sociais, como o facebook ou o twitter.

O pesquisador percebeu que ao ativar essa funcionalidade estando neste link, por exemplo:

https://hackerone.com/blog/introducing-signal

era criado um link de compartilhamento igual a este:

https://www.facebook.com/sharer.php?u=https://hackerone.com/blog/introducing-signal

o pesquisador resolveu, então injetar um parâmetro u com o valor https://vk.com/durov ao fim de sua url do hackerone, ela ficou assim:

https://hackerone.com/blog/introducing-signal?&u=https://vk.com/durov

então, quando a funcionalidade fosse ativada seria gerado um link igual este:

https://www.facebook.com/sharer.php?u=https://hackerone.com/blog/introducing-signal?&u=https://vk.com/durov

como vocês podem perceber pela extensão do arquivo, se tratava de um php, logo o argumento usado foi o último, gerando assim a vulnerabilidade.

Este report pode ser lido [aqui](https://hackerone.com/reports/105953), pesquisador recebeu  $500.

### Como explorar:

Para explorar esta vulnerabilidade de maneira efetiva, o pesquisador deve ter um conhecimento básico sobre as outras vulnerabilidades, além de estar atento a como a aplicação reage a injeção de parâmetros repetidos, pois em alguns casos a mesma pode filtrar apenas o primeiro, e usar o segundo ou vice e versa.

O pesquisador também pode usar esta vulnerabilidade para adquirir informações sobre o servidor, como o tipo do servidor ou a linguagem usada, pelo método como ele reage a parâmetros duplicados.
