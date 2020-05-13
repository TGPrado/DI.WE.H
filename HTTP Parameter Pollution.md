# HTTP Parameter Pollution

### O que é:
Como mencionado anteriormente, parâmetros HTTP são métodos pelos quais se passam informações do cliente para o servidor, esta vulnerabilidade ocorre quando eles não são "higienizados" e o servidor os usa para realizar alguma ação, a periculosidade do HPP geralmente depende de onde ela é executada, caso seja no servidor, alta periculosidade, caso seja no cliente, média periculosidade.Devido ao aumento da complexidade das aplicações WEB fez-se necessário novos parâmetros, gerando assim um aumento na ocorrência dessa vulnerabilidade nos últimos anos.

### Funcionamente técnico:

Antes de ir de cabeça na vulnerabilidade, vou explicar como os parâmetros são analisados do lado do servidor, para isto usarei a linguagem de back-end chamada php, aqui explicarei o básico dela, caso tenha interesse leia sua documentação [aqui.](https://www.php.net/manual/pt_BR/)

