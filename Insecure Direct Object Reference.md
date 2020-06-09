# Insecure Direct Object Reference
### O que é:

O IDOR ocorre quando, o atacante por meio da modificação dos dados enviados ao servidor, consegue acessar diretamente outros objetos, no melhor dos casos isto pode levá-lo a uma escalação de privilégios vertical entretanto é mais comum que isto leve-o à uma escalação horizontal, a primeira se refere a quando o atacante consegue subir seu nível de privilégios por exemplo, se tornar um administrador, na segunda ele consegue apenas acesso a outras contas as quais estão no mesmo nível que ele.

Está falha é gerada por um mal design do back-end da aplicação o qual não verifica o nível de privilégio de um usuário antes de responder a request.

### Funcionamento técnico:

É normal que aplicações manipulem e acessem, por exemplos contas, por meio de parâmetros um exemplo pode ser a seguinte URL:
www.exemplo.com/conta?id=4
é de se esperar que alterando o id possamos acessar a página de outro usuário e não há nenhum problema nisso, desde que não seja é possível acessar dados "confidenciais" de outros usuários como por exemplo a página de edição da conta.

Imagine que uma aplicação use a seguinte URL para editar os dados de uma conta
www.exemplo.com/edit?id=4 teríamos uma falha caso fosse possível editar os dados de um usuário apenas modificando o id para 5, por exemplo. Isto ocorre pois o back-end não checa se o cookie do atacante é equivalente ao do id 5.

Algumas aplicações codificam o id como uma forma de proteção e outras usam o conceito de identificador único universal ou simplesmente UUID com ele é impossível prever o valor de id de um usuário, inviabilizando na **maioria das vezes** a falha.

Esta vulnerabilidade também pode ser encontrada no acesso de arquivos, caso um site use uma URL previsível e não verifique o privilégio do cliente é possível consultar dados de outros usuários, como imagens que deveriam ser privadas, arquivos de texto e etc.

O último lugar onde se encontra esta falha é nos cookies, há casos em que eles são fáceis de serem descobertos assim, o atacante pode simplesmente substituir seu valor e acessar outra conta. 

### Exemplo:

No começo de 2020, Ameya encontrou uma falha IDOR no endpoint da Razer, nela o atacante poderia acessar dados confidenciais dos clientes como email, csrf token e o rzr_id.

Para isto o pesquisador viu que, ao acessar a conta de um usuário encontrava um link parecido com este: ```https://insider.razer.com/index.php?members/kajira.714/```

Ele entendeu que o valor 714 era referente ao user ID e substituiu este valor em outro endpoint da empresa ```https://insider.razer.com/api.php?action=getuserprofile&user_id=714``` com isto ele pôde acessar os dados citados acima.

É interessante citar que o atacante não precisava estar logado para poder acessar esse endpoint.

Esta falha só é possível pois o id é muito previsível, com um simples ataque de força bruta seria possível roubar os dados de centenas de usuários.

O pesquisador recebeu $375 por esta falha, e ela pode ser lida [aqui](https://hackerone.com/reports/723118).

### Como explorar:


Explorar esta falha é fácil, fique atento a valores que sejam referentes ao ID de um usuário como UUID, user, account e etc, ao encontrá-los tente alterá-lo para ver como a aplicação reagirá.

Como citado acima usar o sistema de UUID é muito seguro, pois é impossível "advinhar" um id, entretanto há duas maneiras de se explorar esta falha mesmo com eles:

A primeira é basicamente procurar na aplicação onde este valor possa estar "vazando" isto é comumente encontrado em sistemas de recuperação de senha, de "invite a friend" mas também pode ser encontrado no perfil da vítima.

A segunda forma é atrelar a falha HPP- HTTP Parameter Pollution- onde o atacante tenta duplicar o parâmetro para ver como o servidor reage.