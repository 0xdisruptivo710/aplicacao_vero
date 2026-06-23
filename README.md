# Aplicação Vero — Formulário de qualificação

Formulário de qualificação de leads (estilo Chronos) da **Veromidia**, agência de tráfego pago especializada em clínicas de saúde e estética.

## Fluxo

Capa → Nome → WhatsApp → @Instagram → "Já investe em tráfego pago?" → "Disposto a investir R$3.000/mês?" → Agendamento (calendário + horário) → Confirmação.

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

Payload enviado no submit: `nome`, `telefone`, `instagram`, `q1`, `q2`, `data`, `hora`.
