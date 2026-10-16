# Idempotência de POST com ID do cliente — efeitos colaterais

Aprendizado da PR cloudpark/cloudpark-erp#726 (chat do ticket, 2026-10-16), apontado na revisão do rkoball. Vale para qualquer endpoint que deduplica pelo ID gerado no cliente (ex.: `client_message_id`) e responde 200 com o registro existente no reenvio.

**Status:** recomendação (aplicada na #726; ainda não é regra da equipe).

## O problema
Ao tornar o POST idempotente, o reenvio passa a devolver o registro já gravado **sem refazer nada**. Se a primeira requisição gravou e depois quebrou (exceção, corte de 120 s do Gunicorn), os efeitos que vinham após o commit nunca acontecem. Antes da idempotência, o reenvio duplicava o registro, mas pelo menos refazia tudo. A correção pode trocar duplicação por perda silenciosa.

## Recomendações
- **Separe os efeitos em dois grupos:**
  - **só banco** (SLA, prazos, atribuição): grave na mesma transação do registro (ex.: callback `after_create` dentro do `atomic`). Se falharem, o registro some junto e o reenvio refaz tudo.
  - **externos** (MQTT, push, notificação): ficam depois do commit. Refaça no reenvio só os que têm marcador confiável de "já tentado" (ex.: `realtime_publish_attempted_at`) e cujo receptor deduplica.
- **Efeito sem marcador:** não refaça às cegas (notificaria em dobro). Registre como débito na PR ou crie o marcador.
- **Teste o cenário real:** falha depois do commit seguida do reenvio com o mesmo ID. O teste falha sem a correção.

## Relacionados
- [[App mobile - fila de envio com reenvio automático]]
- [[CloudPark ERP - Notas técnicas]]
