# Git — branch mãe e atividade com dependência de outra

Abrangência: **processo** (repos ParkPro e ERP, fluxo `stable` → mãe → `<n>-test` → `test`). Candidato a padrão; até agora aplicado no #739. Origem: [[739 - Alarme de plantão em celular compartilhado]], 2026-10-19 e 2026-10-20.

## Armadilha
- A branch da atividade é rebaseada sobre a `test` quando a mãe fica para trás. Os commits da atividade passam a conviver com commits de outras tasks que já estão na `test`.
- A mãe continua com os mesmos commits, mas com outros hashes. Não dá para fazer fast-forward nem substituição sem force push.
- Se a atividade depende de código que só existe na `test` (de outra task), os commits novos não aplicam sobre a mãe. O primeiro conflito costuma aparecer já no primeiro commit novo.
- **Rebase de branch com muitos commits e conflito repetido:** cada commit é reaplicado e pode conflitar de novo. Caso 732 (2026-10-21): 49 commits sobre a `test`, e o primeiro já deu conflito. O merge da `test` na branch resolveu tudo numa passada, com 3 conflitos. Trade-off: o merge deixa um commit de merge no histórico. A escolha é do dono da branch.

## Regra
- A mãe só recebe commits da própria atividade.
- Se a atividade depende de outra task que ainda não está na `stable`, não levar nada dessa dependência para a mãe. Esperar a outra task chegar à `stable` e reconstruir a mãe a partir dela, com só os commits da atividade.
- Adaptar o código para não depender da outra task cria uma terceira versão, diferente da que foi revisada. Só vale com aprovação e revisão nova.
- Doc temporário na mãe, para registrar a pendência, deve ser removido na reconstrução.

## Como checar sem mexer na árvore de trabalho
- Separar os commits por assunto: `git log --format='%h|%s' <mãe>..<atividade>`. Os que repetem o assunto de um commit da mãe são replays do rebase; os novos são o trabalho pendente.
- Simular o merge: `git merge-tree --write-tree --name-only <mãe> <atividade>`. Lista os conflitos sem tocar em nada.
- Saber se um commit da atividade já está na mãe com outro hash: `git diff --quiet <mãe> <commit> -- <arquivo>` por arquivo. Se todos os arquivos baterem, não há nada a levar. Caso 732 (2026-10-21): os três commits da 732 que faltavam já estavam na mãe com outros hashes.
- Ao escrever script que resolve conflitos em loop, conferir a lista de arquivos antes de agir. Uma variável de caminho não exportada gerou lista vazia: o script não resolveu nada e parou só pela checagem de marcadores.
- Testar o cherry-pick num worktree temporário (`git worktree add --detach <caminho> <mãe>`) e remover depois (`git worktree remove --force <caminho>` e `git worktree prune`).
- Descobrir se a mãe tem um bug de um commit da `test`: `git merge-base --is-ancestor <commit> <mãe>` (sai com 0 se o commit está na mãe). Confirmar também se o arquivo e a chamada que causam o bug existem na mãe (`git cat-file -e <mãe>:<arquivo>` e `git show <mãe>:<arquivo> | grep <função>`). Caso de 2026-10-20: a mãe do #739 não tinha o bug, então a correção foi só para a `test` (ver [[ParkPro - Notas técnicas]]).

## Caso 723 (2026-10-22): branch da atividade preservada e PR em branch nova
- A branch da atividade (`cloudpark/cloudpark-erp/723`) ficou como histórico e foi publicada. Para o PR, criou-se `cloudpark/cloudpark-erp/723-test` com `git switch -c <723-test> origin/test` e cherry-pick dos commits. Não houve conflito.
- Para corrigir um commit ainda não publicado: `git commit --fixup=<hash>` e `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash <base>`. Não reescrever commit que já foi publicado.
- Antes do PR, rodar typecheck e os testes da branch nova, porque a base mudou e pode ter código diferente.

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[CloudPark ERP - Notas técnicas]] (rebase de branch longa com trecho movido)
