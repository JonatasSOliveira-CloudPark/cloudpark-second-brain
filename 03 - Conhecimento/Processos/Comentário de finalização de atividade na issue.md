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

Relacionado: [[Git - replicar PR mesclada em outra branch base]]
