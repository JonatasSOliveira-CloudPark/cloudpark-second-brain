# #616 — Reforço de plantão (a cada 5 min após o aceite)

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/616
- Repos: ERP (`cloudpark/cloudpark-erp/616`) e app ParkPro (`cloudpark/cloudpark-erp/616-reforco-plantao`, com a `-test` para o PR em `test`; mãe `cloudpark/cloudpark-erp/616`).
- PRs: ERP [#897](https://github.com/cloudpark/cloudpark-erp/pull/897) para `master`; app [#175](https://github.com/cloudpark/cloudpark-parkpro/pull/175) para `test`.
- Decisões: [[2026-11-10 - Reforço de plantão (#616)]].

## Estado (2026-11-10)
- Código das duas partes pronto e revisado. O app teve `CHANGES_REQUESTED` do rkoball; as correções (navegação sobre alarme de outro chamado, cancelamento com id 0) foram publicadas e pedi nova revisão.
- Pendente: testes no aparelho (foreground, background, app fechado, Não perturbar, som, cancelamento na bandeja, iOS).
- Pendente: teste e2e `test_opening_by_link_records_the_agent_view` roda no CI do PR (não rodou localmente).
- Ordem de deploy: o app publicado e adotado antes de ligar `OnCallReinforcementConfig.push_enabled` no ERP. Depois, remover a flag (TODO #616).
- Build nativo novo do app (dois sons em `app.base.json`).

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[CloudPark ERP - Notas técnicas]]
- [[739 - Alarme de plantão em celular compartilhado]]
