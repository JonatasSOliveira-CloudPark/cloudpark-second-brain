# #732 — "Falar com Suporte" no acesso Técnico

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/732
- Repo/branch: `cloudpark/cloudpark-erp`, branch `cloudpark/cloudpark-erp/732`. PR [#768](https://github.com/cloudpark/cloudpark-erp/pull/768), aprovada por rkoball em 2026-10-21 sobre `8ed09187` e **mesclada no master em 2026-10-21 às 13:26 UTC** (merge `95523b56`)
- Doc de trabalho na branch: `docs/tmp/732-falar-com-suporte-tecnico.md` (definições + **contrato para o app**). Temporário: remover ou tornar definitivo antes do merge.
- Decisões e justificativas: [[2026-10-14 - Ticket aberto pelo técnico (#732)]]
- Notas técnicas do ERP: [[CloudPark ERP - Notas técnicas]]

## Objetivo
Técnico de campo (próprio ou de empresa parceira — `fieldcontrol.FieldTechnician`, não é `ParkProUser`) abre ticket direto com o suporte pelo app, sem escolher artigo, e conversa pelo chat. Ticket identificado como aberto por técnico e visível só para ele e para o ERP.

## Estado em 2026-10-14
ERP concluído em 4 passos (commits na branch, testes passando exceto 4 falhas antigas do `parkpro`):

| Passo | Commit | Conteúdo |
|---|---|---|
| 1 | `a99e959a` | `Ticket.Kind.TECHNICIAN`, FKs `opened_by_technician`/`reference_ticket`, `Ticket.display_title`, lado do atendente, filtro "Técnico", escopo do chat ParkPro |
| 2 | `5578c160` | API `api/v1/parkpro/field/`: `parkings/`, `tickets/` (abrir, listar, detalhe); `TechnicianTicketDomain` em `fieldcontrol/domain.py` |
| 3 | `c143e08c` | Chat: `SenderType.TECHNICIAN` + `technician_sender`, rotas `tickets/<pk>/messages/`, mídia, push ao técnico, descrição como 1ª mensagem |
| 4 | `4211a38e` | Exibição no ERP (React): técnico/empresa, ticket de referência, card, chat, busca por técnico (`search_cache`) |
| fix (15/10) | `84c353dc` | Badge "Técnico", sem "N/A", sem bloco fixo de descrição no chat, Soluções/SLA sem carregamento infinito |

Migrations no `support`: `0102` a `0105`.

**Próximo:** planejar o app ParkPro (repo `parkpro`, React Native/Expo) — prompt de planejamento já preparado na sessão; app só começa depois do ERP.

## Estado em 2026-10-15
- Telas do ERP conferidas no navegador com tickets de exemplo (#6776, #6777) no banco isolado da worktree; roteiro de teste entregue ao usuário (ERP React + API do técnico via `curl`).
- Recomendação em aberto: abrir a PR do ERP sem esperar o app (aguardando decisão do usuário).
- Na worktree há 7 alterações de ambiente de desenvolvimento sem commit (acesso pela rede, login sem código em dev), que **não** são desta atividade.
- App ParkPro implementado no repo `parkpro`, branch `cloudpark/cloudpark-erp/732` (5 commits enviados, sem PR, não testado em aparelho). Detalhes no [[2026-10-15]]. Pendente no ERP: recusar abertura pela visita concluída, encerrada com retorno ou cancelada (o app já esconde o botão nesses casos).

## Estado em 2026-10-21
- Mesclada no master (13:26 UTC) e replicada para a release `26.10.16` pela #773.
- Rebase com o master feito. Conflitos de plantão resolvidos com a versão do master (#739), mantendo só a troca de título para `display_title`. Endpoint de disparo manual de plantão (`OnCallTriggerNotificationView`) removido, como no master.
- Migrations de `support` renumeradas para `0111` a `0114`, depois da `0110` do master (commit `c8599aae`). Uma folha por app, conferida por AST.
- Push com `--force-with-lease` em `c475ab18` (fix de fake de teste do alarme). Testes: 709 rodados; restam só as 2 falhas antigas de `parkpro`.
- Stash `732-env-config-rebase` guarda as alterações de ambiente do worktree; backup em `backup/732-pre-rebase-2`.

## Pendências
- Release `26.10.16`: réplica da #768 na PR [#773](https://github.com/cloudpark/cloudpark-erp/pull/773), da branch `cloudpark/cloudpark-erp/732-26.10.16` (um commit `34d3b10b`). Testes: 709, só as 2 falhas antigas de `parkpro`.
- Pendente: atualizar o checklist da issue (ainda diz que a PR está aberta e com revisão pendente) e definir a ordem de release (ERP antes do app).
- Pendente: transformar o doc `docs/tmp/` em definitivo, ou confirmar que a remoção feita na #768 basta.
- Restrição de pátios do técnico: em definição com o QA; hoje vê todos (`TechnicianTicketDomain.available_parkings`).
- Achados de segurança fora do escopo (não tratados):
  - Rotas do app `media` com `AllowAny` no host público (`api/v1/media/media-files/`, `api/v1/tickets/<id>/media/`).
  - MQTT sem autenticação/ACL: qualquer cliente assina/publica nos tópicos de ticket.
  - `TicketCreateSerializer` do cliente (ParkPro) não confere se o pátio é do usuário; `TicketOpeningCheckView` duplica a regra de time sem o redirecionamento fora do horário.
  - #723 (vídeo no chat): o checklist usa o mesmo endpoint de upload de media acima; ver [[723 - Vídeo que não reproduz no chat]].

## Dependências
- **Nova dependência (decidida pelo usuário em 2026-10-16):** as empresas de técnico (`fieldcontrol.TechnicianCompany`) precisarão ter cadastrados os pátios a que têm acesso. A listagem de pátios que o técnico seleciona no app (`GET field/parkings/`) passa a sair desse cadastro da empresa, e não mais de "todos os pátios" (decisão provisória, aguardando o QA).
- **Impacto previsto:**
  - novo vínculo empresa↔pátio (M2M) com migration;
  - a busca de pátios do técnico passa a filtrar pela empresa;
  - a abertura de ticket pelo botão deve conferir que o pátio pertence à empresa do técnico;
  - o app depende desse cadastro estar preenchido.
- **Em aberto (não decidido):**
  - ~~regra para técnico sem empresa~~: decidida em 2026-10-20 (ver decisões);
  - cadastro de pátios pelo admin do ERP;
  - comportamento na abertura pela visita;
  - migration inicial.
- **Decisões de 2026-10-16 (respostas do usuário):**
  - Técnico sem empresa: ainda não decidido; o usuário prefere que todo técnico esteja cadastrado em uma empresa. Pendência: definir se os técnicos próprios entram em uma empresa interna.
  - Cadastro de pátios: no ERP Django, na tela/admin de empresa de técnico (`fieldcontrol.TechnicianCompany`).
  - Abertura pela visita com pátio fora do cadastro: liberar, desde que a visita seja do técnico. O usuário pediu uma verificação de segurança "inteligente", ainda sem definição do que é. Pendência em aberto.
  - Empresas existentes começam sem pátios cadastrados.
  - Restrição provisória com o QA: pergunta em aberto. O usuário não entendeu a formulação; reformular antes de decidir.

## Branches (estado em 2026-10-21)
- Mãe do app: `cloudpark/cloudpark-erp/732`. Recebeu as correções da PR 156 que não dependem do #150 (`e2d915d`, `882e8f1`, `60e342e`).
- PR do app: `cloudpark/cloudpark-erp/732-test` → `test` (cloudpark-parkpro#156), mesclada em `test` em 2026-10-21 pelo usuário. A branch da PR ficou com o merge da `test` (`ce2a85a`), resolvido com os três conflitos aprovados. A aprovação do rkoball é sobre `46a1baa`, anterior a esse merge.
- Só na `732-test` por depender do #150 ou da #739: o merge da `test` e o teste de presença `46a1baa`. A mãe não recebeu nada novo em 2026-10-21, porque os ajustes do merge dependem de código da #739 (`useSessionLogout`) e da #686 (normalização de erro do `Ticket.error.ts`), que não estão na `stable`. Quando o #150 e a #739 chegarem à `stable`, a mãe é reconstruída a partir da `stable` (mesma regra da #739).
- Pendente: teste em aparelho e ordem de release (ERP #768 em produção antes das lojas, conforme a revisão do rkoball).
