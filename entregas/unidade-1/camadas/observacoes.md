# Camadas — agenda clínica

## O que alterei

Em `dominio.py`, no value object `Horario`, troquei a regra de conflito. Ela
comparava a sobreposição dos dois intervalos:

```python
return self.inicio < outro.fim and self.fim > outro.inicio
```

Passei a comparar só o horário de início:

```python
return self.inicio == outro.inicio
```

A linha original ficou como comentário no próprio método, para dar para voltar
atrás lendo o arquivo.

## O que a saída revelou

Na seção 2, o agendamento das 09:15–09:45 para a Dra. Ana deixou de ser recusado.
Antes:

```
HTTP 409 CONFLICT → {'erro': 'Dr(a). Dra. Ana Silva já tem consulta das 09:00 às 09:30.'}
```

Depois:

```
HTTP 201 CREATED → {'id': 4, ... 'horario': '09:15–09:45', 'status': 'agendada'}
```

O efeito continuou aparecendo depois. A agenda da seção 3 passou a listar três
consultas em vez de duas, e a seção 7, que antes terminava com a agenda vazia,
passou a mostrar a consulta 4 ainda de pé.

## Qual responsabilidade isso mostra

A regra que impede duas consultas no mesmo horário mora inteira em um método de
um objeto do domínio. Não precisei tocar em `apresentacao.py`, `servicos.py` nem
`repositorios.py`, e mesmo assim o comportamento do sistema mudou de ponta a
ponta.

Isso responde as duas primeiras questões da oficina. A apresentação converte o
dicionário de entrada em `SolicitacaoAgendamento` e traduz a exceção
`ConflitodeAgendaError` em HTTP 409 — ela formata, não decide. O serviço busca as
consultas existentes e pergunta a cada uma se conflita, mas quem sabe responder é
o `Horario`. Quando o `Horario` mudou de opinião, o serviço obedeceu sem saber que
algo tinha mudado, e a apresentação continuou formatando corretamente: só passou
a formatar um 201 em vez de um 409.

Sobre a terceira questão: trocar o armazenamento em memória por um banco mexeria
só nas classes `InMemoria*` de `repositorios.py`. `AgendamentoServico` depende da
interface `RepositorioConsulta`, não da implementação, então a camada de negócio
fica estável. A própria seção 9 do programa existe para mostrar isso.

## Como reverter

Apagar esta cópia e copiar de novo
`codigos/cap01-estilos-fundamentais/1.2-estilo-em-camadas`.
