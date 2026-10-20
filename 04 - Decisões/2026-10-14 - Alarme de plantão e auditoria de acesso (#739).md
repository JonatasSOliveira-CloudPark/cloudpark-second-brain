# 2026-10-14 — Alarme de plantão e auditoria de acesso (#739)

Abrangência: **projeto** (CloudPark ERP + app ParkPro). Decisões tomadas pelo usuário durante o planejamento. Contexto: [[739 - Alarme de plantão em celular compartilhado]].

## Organização
- **Uma PR por atividade por repositório** (uma no ERP, uma no ParkPro), mesmo com correção urgente. O usuário recusou uma PR antecipada só com o fix do 400. Consequência: o fix só chega a produção com a PR inteira.

## Alarme de plantão
- **Registrar a entrega do push** em cada `OnCallNotificationLog` (enviado / sem dispositivo / falha), só para o alarme de plantão. Justificativa: a falha era invisível e o histórico dizia "notificado".
- **Alerta no canal Webex já existente** (`TeamMessageService.send_message`, canal padrão), não um canal novo; **uma vez por chamado** (trava `cache.add` de 24h). Originalmente só saía quando **nenhum** destinatário da rodada recebia. *Alterado em 2026-10-19:* também sai quando um plantonista escalado não recebe, mesmo com o líder alcançado.
- **Líder acionado na mesma rodada** quando não há plantonista no horário (qualquer dia, não só feriado) **ou** nenhum plantonista recebeu. Segue o padrão do feriado (PR #696): o líder repete a cada rodada até alguém aceitar.
- **Endpoint público de disparo manual removido** (era ferramenta de teste de maio, sem consumidores).

## Auditoria de acesso
- **Tabela única** `AppAccessAuditEvent` com campo de modo de login: **Operador, Dashboard, Técnico e Plantão** (o ParkPro é um app só com modos de login; operador = login com pátio, dashboard = Cognito). Tela no Django admin, só leitura.
- **Retenção de 30 dias** (*alterado pelo usuário em 2026-10-17; antes era 90*), cron diário em produção e QA (mesmo padrão da limpeza de deduplicação de eventos).
- **Login com falha:** só grava se o usuário existir naquele modo **e** no máximo 1 por usuário+IP por minuto. Justificativa: não lotar a tabela com tentativas de e-mails inexistentes ou repetidas.
- **Dispositivo:** grava só mudanças (novo, reatribuído com o usuário anterior, removido com motivo); re-registro do mesmo dono não gera linha. Refresh de token não é auditado.
- Auditoria nunca derruba login/registro (try/except + `log`).

## Logout
- **Endpoint por modo de login**, desvinculando **só o aparelho que pediu** (usuário pode estar logado em vários aparelhos); 204 idempotente.
- `DELETE /devices/<id>/` antigos mantidos com `TODO(cloudpark-erp#739)` para remover após atualização do app.
- **Revogar JWT no logout: adiado** (pendência; exige lista de revogação consultada em toda requisição).

## Decisões de 2026-10-17 a 2026-10-19 (revisão antes da PR)
- **Colunas de entrega anuláveis, em migração nova (`0110`):** a `0109` não é editada, porque pode já estar aplicada em algum banco. Motivo: colunas obrigatórias sem default no banco quebram o código antigo durante o deploy ([[Django - coluna NOT NULL nova quebra o código antigo no deploy]]).
- **Não reenviar ao líder na escalada** se ele já foi acionado no chamado (decisão do usuário).
- **Alerta com plantonista escalado não recebido** (ver alteração acima) e **texto próprio** para esse caso; o líder escalado também conta.
- **Cache do alerta protegido:** se o cache falhar, o alerta ainda sai e o erro vai ao log (para não perder alarme).
- **Log de notificações restrito à permissão de plantão** (a mesma das outras telas de plantão).
- **Auditoria da troca de aparelho** também no técnico e no plantão, como já era no operador.
- **Consumo da chave de 24h pelo alerta parcial:** aceito e registrado na PR, em vez de mudar o código.
- **Marcador de revisão na PR:** o commit revisado é o `9f0ab9f1`; o último commit (só testes) fica registrado na descrição.
- **Canal do alerta de alarme não entregue (2026-10-20, pedido do revisor e aceito):** vai para a sala do time responsável (`ticket.responsible_team.webex_room_id`). Sem time ou sala, cai no canal padrão. Motivo: o canal padrão também recebe os erros do `log()`, e o gestor poderia não ver o alerta.
- **Escalada ao líder já acionado (2026-10-20, pedido do revisor e aceito):** a escalada expira as pendências e grava o histórico, mesmo que o líder já tenha sido acionado em paralelo, mas não cria novo log nem novo push para ele. Correção da guarda, que tinha ficado ampla demais.

## App ParkPro (aprovadas pelo usuário em 2026-10-14, salvo indicação)
- **Branch e PR:** `cloudpark/cloudpark-erp/739` a partir do `stable`. A PR única do app é `cloudpark/cloudpark-erp/739-test → test`, seguindo o fluxo do `CLAUDE.md` do repo.
- **Logout:**
  - Cada repository de auth (operador, técnico, plantão) ganha `logout()`, com `suppressGlobalError` e timeout curto.
  - Um caso de uso por persona chama o servidor **antes** de `store.logout()`.
  - Os hooks de persistência passam a limpar só o local. O `DELETE /devices/<id>/` sai do app.
  - Justificativa: um efeito reativo a `tokens = null` não consegue fazer chamada autenticada.
- **Quais caminhos chamam o servidor:** os três botões "Sair da conta" e os logouts forçados com sessão ainda válida (operador sem pátio em `customer/Home` e o `logout()` do `Login`). Expiração no interceptor, "Entrar novamente" do diálogo global, refresh inválido e bootstrap sem sessão limpam **só o local**. Justificativa: o token já não vale, e o próximo login reatribui o aparelho.
- **Falha no logout:** limpa o local mesmo assim e mostra um toast não bloqueante ("não foi possível desvincular este aparelho; ele será liberado no próximo login"). O botão tem loading e proteção contra toque duplo. O plantão ganha confirmação antes de sair.
- **Dados enviados no logout:** o último token registrado (`LAST_FCM_KEY`) e o `device_uuid`, sem `getToken()` ao vivo. Se não houver nenhum dos dois, o app não chama o servidor.
- **`device_uuid` estável por instalação:** não é apagado no logout, e as três chaves continuam separadas por modo.
  - Justificativa: o ERP trata o campo como "UUID de instalação", único por tabela; o `remove_device_uuid_conflict` limpa tokens antigos do mesmo aparelho; o logout acha o aparelho mesmo se o token mudou.
  - Na abertura de chamado, o ERP busca o aparelho filtrando pelo usuário logado, então não há mistura entre quem divide o celular.
- **Falha de registro:**
  - Plantão: status `pending | registered | failed`, até 3 tentativas automáticas (5s, 15s, 30s), nova tentativa ao voltar ao primeiro plano, banner vermelho com "Tentar novamente" e "Abrir configurações" quando a permissão foi negada. Não bloqueia a Home.
  - Técnico: as mesmas tentativas, mais um toast na falha final.
- **Indicador "alarme ativo neste aparelho":** confirmado pelo 201 do registro e por `GET /support/devices/` ao abrir a Home e ao voltar ao primeiro plano, comparando o token ou o UUID local. Um card "Status do alarme" junta o registro no servidor e o teste de volume.
- **Versão mínima:** no app, só testes. A mudança fica no ERP: `PARKPRO_MIN_APP_VERSION` (env, vazio = desligado) e header `x-min-app-version`, ativado só depois da publicação nas duas lojas. Versões ≤ 2.0.0 ficam para a operação (atualizar o celular compartilhado na mão).
- **2026-10-15:** classe comum `DeviceInstallationStorage` (UUID e último token), usada pelos três services de dispositivo por composição. O `deviceId` numérico (stores, `loadDeviceId`, `*_device_id`) será removido junto com o `DELETE`, e a chave órfã fica no SecureStore porque é inofensiva.

## App ParkPro — decisões de 2026-10-19 (revisão antes da PR #155)
- **Operador sem alerta de falha de registro:** decisão do PO. Só registra no log. Os alertas e as novas tentativas ficam nas personas de plantão e técnico.
- **Erro inesperado no registro é retentado** (5, 15 e 30 s), como a falha de token. Antes, a primeira versão do motivo "inesperado" parava na primeira falha, e a revisão pegou isso como bloqueante.
- **Logout aborta só o POST de registro em andamento** (`AbortSignal` até o axios). Permissão de notificação (até 60 s) e token FCM (15 s) não são abortáveis. O risco de o aparelho ficar vinculado ao atendente anterior até o próximo login está declarado na PR.
- **`Ticket.create` continua com o token do operador:** é fluxo de cliente, sem alarme de plantão.
- **Mãe `cloudpark/cloudpark-erp/739` não recebe commits do #739 enquanto depender do #150:** aplicar sobre a mãe conflita em `232e531`, e adaptar criaria uma terceira versão do código. Decisão tomada para não trazer escopo de outra atividade. Detalhes no doc temporário da mãe (`docs/739-mae-dependencia-150.md`).

## App ParkPro — decisões de 2026-10-20 (PR #155 aprovada)
- **Drenagem do logout espera só o POST de registro em andamento.** Motivo: o rkoball apontou que a espera nas etapas de permissão e de token consumia o orçamento de 5 s sem proteger nada. A época continua impedindo qualquer POST depois do stop.
- **Deduplicação do registro do operador só reaproveita a tentativa com o mesmo `signal`.** Motivo: o abort do logout não alcançava um POST iniciado por outro chamador (o `Ticket.create`). Pode haver dois POSTs, que são idempotentes (o ERP reatribui o token).
- **Correções só na `cloudpark/cloudpark-erp/739-test`**, não na `test` (base protegida da PR) nem na mãe `739` (depende do #150).
