# redis-101

![logo do redis](https://www.logo.wine/a/logo/Redis/Redis-Logo.wine.svg)

Bem-vind@ ao minicurso de redis!

A ideia é te dar um roadmap e explicação sobre o básico do banco de dados mais quente do mercado. Iremos abordar sobre sua arquitetura geral, tipos de dados, operações básica e como setar seu próprio _cluster redir ®_.

Esse minicurso não vai esgotar tudo o que se têm para dizer sobre redis, então recomendo fortemente que
você busque saber mais por si mesmo e explore esse mundo ~muito foda~ de NoSQL. Bons lugares para se começar: [documentação oficial](https://redis.io/docs/latest/), [código fonte](https://github.com/redis/redis) e [wikipedia](https://en.wikipedia.org/wiki/Redis). Espero que goste :)

# Introdução

Mas afinal, o que é o redis? 

Ele é um banco de dados _in-memory_ com persistência opcional bem simples _(yet powerful)_. Você pode pensar nele como um **servidor de estrutura de dados**, isso significa que: 

> O redis provê acesso a estruturas de dados mutáveis através de um conjunto de comandos que são enviados usando um modelo cliente-servidor. Dessa forma, diferentes processos podem realizar pesquisas e modificações nas mesmas estruturas de dados de forma compartilhada. [[1]](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis)

Por conta disso ele é MUITO utilizado como servidor de cache [[2]](https://www.akitaonrails.com/2021/12/08/akitando-110-como-fazer-o-ingresso-com-escalar-conceitos-intermediarios-de-web) [[3]](https://status.openai.com/incidents/fk0tcbydtybr), comunicação entre serviços [[4]](https://www.youtube.com/watch?v=rP9EKvWt0zo) e muito mais.


# Instalação

Vou ser sincero, não quero perder tempo com isso, siga as [instruções](https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/) e seja feliz. 

Durante o minicurso irei assumir que você está em um ambiente UNIX e utilizar docker, docker compose, se você ~ainda~ não têm instalado vai lá no [docker.docs](https://docs.docker.com/engine/install/) e em [compose.docs](https://docs.docker.com/compose/install/) e instala os dois. Voltou? Ótimo! Para rodar o redis basta: 

1. Clonar este repositório:
```bash
git clone git@github.com:diegoreis42/redis-101.git
```

2. Ir para o diretório do projeto:
```bash
cd redis-101
```

3. Rodar as instâncias de redis e redis insight:
```bash
docker compose up 
```

Esse último comando vai criar um único container de docker e expor duas portas:
- `6379` porta padrão do servidor redis;
- `8001` porta para o _Redis Insight ®_ ;

Vamos explorar o _Redis Insight ®_ mais pro final do minicurso, como somos _try hard_ iremos utilizar
majoritariamente a linha de comando. Chega de papo, bora lá!

# Primeiros Passos

Para se conectar no _REPL_ do servidor redis basta rodar:

```bash
docker exec -it redis redis-cli
```

> INFO `redis-cli` é a ferramenta de linha de comando do redis. Você pode saber mais sobre ela [aqui](https://redis.io/docs/latest/operate/rs/references/cli-utilities/redis-cli/) 

Digite `PING` se a reposta for `PONG`, deu bom 😎

Adendo: comando redis, assim como SQL, são _case insensitive_ o que significa que podemos escrever tudo em "caixa-baixa", durante o minicurso vou utilizar "caixa-alta" para seguir um padrão, mas você é livre para fazer da forma que preferir :) 

### [Data Types](https://redis.io/docs/latest/develop/data-types)

Como eu disse, redis é um servidor de estrutura de dados, essas estruturas são definidas como `data types`, vamos passar por cima de cada uma delas, mas antes disso um adendo: as operações nas estruturas do redis são realizadas através de `commands` não vai ser possível mostrar todos os `commands` de cada estrutura então se ficar curios@ poderá buscar mais sobre cada uma [aqui](https://redis.io/docs/latest/commands/):

#### [Strings](https://redis.io/docs/latest/develop/data-types/strings/)

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

Note que não existe um comando para criar uma lista, simplesmente inserimos um dado, se a lista existe um novo dado é incluído, se não existe nosso querido redis se responsbiliza por criar.

Além disso, `LPUSH` e `RPUSH` são _comandos variáticos_ o que significa que somos livres para inserir um ou mais elementos de uma só vez.

Para removermos um elemento da lista utilizamos:

```bash
LPOP fila-espera:cafe 
```
> A mesma lógica de pushs se aplica aqui: `LPOP` remove elementos no início e `RPOP` no final.

Para verificarmos elementos de uma lista utlizamos `LRANGE` (sim `L` é usando tanto como _left_ quanto _list_ that's life). 

```bash
LRANGE fila-espera:cafe 0 3
```

O comando acima busca os 4 primeiros elementos da lista, o primeiro argumento é o índice do início de nosso _range_ e o segundo é o final do _range_. Também podemos utilizar números negativos, então o redis começa a contar de "trás pra frente": -1 é o último elemento, -2 o penúltimo e assim por diante.

```bash
LRANGE fila-espera:cafe 0 -1

# Busca todos os elementos da lista
```

Um caso de uso comum para lista em redis é mantes o n últimos elementos de alguma coisa. Tweets por exemplo [[5]](https://www.infoq.com/presentations/Real-Time-Delivery-Twitter/). O que queremos é uma _capped collection_ nesse caso uma _capped list_ (literalmente lista limitada), utilizaremos `LTRIM`
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

<!-- Incluir mais estruturas de dados aqui -->

# Persistência

So far so good, vimos as principais estruturas de dados que o redis nos fornece mas ainda existe alguns conceitos que precisamos explorar. Um deles é a **persistência**, lembra quando eu disse que o redis é um banco de dados _in-memory_ com persistência opcional, pois é, vamos ver como configurá-la!

### Tipos de persistência

Persistência se refere a escrever dados em um _storage_ durável, como um HD ou SSD. O redis nos fornece 4 opções [[6]](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/):

- **RDB(Redis Database)**: snapshots _point-in-time_ em intervalos específicos;
- **AOF(Append Only File)**: arquivo de log que "appenda" logs de escrita. Essas operações podem ser replicadas quando o servidor restartar para reconstruir o banco com o dataset original. 
- **Sem persistência**: autoexplicativo
- **RDB + AOF**: estratégias RDB e AOF combinadas na mesma instância.

Você pode ler os _trade-offs_ de cada estratégia na documentação. 

### Snapshots

Snapshots funcionam como "máquinas do tempo", você pode tirar quantas "fotos" do seu banco quando quiser na frequência que quiser e pode restaurar seu banco em qualquer ponto do tempo.[[7]](https://medium.com/redis-with-raphael-de-lio/understanding-persistence-in-redis-aof-rdb-on-docker-dcc176ea439)

Redis é _single-threaded_, para que o processo de snapshotting não atrapalhe os clientes ele faz um `fork()` de si e o processo filho se encarrega dessa tarefa. Snapshots são guardados no arquivo binário `dump.rdb`.

![ilustração do funcionamento de snapshots no redis](https://miro.medium.com/v2/resize:fit:720/format:webp/1*0fQ1UKmtXqgIVXkTWJLorw.gif)

Vamos por a mão na massa agora. Se você é ~minimamente~ curios@ viu que temos um arquivo redis.conf na raíz do projeto, ele é um template que o redis disponibiliza [aqui](https://raw.githubusercontent.com/redis/redis/unstable/redis.conf) com tudo o que podemos configurar em nossos servidores (recomendo fortemente que leia os comentários desse arquivo para afiar seu entendimento). Abra o arquivo `redis.conf` no seu editor de 1texto favorito e busque por `SNAPSHOTTING`, essa seção descreve perfeitamente como o redis realiza snapshotting e tudo o que ocorre em volta disso: compressão, tratamento em casos de erro, replicação do arquivo e muito mais.

Por hora vamos nos preocupar somente com a linha `440` onde definimos em qual frequência e o mínimo de mudanças necessárias para realizar o snapshot, descomente essa linha e a altere para:

```conf
save 60 1 

# Ela diz: Faça um snapshot a cada 1m (60s) se ao menos uma mudança foi feita
```

Eu sei, essa é uma frequência *muito* excessiva; em um ambiente real essa quantidade não é recomendada, mas como estamos explorando ~_just for fun_~ tá perfeito.

Restarte seus containers:

```bash
docker compose restart
```

acesse o _REPL_ dentro do container com:

```bash
docker exec -it redis redis-cli
```

digite: `CONFIG GET save`, se o valor que configuramos aparecer significa que o redis reconheceu corretamente nosso arquivo de configuração!

Agora vamos ver funcionando:







