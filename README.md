# Arquitetura de Software — entregas

Davy Woolley · Módulo 1

Oficina de ferramentas: três estilos arquiteturais em código executável. Cada
pasta é uma cópia de um exemplo do capítulo 1 do
[repositório do curso](https://github.com/marco-mendes/arquitetura-software), com
a saída antes e depois de uma alteração e uma nota explicando o que ela revelou.

| Pasta | Estilo | O que alterei | O que apareceu na saída |
|---|---|---|---|
| [`camadas/`](entregas/unidade-1/camadas/) | Camadas | a regra de conflito em `dominio.py` passou a comparar só o horário de início | o agendamento sobreposto virou HTTP 201 em vez de 409 |
| [`pipes-and-filters/`](entregas/unidade-1/pipes-and-filters/) | Pipes and Filters | tirei o `NormalizadorDeCampos` da composição em `main.py` | todos os scores caíram para 0% e o ranking se reordenou |
| [`microkernel/`](entregas/unidade-1/microkernel/) | Microkernel | inverti `ORDEM_CATEGORIAS` em `nucleo.py` | o e-mail passou a avisar o valor bruto, sem impostos |

Os detalhes estão no `observacoes.md` de cada pasta. As alterações estão marcadas
com um comentário `EXPERIMENTO:` no código, ao lado do valor original.

## Como rodar

Python 3.10 ou mais recente, sem dependências. Dentro de cada pasta:

```bash
python main.py
```

No PowerShell, rode `$env:PYTHONIOENCODING="utf-8"` antes — sem isso o console
quebra ao imprimir os caracteres de moldura.
