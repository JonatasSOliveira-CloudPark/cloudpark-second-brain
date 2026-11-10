# CloudPark ERP — Notas técnicas

Conhecimento do projeto `cloudpark/cloudpark-erp` confirmado em sessões de trabalho. Regras gerais do repo ficam no `CLAUDE.md` dele; aqui só o que não está lá.

## Convenções do repo (confirmadas pelo usuário)
- Branch `cloudpark/cloudpark-erp/<nº issue>`; commit `<branch>: <tipo>(escopo): <descrição>` em português (ex.: `cloudpark/cloudpark-erp/#732: feat(support): ...`).
- Sem linha de atribuição de IA em commits/PRs; autor e committer com o e-mail corporativo.
- PR para `master`: assignee = autor, reviewer = rkoball; ao abrir, comentar na issue o "Checklist de Finalização de Tarefa".
- PR de release (ex.: base `26.10.16`, com alterações já mescladas no `master`): só assignee. Sem reviewer e sem comentário do checklist na issue (regra do usuário, 2026-10-20, PR #769; já vinha da PR #744 em 2026-10-14).
- Comparar sempre com `origin/master`: o `master` local pode estar atrasado e mostrar centenas de arquivos que não são da branch (2026-10-19).
- Marcador de revisão do `/revisar` aponta para o commit revisado. A label `revisado` só entra depois do `/revisar` das correções finais. Commit posterior só de teste deve ser citado na descrição da PR. Se o checklist já existe (por exemplo, depois de ajustes do review), editar o comentário existente e acrescentar só o que mudou para o produto. Não postar um segundo checklist, para não confundir o QA (escolha do usuário em 2026-10-15, #686).
- Atualizar PR já revisada com o master: o usuário escolheu **rebase + force push** (`--force-with-lease`), não merge, mesmo avisado de que a revisão perde o diff incremental (#726, 2026-10-16).
- Código sem comentários óbvios; planejar respeitando a arquitetura existente e levantar dúvidas em perguntas numeradas.

## Suporte (`support`)
- `Ticket.Kind`: CLIENT=1, INTERNAL=2, TECHNICIAN=3 (desde a #732). Vários fluxos ramificam por `kind`; ao criar um tipo novo, revisar `support/views.py` (envio de mensagem do ERP), `OnCallDomain._send_message`, `TicketClosureDomain._notify_closed` e o filtro do ERP.
- Ticket sem `request_type` quebrava pontos que faziam `ticket.title or ticket.request_type.title`; usar `Ticket.display_title`.
- `Ticket.search_cache` é preenchido por trigger **só no INSERT** (`fn_create_support_ticket_cache`, última versão na migration `0105`); campos separados por `|` e lidos por posição em `TicketTextSearchSerializer` e no seletor de ticket do fieldcontrol. Mudar a função exige migration + backfill.
- Chat: estado da conversa em `TicketConversationStateDomain` (`last_chat_message_sender`, `pending_customer_messages`); efeitos do envio do solicitante em `TicketRequesterMessageDomain.on_message_sent`.
- Envio de mensagem do chat é idempotente por `client_message_id` + `sender_type` no ticket (`TicketMessageDomain.save_once`, sob `select_for_update` do ticket; desde a #726). Ajustes de SLA/NRT e atribuição do agente rodam no `after_create`, na mesma transação. No reenvio, `TicketRealtimeDomain.republish_message_if_not_attempted` refaz o publish (e o push do agente) se `realtime_publish_attempted_at` for nulo; abas e notificação ao suporte não são refeitas (débito aceito).
- `fieldcontrol.domain` importa `support.domain` → regra que precisa dos dois fica no `fieldcontrol` (o inverso dá import circular).
- **Visibilidade de ticket** (confirmado no código em 2026-10-26): `TicketAccessScopeDomain` (interno) e `ExternalSupportScopeDomain` (time externo, `Team.is_external`) filtram lista, detalhe, mensagens e histórico por `responsible_team_id` **OR** `created_by_team_id`, restritos a `agent.teams`. `is_master` vê tudo. `internal_requester` não dá acesso. Não há critério por `created_by_agent`.
- `created_by_team_id` é um **snapshot** do `main_team` do criador na criação (`get_ticket_creator_data`, [support/serializers.py](support/serializers.py)). Se o `main_team` não estava definido, fica nulo sem erro. Transferir o ticket não altera esse campo, e mudar o `main_team` depois não corrige tickets antigos.
- Caso PRD 2026-10-26: agente externo (ParkDesk) criou ticket interno para o suporte; depois da transferência ele deixou de ver. O `main_team` dele estava com outro time na criação. Corrigido com ajuste manual de `created_by_team_id` no banco, sem migração nem correção de código.
- Pendências não tratadas: (1) a visibilidade do criador não usa `created_by_agent`; (2) ao transferir, o time de origem perde a visão (`TicketAccessScopeDomain`), o que pode ser intencional ou não, a confirmar com o produto; (3) `TicketAdmin` não permite edição (`has_change_permission` falso), então o campo só aparece em leitura; (4) não há teste cobrindo criação por agente externo e transferência.

## Plantão (`OnCallDomain`, desde a #739)
- `OnCallNotificationLog` guarda a entrega do push (`delivery_status`: enviado / sem dispositivo / falha, contagens e erro do FCM). O histórico do chamado usa esse resultado.
- Sem plantonista no horário ou nenhum plantonista alcançado → líder acionado na mesma rodada. Ninguém recebeu → alerta Webex único por chamado (`cache.add` `oncall:undelivered-alert:<id>`).
- `get_active_attendants_for_datetime` exige `OnCallDaySchedule` da data; só horário semanal dá lista vazia.
- Logs de notificação consultáveis no admin (só leitura).
- A janela de plantão depende do pátio por causa do feriado local: `get_effective_ranges(date, parking)`. Quem decide se é plantão deve passar o `parking`.
- Aceite da cobrança de plantão (PR #700 da #686, em revisão em 2026-10-15): a regra única é `OnCallChargeAcceptanceDomain.is_required(parking, request_type, team=None)`. Time com horário de atendimento não exige o aceite; senão, vale `OnCallDomain.is_on_call_window_now(parking)`. Os aceites ficam no histórico `OnCallChargeAcceptance`, por ciclo do chamado (`ticket_opened_at` = `creation_date`), e não mais nas colunas do `Ticket`. Ver [[686 - Aceite de plantão na reabertura de chamado]].

- **Reforço de plantão (#616, 2026-11-10):** `OnCallReinforcementEvent` com os tipos `reinforcement`, `stopped`, `agent_viewed`, `first_message` e `error`. Motivos de parada: `first_response`, `ended`, `reassigned`, `superseded` e `awaiting_customer`. Após 3 erros seguidos na mesma rodada, o reforço não é reagendado.
- Migrations 0116 a 0124 do `support`. A 0123 junta a 0122 com a 0116 do master; a 0124 adiciona o tipo `error`.
- Envio atrás de `OnCallReinforcementConfig.push_enabled` (desligada por padrão), com TODO #616 para remover depois do deploy do app.
- Suíte local: `FAKE_MQTT_INTEGRATION=true` no ambiente. O CI já usa o padrão verdadeiro. O `core/testing.py` não força mais a flag.

## Dispositivos e acesso do app ParkPro (`parkpro`, desde a #739)
- Um app com 4 **modos de login**: operador (login com pátio), dashboard (Cognito, sem pátio), técnico, plantão. Modelos de dispositivo: `Device`, `TechnicianDevice`, `SupportAgentDevice` (`AppDeviceModel`, `fcm_token` único).
- `parkpro/device_registry.py` centraliza: conflito de `device_uuid`, mesmo token em outro modo, `delete_devices()` (apaga gravando auditoria), registro auditado e `logout_device()`. Remoção de dispositivo nova deve passar por `delete_devices`.
- Auditoria `AppAccessAuditEvent` (`AppAccessAuditService.record` / `record_login`, admin, limpeza 90 dias). IP via `core.utils.get_client_ip`.
- Serializers de registro de plantão/técnico não podem validar unicidade do token: [[DRF - ModelSerializer com campo unique bloqueia upsert]].

## Testes e verificação local
- Testes com o pyenv `cloudpark` e `DOTENV_FILE` apontando para o `.env` do checkout principal (worktree não tem `.env`).
- 4 falhas pré-existentes no app `parkpro` (2026-10-14). Com a #739, as 2 de registro de dispositivo eram o bug real e passam; restam `test_cloudpark_permissions_url_and_header` e `test_service_hours_response`.
- O banco de teste (`test_cloudpark_erp`) é **compartilhado entre worktrees**: duas sessões rodando testes ao mesmo tempo colidem ("being accessed by other users"); esperar o outro processo terminar. Para esperar via `pgrep -f`, ancorar o padrão no binário (`^/home/jonatas/.pyenv/versions/cloudpark/bin/python manage.py test`), senão o padrão casa com o próprio comando de espera e o loop não termina. Trocar `DATABASE_URL` por variável de ambiente não funciona (o `.env` prevalece).
- `manage.py` (inclusive `test`) busca segredos no OpenBao ao iniciar; com o OpenBao fora do ar dá `ConnectTimeout` e nada roda (em dev só o `RuntimeError` é tolerado).
- `resolve()` nunca falha em URL inexistente: o `catch_all_view` do admin casa tudo. Para testar que uma rota saiu, usar `reverse(nome)` esperando `NoReverseMatch`.
- Nome de índice no Django tem limite de 30 caracteres (`models.E034`).
- Migration ainda não aplicada em lugar nenhum (só na branch, sem merge) pode ser corrigida no próprio arquivo em vez de gerar outra. Se ela colidir com o master depois do rebase (duas folhas no app), renumerar e apontar a dependência para a folha atual, em vez de criar uma merge migration. Conferir com `MigrationLoader(None).graph.leaf_nodes("<app>")` e `detect_conflicts()`.
- Rebase de branch longa: quando a branch moveu ou duplicou um trecho que o master alterou, o conflito aparece só em uma das cópias. A outra continua com o código antigo sem acusar nada. Exemplo da #686: a branch tinha subido o `TicketOpeningCheckBase` e o master tinha passado `parking` ao `_is_on_call_window_now`. Depois do rebase, procurar as chamadas da branch ao código que o master mudou.
- Rebase com o master que reescreveu o plantão (#739): nos conflitos de `support/domain.py`, manter a versão do master e só adaptar o título para `display_title`. Testes com `SimpleNamespace` como ticket precisam ganhar `display_title` (2026-10-21).
- Migrações: ao renumerar as da branch depois da folha do master, conferir uma folha por app com AST sobre os arquivos (`dependencies`). `manage.py` pode travar por OpenBao fora do ar; o `curl` em `/v1/sys/health` confirma.
- `makemigrations --check` acusa drift antigo (`ticketclosinghistory.id`, `cronjob*`) que não é da task; remover das migrations geradas.
- Frontend React: o ESLint do projeto não cobre `.ts/.tsx`; o `tsc --noEmit` tem ~39 erros pré-existentes — comparar a contagem antes/depois e rodar `vite build` com `--outDir` fora do repo. Worktree usa symlink de `frontend_react/node_modules` para o checkout principal.

## Admin com seleção múltipla em volume (fieldcontrol, 2026-10-21 a 23)
- `DATA_UPLOAD_MAX_NUMBER_FIELDS` (padrão 1000) limita campos do POST. Um select múltiplo do admin envia um campo por item: com mais de 1000 pátios o salvamento dá "Max number of fields exceeded". Solução usada: o script da página junta os ids em um campo só (`parkings=1,2,3`), e o servidor valida cada id contra a lista visível. Não aumentar o limite.
- Widget nativo (`FilteredSelectMultiple`) com valor único: sobrescrever `format_value` (o `Select` do Django trata string como um item só) e `has_changed` (senão o histórico registra "alterado" em todo salvamento).
- Corrigido na #780 (merge 23/10). Réplica para release: #784 `26.10.16`, sem reviewer.

## Ambiente local (worktree e rede) — confirmado em 2026-10-15
- Procedimento de worktree e de banco com dump de produção: `docs/infra/worktrees.md` (no repo do ERP).
- OpenBao/`manage.py` com timeout costuma ser o NetBird em `NeedsLogin` (login do peer expirou). `netbird up` refaz o SSO sem `down`; depois, `bao-login` se o token do OpenBao também expirou.
- Worktree sem `.env`: rodar `CCSWITCH_REPO_PATH=<checkout principal> CCSWITCH_SESSION_NAME=<nome> ./scripts/worktree-setup.sh` (copia o `.env` e cria os links). Sem `backups/prod-dumps/producao.dump`, o setup **não** cria banco e o `.env` fica apontando para o `cloudpark_erp` compartilhado — não rodar `migrate` de branch nele.
- Banco próprio sem dump (feito à mão na #732; automação pendente): `pg_dump -Fc` do `DATABASE_URL` do checkout principal → `scripts/restore-prod-dump.sh <arquivo> <nome>` (Postgres isolado na 5434, ajusta o `.env`), gravando antes o nome em `.worktree-proddump-db` (`worktree_proddump_record_db_name`) para o archive limpar depois.
- Acesso pela rede local em dev: `runserver 0.0.0.0:8000`; o template `application_react.html` fixa o Vite em `localhost:5173`, e o Vite 7 só libera CORS para localhost — exige ajuste no template e `server.host`/`server.cors` no `vite.config.js` (feito na worktree da #732, **sem commit**). O CSRF aceita o IP porque o `.env` local tem `PUBLIC_BASE_URL=http://<IP>:8000`.
- Login do ERP exige código de verificação mesmo em dev; o IP já é liberado em `STAGE=dev` (`is_authorized_erp_login_ip`). 5 tentativas erradas bloqueiam por 15 min (`FailedLogin`).
- Não usar `git checkout -B <branch>` numa worktree quando a branch já está aberta em outra: ele move a ref compartilhada e deixa a outra worktree com o índice desatualizado (arquivos aparecem como modificados). Antes, conferir com `git worktree list`; se a branch estiver aberta em outra, trabalhar numa branch local própria e fazer o push com `<local>:<branch>`. Aconteceu na #686 em 2026-10-16. Foi corrigido com `git update-ref` de volta ao commit original.

- Banco isolado da worktree com dados: `pg_dump -Fc` do `cloudpark_erp` compartilhado e `pg_restore --data-only` num banco com o schema da branch (2026-10-21). Ajustes necessários: colunas que só existem no banco base (`support_ticket.first_response_at`), NOT NULL que o banco base viola (`store_order.status`, `financial_financialconfig.legal_dossier_email`), e tabelas de outras branches (celery results, WhatsApp) que dão erro e são ignoradas. Não use `TRUNCATE ... CASCADE` para reexecutar: derruba tabelas relacionadas.
- Migrate com histórico divergente (banco base de outra linha de migrations): o erro aparece como "já existe" (coluna ou tabela). Faked migrate um por um não converge; recriar o banco vazio e migrar do zero é mais rápido.
- `unaccent` pode aparecer em `pg_extension` e não ter função: `DROP EXTENSION unaccent; CREATE EXTENSION unaccent;` no banco de teste.
- Nome do banco de teste = `test_` + nome do banco. Um banco de worktree com nome longo (`cloudpark_erp_proddump_claude_worktree_atividade_732_fdca22`) passa de 63 caracteres e o teste não sobe; rodar os testes com o `.env` principal (banco `test_cloudpark_erp`).

## Edge público, middleware e WAF (verificado em 2026-10-26)
- Instância pública: CloudFront → Caddy (`docker/caddy/Caddyfile.public`) → Django. O Caddy responde `Forbidden` em texto puro com 403 quando a requisição não traz o header `CLOUDFRONT` com o segredo da distribuição. Essa resposta não chega ao Django, então não aparece nos logs do ERP.
- `PublicRouteGuardMiddleware` (`core/middleware.py`) responde 403 em JSON com `{"detail": "Authentication credentials were not provided."}` quando `PUBLIC_ROUTE_GUARD_ENABLED` está ligado (o default é desligado) e o host ou o path não passa em `core/route_access.py`.
- `api/v1/parkpro/` e `api/v2/parkpro/` estão em `PUBLIC_EDGE_PREFIXES`. Pelo código, a rota de foto não deveria ser barrada por esse guard em host público.
- WAF (AWS): `SizeRestrictions_BODY` está em Count nas distribuições de prod e QA (informado por infra em 2026-10-26). Count só registra, não bloqueia.
- Pendente: suspeita de `CrossSiteScripting_BODY` (grupo gerenciado) em Block, bloqueando o upload multipart de foto (`/api/v2/parkpro/field/visits/{id}/photos/`). Não confirmada: depende do log do WAF com `terminatingRuleId`. Ver [[Erro HTTP com mensagem genérica - origem fora da view]].

## Relacionados
- [[732 - Falar com Suporte (técnico)]]
- [[Erro HTTP com mensagem genérica - origem fora da view]]
- [[739 - Alarme de plantão em celular compartilhado]]
- [[686 - Aceite de plantão na reabertura de chamado]]
- [[Django + PostgreSQL - recriar índice sem travar a tabela]]
- [[Idempotência de POST com ID do cliente - efeitos colaterais]]
