# Arquitetura de Software — entregas

Davy Woolley

## Módulo 1 — Oficina: três estilos em código

Cada pasta é uma cópia de um exemplo do capítulo 1 do
[repositório do curso](https://github.com/marco-mendes/arquitetura-software), com
a saída antes e depois de uma alteração e uma nota explicando o que ela revelou.

| Pasta | Estilo | O que alterei | O que apareceu na saída |
|---|---|---|---|
| [`camadas/`](entregas/unidade-1/camadas/) | Camadas | a regra de conflito em `dominio.py` passou a comparar só o horário de início | o agendamento sobreposto virou HTTP 201 em vez de 409 |
| [`pipes-and-filters/`](entregas/unidade-1/pipes-and-filters/) | Pipes and Filters | tirei o `NormalizadorDeCampos` da composição em `main.py` | todos os scores caíram para 0% e o ranking se reordenou |
| [`microkernel/`](entregas/unidade-1/microkernel/) | Microkernel | inverti `ORDEM_CATEGORIAS` em `nucleo.py` | o e-mail passou a avisar o valor bruto, sem impostos |

Os detalhes estão no `observacoes.md` de cada pasta. As alterações estão marcadas
com um comentário `EXPERIMENTO:` no código, ao lado do valor original.

**Como rodar:** Python 3.10+, sem dependências. Dentro de cada pasta,
`python main.py`. No PowerShell, rode `$env:PYTHONIOENCODING="utf-8"` antes — sem
isso o console quebra ao imprimir os caracteres de moldura.

## Módulo 2 — Oficina: contrato, execução e comparação

[`entregas/unidade-2/plataforma-hospitalar/`](entregas/unidade-2/plataforma-hospitalar/)
— a API de elegibilidades do laboratório, com as evidências em `evidencias/`.

| Evidência | O que mostra |
|---|---|
| `testes-contrato.txt` | 7 testes de contrato aprovados |
| `spectral-valido.txt` | contrato sem erros no Spectral 6.16.1 |
| `spectral-falha-deliberada.txt` | `oas3-valid-media-example` com código 1, o experimento do exemplo inválido |
| `resposta-post-202.txt` | 202 Accepted com protocolo e cabeçalho `Location` |
| `resposta-get-200.txt` | 200 recuperando pelo protocolo, e 404 para protocolo inexistente |
| `resposta-422.txt` | 422 sem `cpf` e 422 com `cpf` fora do padrão |
| `openapi-experimento.yaml` | a cópia com a falha deliberada, preservada |
| `openapi-gerado.json` | o contrato que o FastAPI gera, para comparar com o escrito |
| `bruno/elegibilidades/` | coleção Bruno com as quatro requisições |

A nota com a comparação e as questões exploratórias está em
[`evidencias/observacoes.md`](entregas/unidade-2/plataforma-hospitalar/evidencias/observacoes.md).
Nela fica registrada a divergência encontrada: o contrato gerado declara um `422`
no `GET /elegibilidades/{protocolo}` que o contrato escrito não menciona, e
nenhuma das ferramentas reclama disso sozinha.

### Como rodar

```bash
cd entregas/unidade-2/plataforma-hospitalar
py -m venv .venv
.venv\Scripts\python.exe -m pip install -e ".[dev]" pyyaml
.venv\Scripts\python.exe -m pytest tests/test_api_contract.py -q
.venv\Scripts\python.exe -m uvicorn hospital.api.main:app --reload
```

O `pyyaml` entra à parte porque o `pyproject.toml` do laboratório não o declara,
embora `tests/test_api_contract.py` importe `yaml`. Com a API no ar, abra
http://127.0.0.1:8000/docs ou importe `evidencias/bruno/elegibilidades/` no Bruno.
