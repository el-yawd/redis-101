<!-- Logo 4noobs -->

<p align="center">
  <a href="https://github.com/he4rt/4noobs" target="_blank">
    <img src="https://raw.githubusercontent.com/he4rt/4noobs/master/.github/header_4noobs.svg">
  </a>
</p>

<!-- Title -->

<p align="center">
  <h2 align="center">Redis 101</h2>

  <h1 align="center"><img src="https://www.logo.wine/a/logo/Redis/Redis-Logo.wine.svg" alt="Logo do Redis" width="220"></h1>

  <p align="center">
    <br />
    <a href="#ROADMAP"><strong>Explore a documentação »</strong></a>
    <br />
    <br />
    <a href="https://github.com/el-yawd/redis-101/issues/new">Report Bug</a>
    ·
    <a href="https://github.com/el-yawd/redis-101/issues/new">Request Feature</a>
  </p>
</p>

 <!-- ABOUT THE PROJECT -->

Bem-vindo ao minicurso de redis!

A ideia é te dar um roadmap e explicação sobre o básico do banco de dados mais quente do mercado.
Iremos abordar sobre sua arquitetura geral, tipos de dados, operações básicas, pesistência e como setar seu próprio _cluster redis ®_.

Esse minicurso não vai esgotar tudo o que se têm para dizer sobre redis, então recomendo fortemente que
você busque saber mais por si mesmo e explore esse mundo ~muito foda~ de NoSQL. Bons lugares para se começar: [documentação oficial](https://redis.io/docs/latest/), [código fonte](https://github.com/redis/redis) e [wikipedia](https://en.wikipedia.org/wiki/Redis). Espero que goste :)

<!-- ROADMAP OF PROJECT -->

## ROADMAP

- [Introdução](#Introdução)
- [Instalação](#Instalação)
- [Primeiros Passos](#Primeiros-passos)
  - [Data Types](#Data-types)
  	- [Strings](#Strings)
    - [Lists](#Lists)
    - [Sets](#Sets)
    - [Hashes](#Hashes)
    - [Sorted Sets](#Sorted-Sets)
- [Persistência](#Persistência)
  - [RDB](#RDP)
  - [AOF](#AOF)
- [Sistemas Distribuídos](#Sistemas-Distribuídos)
  - [Replicação](#Replicação)
  - [Clustering](#Clustering)
- [Redis Insight](#Redis-Insight)
- [Conclusão](#Conclusão)

<!-- CONTRIBUTING -->

## Como Contribuir

Contribuições fazem com que a comunidade open source seja um lugar incrível para aprender, inspirar e criar. Todas contribuições
são **extremamente apreciadas**

1. Realize um Fork do projeto
2. Crie um branch com a nova feature (`git checkout -b feature/featureBraba`)
3. Realize o Commit (`git commit -m 'Adicionado conteudo brabo'`)
4. Realize o Push no Branch (`git push origin feature/featureBraba`)
5. Abra um Pull Request

## Autores

- **Diego Reis** - _Main author_ - [@el_yawd](https://x.com/el_yawd)

---

# Introdução

Mas afinal, o que é o redis?

Ele é um banco de dados _in-memory_ com pesistência opcional bem simples _(yet powerful)_. Você pode pensar nele como um **servidor de estrutura de dados**, isso significa que:

> O redis provê acesso a estruturas de dados mutáveis através de um conjunto de comandos que são enviados usando um modelo cliente-servidor. Dessa forma, diferentes processos podem realizar pesquisas e modificações nas mesmas estruturas de dados de forma compartilhada. [[1]](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis)

Por conta disso ele é MUITO utilizado como servidor de cache [[2]](https://www.akitaonrails.com/2021/12/08/akitando-110-como-fazer-o-ingresso-com-escalar-conceitos-intermediarios-de-web) [[3]](https://status.openai.com/incidents/fk0tcbydtybr), comunicação entre serviços [[4]](https://www.youtube.com/watch?v=rP9EKvWt0zo) e muito mais.


# Instalação

Vou ser sincero, não quero perder tempo com isso, siga as [instruções de instalação](https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/) e seja feliz.

Durante o minicurso irei utilizar docker, se você ~ainda~ não têm instalado vai lá no [docker.docs](https://docs.docker.com/engine/install/) e instala. Voltou? Ótimo! Para rodar o redis basta executar:

```bash
    docker run -d --name redis-stack -p 6379:6379 -p 8001:8001 redis/redis-stack:latest
```

Esse comando vai criar um único container de docker e expor duas portas:
- `6379` porta padrão do servidor redis;
- `8001` porta para o _Redis Insight ®_ ;

Vamos explorar o _Redis Insight ®_ mais pro final do minicurso, como somos _try hard_ iremos utilizar
majoritariamente a linha de comando. Chega de papo, bora lá!

# Primeiros Passos

Para se conectar no _REPL_ do servidor redis basta rodar:

```bash
docker exec -it redis-stack redis-cli
```

> INFO `redis-cli` é a ferramenta de linha de comando do redis. Você pode saber mais sobre ela [aqui](https://redis.io/docs/latest/operate/rs/references/cli-utilities/redis-cli/)

### [Data Types](https://redis.io/docs/latest/develop/data-types)

Como eu disse, redis é um servidor de estrutura de dados, essas estruturas são definidas como `data types`, vamos passar por cima das principais, mas antes disso um adendo: as operações nas estruturas do redis são realizadas através de `commands`; não vai ser possível mostrar todos os `commands` de cada estrutura então se ficar curioso poderá buscar mais sobre cada uma [na documentação de comandos](https://redis.io/docs/latest/commands/):

### [Strings](https://redis.io/docs/latest/develop/data-types/strings/)

A forma mais simples de estrutura de dados no redis é o chave-valor. Chaves são strings e valores também, dentro do _REPL_ digite:

```bash
SET melhor-aula nosql
```

O comando `SET` "seta" uma chave com um valor associado, nesse caso a chave `melhor-aula` está associada com o valor `nosql`. Para acessar o valor basta digitar:

```bash
GET melhor-aula

# Resultado -> "nosql"
```

> WARNING Se rodarmos `SET` quando uma chave já existe, seu valor anterior é sobrescrito. Para
> evitar esse comportamento é necessário passar o parâmetro `nx` no final do comando.

E você pode usar qualquer string como valor (com tamanho de até 512MB), até mesmo imagens!

Imagine que está desenvolvendo uma rede social com altos requerimentos de escalabilidade e precisa guardar quantos views um post possui, você provavelmente estará usando vários serviços, espalhados no mundo todo. Como garantir que **diferentes serviços compartilhem esse dado**?

Yup, você acertou, com redis;

O comando `INCR` associa uma chave a um valor numérico, e toda vez que `INCR` é chamado com a chave ele incrementa +1 no valor atual, por exemplo:

```bash
INCR user:post-1:views
```

Execute várias vezes, você verá o valor incrementando.

```bash
INCRBY user:post-1:views 42
```

Incrementa o valor +42, (você também pode incrementar números negativos).

### [Lists](https://redis.io/docs/latest/develop/data-types/lists/)

Listas em redis são [listas ligadas](https://en.wikipedia.org/wiki/Linked_list) com ponteiros para o início e final da fila, isso significa que operações de _pop_ e _push_ são feitas em _O(1)_. O seu _trade-off_ são operações que utilizam índices, pois cria a necessidade de percorrer cada elemento da lista. O redis fornece alternativas caso necessitemos utilizar indíces mas isso vai ficar pra depois :)

Para inserirmos um novo elemento em uma lista utilizamos o comando `LPUSH` ou `RPUSH`, por exemplo:

```bash
LPUSH fila-espera:cafe Alice Bob Joe
```

> `LPUSH` insere um novo elemento no início da lista (_head_) e `RPUSH` faz o mesmo mas no final da fila (_tail_).

Note que não existe um comando para criar uma lista, simplesmente inserimos um dado. Se a lista existe um novo dado é incluído, se não existe nosso querido redis se responsabiliza por criar; isso se aplica para todo data type :).

Além disso, `LPUSH` e `RPUSH` são _comandos variáticos_ o que significa que somos livres para inserir um ou mais elementos de uma só vez.

Para removermos um elemento da lista utilizamos:

```bash
LPOP fila-espera:cafe
```
> A mesma lógica de pushs se aplica aqui: `LPOP` remove elementos no início e `RPOP` no final.

Para verificarmos elementos de uma lista utlizamos `LRANGE` (sim `L` é usando tanto como _left_ quanto _list_, that's life).

```bash
LRANGE fila-espera:cafe 0 3
```

O comando acima busca os 4 primeiros elementos da lista, o primeiro argumento é o índice do início de nosso _range_ e o segundo é o final do _range_. Também podemos utilizar números negativos, então o redis começa a contar de "trás pra frente": -1 é o último elemento, -2 o penúltimo e assim por diante.

```bash
LRANGE fila-espera:cafe 0 -1

# Busca todos os elementos da lista
```

Um caso de uso comum para lista em redis é manter o n últimos elementos de alguma coisa. Tweets por exemplo [[5]](https://www.infoq.com/presentations/Real-Time-Delivery-Twitter/). O que queremos é uma _capped collection_ nesse caso uma _capped list_ (literalmente lista limitada), utilizaremos `LTRIM`
para fazer isso.

`LTRIM` é muito parecido com `LRANGE` com a única diferença de que `LRANGE` é feito para consultas (logo não modifica a lista) e `LTRIM` literalmente apara tudo o que não estiver dentro do range, exemplo:

```bash
LPUSH last:3:custumers Chris Ana Anthony

LRANGE last:3:custumers 0 2

LPUSH last:3:custumers Paul

LTRIM last:3:custumers 0 2
# cuidado com erros off-by-one

LRANGE last:3:custumers 0 2
```

Seguindo essa sequencia vemos que Chris é "aparado" da lista visto que é o 4° mais recente.

### [Sets](https://redis.io/docs/latest/develop/data-types/sets/)

Sets (conjuntos) é uma coleção não ordenada de strings únicas. Elas são muito úteis para representar
pertencimento de um item a um conjunto, valores únicos, etc.

Vamos adicionar um novo item no set com:

```bash
SADD nosql:students Joaozinho Mariazinha
```

Para checarmos pertencimento utilizamos:

```bash
SISMEMBER nosql:students Joaozinho

# (integer) 1

SISMEMBER nosql:students Jack
# (integer) 0
```

Podemos também fazer operações de intersecção!

```bash
SADD bd-2:students Joaozinho Caio

SINTER nosql:students bd-2:students

# 1) "Joaozinho"
```

E cardinalidade:

```bash
SCARD nosql:students

# (integer) 2
```

Para removermos um elemento:

```bash
SREM nosql:students Joaozinho

SMEMBERS nosql:students # Lista todos os elementos da lista
```

Para maioria das operações a complexidade é _O(1)_, contudo para sets grandes
deve-se utilizar o comando _SMEMBERS_ com cuidado, já que ele possui complexidade _O(n)_.

### [Hashes](https://redis.io/docs/latest/develop/data-types/hashes/)


Hashes em redis funcionam como uma coleção de `strings`, podemos usá-los para representar objetos básicos dentre outros dados.

```bash
HSET aluno:1 nome Jacob curso "Sistemas de informacao" hobbie "virar lobisomem"

HGETALL aluno:1
# 1) "nome"
# 2) "Jacob"
# 3) "curso"
# 4) "Sistemas de informacao"
# 5) "hobbie"
# 6) "virar lobisomem"

HGET aluno:1 nome
# "Jacob"
```

> NOTE: Você deve ter reparado que utilizamos ':' para separar tokens nos nomes, isso é
> apenas uma convenção. Nomes e chaves devem ter eles mesmos significado. Por exemplo, 'aluno:1'
> provavelmente é um aluno com id 1, o redis não faz a mínima ideia desse id, portanto sua aplicação deve
> se responsbilizar por manter um padrão coerente e que não sobrescreva incorretamente dados anteriores.

Também podemos manter contadores em nossos "objetos"

```bash
HINCRBY aluno:1 faltas 1

HGETALL aluno:1
# 1) "nome"
# 2) "Jacob"
# 3) "curso"
# 4) "Sistemas de informacao"
# 5) "hobbie"
# 6) "virar lobisomem"
# 7) "faltas"
# 8) "1"
```

## [Sorted Sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/)

Lembra quando eu disse que o redis fornecia uma estrutura de dados para usar índices.
Pois bem, _voilà_.

_Sorted sets_ são como um mix entre _Sets_ e _Hashes_, são formados por
elementos não-repetíveis, porém agora cada elemento é associado a um _float point_
chamado _score_ como um _hash_. Este score é utilizado para comparar valores, então se:
`A > B` logo `A.score > B.score`. Um fator interessante é que os elementos são guardados
já ordenados, então buscas ordenadas não custam tanto.

Para incluir novos elementos:

```bash
ZADD brasileirao:tabela 0 Botafogo 1 Palmeiras 3 Fortaleza 4 Flamengo

ZRANGE brasileirao:tabela 0 -1
# 1) "Botafogo"
# 2) "Palmeiras"
# 3) "Fortaleza"
# 4) "Flamengo"

ZREVRANGE brasileirao:tabela 0 -1
# 1) "Flamengo"
# 2) "Fortaleza"
# 3) "Palmeiras"
# 4) "Botafogo"
```

Podemos passar o argumento `WITHSCORES` nos comandos de range para também
devolver os ranges:

```bash
ZRANGE brasileirao:tabela 0 -1 WITHSCORES

# 1) "Botafogo"
# 2) "0"
# 3) "Palmeiras"
# 4) "1"
# 5) "Fortaleza"
# 6) "3"
# 7) "Flamengo"
# 8) "4"
```

Podemos realizar comando em scores:

```bash
ZRANGEBYSCORE brasileirao:tabela -inf 3 # busca todos os elementos com score entre -infinto e 3.
# 1) "Botafogo"
# 2) "Palmeiras"
# 3) "Fortaleza"
```

Também podemos buscar o score de um valor com:

```bash
ZRANK brasileirao:tabela Palmeiras
# (integer) 1
```

## Persistência

Lembra quando eu disse que redis possui `pesistência opcional`? Vamos ver como configurá-la agora! O Redis tem 4 tipos
de persistência:

- **RDB (Redis Database)**: Snapshots que são realizados em intervalos específicos;
- **AOF (Append Only File)**: Arquivo de log de toda operação de escrita, que podem ser replicadas quando o servidor reinicia, reconstruindo
  seu estado original.
- **Sem persistência**: Autoexplicativo né. Se o servidor cair não tem jeito de recuperar os dados.
- **RDB + AOF**: Combinação das estratégias de _logging_ e _snapshotting_

Você pode ler mais sobre seus _trade-offs_ [aqui](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/).

### RDB

Redis é _single-threaded_ então como podemos fazer o snapshot sem bloquear a _thread_ e perder operações? O processo realiza um _fork()_ e esse _fork()_ que realiza essa operação, enquanto o processo pai continua ativo recebendo requisições.[[6]](https://medium.com/redis-with-raphael-de-lio/understanding-persistence-in-redis-aof-rdb-on-docker-dcc176ea439)


![gif explicando seu funcionamento: o processo pai faz um fork do filho e este filho que realiza o snapshot (uma "foto") do banco](https://miro.medium.com/v2/resize:fit:720/format:webp/1*0fQ1UKmtXqgIVXkTWJLorw.gif)

Se você reparou, na raíz do projeto existe um arquivo chamado `redis.conf`, lá
podemos configurar nosso servidor redis. Ele também possui comentários muito
úteis que os próprios desenvolvedores disponibilizaram para que a comunidade entenda melhor cada parâmetro e seu funcionamento interno.

Abra-o e pesquise por `SNAPSHOTTING` essa seção nos fornece toda a configuração relacionada ao `RDB`, descomente a linha 440 e a mude para `save 5 1`. Ela
significa que a cada 5s iremos realizar um snapshot se ao menos uma _key_ mudou.

> WARNING Essa é uma quantidade excessiva de snapshotting, ela é apenas para experimentarmos mais facilmente.

Bacana, a partir de agora vamos utilizar `docker compose`, ele é uma extensão de docker que nos permite, declarativamente, configurar
múltiplos containers. Se não o tiver basta seguir a [[documentação]](https://docs.docker.com/compose/install/).

Pare a instância que estavamos rodando com `docker container stop redis-stack` e
utilize `docker compose up` e acesse o novo container com `docker compose exec -it redis redis-cli`. Para nos certificarmos que estamos com a nova configuração rode:

```bash
CONFIG GET save

# Resposta esperada
# 1) "save"
# 2) "5 1"
```

Cool, crie uma nova chave e espere 5s. Em outro terminal rode:

```bash
docker compose cp redis:/data/dump.rdb ./dump.rdb
```

Esse comando copia o arquivo de dump para nosso diretório atual. Com ele podemos restaurar nossos dados caso o servidor caia.

Agora vamos parar o container e apagar sua imagem com:

```bash
docker stop redis

docker rm redis
```

Agora restaure a imagem com `docker compose up`, copie o arquivo para dentro do container: `docker cp ./dump.rdb redis:/data/dump.rdb` e o reinicie com:

```bash
docker compose restart
```

Basta entrar no container e verificar os dados que salvamos no dump :)

### AOF

![gif do funcionamento do AOF: cada operação de escrita vira um bloquinho e vai para uma lista de blocos. Quando o servidor é reiniciado os blocos da lista vão sendo consumidos pelo redis para recriar o estado anterior do banco](https://miro.medium.com/v2/resize:fit:720/format:webp/1*-nu-a_xIAH4OwIVdkszVzA.gif)

Um problema de _snapshotting_ é que se o sevidor cair perdemos os dados que foram escritos depois do último snapshot, se não podemos perder nem um dado se quer devemos considerar utilizar AOF.

Para habilitá-lo vá em `redis.conf` e busque por `APPEND ONLY MODE` essa é a seção de configuração do AOF. Mude a linha `appendonly no` para `appendonly yes`.

Resete o servidor, o acesse e faça alguma operação de escrita.

Para recuperar o diretório de logs rode:

```bash
❯ docker compose cp redis:data/appendonlydir ./appendonlydir
```

se vermos o conteúdo desse diretório com `ls appendonlydir` veremos que temos 3 arquivos, o que são eles:

- `appendonly.aof.1.base.rdb`: É um snapshot de todo o banco no momento em que esse arquivo foi criado.
- `appendonly.aof.1.incr.aof`: Todos os comandos que alteraram o banco que foram rodados depois do snapshot.
- `appendonly.aof.manifest`: Gerencia os outros arquivos e garante a ordem em que devem ser executados.

Para saber mais sobre o assunto de persistência recomendo começar por [[aqui]](https://medium.com/redis-with-raphael-de-lio/understanding-persistence-in-redis-aof-rdb-on-docker-dcc176ea439) e [[aqui]](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/).


## Sistemas Distribuídos

Tudo o que vimos até agora acontece em uma instância de redis, manipulação de estruturas de dados e recuperação de dados.
Mas e se quisermos garantir um downtime baixo com alta disponibilidade? Ou se precisamos aumentar o _throughput_? Ai que entra a distribuição.

### [Replicação](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)

Uma forma simples de aumentarmos o _throughput_ é através da replicação. Dividimos nossas intâncias de redis em dois tipos:
líder (master) e seguidores (slaves). Operações de escrita só podem ser realizadas no líder e este por sua vez replica os dados para seus seguidores. Dessa forma, tiramos a carga de leitura de um servidor e distribuimos entre os seguidores, enquanto o líder só se ocupa com as operações de escrita e replicação. Vamos ver isso funcionando.

### Passos para executar

Na pasta replication, execute o seguinte comando para iniciar o replica-set:

```bash
docker-compose up
```

Simples não? Agora abra dois terminais e execute os comandos abaixo, um pra cada terminal:

```bash
docker exec -it redis-master redis-cli

docker exec -it redis-slave-1 redis-cli
```

Tente escrever algo no `redis-slave-1`, vai dar erro, por quê? Como ele é um servidor de réplica
somente ações `READ ONLY` podem ser executadas. Ainda no `redis-slave-1` rode:

```bash
GET melhor-aula

# (nil)
```

Agora no servidor master rode `SET melhor-aula nosql`, se dermos `GET` em ambos os servidores o dado
`melhor-aula` estará disponível!

### [Clustering](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/#create-a-redis-cluster)

Replicação é legal mas ela possui alguns drawbacks:

- Muita dependencia do master;
- Não escala horizontalmente por que a escrita é "engargalada" pela master;
- Se algum nó cai o redis não se recupera sozinho

Uma das maneiras de resolver isso é com [sentinelas](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
que nada mais são que outros processos que ficam "vigiando" nosso replica-set, ele cuida de todo sistema de eleição de líder,
notificação, etc. ~Não falaremos dele aqui por que o escritor perdeu 3 anos de vida útil tentando configurar~ Fique livre para estudar esse modelo :)

Outra forma é com clusterização, com ele o redis faz automagicamente um processo de _sharding_ que nada mais é que separar sua base de dados entre os nós, ao invés de cada um ter uma cópia da base toda. Assim, novos nós podem ser adicionados e removidos da rede de forma a reorganizar os dados facilmente.

![Imagem de exemplo do schema de clusterização: um cliente pesquisa por três keys onde cada uma está em um shard, cada shard é composto por duas instâncias de redis: um master e um slave](https://www.aeraki.net/blog/2023/manage-redis-traffic-with-istio-and-aeraki/redis-cluster.png)


Para rodar as instâncias redis em sua máquina, vá para o diretório `cluster` e rode `docker compose up`.
Para iniciar o cluster, rode:

```bash
docker exec -it cluster-redis-node-1-1 redis-cli --cluster create \
  redis-node-1:6379 \
  redis-node-2:6379 \
  redis-node-3:6379 \
  redis-node-4:6379 \
  redis-node-5:6379 \
  redis-node-6:6379 \
  --cluster-replicas 1
```

Ele irá criar nosso cluster com os nodes passados com um fator de replicação de 1, o que significa que cada
escrita deve ser replicada para pelo menos 1 nó.

Conecte no primeiro nó com `docker exec -it cluster-redis-node-1-1 redis-cli -c` e rode:

```bash
SET key1 "value1"
SET key2 "value2"
SET key3 "value3"
SET key4 "value4"
```

Para algumas chaves você deve ter recebido `-> Redirect to slot <slot_number> located at <slot_host>` antes do `OK`, isso significa que a chave que setamos não se encontra no `node-1`, porém se dermos `GET` em uma chave "movida" conseguimos, por quê?

Reparou no `-c` que passamos junto ao `redis-cli`? Ele torna o nosso _redis client_ ciente que está em ambiente de cluster, se tentarmos acessar sem o `-c` iremos receber um `(error) MOVED <slot_number> <slot_host>`

Podemos consultar qual _keyslot_ uma chave pertence com `CLUSTER KEYSLOT <keyslot>`, para sabermos quais nodes
pertencem ao nosso cluster podemos rodar: `CLUSTER NODES`.

Vamos adicionar um novo nó no cluster. Dentro do arquivo `docker-compose.yml` descomente o serviço `redis-node-7` e execute: `docker compose up redis-node-7`

Agora acesse o nó 1 com `docker exec -it cluster-redis-node-1-1 bash` e rode `redis-cli --cluster add-node redis-node-7:6379 redis-node-1:6379`. Entrando no cliente redis com `redis-cli` podemos pesquisar pelos nós como explicado acima, e verificar que ele foi de fato aceito. Agora podemos redistribuir as chaves rodando: `redis-cli --cluster rebalance redis-node-1:6379`

### Redis Insight

Redis Insight é uma ferramenta web que nos permite interagir com o redis de forma intuitiva e prática. Eu sei eu sei, poderíamos estar usando ela desde o começo mas onde está a didática em não sofrer?

Para roda-lo basta descomentar o serviço de redis-insight no docker-compose e rodar: `docker compose up redis-insight`, ele estará exposto na porta `8001`.

Por lá podemos cadastrar nossos nós do cluster e importar os dados que estão na raiz do diretório `cluster`.

## Conclusão

Redis é uma tecnologia bem interessante, embora simples ela nos dá flexibilidade e performance para resolver problemas de escala no mundo real, sem dúvidas uma ótima carta na manga para qualquer desenvolvedor.

Espero que você tenha gostado desse mini-curso, se ele foi útil para você de alguma forma não esqueça de deixar uma estrelinha e compartilhar com os amigos. Até mais e `accelerate, anon`.
