# redis-101

![logo do redis](https://www.logo.wine/a/logo/Redis/Redis-Logo.wine.svg)

Bem-vind@ ao minicurso de redis!

A ideia é te dar um roadmap e explicação sobre o básico sobre o banco de dados mais quente do mercado. Iremos abordar sobre sua arquitetura geral, tipos de dados, operações básica e como setar seu próprio _cluster redir ®_.

Esse minicurso não vai esgotar tudo o que se têm para dizer sobre redis, então recomendo fortemente que
você busque saber mais por si mesmo e explore esse mundo ~muito foda~ de NoSQL. Bons lugares para se começar: [documentação oficial](https://redis.io/docs/latest/), [código fonte](https://github.com/redis/redis) e [wikipedia](https://en.wikipedia.org/wiki/Redis). Espero que goste :)

# Introdução

Mas afinal, o que é o redis? 

Ele é um banco de dados _in-memory_ bem simples _(yet powerful)_. Você pode pensar nele como um **servidor de estrutura de dados**, isso significa que: 

> O redis provê acesso a estruturas de dados mutáveis através de um conjunto de comandos que são enviados usando um modelo cliente-servidor. Dessa forma, diferentes processos podem realizar pesquisas e modificações nas mesmas estruturas de dados de forma compartilhada. [[1]](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis)

Por conta disso ele é MUITO utilizado como servidor de cache [[2]](https://www.akitaonrails.com/2021/12/08/akitando-110-como-fazer-o-ingresso-com-escalar-conceitos-intermediarios-de-web) [[3]](https://status.openai.com/incidents/fk0tcbydtybr), comunicação entre serviços [[4]](https://www.youtube.com/watch?v=rP9EKvWt0zo) e muito mais.


# Instalação

Vou ser sincero, não quero perder tempo com isso, siga as [instruções](https://redis.io/docs/latest/operate/oss_and_stack/install/install-stack/) e seja feliz. 

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

Como eu disse, redis é um servidor de estrutura de dados, essas estruturas são definidas como `data types`. Vamos passar por cima de cada uma delas:

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


