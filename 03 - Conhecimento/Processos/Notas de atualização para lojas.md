# Notas de atualização para lojas

Status: padrão usado na 2.2.0 do ParkPro (2026-10-23). Recomendação de processo, não regra formal.

## Formato
- Só o que é novo para o usuário, em três blocos no máximo: novidades, melhorias e correções visíveis.
- Frases curtas em pt-BR, sem nomes de classe, issue, endpoint ou detalhe técnico.
- Não incluir correção invisível ao usuário (log, teste, refatoração).
- O dev confirma o texto final antes de colar na loja.

## Como levantar o conteúdo
- Diff entre a branch de release e a `stable`: `git log --no-merges --format=%s origin/stable..origin/release_candidate`.
- Filtrar `feat` e `fix` e descartar `test`, `refactor` e `chore`.
- Agrupar por área (ex.: chat, plantão, acesso técnico) e reescrever cada item do ponto de vista do usuário.
- Cruzar com as issues para não perder entrega que não aparece no título do commit.

## Relacionados
- [[ParkPro - Notas técnicas]]
