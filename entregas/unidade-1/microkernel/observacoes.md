# Microkernel — faturamento por plugins

## O que alterei

Em `nucleo.py`, inverti a ordem das categorias. A notificação passou a rodar
antes dos impostos:

```python
# EXPERIMENTO: ordem original era ["impostos", "frete", "notificacao"]
ORDEM_CATEGORIAS = ["notificacao", "impostos", "frete"]
```

Nenhum plugin foi tocado. Nenhuma fatura foi tocada. Só uma lista no núcleo.

## O que a saída revelou

As quatro faturas passaram a mandar e-mail com o valor errado. A fatura #1001:

```
antes:  [Email → financeiro@techcorp.com] ... Total: R$13,400.00
depois: [Email → financeiro@techcorp.com] ... Total: R$12,000.00
```

R$12.000 é o valor bruto. O e-mail saiu antes de o ICMS-SP e o ISS-SP entrarem na
conta. Aconteceu o mesmo nas quatro faturas: #1002 avisou R$8.400 em vez de
R$10.080, #1003 avisou R$75.000 em vez de R$84.000, #1004 avisou R$45.000 em vez
de R$53.100.

O detalhe importante é que o bloco "Resultado da emissão" continuou certo. ICMS,
ISS, frete e total finais não mudaram em nenhuma fatura. Quem ficou errado foi só
quem leu o resultado cedo demais.

## Qual responsabilidade isso mostra

O núcleo não sabe o que é ICMS, o que é frete nem como se manda um e-mail. Ele
conhece um contrato — `processar(fatura, resultado)` devolvendo um resultado — e
uma ordem de categorias. Toda a regra fiscal está nos plugins, e cada um decide
sozinho se o contexto se aplica: ICMS-RJ não contribuiu nada nas faturas de SP e o
ICMS-MG só apareceu depois de ser registrado, na última seção, sem que o núcleo
fosse recompilado.

Essa divisão é o que torna a mudança tão barata e tão perigosa. Barata porque
acrescentar MG custou uma linha de registro. Perigosa porque `ORDEM_CATEGORIAS` é
uma dependência que não aparece em lugar nenhum do contrato: o plugin de
notificação depende de os plugins de imposto já terem rodado, mas nada no código
expressa isso. Trocar a ordem no núcleo é uma linha, e quebra silenciosamente um
plugin que ninguém abriu.

Responde a segunda e a terceira questões da oficina: a ordem afeta a notificação
mas não o total, e a saída mostra a contribuição de cada regra na lista do
"Resultado da emissão" — SP recebe ICMS-SP e ISS-SP, RJ recebe só ICMS-RJ, e o
frete aparece zerado nas faturas acima de R$5.000.

## Como reverter

Apagar esta cópia e copiar de novo
`codigos/cap01-estilos-fundamentais/1.4-microkernel`.
