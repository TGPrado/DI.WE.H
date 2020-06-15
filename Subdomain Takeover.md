# Subdomain Takeover

### O que é:

Esta falha ocorre quando um atacante consegue adquirir um subdomínio de uma empresa, podendo usá-lo para criar páginas falsas com o objetivo de adquirir dados de usuários(*phishing*), bem como levá-lo a outras falhas como o XSS; este tipo de vulnerabilidade ocorre devido ao excesso de confiança do navegador no Domain Name Server(DNS).

### DNS:

Ao fazer uma request para um site, inicialmente é feito uma tradução do nome de domínio,como ```www.example.com```, para seu respectivo Internet Protocol address(*IP*), é inimaginável um usuário ter que decorá-lo para toda página que desejar acessar, logo foram criados servidores que armazenam tabelas, as quais relacionam os valores do nome de um site e de seu respectivo endereço ip.

A request para o DNS funciona da seguinte forma:

![](https://i.imgur.com/hMtTksj.png)
###### source:https://0xpatrik.com/content/images/2018/08/resolution.png

1. É feita uma requisição para um servidor DNS recursivo, ele é responsável por procurar em outros servidores a resposta para a "tradução".
2. O DNS recursivo envia uma request para um ROOT nameserver, este possui tabelas que relacionam a última parte de um domain name como ```.com```,```.net```, ```.gov``` e ```.edu``` a um, Top-Level Domain (*TDL*), indicando para onde a request deve ser enviada.
4. O TLD, por sua vez consulta e retorna um authoritative DNS.
6. Este, retorna as informações sobre um domínio, como o endereço IP pois possui autoridade para isso.
9. Com o IP em "mãos" uma conexão é estabelecida e dados são trocados.

É interessante saber que os DNS's são um dos responsáveis pelo funcionamento da internet, no momento de escrita deste artigo, existiam apenas 13 DNS's do tipo ROOT, caso todos eles parem de funcionar, seja por falha de segurança ou qualquer outro motivo, a internet também para.

### CNAME:


Canonical name é usado para um serviço que aponta um nome de domínio para outro, como na imagem abaixo:

![](https://imgur.com/9U8h1Eg.png)


Inicialmente, é feita uma consulta igual a qualquer outra, entretanto quando o TLD indica um DNS autoritativo responsável pelo CNAME ele aponta para outro já que aquele não é o real nome do domínio, por fim o IP do servidor é retornado e uma conexão é criada.

Tudo isso ocorre "por baixo dos panos", afinal nada é mostrado para o usuário final.


### Funcionamento técnico:

Primeiro imagine que temos um domínio ```exemplo.com```, este por sua vez possui um subdomínio ```sub.exemplo.com```, o qual aponta para ```dominio.com```; criado nosso cenário, explicarei em alguns passos o funcionamento da falha:

O dono de ```exemplo.com``` deseja excluir seu subdomínio, entretanto o mesmo exclui somente ```dominio.com```, com isto ```sub.exemplo.com``` ainda está apontando para algum lugar, só que este não existe, caso um atacante reivindique-o ele poderá ter total acesso a pagina e fazer o que quiser com ela.

### Exemplo:

Em abril de 2020 o hacker geekboy descobriu uma falha no subdomínio ```devrel.roblox.com``` da empresa Roblox,  com ela o pesquisador pôde reivindicar o domínio e gerar a página que quisesse.

O mesmo gerou uma PoC na qual provou que poderia roubar os cookies do usuário e enviá-los pelo chat Roblox,para isto ele usou php js e HTML; por serem, teoricamente, domínios da mesma empresa a política do Cross-origin resource sharing(*CORS*) permitiu o envio da mensagem.

O pesquisador recebeu $2500 pela falha, a mesma pode ser lida por completo [aqui.](https://hackerone.com/reports/335330)

### Como explorar:

Para descobrir possíveis aquisições de subdomínios primeiro devemos conhecê-los, para isto existem diversas ferramentas algumas pagas e outras gratuitas, devido a esta diversidade não há como falar sobre todas, por isto explicarei somente a que acredito ser a mais interessante: o site https://crt.sh/ procura por certificados SSL registrados por domínios e subdomínios, esta informação é acessível pois ao criar um certificado ele deve ser registrado com uma autoridade de certificação para que os navegadores possam confirmar sua veracidade.

Descoberto os subdomínios é interessante acessá-los e procurar por erros, geralmente quando esta falha ocorre é gerado um erro do tipo 404 ao acessar a página, pois ela tecnicamente não existe, isto é um grande indicador de aquisição de subdomínios.

Feito estes passos use o comando dig do linux para descobrir para onde o subdomínio aponta.

Este problema é encontrado comumente nos servidores em nuvem da AWS e da AZURE. 

Em alguns poucos casos o administrador do bug bounty pode solicitar um certificado SSL para que a recompensa seja paga, mas não desanime não é tão difícil adquirir um.