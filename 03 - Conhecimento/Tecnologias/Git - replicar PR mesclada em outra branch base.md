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
- `git log origin/stable..origin/<test>` mostra os commits de todas as atividades que já estão na `test`, não só os da atividade. Com a `test` na frente da RC, filtrar pelo prefixo `cloudpark/<repo>/#<n>:` (ver caso 172-rc abaixo) ou pelo merge da PR, senão o cherry-pick traz trabalho alheio.
- Dependência de outra atividade já promovida à release por cherry-pick tem hash diferente. `git merge-base --is-ancestor <hash-da-test>` dá falso mesmo com o conteúdo presente. Conferir por mensagem de commit (`git log --grep`) ou por `git cherry <release> <head>` antes de concluir que falta.

## Relacionados
- [[Git - branch mãe e atividade com dependência de outra]]
- [[CloudPark ERP - Notas técnicas]] (convenção de PR de release: sem reviewer)

## Caso 172-rc (2026-11-09): filtrar a atividade quando a test está muito à frente
- `git log --no-merges --format='%h %s' origin/stable..origin/<atividade>-test` deu 57 commits. Só 11 eram da atividade, com prefixo `cloudpark/cloudpark-parkpro/#172:`. Esses 11 foram aplicados em ordem cronológica numa branch nova a partir de `origin/release_candidate`, sem conflito.
- A branch da RC nasceu com `git worktree add -b <rc> <caminho> origin/release_candidate`. Para entrar nela no harness, usar `EnterWorktree` com o caminho, porque `git -C` e `git --git-dir` podem ser recusados em worktree isolado.
- Testes: `node_modules` é um link para o worktree da atividade. Não precisou reinstalar. Remover o link depois de rodar o Jest.
- Confirmar o resultado com `gh pr view <n> --json baseRefName,reviewRequests,assignees`: base `release_candidate`, sem revisor, assignee o dev.
- PR #177, no repo parkpro. Ver [[ParkPro - Notas técnicas]].

## Caso 732-rc (2026-10-23): PR já na test, RC contida na test
- Antes de montar a branch, `git cherry origin/test origin/release_candidate | grep '^+'` vazio = a RC não tem commit fora da `test`. Aí o head da PR (já mesclada na `test`) pode entrar na RC por merge sem perder nada da RC.
- Merge do head com `-X theirs` resolve o conteúdo, mas deixa conflitos modify/delete (arquivos que a PR moveu). Remover com `git rm` ou apagar o arquivo: `git rm --cached` só tira do índice e o arquivo continua no disco como não rastreado, quebrando o typecheck.
- Critério de conclusão: `git diff --name-status <head> -- .` vazio. Exceção aceita: um arquivo de teste que a RC tinha a mais (testes da #686), mantido de propósito.
- O merge traz para a RC itens de outras atividades que entraram na `test` pelo merge do head (ex.: token de tracking no `eas.json`). Listar os arquivos antes de abrir a PR e sinalizar no corpo.
- Typecheck: comparar a contagem de erros com a que a PR registrou como preexistente na `test`, não só "passou".
- Referência: [[732 - Falar com Suporte (técnico)]].

## Caso 723-rc (2026-10-30): base avançou, comparar pelo merge-base
- Com o `stable` (ou a `test`) à frente da base da atividade, `git diff --name-only origin/<base> origin/<atividade>` lista arquivos de outras tasks. Não é o que a atividade mudou.
- Para pegar só a atividade: `git merge-base origin/<base> origin/<atividade>` e, a partir dele, `git rev-list --reverse --no-merges <merge-base>..origin/<atividade>` (commits) e `git diff --name-only <merge-base> origin/<atividade>` (arquivos).
- Caso 723: 4 commits e 6 arquivos. Cherry-pick sobre `origin/release_candidate` sem conflito, com um merge automático em `Actions.tsx`. Conferido com `git diff --stat origin/release_candidate HEAD` (6 arquivos).

## Caso 179 (2026-11-13): atividade com base na test
- A branch da atividade partia da `test`, não da `stable`. `git rev-list stable..atividade` trouxe cerca de 190 commits de outras tasks. Para achar a atividade, filtrar pelo prefixo: `git log --no-merges --reverse --grep='#179:' --format=%H origin/stable..<atividade>` (11 commits).
- O commit de merge de `test` dentro da branch (`chore(merge)`) não foi aplicado: traz o trabalho de outras tasks.
- `package.json` e `pnpm-lock.yaml` conflitam em cada base (a RC e a stable têm outras versões). Resolver o `package.json` à mão, sem o pacote removido, e regenerar o lock com `pnpm install --lockfile-only --ignore-scripts`. Depois, seguir os cherry-picks.
- Conferência: `git diff --stat origin/<base> HEAD` com os arquivos da atividade e `pnpm exec jest` nas áreas alteradas.
