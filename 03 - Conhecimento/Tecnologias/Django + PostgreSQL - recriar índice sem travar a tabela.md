# Django + PostgreSQL — recriar índice sem travar a tabela

Abrangência: tecnologia (Django 4.x + PostgreSQL). Aplicado no CloudPark ERP em 2026-10-14 (#732, migration `support/0104`), seguindo o padrão já usado na `support/0076`.

## Quando usar
Mudar um índice (ex.: condição de índice parcial) em tabela grande que recebe escrita constante. O `RemoveIndex` + `AddIndex` gerados pelo Django travam a tabela durante a criação.

## Como
- Migration separada com `atomic = False` (CONCURRENTLY não roda em transação); deixe as alterações de campo em outra migration, atômica.
- `SeparateDatabaseAndState`:
  - `database_operations`: `RunSQL("CREATE INDEX CONCURRENTLY IF NOT EXISTS <novo> ...")` **antes** de `RunSQL("DROP INDEX CONCURRENTLY IF EXISTS <antigo>")`, cada um com o `reverse_sql` inverso.
  - `state_operations`: `RemoveIndex(<antigo>)` + `AddIndex(<novo>)`.
- Dê **outro nome** ao índice novo, para criá-lo antes de apagar o antigo e a consulta nunca ficar sem índice. Atualize o `name` no `Meta.indexes` do model.
- Confira com `makemigrations --dry-run` que não sobra diferença.
