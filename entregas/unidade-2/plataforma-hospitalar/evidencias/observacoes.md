# Oficina do módulo 2 — contrato, execução e comparação

Davy Woolley · API de elegibilidades da plataforma hospitalar

## O que rodei

| Ferramenta | O que examinou | Evidência |
|---|---|---|
| pytest / TestClient | a implementação | `testes-contrato.txt` — 7 passed |
| Spectral 6.16.1 | o documento | `spectral-valido.txt` — sem erros |
| Spectral, falha deliberada | o documento com exemplo inválido | `spectral-falha-deliberada.txt` — código 1 |
| curl contra o Uvicorn | a experiência do consumidor | `resposta-post-202.txt`, `resposta-get-200.txt`, `resposta-422.txt` |
| Bruno | consumidor manual | coleção em `bruno/elegibilidades/` |

O POST devolveu `202 Accepted` com `location: /elegibilidades/1650dcbc-…`, o GET
com esse protocolo devolveu `200`, o GET com um protocolo inventado devolveu
`404 elegibilidade_nao_encontrada`, e o POST sem `cpf` devolveu
`422 dados_invalidos` apontando `body.cpf`.

## Contrato explícito, contrato gerado e execução

As três coisas se sobrepõem, mas não são a mesma. Encontrei uma divergência real
comparando `contratos/openapi.yaml` com o `/openapi.json` que o FastAPI gera:

```
GET /elegibilidades/{protocolo}
    escrito: ['200', '404']
    gerado : ['200', '404', '422']
```

O FastAPI documenta sozinho um 422 no GET, porque valida o parâmetro de caminho.
O contrato escrito não menciona esse status. Nenhum dos sete testes reclama, e o
Spectral também não — ele lê o documento, não a aplicação. É uma divergência que
só aparece quando alguém compara os dois de propósito.

Isso resume a divisão de trabalho: o documento explica a intenção, o teste
escolhe algumas amostras, e a execução mostra o que realmente acontece. O
Spectral acha problema estrutural sem nunca chamar a API; o TestClient verifica a
API sem garantir que ela obedece ao documento; o Bruno mostra como é ser
consumidor, mas uma execução manual não é regressão. Nenhuma das três substitui a
leitura humana do contrato.

## A falha deliberada

Em `openapi-experimento.yaml` troquei o `cpf` do exemplo de mídia de
`'12345678901'` para `'123'`, só na linha 35. O Spectral reprovou:

```
35:24  error  oas3-valid-media-example  "cpf" property must match pattern "^\d{11}$"
```

Isso não é defeito pendente — é o resultado esperado do experimento. O
`contratos/openapi.yaml` original continua válido e não foi tocado.

A parte interessante é a simetria: o mesmo valor `123` enviado ao servidor
também é recusado, com `422` e `String should match pattern '^\d{11}$'`
(está em `resposta-422.txt`). O documento reprova antes de qualquer chamada
porque o exemplo é verificável contra o próprio schema; o servidor reprova
depois porque aplica o mesmo `pattern` em tempo de execução. Duas verificações
independentes, a mesma regra, e é justamente por serem independentes que vale
manter as duas.

## Duas coisas do ambiente

O `pyproject.toml` do laboratório não declara `pyyaml`, mas
`tests/test_api_contract.py` importa `yaml` na linha 5. Sem instalar o pacote à
parte, a coleta do pytest falha antes de rodar qualquer teste. Instalei `pyyaml`
no `.venv` e não alterei o `pyproject.toml`.

O roteiro fala em seis testes em `test_api_contract.py`; a versão atual do
repositório tem **sete**, todos passando. Rodando a suíte inteira dá
`18 passed, 2 failed, 3 skipped` — as duas falhas estão em
`test_event_idempotency.py`, que é material dos módulos seguintes e não faz
parte desta oficina.

## Questões exploratórias

**O que 202 permite ao provedor mudar sem quebrar o consumidor?**
O 202 diz "recebi e vou tratar", não "está pronto". Com isso o provedor pode
mudar quanto tempo leva, quantas etapas existem, se consulta a operadora na hora
ou depois, e se processa sozinho ou em fila — nada disso está prometido na
resposta. O que ele não pode mudar é o protocolo devolvido e o fato de existir um
lugar para consultar. Um 200 no lugar do 202 teria fechado essa porta: prometeria
decisão concluída, e todo consumidor passaria a depender disso.

**Por que Location é melhor que pedir uma URL por convenção?**
Convenção é um acordo que não está escrito em lugar nenhum. Se o consumidor monta
`/elegibilidades/{protocolo}` de cabeça, o dia em que a rota virar
`/v2/elegibilidades/{protocolo}` ou passar por um gateway quebra todo mundo ao
mesmo tempo, sem aviso. Com `Location`, quem decide o endereço é quem é dono
dele, e a mudança viaja na própria resposta.

**Qual divergência entre OpenAPI e aplicação os testes ainda não detectam?**
A que achei acima: o 422 que o FastAPI gera para o GET e o documento escrito não
declara. Os testes comparam exemplos e comportamento, mas ninguém compara a lista
de status documentados com a lista gerada. Também não é detectada divergência de
descrição — um `summary` desatualizado passa por todas as ferramentas.

**Quando uma chave de idempotência passaria a ser necessária?**
Quando o mesmo pedido puder chegar duas vezes sem que o consumidor saiba. Hoje
não pode: cada POST gera protocolo novo, e se a resposta se perder o consumidor
simplesmente não tem protocolo nenhum. O problema aparece no momento em que
houver retry automático — no cliente, num gateway ou numa fila — porque aí o
segundo envio é idêntico ao primeiro e a API não tem como distinguir "de novo" de
"outro pedido". A chave precisa vir do consumidor e ser estável entre as
tentativas.

**Que parte deixaria de funcionar com duas instâncias e memória separada?**
O GET. O protocolo criado pelo POST fica no dicionário em memória de um processo;
com duas instâncias atrás de um balanceador, o GET tem chance de cair na
instância que nunca viu aquele protocolo e responder `404` para um pedido que
existe. O POST continuaria funcionando — e é justamente por isso que o problema é
traiçoeiro: só a leitura falha, de forma intermitente. É o limite que o próprio
laboratório assume ao guardar tudo em memória.
