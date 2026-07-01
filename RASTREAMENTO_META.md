# Rastreamento Meta — Formulário Vero (Veromidia)

Configuração de rastreamento de conversões do formulário, no padrão do PRD do formulário Aios.
Arquitetura escolhida: **client-side** (Meta Pixel no navegador + leads enviados direto ao n8n). Sem backend/CAPI no site.

Última atualização: 2026-06-30.

---

## 1. Identificadores

| Recurso | Valor |
|---|---|
| Dataset / Pixel | `1624871735228400` — "Protocolo Fecha-tudo" |
| Business | `1418285908753284` — "BM - Vero Midia" |
| App do token (System User) | `995081473496483` — "Vero" |
| Conta de anúncios (conversões) | `act_752375456840924` — CA1 |
| Versão Graph API | `v21.0` |
| Webhook n8n | `https://aios-n8n-webhook.yspmhc.easypanel.host/webhook/aplicacao` |

> ⚠️ O App ID informado no setup (`1700536861125654`) **não** é o app do token. O token pertence ao app `995081473496483` ("Vero"). Para CAPI/`appsecret_proof` o que vale é esse app.

## 2. Eventos rastreados (Pixel, no navegador)

| Evento | Tipo | Dispara quando |
|---|---|---|
| `PageView` | padrão | carrega a página |
| `ViewContent` | padrão | vê a capa |
| `IniciouFormulario` | custom | clica "Quero lotar minha agenda" |
| `Lead` | padrão | qualifica e chega no calendário |
| **`Schedule`** | padrão | **confirma o agendamento OU clica no encaixe via WhatsApp** |
| `Purchase` | padrão | espelho do agendamento (valor R$1, p/ otimizar por Vendas) — dispara no agendamento **e no encaixe via WhatsApp** |
| `Contact` | padrão | clica "Tentar encaixe via WhatsApp" |
| `Desqualificado` | custom | responde "Não quero investir agora" na Q2 |

Cada evento leva um `eventID` único + `fbp`/`fbc` (cookies) — preparado para futura deduplicação caso ligue CAPI.

## 3. Conversões personalizadas (criadas na CA1)

| Nome | Categoria | Evento base | ID |
|---|---|---|---|
| Agendamento - Vero | SCHEDULE | `Schedule` | `1468081591673990` |
| Lead Qualificado - Vero | LEAD | `Lead` | `2269021987239009` |
| Agendamento (Vendas) - Vero | PURCHASE | `Purchase` | `899858385740404` |

Regra de cada uma: `{"and":[{"event":{"eq":"<Evento>"}},{"url":{"i_contains":"http"}}]}` (a cláusula de URL é obrigatória na API; `http` = qualquer página deste pixel, que é dedicado ao Vero). Para restringir a um domínio depois, troque `http` pelo domínio em produção.

## 4. Fluxo do formulário

```
Capa → Nome → WhatsApp → Instagram → Q1 → Q2
   • Q2 = "Não quero investir agora"  ➜ DESQUALIFICA (tela "Obrigado" + Desqualificado + n8n status=desqualificado)
   • senão ➜ Lead ➜ Calendário ➜ Agendar ➜ Schedule + Purchase + n8n status=agendado ➜ Sucesso
```

### Agenda (escassez)
- **3 dias livres por semana:** Segunda, Quarta e Sexta (`FREE_WEEKDAYS=[1,3,5]`). Demais dias úteis no horizonte aparecem riscados como "ocupado"; fim de semana e fora da janela ficam apagados.
- **3 horários livres por dia:** escolhidos por data a partir de `ALL_TIMES` (`freeTimesFor`); o resto aparece "ocupado".
- **Botão "Tentar encaixe via WhatsApp"** abaixo do "Agendar": abre `wa.me/351961342444` (+351 961 342 444) com mensagem pré-preenchida (nome + @), dispara `Contact` + `Schedule` + `Purchase` (o encaixe conta como "agendamento feito", inclusive p/ otimizar campanha de Vendas) e envia ao n8n com `status=encaixe_whatsapp`. Número configurável em `WA_NUMBER`.

## 5. Payload enviado ao n8n

```json
{
  "origem": "vero",
  "status": "agendado | desqualificado",
  "lead_id": "<uuid estável do lead>",
  "nome": "", "telefone": "", "instagram": "",
  "q1": "", "q2": "", "data": "", "hora": "",
  "fbp": "", "fbc": "", "pagina": "", "ts": "<ISO>",
  "motivo": "<quando desqualificado ou encaixe_whatsapp>"
}
```

`status` possíveis: `agendado` · `desqualificado` · `encaixe_whatsapp`.

`lead_id` é o mesmo em todos os envios do mesmo visitante — use para dedupe/upsert no n8n.
Envio é client-side (a URL do webhook fica visível no navegador — trade-off da arquitetura sem backend).

## 6. Verificação feita (2026-06-30)

- ✅ Pixel inicializa com o ID certo (config carregado no navegador).
- ✅ Fluxo qualificado dispara, em ordem: `IniciouFormulario → Lead → Schedule → Purchase ($1 BRL)`; tela final "sucesso"; payload n8n `status=agendado` correto.
- ✅ Fluxo de desqualificação: `IniciouFormulario → Desqualificado`; tela "Obrigado pelo interesse!"; payload n8n `status=desqualificado` + `motivo`.
- ✅ Dataset aceita eventos: CAPI test event (`Schedule`) → `events_received: 1` (Test Events `TEST_VERO_001`, não conta nos dados reais).
- ⚠️ A entrega do beacon `/tr` não é testável em headless/localhost. **Verifique em produção** com a extensão **Meta Pixel Helper** ou em **Gerenciador de Eventos → Testar eventos** (cole a URL ao vivo).

## 7. Pendências / limpeza

- 🔴 **Segurança:** o App Secret foi digitado em texto no setup. **Rotacione** no painel do app "Vero" (`995081473496483`). Ele não está em nenhum arquivo do projeto.
- 🟡 **Apagar 2 conversões de teste** "ZZ APAGAR (teste)" na CA1 (Gerenciador de Eventos → Conversões personalizadas). A Graph API **não permite apagar** conversão personalizada (só renomear), por isso ficaram marcadas.
- 🟡 (Opcional) Restringir a regra das conversões ao domínio de produção (trocar `http`).
- 🟡 (Opcional) Ajustar `META_BOOKING_VALUE` (hoje R$1) se quiser otimizar por valor.

## 8. Notas de implementação

Tudo no `index.html` (sem build). Constantes no topo do `<script>`:
`WEBHOOK_URL`, `BOOKING_VALUE`, `ORIGEM`. O Pixel base fica no `<head>`.
Pixel ID, dataset, conta e IDs de conversão **não são segredos** (podem aparecer no cliente/doc). Token e App Secret **nunca** vão para o código.
