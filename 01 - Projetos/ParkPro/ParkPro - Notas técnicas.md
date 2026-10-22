# ParkPro — Notas técnicas

Conhecimento do app `cloudpark/parkpro` (React Native/Expo) confirmado no código. As regras gerais do repo ficam no `CLAUDE.md` e no `AGENTS.md` dele; aqui entra só o que não está lá. Data da verificação: 2026-10-15.

## Chat de ticket (cliente)
- Arquivos: `src/presentation/pages/customer/support/Chat/` (hooks e stores) e `src/presentation/components/organism/Chat/`.
- **Está preso ao cliente em vários pontos:**
  - rotas `/v1/parkpro/...` fixas em `HttpTicketMessage.repository`, `HttpMedia.repository` e `HttpTicket.repository`;
  - `SenderType.CUSTOMER` fixo na mensagem otimista (`organism/Chat/index.tsx`) e na presença;
  - perfil do ParkPro em `useChatUser`;
  - status pelo tópico `user/:userId/tickets/status`;
  - "mensagem minha" comparando só o id (`CustomBubble`, `AudioPlayer` e o `user._id` do GiftedChat; ver [[react-native-gifted-chat - identidade do remetente]]).
- **Envio e reenvio (PR #151, ainda não mesclada em 2026-10-15):** a fila fica em `useTicketMessages`.
  - Só segura as mensagens seguintes a falha com `sendFailure.waitsForConnection` (conexão, timeout, 5xx, 408, 429).
  - O anexo recusado no aparelho lança `AttachmentRejectedError` e vira falha definitiva.
  - Timeout e erro de rede com o aparelho conectado usam o temporizador de 5 s, 20 s e 60 s.
  - Offline, o app espera o MQTT reconectar, a volta ao app, um push ou o NetInfo.
  - Envio e anexo usam `CHAT_SEND_GLOBAL_ERRORS`, que suprime o modal global também em 5xx.
  - Aprendizados: [[App mobile - fila de envio com reenvio automático]].
- O `TicketErrorFactory` troca a mensagem do 409 `ACTIVE_TICKET_EXISTS` por um texto fixo ("artigo nesse pátio"). Num fluxo novo, use a mensagem que o backend devolve.
- Vídeo que não reproduz (#723): conversão para H.264 720p em andamento no PR cloudpark-parkpro#161 (branch `cloudpark/cloudpark-erp/723-test`, base `test`). Confirmado no código do PR: o picker da galeria usa `videoExportPreset` H.264 720p (`chatMediaPicker.options.ts`) e a câmera grava com `videoCodec: "h264"`; a mensagem do commit diz que a exportação vale no iOS e o Android segue com o arquivo original. Decisão: [[2026-10-22 - Vídeo que não reproduz no chat (#723)]]; navegador: [[Vídeo HEVC em MOV - reprodução e download no navegador]].

## Bug conhecido: status do chat apagado por evento de SLA
- **Sintoma:** com o chat do cliente aberto, `status` e `status_label` do ticket ficam `undefined` no `ticket.store`. Isso aparece no chat e na `TicketInfo`, que usa o mesmo store, até recarregar.
- **Causa:** o ERP publica no `user/:userId/tickets/status` o evento `ticket_sla_states_updated`, que vem sem `status` (task `update_sla_states_task`, só para ticket não `ENDED`, quando o estado de SLA muda). O `useTicketStatus` não filtra `event_type` e aplica qualquer evento; o `TicketStatusEventDTO` só declara `ticket_closed` e `ticket_reopened`.
- **Impacto:** o envio de mensagens não é liberado indevidamente, porque o evento só chega para ticket aberto.
- **Correção:** aplicar só os eventos que trazem status: `ticket_closed`, `ticket_reopened` e `ticket_end_pause` (fim da pausa, que o cliente já aplicava corretamente). Ignorar `ticket_sla_states_updated`. Feita em 2026-10-15 na branch `cloudpark/cloudpark-erp/732` (`TicketStatusService` + `isTicketStatusEvent`, commit `240398c`), ainda sem PR.

## Vídeo no chat (erp#723, 2026-10-22)
- Decisão: [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]].
- Código: `Chat/components/chatMediaPicker.constants.ts` (preset da galeria no iOS), `Chat/components/Actions.tsx` (aviso de falha) e `CameraOverlay/index.tsx` (H.264 e 720p só no modo vídeo).
- Estado: PR [#161](https://github.com/cloudpark/cloudpark-parkpro/pull/161) para `test`. Teste em aparelho pendente. Causa ainda não confirmada (falta o `ffprobe` do #2330).
- Libs: [[expo-image-picker - preset de exportação de vídeo (iOS)]] e [[react-native-vision-camera - gravação em H.264 e formato 720p]].
- Mock de teste com JSX: [[Jest - jest.mock com JSX e NativeWind]].

## MQTT
- O ERP publica em `parkpro-{stage}/ticket/:ticketId/status` os eventos `ticket_closed`, `ticket_reopened`, `ticket_end_pause` e `ticket_sla_states_updated`. Quem consome esse tópico tem de filtrar por `event_type`.
- O app assina `ticket/:ticketId/queue/receive`, mas o ERP não publica nada nesse tópico (ele só existe no enum `WsTopics`).

## Navegação e push
- O `AppNavigator` monta um stack por persona dentro do mesmo `NavigationContainer`. O `OpenScreenNotificationHandler` só navega se o nome da tela estiver nas `routeNames` do stack atual. Por isso uma tela que o push abre precisa ter o mesmo nome (ex.: `"Chat"`) em todos os stacks que a usam.
- O `TicketChatNotificationHandler` chama `notifications/read` (rota do cliente) quando o chat está aberto em primeiro plano. Na branch da #732, a sessão do técnico só limpa a notificação local.
- **React Navigation 7:** `navigate(nome)` para uma tela que já está mais atrás no stack **empilha uma nova** (só volta com `pop: true`). Para voltar a uma tela existente, use `popTo(nome)`. Com a home do técnico (#732) na base do stack, `popToTop` passou a levar à home, e os retornos para a lista de visitas viraram `popTo("VisitList")`.

## HTTP e sessão
- Um único `axiosClient`. O token vem de `getActiveSession()`, com prioridade suporte > técnico > operador.
- **Rota de outra persona com o token errado abre o diálogo global de erro** (`globalError.interceptor`) e não falha em silêncio. Toda chamada precisa respeitar a persona ativa.
- O MQTT (`src/infra/mqtt/client.ts`) conecta sem credencial por persona. O técnico usa os mesmos tópicos de ticket.
- **Renovação do técnico precisa aceitar o `refresh_token` novo.** O ERP devolve `access_token` e `refresh_token` no refresh. O ramo do técnico em `AuthInterceptor.ts` faz `setTokens({...session.tokens, ...result.value})`, como o do operador. Se ele só copiar o `access_token`, o técnico perde o `technician_id` e o chat trava (bloqueante do rkoball na PR #156, 2026-10-20). O ramo de suporte ainda copia só o `access_token`: não verificado se o ERP rotaciona o refresh do suporte.

## Registro de dispositivo e logout (verificado em 2026-10-14, #739)
- **Logout não desvincula o aparelho no servidor.** Os hooks `supportSessionPersistence` e `fieldTechSessionPersistence` chamam `DELETE /devices/<id>/` depois que o `logout()` do store zerou os tokens → requisição sem `Authorization`, recusada; o `finally` limpa o local mesmo assim. No operador/dashboard (`sessionPersistence` + `Device.service`), o `deleteDevice` nem chama o servidor.
- O registro roda ao montar o stack do modo (`SupportStack.tsx`); falha só vira `Logger.warn`.
- Versões: metadados do dispositivo só a partir da **2.1.1** (01/10/2026). Na 2.1.0, o registro pulava o POST se o token fosse igual ao último salvo (`LAST_FCM_KEY`).
- O `deleteDevice` apaga também o `device_uuid` salvo, gerando UUID novo a cada login. A decisão é mantê-lo estável; ver [[2026-10-14 - Alarme de plantão e auditoria de acesso (#739)]].
- **Efeito visível:** a rota `/support/devices/<id>/` não está em `AUTH_ENDPOINT_PATTERNS`, então o 401 do `DELETE` sem token abre o diálogo "Sessão expirada / Entrar novamente" depois de **todo** logout do plantão.
- **O técnico já desvinculava no botão "Sair da conta":** `VisitList` chama `deleteDevice` antes do `logout()`. Os outros caminhos de logout do técnico não fazem isso.
- Na branch da 732 (2026-10-21), o logout da `Home` usa `useSessionLogout(fieldTechSessionLogout)`, como a `VisitList` da `test`. A branch mãe ainda tem o hook antigo (`deviceId` e `deleteDevice`) até o #739 chegar à `stable`.
- **Os hooks de persistência também rodam no boot sem sessão,** porque o bootstrap chama `logout()`. O operador perdia o `parkpro.device_uuid` a cada abertura do app.
- O `listenTokenRefresh` do plantão não devolve unsubscribe. Cada login acumula um listener, e depois do logout ele faz POST sem auth.
- O técnico (`FieldTechStack`) ignora o resultado do registro.
- `suppressGlobalError` (`true` ou uma lista de status) no config do axios evita o diálogo global. As rotas `.../auth/logout/` novas já casam com `AUTH_ENDPOINT_PATTERNS`, que suprime 401, 403 e 404.
- O `SecureStorageService` lê a chave `mypark.*` antiga quando a `parkpro.*` não existe.
- O `id` do `/support/devices/` só existia para o `DELETE`.
- `GET /support/devices/` devolve uma lista sem paginação, com `fcm_token` e `device_uuid`.
- Contrato novo do ERP (logout por modo, reatribuição de token): [[739 - Alarme de plantão em celular compartilhado]].

## Registro e logout após a PR #155 (estado em 2026-10-20)
- O logout espera só o POST de registro em andamento (`stopAndDrain`). Se o registro estiver na permissão de notificação ou no token FCM, o drain resolve na hora, e nenhum POST sai depois do stop (`ensureSameEpoch`). A espera antiga consumia o orçamento de 5 s sem proteger nada.
- O serviço avisa o início do POST pelo callback `onPostStarted`. Não há timers nem sleeps nisso.
- O aparelho só fica "ativo" quando o token FCM atual é o salvo e está na lista do servidor.
- Falha inesperada vira motivo `unexpected`, retentado em 5, 15 e 30 s. `push_unavailable` fica só para falha de token ou de permissão.
- Cancelamento (`AbortError` ou `CanceledError`) vira `cancelled`: não retenta e não mostra alerta.
- O alerta de falha de registro é adiado nas telas de plantão e volta ao sair. Alerta que o usuário dispensou não volta.
- O operador não mostra alerta de falha de registro (decisão do PO); só registra no log.
- No logout, com aparelho registrado e sem refresh válido, o resultado é erro com o toast "será liberado no próximo login". Sem aparelho registrado, o logout é sucesso.
- O app não chama mais `DELETE /devices/<id>/`. A rota continua no ERP para apps antigos (`TODO(cloudpark-erp#739)`).
- Deduplicação do registro do operador: reaproveita a tentativa em andamento só com o mesmo `signal`. Chamada com outro `signal` faz um POST novo, que o abort alcança.
- PR #155 aprovada pelo rkoball e mesclada em `test` em 2026-10-20. Pendente: validar no aparelho (roteiro de 11 cenários) depois do deploy do ERP #766.

## Versão mínima e atualização forçada (verificado em 2026-10-14)
- **App:**
  - O `globalError.interceptor` lê os headers de versão mínima (`MINIMUM_APP_VERSION_HEADERS`) e também trata o status 426 e os códigos `APP_UPDATE_REQUIRED` e afins. Isso abre um diálogo bloqueante "Atualizar aplicativo".
  - O header só é lido em respostas de sucesso.
  - Nenhum teste cobre esse caminho do header até 2026-10-14.
- **O ERP não envia nenhum header de versão mínima** (verificado na branch 739).
- **Por versão:**

  | Versão | Interceptor de erro global | Headers lidos | `AppVersionGuard` |
  |---|---|---|---|
  | 2.0.0 (15/07/2026) | não | — | não |
  | 2.1.0 (20/07/2026) | sim | `x-min-app-version`, `x-minimum-app-version`, `x-mypark-min-app-version` (sem `x-parkpro-*`) | sim |
  | 2.1.1 (01/10/2026) | sim | inclui `x-parkpro-min-app-version` | sim, e corrige o crash no iOS (fca90ea) |

- O `AppVersionGuard` bloqueia sempre que a loja tem versão mais nova (pela API de atualização da loja). Funciona no Android desde a 2.1.0 e no iOS só a partir da 2.1.1.

## Testes (Jest)
- **Importar o `axios` em tempo de execução pode derrubar a suíte** com `TypeError: Cannot cancel a stream that already has a reader`. O erro vem do adapter `fetch` do axios rodando sobre o `expo/virtual/streams`, e aconteceu em 2026-10-15 num teste de util que importava o `normalizeGlobalHttpError`.
  - Contorno: um helper que só precisa de tipo do axios fica em módulo próprio, com `import type` (ex.: `src/infra/http/isTimeoutError.ts`).
  - Alternativa: usar `jest.mock("@/infra/http/client")`, como fazem os testes de repositório.
- **Checar se um teste de regressão falha sem a correção, sem perder o trabalho:** reverter só os arquivos de produção para o commit anterior (`git show <pai>:<arquivo> > <arquivo>`), manter os testes novos, rodar a suíte e restaurar com `git checkout HEAD -- <arquivos>`. Se o teste importa um export que o código antigo não tem, a falha não prova o comportamento; é preciso um teste de comportamento.
- Teste que dá timeout de 5 s com o código antigo pode estar esperando uma promessa que nunca resolve, e não falhando numa asserção. Ver o motivo antes de concluir.
- **Mock de módulo incompleto quebra o teste antes da asserção.** Se o `jest.mock` não expõe uma função que o código chama, o erro aparece como `X is not a function`, e o teste falha mesmo com o código certo. Ao adicionar uma chamada nova a um módulo mockado, atualizar o mock dos testes que importam esse módulo. Caso: `clearRenewal` em `logoutInBackground.test.tsx` (2026-10-20, PR #157). Outro caso: `getTokenAccount` em `fieldTechRefresh.test.ts`, que o `AuthInterceptor` passou a chamar (2026-10-21).
- Em 2026-10-20 o `pnpm typecheck` da `test` tinha erros em 6 arquivos de teste e um tipo em `react-native-gifted-chat` (ver a nota da lib). Não são de código de produção. Ficaram para outra atividade.

## Relacionados
- [[732 - Falar com Suporte (técnico)]]
- [[739 - Alarme de plantão em celular compartilhado]]
- [[CloudPark ERP - Notas técnicas]]
