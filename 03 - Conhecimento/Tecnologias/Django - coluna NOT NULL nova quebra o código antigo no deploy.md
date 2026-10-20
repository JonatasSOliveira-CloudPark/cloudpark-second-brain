# Django — coluna NOT NULL nova quebra o código antigo no deploy

Abrangência: **tecnologia** (Django com migrations). Confirmado em 2026-10-19 no CloudPark ERP ([[739 - Alarme de plantão em celular compartilhado]]).

## Problema
Uma migração `AddField` com campo obrigatório e `default` no Django cria a coluna `NOT NULL`, mas o `default` fica só no código e não no banco (Django 4.2 não tem `db_default`). Durante o deploy, o código antigo ainda está no ar e faz `INSERT` sem a coluna nova. O banco recusa a gravação, e a operação falha no meio, sem aviso claro.

## Como aparece
- Só durante a janela entre o `migrate` e o fim do rollout. Depois, o código novo grava a coluna e o problema some, o que dificulta a investigação.
- Afeta qualquer modelo gravado por código antigo: tarefas, cron, webhooks e views em instâncias que ainda não foram atualizadas.

## Como evitar
- Colunas novas de model gravado em produção: `null=True` no model e na migração. O `default` pode ficar no código.
- Se a coluna precisar ser obrigatória, dividir em dois deploys: primeiro anulável com backfill, depois `NOT NULL` (ver regra de migrations no CLAUDE.md do ERP).
- Editar uma migração já aplicada não muda o banco. Em vez disso, criar uma migração nova com `AlterField` que corrige a coluna.
