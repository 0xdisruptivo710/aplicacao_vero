# Q2 — de teste de compromisso para faixas de investimento

**Data:** 2026-07-11
**Arquivo afetado:** `index.html` (único arquivo do projeto)

## Problema

A pergunta 5 do formulário (`data-step="q2"`) era um teste de compromisso, não uma pergunta:

> Está disposto a investir pelo menos R$3.000/mês em anúncios, sem contar nossa mão de obra?
> *Se marcar "não", não vamos te convencer do contrário.*
> ✅ Sim, preciso de tráfego profissional / ❌ Não quero investir agora

Três coisas a tornavam agressiva: o verbo pedia compromisso antes de qualquer conversa; o subtexto avisava que uma das respostas encerrava o assunto; e o ❌ marcava explicitamente uma resposta como errada. Quem marcasse a opção B era desviado para uma tela de desqualificação e nunca via o calendário.

## Decisão

A Q2 vira uma pergunta de planejamento com três faixas, e **ninguém sai do funil** — todas as respostas seguem para o agendamento. A qualificação deixa de ser um portão e passa a ser um dado.

### Nova copy

> **Qual investimento mensal em anúncios faz sentido para a sua clínica hoje?** \*
>
> Esse valor é só a verba que vai para as plataformas — não inclui a nossa mão de obra. Não tem resposta errada; é para a gente preparar a conversa certa para o seu momento.
>
> - **A.** Consigo investir R$3.000/mês ou mais
> - **B.** Depende do retorno — quero entender melhor antes de definir
> - **C.** Hoje ainda não consigo esse valor

Sem emojis de aprovação/reprovação nas opções.

### Fluxo

O ramo de desqualificação é removido por inteiro: o atributo `data-disqualify`, a flag `q2Disqualify`, a função `disqualify()` e a tela `data-step="desqualificado"`. Nada mais alcançava essa tela, então ela sai junto em vez de virar código morto.

### Sinal para o Meta

Como todo mundo passa a agendar, `Purchase` com valor fixo ensinaria o algoritmo que lead frio vale o mesmo que lead quente. Cada opção ganha um `data-score` e o `Purchase` passa a disparar com esse valor:

| Opção | score | `qualificacao` | `Purchase.value` |
|---|---|---|---|
| A — R$3.000+ | 3 | `alta` | 3 |
| B — depende do retorno | 2 | `media` | 2 |
| C — ainda não consigo | 1 | `baixa` | 1 |

`Schedule` e `Lead` continuam iguais para todos. O mesmo valor por faixa vale para o agendamento normal (`book()`) e para o encaixe via WhatsApp.

### Payload do n8n

Além de `q2` (texto literal da opção, mantido), o payload ganha `qualificacao` com `alta` / `media` / `baixa`, para permitir roteamento e priorização no n8n sem depender de comparação de strings.

## Capa

Com a Q2 deixando de barrar, a linha final da capa virava a única barreira do funil — e era mais dura que a pergunta suavizada:

> → Exclusivo para clínicas de saúde e estética dispostas a investir no mínimo R$3.000/mês em anúncios.

O valor sai da capa por completo e passa a aparecer só na Q2. A linha vira um convite:

> → Feito para clínicas de saúde e estética que querem crescer com previsibilidade, não com sorte.

Consequência aceita: some a autosseleção antecipada. Mais gente entra no formulário e descobre a faixa de investimento no meio do caminho, em vez de desistir na capa. O `Purchase` ponderado por faixa é o que impede esse volume extra de virar sinal ruim para o Meta.
