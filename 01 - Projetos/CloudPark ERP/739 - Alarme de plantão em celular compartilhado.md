# #739 — Alarme de plantão não tocou (celular compartilhado)

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/739
- Repo/branch ERP: `cloudpark/cloudpark-erp`, branch `cloudpark/cloudpark-erp/739` (enviada; **sem PR** em 2026-10-14)
- Decisões e justificativas: [[2026-10-14 - Alarme de plantão e auditoria de acesso (#739)]]
- Notas técnicas: [[CloudPark ERP - Notas técnicas]], [[ParkPro - Notas técnicas]]

## Problema
4 atendentes de plantão revezam o mesmo celular no modo plantão do ParkPro. No fim de semana e no feriado de 12/10/2026 o alarme não tocou para o plantonista escalado. Em "Dispositivos de Atendente" só havia o registro de outro atendente, criado em 30/09 e nunca atualizado.

## Causa raiz (confirmada no código)
1. **Registro recusado com 400:** `SupportAgentDeviceSerializer` e `FieldTechnicianDeviceSerializer` são `ModelSerializer` e o `fcm_token` é `unique` no modelo; o DRF adiciona `UniqueValidator` e rejeita token já cadastrado antes do `update_or_create` que reatribuiria. No celular compartilhado, todo login depois do primeiro falhava; o app só fazia `Logger.warn`. Ver [[DRF - ModelSerializer com campo unique bloqueia upsert]]. Os 2 testes "pré-existentes" de registro de dispositivo que falhavam no `parkpro` apontavam exatamente isso.
2. **Logout não desvinculava no servidor (app):** o `DELETE` do dispositivo sai depois de apagar os tokens → sem `Authorization` → recusado. No operador/dashboard o logout nem chama o servidor.
3. **Falhas silenciosas no ERP:** "sem token" era `logging.info`, histórico dizia "Plantão notificado" sempre, e sem plantonista na escala o `notify_round` dava `return` sem escalar.
- Evidência: registro de 30/09 sem metadados → app ≤ 2.1.0 (metadados só a partir da 2.1.1). Sem acesso ao banco de produção, não houve confirmação por dados.

## Entregue no ERP (14/10/2026, todos com testes)
| Commit | Conteúdo |
|---|---|
| `55bd4d9a` | Resultado da entrega em `OnCallNotificationLog` (`delivery_status` etc., migration `support/0102_oncallnotificationlog_delivery`) e histórico real |
| `0f59871b` | Admin só leitura dos logs de notificação do plantão |
| `21409350` | Removido endpoint público sem auth `.../support/on-call/trigger-notification/` |
| `0b5abdef` | Líder acionado sem plantonista ou com plantonistas inalcançáveis; alerta Webex 1x por chamado |
| `141613b7` | Auditoria de acesso `AppAccessAuditEvent` (migration `parkpro/0024`), admin, limpeza de 90 dias |
| `6a42479e` | **Fix da causa raiz** (sem `UniqueValidator` no `fcm_token`) |
| `5f2cb69e`, `a20e3b53`, `d28219c7` | Modos Operador/Dashboard/Técnico/Plantão; eventos de login e dispositivo; metadados `null` aceitos |
| `bef4b892` | Logout por modo de login desvinculando só o aparelho que pediu |
| `9636900e` | Merge do master |

## Contrato para o app ParkPro
- `POST api/v1/parkpro/auth/logout/`, `.../field/auth/logout/`, `.../support/auth/logout/`; access token do modo; body `fcm_token` e/ou `device_uuid` (≥1, senão 400); 204 idempotente; sem rota no legado `mypark`.
- Registro de plantão/técnico reatribui token já cadastrado (201); os 3 registros aceitam metadados `null`.
- `DELETE .../support|field/devices/<id>/` mantidos com `TODO(cloudpark-erp#739)` até o app novo estar em uso.
- O fix do 400 já vale com o app instalado (o logout da 2.1.x limpa o token local e o próximo login reenvia o registro).

## App ParkPro — plano (aprovado em 2026-10-14)
Repo `cloudpark/parkpro`, branch `cloudpark/cloudpark-erp/739` a partir do `stable`, PR `739-test → test`. Decisões: [[2026-10-14 - Alarme de plantão e auditoria de acesso (#739)]] (seção "App ParkPro").
0. Preparação: `pnpm install` e rodar `typecheck`, `lint` e `test:ci` para ter a referência antes das mudanças.
1. Services de dispositivo: `DeviceInstallationStorage`, UUID estável, `getLogoutPayload()`, unsubscribe do listener do plantão, `await` da permissão no Android, acento.
2. Logout no servidor: DTO, `logout()` nos 3 repositories de auth e casos de uso por persona.
3. Ligar o logout nas telas: hooks de persistência só locais, botões "Sair da conta" e logouts forçados; remoção do `DELETE` e do `deviceId`.
4. Status de registro e novas tentativas (plantão e técnico).
5. Indicador "alarme ativo neste aparelho" na Home do plantão (`GET /support/devices/`).
6. Testes do header `x-min-app-version` no `globalError.interceptor`.
7. Validação final, roteiro de teste manual com a auditoria do ERP e rascunho da PR.

### Estado em 2026-10-15 (fim do dia)
Branch local, **nada enviado ao remoto**, sem PR. Histórico do dia em [[2026-10-15]].

| Passo | Estado | Commits |
|---|---|---|
| 0 | Concluído. Antes das mudanças: 1081 testes passando, 10 erros de typecheck que já existiam, 7 avisos de lint | — |
| 1 | Concluído. Os commits 1 e 2 do plano viraram um só | `5796411`, `3a08dfe`, `efcc076`, `bbca811` |
| 2 | Concluído (`SessionLogoutService`, classe única com 3 instâncias; logout com timeout de 5s) | `459567b`, `17cda44` |
| 3 | Concluído (`useSessionLogout`; confirmação ao sair do plantão; `DELETE` e `deviceId` removidos) | `0f666b0`, `044fb67` |
| 4 | Concluído (4 motivos de falha, novas tentativas em 5s/15s/30s, modal uma vez por motivo em cada login) | `efbabf6`, `200bd17`, `0703565` |
| 5 | Implementado e validado, **sem commit** (indicador compara só pelo token atual) | — |
| 6 | Não iniciado | — |
| 7 | Não iniciado | — |

- Ao fim do dia: 207 suítes e 1184 testes passando, typecheck só com os 10 erros que já existiam, lint com 0 erros. Nada foi testado em aparelho.
- **App antigo:** a versão nova não usa mais `DELETE .../devices/<id>/`. As rotas continuam no ERP para os apps já instalados (`TODO(cloudpark-erp#739)`).

## Pendências
- Abrir a PR do ERP (assignee @me, reviewer rkoball) + checklist na issue; comentar na issue a causa raiz e as evidências (combinado para o fim da atividade).
- **Conflito de migrations previsto:** a #732 também cria `support/0102`…`0105` a partir de `0101_merge`. A segunda a entrar no master precisa de merge migration (uma folha por app).
- App ParkPro: Passos 0 a 4 commitados; Passo 5 sem commit; faltam os Passos 6 e 7, o push e a PR `739-test → test` (assignee @me, reviewer rkoball). Veja o plano e o estado acima.
- **Versão mínima (ERP):** hoje o ERP não envia header de versão mínima. Falta implementar `PARKPRO_MIN_APP_VERSION` + `x-min-app-version` (lido pelo app desde a 2.1.0) e ativá-lo depois que o app novo for publicado.
- Revogar JWT no logout: adiado por decisão do usuário.
- `fieldcontrol.views.TechnicianDeviceViewSet` está sem rota (código morto); não tratado.
- Ação operacional sugerida: cada plantonista conferir em "Dispositivos de Atendente" o próprio registro antes do plantão até o deploy.
