# Race Condition

### O que é:

Conhecida no brasil como 'condição de corrida' esta falha ocorre pelo fato de dois ou mais processos, teoricamente simultâneos, dependerem do mesmo recurso. 

Isto pode levar à curtidas infinitas em uma postagem na sua rede social ou à retirada de dinheiro infinito de sua conta bancária, por exemplo.

### Programação assíncrona:



Acredito que antes de definir assincronismo devo definir sua diferença comparada ao sincronismo, então vamos lá!

O último ocorre quando uma aplicação precisa primeiro executar uma sequencia de comandos inteira para depois partir para outros comandos, já o assincronismo ocorre quando pode-se executar comandos mesmo sem o anterior ter sido finalizado, este tipo de programação é usada para diminuir a velocidade de execução de programas, para isto divide-se sua execução em diversas 'thread' ou processos.

![](https://imgur.com/xjZ22qq.png)
###### source:https://imgur.com/xjZ22qq.png

Abaixo tem-se um exemplo de uma aplicação *synchronous*.

```python
#!/usr/bin/python 
def Primeiro():
	x = 0
  	while x < 1000000000000:
  		x += 1
  		return(print('Primeiro'))
  Primeiro()
  print('Segundo')                                                           
```
Neste caso a saída será:

```
Primeiro
Segundo
```

Para executar isto de maneira assync no python é simples.

```python
#!/usr/bin/python 
import threading
import time

def Primeiro():
	x = 0
    while x < 100000000:
    	x += 1
    return(print('Primeiro'))
t = threading.Thread(target=Primeiro) # Aqui definimosqual funcão deve ser executada ao iniciar a thread
t.start() # iniciamos a thread
print('Segundo')    
```
Neste caso tivemos um problema pois a saída foi esta:

```
Segundo
Primeiro
```

Isto ocorreu pois o comando print('segundo') foi executando antes da thread ser terminada e é um problema parecido que nos levará à falha deste artigo.


### Funcionamento técnico:

Imagine o seguinte cenário:



1. Você tem US $ 500 em sua conta e precisa enviar o valor para um amigo.
2. Usando seu telefone, você loga no seu banco e solicita uma transferência de US $ 500 para ele.
3. Após 10 segundos, a solicitação ainda não terminou. Então, você entra no banco pelo seu laptop, vê que seu saldo ainda é de US $ 500 e solicita a transferência novamente.
5. Ambas as solicitações terminam.
6. Sua conta bancária agora é de US $ 0.
7. Seu amigo lhe fala que recebeu US $ 1.000.
8. Você atualiza sua conta e seu saldo ainda é de US $ 0.


Como isto ocorreu? Simples, o banco faz somente uma validação inicial do valor em sua conta, e como a primeira solicitação ainda não havia terminado quando a segunda foi feita, ela retornou verdadeiro e o valor foi transferido.

Então como podem ver, a falha ocorre quando dois processos simultâneos acessam necessitam e modificam o mesmo recurso.

### Exemplo:

A falha deste exemplo foi descoberta no programa de bug bounty da empresa hackerone, não foi revelado quanto o hacker recebeu por ela, mas acredito que ela seja muito interessante por isto a escolhi.

Como citado no começo no primeiro artigo para se hackear algo deve-se entender primeiro como aquilo funciona, por exemplo este race condition foi achado no sistema de convite à bug bounties, para encontrá-lo o pesquisador imaginou como ele funcionaria e chegou as seguintes conclusões:

1. Os convites são únicos, logo será feito uma pesquisa em um banco de dados para verificação da existência do mesmo.
2. Depois disto, será adicionado uma permissão à conta do usuário.
3. Por fim o token será excluído da base de dados, para ninguém mais usá-lo.

O problema aqui, ocorre pelo fato da sequência de comandos depender apenas da primeira verificação e delas demorarem para ser executadas.

Tendo apenas esta ideia ele criou três novas contas na empresa, a primeira como 'empresa', e as outras duas como pesquisador, chamaremos elas de A,B e C, respectivamente.

Por fim o mesmo enviou um convite de A para B, ele acessou o link logado na conta B, e por meio de uma guia anônima logou na conta C e abriu-o também, ele deixou o botão de aceitar lado a lado e clicou em ambos o mais rápido que pôde na primeira tentativa ele falhou e não conseguiu acessar com as duas contas, entretanto na segunda ele obteve sucesso e adicionou duas contas com o mesmo link de convite.

O mesmo também poderia ter criado um script para enviar as requests pelas duas contas ao mesmo tempo, isso lhe pouparia o tempo de fazer duas tentativas.

### Como explorar:

Para explorar esta falha, o pesquisador deve ter um grande entendimento de como funcionam os sistemas, pois somente assim conseguirá entender que um simples 'aceitar convite' pode gerar uma vulnerabilidade.

Também é interessante saber que muitas vezes as requests são resolvidas muito rapidamente, entretanto o processo no lado servidor que executa a sequência de comandos pode demorar muito para terminar, isto pôde ser visto no Exemplo, lá o processo de aceitação foi extremamente rápido, entretanto o processo de execução no server side provavelmente durou um tempo maior, o qual não é mostrado para o cliente.
