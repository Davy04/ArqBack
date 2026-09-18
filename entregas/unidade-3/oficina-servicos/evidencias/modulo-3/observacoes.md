# Oficina do módulo 3: dois serviços, dois bancos e uma falha parcial

Davy Woolley

Subi os quatro contêineres com Docker Compose (Docker 29.8.0), usando as portas
18001 e 18002, e segui o roteiro inteiro: estado nominal, falha parcial,
recuperação e limpeza. Os arquivos de evidência estão numerados na ordem em que
foram gerados.

| Arquivo | O que mostra |
|---|---|
| `0-versoes.txt`, `0-config.txt`, `0-subida.txt` | versões, Compose válido com os quatro serviços e a subida com `--wait` até todos ficarem saudáveis |
| `1-estado-nominal.txt` | os quatro `healthy` no `ps` e os dois `/health` com 200 |
| `2-exame-201.txt` | `POST /exames` para paciente-001 devolvendo 201 com `solicitacao_id` 1 |
| `3-inelegivel-e-fronteira-de-rede.txt` | o 422 de um beneficiário inelegível e o teste de rede feito de dentro do contêiner de Exames |
| `4-falha-parcial-503.txt` | Elegibilidade parada: 503 na operação e 200 no health, da mesma instância |
| `5-latencia-do-503.txt` | quanto tempo o 503 leva para chegar |
| `6-recuperacao.txt` | tudo saudável de novo e um novo 201 |
| `7-testes-fronteira.txt` | os quatro testes de fronteira passando |
| `8-limpeza.txt` | `down -v` removendo contêineres, redes e volumes |

## O momento da falha parcial

Com a Elegibilidade parada, o `docker compose ps` continuou mostrando Exames como
`healthy`. Mandei o mesmo pedido que antes tinha dado 201 e recebi:

```
HTTP/1.1 503 Service Unavailable
{"detail":{"codigo":"dependencia_indisponivel"}}
```

Três segundos depois, o health da mesma instância respondeu:

```
HTTP/1.1 200 OK
{"status":"ok","servico":"exames"}
```

As duas respostas estão certas, cada uma do seu ponto de vista. O health de
Exames só pergunta se o processo está vivo e se o próprio banco responde, e as
duas coisas eram verdade. A operação de negócio precisa perguntar à Elegibilidade
se o paciente pode ser atendido, e não havia ninguém do outro lado para
responder. O processo estava no ar, mas a capacidade de pedir exame não estava.

Se o health de Exames também consultasse a Elegibilidade, o Docker marcaria
Exames como doente e poderia reiniciá-lo. Isso não resolveria nada, porque o
problema estava no vizinho, e ainda apagaria a informação de que Exames em si
estava bem.

**Respondendo à questão exploratória:** quem permaneceu saudável foi o próprio
Exames, com seu processo e seu banco. O que deixou de ser concluído foi a
solicitação de exame, porque ela atravessa a fronteira até a Elegibilidade.

## A fronteira de dados está na rede

Essa foi a evidência que mais me convenceu. De dentro do contêiner de Exames,
tentei resolver três nomes:

```
db_exames         resolve: 172.19.0.2
db_elegibilidade  NAO resolve: [Errno -2] Name or service not known
elegibilidade     resolve: 172.20.0.2
```

Exames enxerga o próprio banco e enxerga o serviço de Elegibilidade, mas nem
sabe que o banco da Elegibilidade existe. Não é uma questão de permissão negada
na hora de conectar. O nome simplesmente não existe na rede dele. Mesmo que
alguém escrevesse um `SELECT` na tabela de beneficiários dentro de `exames.py`, a
conexão não teria para onde ir. O quarto teste verifica isso no código-fonte,
mas é o `internal: true` do Compose que garante na prática.

## O prazo de 2 segundos não foi o tempo que eu medi

O cliente HTTP de Exames tem `timeout=2.0`, então eu esperava o 503 em até uns
dois segundos. Medi três vezes com a Elegibilidade parada:

```
POST /exames -> 503 em 3.33s
POST /exames -> 503 em 3.31s
POST /exames -> 503 em 3.34s
GET /health  -> 200 em 0.01s
```

O 503 chegou em cerca de 3,3 segundos, todas as vezes. Minha leitura é que o
prazo do httpx vale para conectar e ler, mas não para resolver o nome
`elegibilidade`. Com o contêiner parado, o nome some do DNS interno do Docker, e
essa busca consome o tempo a mais antes de o httpx sequer tentar conectar.

Isso não muda o resultado, que continua sendo um 503 correto e sem travar. Mas
mostra que um prazo declarado no código não é o mesmo que o tempo que o
consumidor espera. Se alguém dimensionasse um timeout lá na ponta achando que
Exames sempre responde em 2 segundos, ia errar.

## Decisão de negócio não é falha técnica

Com tudo no ar, pedi exame para o paciente-002, que está cadastrado como não
elegível, e recebi:

```
HTTP/1.1 422 Unprocessable Entity
{"detail":{"codigo":"beneficiario_inelegivel"}}
```

É uma resposta diferente do 503, e faz sentido que seja. O 422 é a Elegibilidade
funcionando e dizendo "não". O 503 é a Elegibilidade sem responder. Se os dois
casos virassem o mesmo erro, quem consome Exames não teria como saber se deve
tentar de novo mais tarde ou avisar o paciente de que o plano não cobre o exame.

## A recuperação preservou o que foi gravado

Depois de subir a Elegibilidade de novo, o mesmo pedido voltou a dar 201, agora
com `solicitacao_id` 2. O 1 continuava lá, gravado antes da falha. A queda do
vizinho não mexeu no banco de Exames, e a falha ficou restrita ao caminho que
cruza a fronteira.
