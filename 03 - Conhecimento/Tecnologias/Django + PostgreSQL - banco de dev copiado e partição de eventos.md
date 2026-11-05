# Django + PostgreSQL — banco de dev copiado e partição de eventos

**Abrangência:** tecnologia (Django + PostgreSQL), observado no CloudPark ERP. **Status:** confirmado (observado); causa do erro do `migrate`: pendente.

## Situação
- Ao copiar o banco de dev para uma worktree, o `migrate` falhou com `realtime_publish_attempted_at already exists`.
- Depois, testes e dev exigiram a partição de `parkpro_app_event`, que não existia no banco copiado.

## Recomendação
- Depois de copiar o banco, rodar `ensure_parkpro_event_partitions` antes de testes ou dev.
- Se o `migrate` falhar com coluna que já existe, não forçar: conferir o estado das migrations (`showmigrations`) antes de mexer no banco copiado.

## Pendente
- Causa do erro `realtime_publish_attempted_at already exists` (provável divergência entre o dump e o histórico de migrations; não investigado).

Relacionado: [[CloudPark ERP - Notas técnicas]], [[ParkPro - Notas técnicas]]
