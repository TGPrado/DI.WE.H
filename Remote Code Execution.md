# Remote Code Execution

### O que é:

Esta, além de uma falha também pode ser considerada uma categoria, dado isto, não me extenderei muito neste artigo e deixarei para fazer isto nos subsequentes.

RCE ocorre quando o atacante consegue injetar ou executar código malicioso no servidor, podendo no melhor dos casos, levar a uma escalação de privilégios e por fim, à um completo domínio dele.

### Funcionamento técnico:
	
Há basicamente dois modos de explorar esta falha, o primeiro é executando comandos shell no lado servidor, e o segundo é executando funções ou comandos na linguagem de programação usada.

#### Executando comando shell:

Vamos imaginar que um site use esse código php para verificação do ping:

```php
<?php
$domain = $_GET[domain];
echo shell_exec("ping -c 1 $domain");
```
Aqui, o servidor pega o valor injetado no parâmetro domain e usa-o para executar o comando shell, ping, e por fim mostra o resultado ao usuário.

Caso o atacante injete ```www.<example>.com?domain=google.com;id```, o servidor executaria o seguinte comando: ```ping -c1 google.com;id``` o ponto e vírgula indica o fim de um comando e o começo de outro, ao fazer isto o hacker poderia acessar o grupo no qual o usuário do servidor está.

O mesmo também poderia ler o famigerado arquivo ```/etc/passwd``` para isto, injetaria: ```www.<example>.com?domain=google.com;cat /etc/passwd```; ele também pode modificar arquivos, baixar arquivos e etc.

Percebe-se que dependendo de como o servidor esteja configurado, esta falha pode levar à um total domínio do mesmo.

#### Executando funções:

Neste caso o atacante não executa comandos shell, mas chama os usados na linguagem de desenvolvimento, para este exemplo usarei o php, mas a falha pode ocorrer em qualquer linguagem.

```php
<?php
$action = $_GET['action'];
$id = $_GET['id'];
echo call_user_func($action, $id);
```

No código acima são passados, por meio de parâmetros, qual função deve ser executada e seus respectivos argumentos, logo o atacante pode se aproveitar de qualquer função que a linguagem disponha.

É interessante notar que neste caso o atacante, ao invés de injetar comandos shell diretamente, injeta funções que serão executadas, elas podem ser desde comandos php normais até comandos php que executam comandos shell.

Pode-se injetar a seguinte url maliciosa: ```www.<example>.com?action=file_get_contents&id=/etc/passwd```, assim será executado: ```file_get_contents(/etc/passwd)```; o atacante também pode requerir informações do servidor ao chamar a função ```phpinfo```, como nesta url: ```www.<example>.com?action=phpinfo&id=```

### Exemplo:

Em 2017 o hacker yaworsk encontrou um RCE no site da empresa Unikrn, ao acessar o site da empresa o pesquisador percebeu que o mesmo usava AngularJs e ficou atento para possíveis ataques de [Template Injection](Template%20Injection.md), ele pesquisou pelo site mas não encontrou nenhum possível vetor de ataque, até que percebeu uma função que permitia o convite de amigos para a plataforma, yaworsk resolveu testar essa funcionalidade e injetou o seguinte código:```{{7*7}}```, ele recebeu um email igual este:

![](https://i.imgur.com/3GqtiUf.png)

Pronto! foi achado um template injection, mas o pesquisador não se contentou com isto, ele tentou recuperar a versão do Smarty, um template engine php, com o seguinte código: ```{self::getStreamVariable(“file:///proc/self/loginuuid”)}```, foi retornado o seguinte email:

![](https://imgur.com/h3WeuGR.png)

Como pode-ser ver foi retornado o número da versão do php usado, mas yaworsk queria o arquivo ```/etc/passwd```, para isto injetou ``` {php}$s=file_get_contents(‘/etc/passwd’);var_dump($s);{/php}```, entretanto para sua surpresa nada foi retornado, isto ocorreu pois o valor retornado era muito grande, logo o servidor preferiu não retornar um valor vazio. O pesquisador então, resolveu limitar a quantidade de caracteres retornados injetando: ```{php}$s=file_get_contents(‘/etc/passwd’,NULL,NULL,0,100);var_dump($s);{/php}```, isto retornou o esperado.

![](https://imgur.com/ciIFMHb.png)

O pesquisador recebeu $400 por esta falha o report pode ser lido por completo [aqui](https://hackerone.com/reports/164224), mas não fique supreso esta vulnerabilidade costuma ser muito bem recompensada, entretanto, esse programa de bug bounty em específio pagava pouco.

### Como explorar:

Na maioria das vezes esta falha pode ser encontrada em uploads de arquivos, pois ao bypassá-lo você pode injetar uma página php com um código de shell execution, e acessá-la para passar comandos, portanto fique atento a eles.

Também é interessante notar que os programas de bug bounty costumam ter o escopo limitado para esta falha, logo não saia acessando quaisquer arquivos do mesmo, pois isto pode resultar em um não pagamento do bounty.

Caso não esteja familiarizado com Privilege escalation a consulta id dirá o quão vulnerável a aplicação está, logo ao encontrar um RCE execute este comando ou o phpinfo.

Também fique atento a possíveis parâmetros que possam estar ligados a comandos shell, ou à chamada indiscriminada de funções.