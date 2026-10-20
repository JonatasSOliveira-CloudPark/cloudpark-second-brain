# Git — replicar PR mesclada em outra branch base

Abrangência: **processo** (repo `cloudpark/cloudpark-erp`, fluxo de release). Candidato a padrão; aplicado uma vez, na réplica da #766 para a release `26.10.16` (PR [#769](https://github.com/cloudpark/cloudpark-erp/pull/769), 2026-10-20). Origem: [[739 - Alarme de plantão em celular compartilhado]].

## Quando usar
- Uma PR já foi mesclada no `master` e a mesma alteração precisa ir para uma branch de release (ex.: `26.10.16`), sem reescrever nada.

## Passos
1. Achar a base de merge da PR: `git merge-base <master antes do merge> <head da PR>`. Use o `master` de antes do merge, não o atual: o GitHub calcula o diff da PR por essa base.
2. Listar os commits da PR, sem merges: `git rev-list --reverse --no-merges <base>..<head>`.
3. Criar a branch a partir da release atualizada: `git switch -c <nova> origin/<release>`.
4. Aplicar os commits em ordem: `git cherry-pick $(lista)`.
5. Verificar que o conteúdo é o mesmo:
   - lista de arquivos de `git diff <base> <head>` igual à de `git diff origin/<release> HEAD`;
   - linhas `+` e `-` iguais nas duas (sem as linhas de cabeçalho e índice, que mudam por causa do blob base diferente).
6. Se o projeto tem migrations, conferir a cadeia na branch nova: cada migration nova deve depender de uma migration que existe na release, com uma folha só por app.

## Armadilhas
- O `master` local pode estar atrasado; a base tem que vir do `origin`.
- `git fetch origin <branch>` pode falhar com "cannot lock ref" quando outra operação mexe na mesma ref; rodar de novo resolve.
- Cherry-pick sem conflito não prova que o diff é o mesmo: a etapa 5 é o que prova.

## Relacionados
- [[Git - branch mãe e atividade com dependência de outra]]
- [[CloudPark ERP - Notas técnicas]] (convenção de PR de release: sem reviewer)
