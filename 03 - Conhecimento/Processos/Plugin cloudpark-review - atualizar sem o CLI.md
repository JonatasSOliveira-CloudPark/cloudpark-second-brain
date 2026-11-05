# Plugin cloudpark-review — atualizar sem o CLI

**Abrangência:** processo (ferramenta do time), válido nesta máquina. **Status:** confirmado nesta máquina; carregamento da versão nova em sessão nova: pendente.

## Situação
- O `claude` CLI não estava no PATH, então o `/revisar` não conseguiu se atualizar sozinho.
- A atualização para a 0.9.2 foi feita à mão.

## Procedimento
1. Fazer fast-forward do clone em `~/.claude/plugins/marketplaces/cloudpark` (branch `main`).
2. Copiar a versão nova para `~/.claude/plugins/cache/cloudpark/cloudpark-review/<versão>`.
3. Apontar as entradas `cloudpark-review@cloudpark` de `~/.claude/plugins/installed_plugins.json` para a nova versão. Fazer backup do arquivo antes.
4. Abrir uma sessão nova (ou recarregar os plugins) e conferir a versão.

## Pendente
- Confirmar que a 0.9.2 aparece numa sessão nova, sem reiniciar o procedimento.

Relacionado: [[Revisão com modelo errado - revisar com Haiku]]
