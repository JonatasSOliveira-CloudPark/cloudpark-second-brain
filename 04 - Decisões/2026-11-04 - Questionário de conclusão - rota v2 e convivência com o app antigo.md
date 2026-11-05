# Questionário de conclusão — rota v2 e convivência com o app antigo

**Abrangência:** projeto (ParkPro e CloudPark ERP). **Status:** confirmado. **Data:** 2026-11-04. **Atividade:** parkpro#166 e erp#728 (PRs parkpro#171 e erp#850, mescladas).

## Contexto
O app instalado chama `POST /api/v1/parkpro/field/visits/<id>/conclude/` e manda só a descrição. A conclusão com questionário exige respostas obrigatórias e a versão atual, então a mesma rota não pode mudar de comportamento sem quebrar o app antigo.

## Decisão
- A v1 (`.../conclude/`) continua só com a descrição, mesmo em visita com questionário. Não grava respostas.
- A v2 (`.../conclude/v2/`) aplica a regra completa: respostas obrigatórias, `answers` não nulo e `questionnaire_version_id` igual à versão atual.
- O app novo chama a v2.

## Justificativa
- Não quebrar quem já tem o app instalado.
- Evitar que o app antigo dependa de uma regra que ele não envia.

## Alternativas descartadas
- Versão mínima do app com bloqueio no ERP: exigiria publicar o app nas lojas antes de ativar a regra, e não resolvia o caso de quem não atualiza.

## Consequências
- Enquanto o app antigo existir, a v1 aceita conclusão de visita com questionário sem respostas. Isso é um risco conhecido e está declarado no contrato (`docs/fieldcontrol/questionario-contrato-app.md`, seção 3.1).
- Quando o app novo estiver em todos os técnicos, a v1 deve ser desativada.

Relacionado: [[ParkPro - Notas técnicas]], [[CloudPark ERP - Notas técnicas]], [[2026-11-04 - Questionário indisponível sem fallback offline (#166)]]
