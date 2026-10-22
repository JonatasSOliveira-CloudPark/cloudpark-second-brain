# 736 — Máscara do horário da visita para o cliente

Abrangência: **projeto CloudPark ERP** com impacto no app ParkPro. Decisões em [[2026-10-22 - Máscara de período do horário da visita (#736)]].

## Entregas
- ERP: PR [#782](https://github.com/cloudpark/cloudpark-erp/pull/782), contra `master`. Revisado com /revisar no commit `3c63bcdb`; os commits de correção e o merge com o master não foram revisados de novo.
- App ParkPro: PR [#160](https://github.com/cloudpark/cloudpark-parkpro/pull/160), contra `test`. Mostra "Período: …" com `visit_period_label`.
- Checklist de finalização postado na #736 (Entrega, Pull Request, Documentações e Teste).

## Pendências
- Vídeo de validação: pendente.
- Decisão da PO sobre a linha do tempo da visita (em deslocamento, chegou no local): mostrar horário exato ou só a data/período?
- Remover `scheduled_time_label` do ERP quando os apps forem atualizados.
- Issue [#781](https://github.com/cloudpark/cloudpark-erp/issues/781): autenticação no endpoint do PDF da ordem de serviço. Independente desta atividade.
- Status no board do projeto: o `gh` não tem o escopo `read:project`, então não foi atualizado.

## Itens fora de escopo que ficaram claros
- Expediente de 08:00 às 18:00: o caso "Noite" não aparece hoje. A faixa foi mantida como tratamento defensivo.
- O PDF da OS continua com horário exato, e isso vale enquanto a PO não mudar a decisão.

Relacionadas: [[2026-10-22 - Máscara de período do horário da visita (#736)]].
