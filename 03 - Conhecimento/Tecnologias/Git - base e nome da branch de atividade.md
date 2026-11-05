# Git — base e nome da branch de atividade

Abrangência: **processo** (repos `cloudpark/cloudpark-erp` e `cloudpark/cloudpark-parkpro`). Status: **confirmado** no ParkPro (atividade #736, 2026-10-22); a regra de base e nome vem da prática do time.

## Nome e base
- A atividade parte da `stable`.
- O nome segue `cloudpark/<repo-de-origem-da-issue>/<n>-<slug>`. O slug é um resumo curto em kebab-case, minúsculas e sem acentos. O repo é o da **issue**, não o repo onde o código muda. Ex.: issue #736 nasce no ERP, então a branch do app é `cloudpark/cloudpark-erp/736-mascarar-horario-visita`. Atualizado em 2026-10-28 (antes o padrão era só `<n>`; as branches antigas continuam válidas).
- As branches derivadas são `<branch>-test` e `<branch>-rc`.
- Exceção histórica: a atividade erp#723 (2026-10-22) tem a branch `cloudpark/cloudpark-erp/723`, sem slug, criada antes da regra de 2026-10-28. As derivadas `723-test` e `723-rc` seguiram o mesmo nome base, e a skill `promover-rc` aceitou esse nome.
- O prefixo dos commits sai do nome, com `#` antes do número e sem o slug: `cloudpark/cloudpark-erp/#736: feat(...)`.

## Branch que nasce de HEAD errado (worktree)
- A branch criada pelo ambiente de worktree pode partir de um HEAD que não é a `stable`. Caso de 2026-10-22: o HEAD estava 110 commits à frente da `stable` e trazia merges de outras atividades.
- Antes de criar a branch da atividade, conferir:
  - `git merge-base --is-ancestor origin/stable HEAD`
  - `git rev-list --count origin/stable..HEAD`
- Se a base estiver errada, recriar a partir da `stable` sem perder as alterações:
  1. `git diff > <arquivo>.patch`
  2. `git checkout -- .` (só depois de salvar o patch)
  3. `git switch -C <nome> origin/stable`
  4. `git apply <arquivo>.patch`
- `git switch` recusa o switch com alterações locais. Por isso o patch vem antes.

## PR de atividade para `test`
- A PR da atividade vai para `test` pela branch `<branch>-test`, criada a partir de `origin/test` (não da `stable`).
- Os commits da branch mãe entram na `-test` por cherry-pick, na ordem cronológica. A branch mãe continua no remoto.
- Caso parkpro#166 (2026-11-04): a PR foi aberta da branch mãe, que partia da `stable`. Foi fechada e refeita pela `-test`, com os mesmos 16 commits, sem conflito. O corpo da PR precisa apontar para o HEAD da `-test`.

## Relacionado
- [[Git - branch mãe e atividade com dependência de outra]]
- [[Git - replicar PR mesclada em outra branch base]]
