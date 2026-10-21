# Git — replicar PR mesclada em outra branch base

Abrangência: **processo** (fluxo de release, repos `cloudpark/cloudpark-erp` e `cloudpark/cloudpark-parkpro`). Candidato a padrão; aplicado na réplica da #766 para a release `26.10.16` (PR [#769](https://github.com/cloudpark/cloudpark-erp/pull/769), 2026-10-20) e, no parkpro, na promoção para `release_candidate` das branches `cloudpark/cloudpark-parkpro/150-rc` ([#159](https://github.com/cloudpark/cloudpark-parkpro/pull/159)) e `cloudpark/cloudpark-erp/686-rc` ([#158](https://github.com/cloudpark/cloudpark-parkpro/pull/158), 2026-10-21). Origem: [[739 - Alarme de plantão em celular compartilhado]].

## Quando usar
- Uma PR já foi mesclada no `master` e a mesma alteração precisa ir para uma branch de release (ex.: `26.10.16`), sem reescrever nada.

## Passos
1. Achar a base de merge da PR: `git merge-base <master antes do merge> <head da PR>`. Use o `master` de antes do merge, não o atual: o GitHub calcula o diff da PR por essa base.
2. Listar os commits da PR, sem merges: `git rev-list --reverse --no-merges <base>..<head>`.
3. Criar a branch a partir da release atualizada: `git switch -c <nova> origin/<release>`.
4. Aplicar os commits em ordem: `git rev-list --reverse --topo-order <base>..<head> | xargs git cherry-pick`.
5. Verificar que o conteúdo é o mesmo:
   - lista de arquivos de `git diff <base> <head>` igual à de `git diff origin/<release> HEAD`;
   - linhas `+` e `-` iguais nas duas (sem as linhas de cabeçalho e índice, que mudam por causa do blob base diferente).
6. Se o projeto tem migrations, conferir a cadeia na branch nova: cada migration nova deve depender de uma migration que existe na release, com uma folha só por app.

## Alternativa: uma réplica só (PR grande, com commits que se desfazem)
- Usada na #768 → #773 (14 commits, incluindo renumeração de migrations e um doc criado e removido). Os cherry-picks teriam conflitos repetidos.
- Base e head saem do merge commit: `git rev-parse <merge>^1` (base) e `<merge>^2` (head).
- Gerar o diff com `--binary`: `git diff --binary <base> <head> > x.patch`. Na branch nova: `git apply -3 --index x.patch`. Aplicou sem conflito, e o resultado é o mesmo que a etapa 5 confere.
- Se o diff incluir migrations, conferir a folha por app (passo 6) antes do commit.
- Commit único, com a mesma mensagem da PR original.

## Armadilhas
- O `master` local pode estar atrasado; a base tem que vir do `origin`.
- `git fetch origin <branch>` pode falhar com "cannot lock ref" quando outra operação mexe na mesma ref; rodar de novo resolve.
- Cherry-pick sem conflito não prova que o diff é o mesmo: a etapa 5 é o que prova.
- No zsh, `git cherry-pick $(lista)` não separa os hashes (vira um argumento só, com quebras de linha). Usar `xargs`, como no passo 4.
- Cherry-pick com a lista na ordem errada aplica o commit mais novo primeiro e gera conflito. Ordem cronológica: `--reverse --topo-order`.

## Relacionados
- [[Git - branch mãe e atividade com dependência de outra]]
- [[CloudPark ERP - Notas técnicas]] (convenção de PR de release: sem reviewer)
