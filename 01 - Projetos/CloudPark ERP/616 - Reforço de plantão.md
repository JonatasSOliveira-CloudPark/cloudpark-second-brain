# #616 — Reforço de plantão (a cada 5 min após o aceite)

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/616
- Repos: ERP (`cloudpark/cloudpark-erp/616`) e app ParkPro (`cloudpark/cloudpark-erp/616-reforco-plantao`, com a `-test` para o PR em `test`; mãe `cloudpark/cloudpark-erp/616`).
- PRs: ERP [#897](https://github.com/cloudpark/cloudpark-erp/pull/897) mesclada em `master` (11/11); app [#175](https://github.com/cloudpark/cloudpark-parkpro/pull/175) mesclada em `test` (11/11, aprovada); RC [#185](https://github.com/cloudpark/cloudpark-parkpro/pull/185) para `release_candidate`, aberta, sem revisor.
- Decisões: [[2026-11-10 - Reforço de plantão (#616)]].

## Estado (2026-11-18)
- ERP e app em `master` e `test`. Promoção para RC aberta (PR #185) e comentada na issue. Sem merge em `release_candidate` ou `stable`.
- QA aprovou em 18/11 na versão master, com arquivo de evidência anexado à task.
- Testes locais em 16/11: alarme entregue, aceite registrado e rodadas de reforço saindo a cada 5 min; parada por primeira resposta confirmada no chamado 6781.
- Pendente: testes no aparelho (Não perturbar, iOS, toque com app fechado, empilhamento de reforços).
- Pendente: decidir `push_enabled` no ERP de produção antes do deploy do app. Depois, remover a flag (TODO #616).
- Build nativo novo do app (dois sons em `app.base.json`).
- Hipótese, não confirmada: `record_agent_view` usa `request.user.pk`, enquanto o aceite usa `agent.user_id`.

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[CloudPark ERP - Notas técnicas]]
- [[739 - Alarme de plantão em celular compartilhado]]
