# Pipes and Filters — triagem de currículos

## O que alterei

Em `main.py`, comentei uma linha da composição do pipeline: tirei o
`NormalizadorDeCampos` de dentro do fluxo.

```python
.adicionar(ValidadorDeCurriculo())
# EXPERIMENTO: filtro removido da composição
# .adicionar(NormalizadorDeCampos())
.adicionar(FiltroPorExperienciaMinima(vaga))
```

Nenhum filtro foi alterado por dentro. Só a ordem de montagem mudou.

## O que a saída revelou

Três coisas de uma vez.

A própria linha que descreve o pipeline encolheu:

```
Pipeline: Pipeline(LeitorDeCurriculos → ValidadorDeCurriculo → FiltroPorExperienciaMinima → ...)
```

Os descartes continuaram idênticos. Currículo id=3 caiu por nome ausente, Bruno
por experiência e Clara por pretensão salarial — os mesmos três, pelos mesmos
motivos. Continuaram sendo 3 aprovados.

Mas todo mundo zerou o score:

```
1.   ana lima
   Score: ░░░░░░░░░░ 0%
   Habilidades compatíveis: —
```

O nome saiu como veio do dado bruto, com espaços e minúsculas. E a ordem do
ranking mudou: Diego passou na frente de Elena, porque com todo mundo empatado em
zero o desempate virou outro.

## Qual responsabilidade isso mostra

O `CalculadorDeScore` compara as habilidades do candidato com as da vaga em
minúsculas. Enquanto o normalizador estava no fluxo, ele entregava as habilidades
já padronizadas. Sem ele, o cálculo passou a comparar `"Python"` com `"python"` e
não achou nada em comum.

O que isso mostra do estilo: cada filtro faz uma coisa só e confia no formato que
o anterior entregou. Essa é a força e o preço do pipes and filters ao mesmo tempo.
A força é que dá para tirar, pôr e reordenar etapas mexendo só na composição, sem
abrir nenhum filtro. O preço é que o contrato entre eles é implícito — o
calculador não avisa que depende da normalização, ele simplesmente devolve zero.

Sobre as questões da oficina: o `LeitorDeCurriculos` é quem recebe o dado bruto e
o `RelatorioDeTriagem` é quem apresenta o resultado. Itens saem do fluxo nos
testers (validador e os dois filtros); nos transformers ninguém é descartado, só
alterado. O ranking fica no fim porque ordenar exige o score já calculado, e o
score exige que os descartes já tenham acontecido — reordenar não é livre, o que
este experimento mostrou na prática.

## Como reverter

Apagar esta cópia e copiar de novo
`codigos/cap01-estilos-fundamentais/1.3-pipes-and-filters`.
