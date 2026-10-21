# #686 — Aceite de plantão na reabertura de chamado

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/686
- PR ERP: https://github.com/cloudpark/cloudpark-erp/pull/700 (branch `cloudpark/cloudpark-erp/686`)
- PR app: https://github.com/cloudpark/cloudpark-parkpro/pull/145
- PR app para `release_candidate`: https://github.com/cloudpark/cloudpark-parkpro/pull/158 (branch `cloudpark/cloudpark-erp/686-rc`, mesclada em 2026-10-21). A branch fica no repo `cloudpark-parkpro`, não no ERP; o ERP não tem branch `release_candidate`.
- Notas técnicas: [[CloudPark ERP - Notas técnicas]]
- Situação em 2026-10-16: ajustes da 2ª rodada de review enviados (`58067b0e`, `7f88392c`); o usuário pediu a nova revisão ao rkoball.

## Problema
O chamado reaberto durante o plantão não passava pela regra de plantão da abertura. Não exigia o aceite da cobrança nem avisava o plantonista, e o chamado ficava sem atendimento no fim de semana.

## O que a PR entrega
- O `reopen` do app exige `on_call_charge_accepted` em plantão (`400 ON_CALL_ACCEPTANCE_REQUIRED`) e chama `OnCallDomain.notify_round` no commit.
- Tolerância temporária: o campo **ausente** não bloqueia a reabertura (é o app em produção), o plantonista é avisado e a ação fica no log. `TODO(cloudpark-erp#686)` no `reopen`.
- Chamado encerrado por inatividade pode aguardar na fila normal (`on_call_wait_for_queue`), sem aceite e sem aviso ao plantão.
- Os aceites vão para um histórico próprio, `OnCallChargeAcceptance`. O ciclo é identificado por `ticket_opened_at` igual à `creation_date` do chamado.
- O `reopen` passou a respeitar o escopo do usuário: chamado de outro pátio dá 404. Ele também trava a reabertura com `select_for_update`.
- `reset_fields_for_reopen` zera os campos de atendimento de plantão do ciclo anterior.

## Review (2026-10-14) e ajustes (2026-10-15)
- **Migration:** a `0099` gerava duas folhas no `support`. Depois do rebase no master ela virou `0103_on_call_charge_acceptance`, dependendo da `0102_merge_20261014_1620`. Renumerar foi preferido a uma merge migration porque ela nunca tinha sido aplicada fora do ambiente local.
- **Regra do aceite duplicada:** ficou em `OnCallChargeAcceptanceDomain.is_required(parking, request_type, team=None)` (o `is_on_call_window_now` saiu na 2ª rodada, ver abaixo). Usam essa regra a abertura, o `reopen`, o `opening-check` e o `on-call-charge-check`.
- **Getters do aceite duplicados:** foram para o `OnCallChargeAcceptanceFieldsMixin` em `support/serializers.py`. O campo de data continua em cada serializer: `_at_label` no app e `_at` no ERP.
- No rebase, o master tinha passado a considerar feriado por pátio (`get_effective_ranges(date, parking)`). A reabertura passou a seguir isso, e o item entrou no checklist da issue.

## 2ª rodada de review (2026-10-15) e ajustes (2026-10-16)
Motivo: a #731 entrou no master com o critério único de plantão e novas migrations do `support`.
- **Regra do aceite:** `is_required` resolve o time por `TicketResponsibleTeamDomain.resolve` e delega para `OnCallDomain.is_on_call_ticket(team, parking)`: o chamado é de plantão quando está na escala e o time está fora do próprio expediente, ou quando é feriado. Antes, time com horário de atendimento nunca exigia aceite, e a reabertura divergia da abertura. `OnCallDomain.is_on_call_window_now` foi removido (repetia o `is_on_call_time`). Commit `7f88392c`.
- **Migration:** virou `0104_on_call_charge_acceptance`, dependendo de `0103_merge_20261015_1601`. Commit `58067b0e`.
- No rebase ficou mantida a marcação `is_on_call` / `on_call_origin` da #731 na abertura pelo app.
- Testes do `parkpro` partiram da versão do master, que faz patch de `OnCallDomain.is_on_call_ticket`. O patch vale para o `is_required`, porque é feito no atributo da classe.

## Pendências
- Depois do deploy: PR que faz `DROP COLUMN` de `support_ticket.on_call_charge_accepted_by_id` / `on_call_charge_accepted_at`, copiando antes os aceites gravados durante o rolling update. Marcado com `TODO(cloudpark-erp#686)` na `0104`.
- Depois que a versão do app que envia o aceite estiver adotada: remover a tolerância ao campo ausente.
- Fora do escopo: o `reopen` decide pelo time do tipo de chamado, mas desde a #691 o chamado reaberto fica no time anterior. Alinhar exige mudar o app junto.
- Fora do escopo: `tickets/{id}/messages/` busca o chamado sem escopo do usuário.
