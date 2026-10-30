# 2026-10-28 - Flag para pular verificação de versão só em desenvolvimento

**Abrangência:** projeto (ParkPro). **Status:** decidido e implementado na branch da #774 e na PR #167.

## Contexto
Em desenvolvimento local, o `AppVersionGuard` mostrava "nova versão disponível", porque consultava a loja. Trocar o `EXPO_PUBLIC_STAGE` para `dev` contornaria o problema, mas esse valor também define o tópico MQTT, então não serve.

## Decisão
Criar `EXPO_PUBLIC_SKIP_APP_VERSION_CHECK`. Com o valor `"true"` e `__DEV__` verdadeiro, `AppVersionService.needsUpdate` retorna "sem atualização" sem consultar o adaptador.

## Justificativa
- `__DEV__` é falso em builds de release. Assim, QA e produção sempre verificam, mesmo que um `.env` com a flag seja lido durante o build.
- O critério "stage diferente de prod" foi descartado: o QA continuaria pulando a verificação se a flag estivesse presente.

## Consequências
- A variável fica no `.env` local, que é git-ignored. Não vai para o `eas.json`.
- Para aplicar, rode `pnpm start -c` no worktree certo. Ver [[ParkPro - Notas técnicas]].

Relacionado: [[ParkPro - Notas técnicas]]
