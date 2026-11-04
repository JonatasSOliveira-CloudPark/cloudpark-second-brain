# Comentário de finalização de atividade na issue

**Abrangência:** processo (repos `cloudpark/cloudpark-erp` e `cloudpark/cloudpark-parkpro`). **Status:** em uso. Origem na prática do time, nas issues #116, #117, #131, #588, #653 e #774.

## Quando publicar
- Antes do merge, pode haver um comentário "Aguardando revisão".
- Com a PR de `test` pronta, o checklist completo.
- Quando a PR de RC for mesclada, um comentário curto com o link dela.

## Estrutura do checklist
1. `# Checklist de Finalização de Tarefa.`
2. `## ✅ Entrega`: bullets do ponto de vista de quem usa, não do código. Termina com a preservação do comportamento existente e com os testes automatizados.
3. `## 📎 Pull Requests`: só os PRs do escopo desta issue.
4. `## 📝 Documentações`: vídeo de validação e Guia de Usuário, com link em markdown.
5. `## 💬 Teste`: em quais branches foi mesclado e onde está disponível. Só escreva isso depois do merge real.

## Regras
- Links em markdown, com texto de âncora: `[PR #165](url)`, não a URL nua.
- No comentário sobre a RC, cite só a PR de RC.
- Não afirme que algo está mesclado sem conferir com `gh pr view <n> --json state,mergedAt`.
- A issue é fechada manualmente, depois do deploy e da aprovação do QA. Nunca por palavra-chave na PR.

## Relatório mensal de atividades entregues
**Status:** confirmado em 2026-11-03, ao montar o relatório de outubro/2026.

- **Fonte:** itens do board "Entregas CloudPark" (projeto 9 da org `cloudpark`) com status `Deploying` ou `Done` e atribuídos ao usuário. Ler o board exige o escopo `read:project` no `gh`; sem ele, `gh project list` falha.
- **Período:** o board não informa quando o status mudou. Use a data do comentário "Checklist de Finalização" como aproximação e avise no relatório que é uma aproximação. Itens sem checklist no mês não entram, mesmo que tenham mudado de status nele.
- **Conteúdo:** copie o bloco `## ✅ Entrega` do checklist. Algumas issues trazem subtítulos próprios ou usam "Checklist de Entrega Adicional"; revise esses casos à mão.

Relacionado: [[Git - replicar PR mesclada em outra branch base]]
