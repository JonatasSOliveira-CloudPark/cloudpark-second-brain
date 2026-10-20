# Git — branch mãe e atividade com dependência de outra

Abrangência: **processo** (repos ParkPro e ERP, fluxo `stable` → mãe → `<n>-test` → `test`). Candidato a padrão; até agora aplicado no #739. Origem: [[739 - Alarme de plantão em celular compartilhado]], 2026-10-19 e 2026-10-20.

## Armadilha
- A branch da atividade é rebaseada sobre a `test` quando a mãe fica para trás. Os commits da atividade passam a conviver com commits de outras tasks que já estão na `test`.
- A mãe continua com os mesmos commits, mas com outros hashes. Não dá para fazer fast-forward nem substituição sem force push.
- Se a atividade depende de código que só existe na `test` (de outra task), os commits novos não aplicam sobre a mãe. O primeiro conflito costuma aparecer já no primeiro commit novo.

## Regra
- A mãe só recebe commits da própria atividade.
- Se a atividade depende de outra task que ainda não está na `stable`, não levar nada dessa dependência para a mãe. Esperar a outra task chegar à `stable` e reconstruir a mãe a partir dela, com só os commits da atividade.
- Adaptar o código para não depender da outra task cria uma terceira versão, diferente da que foi revisada. Só vale com aprovação e revisão nova.
- Doc temporário na mãe, para registrar a pendência, deve ser removido na reconstrução.

## Como checar sem mexer na árvore de trabalho
- Separar os commits por assunto: `git log --format='%h|%s' <mãe>..<atividade>`. Os que repetem o assunto de um commit da mãe são replays do rebase; os novos são o trabalho pendente.
- Simular o merge: `git merge-tree --write-tree --name-only <mãe> <atividade>`. Lista os conflitos sem tocar em nada.
- Testar o cherry-pick num worktree temporário (`git worktree add --detach <caminho> <mãe>`) e remover depois (`git worktree remove --force <caminho>` e `git worktree prune`).
- Descobrir se a mãe tem um bug de um commit da `test`: `git merge-base --is-ancestor <commit> <mãe>` (sai com 0 se o commit está na mãe). Confirmar também se o arquivo e a chamada que causam o bug existem na mãe (`git cat-file -e <mãe>:<arquivo>` e `git show <mãe>:<arquivo> | grep <função>`). Caso de 2026-10-20: a mãe do #739 não tinha o bug, então a correção foi só para a `test` (ver [[ParkPro - Notas técnicas]]).

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[CloudPark ERP - Notas técnicas]] (rebase de branch longa com trecho movido)
