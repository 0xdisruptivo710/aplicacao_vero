# Aplicação Vero — Formulário de qualificação

Formulário de qualificação de leads (estilo Chronos) da **Veromidia**, agência de tráfego pago especializada em clínicas de saúde e estética.

## Fluxo

Capa → Nome → WhatsApp → @Instagram → "Já investe em tráfego pago?" → "Qual investimento mensal faz sentido?" → Agendamento (calendário + horário) → Confirmação.

Ninguém é desqualificado: as três faixas de investimento da última pergunta seguem para o agendamento. A faixa escolhida vira o valor do evento `Purchase` do Meta (3 = a partir de R$3.000/mês, 2 = depende do retorno, 1 = ainda não consigo), para o algoritmo continuar otimizando para quem tem orçamento.

## Stack

HTML/CSS/JS estático em um único arquivo (`index.html`). Fonte Lato via Google Fonts. Zero dependências, zero build.

## Deploy (Vercel)

Site estático, detectado automaticamente:

```bash
npx vercel          # ou: npm i -g vercel && vercel
```

Ou arraste a pasta em [vercel.com/new](https://vercel.com/new).

## Integração com CRM

No topo do `<script>` em `index.html`, defina a URL do webhook para enviar os leads automaticamente:

```js
var WEBHOOK_URL = "https://seu-crm/webhook";
```

Payload enviado no submit: `nome`, `telefone`, `instagram`, `q1`, `q2`, `qualificacao` (`alta` | `media` | `baixa`), `data`, `hora`, além de `status`, `lead_id`, `fbp`/`fbc` e `origem`.

Use `qualificacao` para rotear ou priorizar o atendimento — é a faixa de investimento da última pergunta, já normalizada (o campo `q2` traz o texto literal da opção).
