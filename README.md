# Arquitetura de Software — entregas

Davy Woolley · Módulo 1

Repositório com as entregas da disciplina. O código dos exemplos vem do
repositório do curso ([marco-mendes/arquitetura-software](https://github.com/marco-mendes/arquitetura-software)),
copiado para `entregas/` e alterado só nas cópias, como pede a oficina.

## Unidade 1

### Estudo de caso — plataforma hospitalar

`entregas/unidade-1/estudo-de-caso/`

Os quatro exercícios num documento só: cenários mensuráveis, matriz de estilos,
estrutura inicial com diagrama e o ADR-001. Tem versão em Markdown e em PDF.

### Oficina de ferramentas — três estilos em código

Cada pasta é uma cópia de um exemplo do capítulo 1, com a saída antes e depois de
uma alteração e uma nota explicando o que ela revelou.

| Pasta | Estilo | O que alterei | O que apareceu na saída |
|---|---|---|---|
| `entregas/unidade-1/camadas/` | Camadas | a regra de conflito em `dominio.py` passou a comparar só o horário de início | o agendamento sobreposto virou HTTP 201 em vez de 409 |
| `entregas/unidade-1/pipes-and-filters/` | Pipes and Filters | tirei o `NormalizadorDeCampos` da composição em `main.py` | todos os scores caíram para 0% e o ranking se reordenou |
| `entregas/unidade-1/microkernel/` | Microkernel | inverti `ORDEM_CATEGORIAS` em `nucleo.py` | o e-mail passou a avisar o valor bruto, sem impostos |

Os detalhes estão no `observacoes.md` de cada pasta.

## Como rodar

Python 3.10 ou mais recente, sem dependências. Dentro de cada pasta:

```bash
python main.py
```

No PowerShell do Windows, o console usa cp1252 por padrão e os programas quebram
com `UnicodeEncodeError` ao imprimir os caracteres de moldura. Defina a codificação
antes de rodar:

```bash
$env:PYTHONIOENCODING="utf-8"
py main.py
```

Isso é ajuste de terminal, não de código — os arquivos dos exemplos não foram
alterados por causa disso.

## Voltar ao original

As alterações estão marcadas com um comentário `EXPERIMENTO:` no ponto exato, com
o valor anterior ao lado. Para desfazer por completo, apague a cópia e copie de
novo o diretório correspondente em `codigos/cap01-estilos-fundamentais/` do
repositório do curso.
