# #732 — "Falar com Suporte" no acesso Técnico

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/732
- Repo/branch: `cloudpark/cloudpark-erp`, branch `cloudpark/cloudpark-erp/732` (sem PR em 2026-10-14)
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

## Pendências
- Antes do merge: o usuário executar o roteiro de teste, sincronizar com o master (uma folha de migration por app; conflito previsto de `support/0102` com a #739), transformar o doc `docs/tmp/` em doc definitivo (sugestão: `docs/support/`), abrir PR (assignee = autor, reviewer = rkoball) e comentar o checklist na issue.
- Restrição de pátios do técnico: em definição com o QA; hoje vê todos (`TechnicianTicketDomain.available_parkings`).
- Achados de segurança fora do escopo (não tratados):
  - Rotas do app `media` com `AllowAny` no host público (`api/v1/media/media-files/`, `api/v1/tickets/<id>/media/`).
  - MQTT sem autenticação/ACL: qualquer cliente assina/publica nos tópicos de ticket.
  - `TicketCreateSerializer` do cliente (ParkPro) não confere se o pátio é do usuário; `TicketOpeningCheckView` duplica a regra de time sem o redirecionamento fora do horário.

## Dependências
- **Nova dependência (decidida pelo usuário em 2026-10-16):** as empresas de técnico (`fieldcontrol.TechnicianCompany`) precisarão ter cadastrados os pátios a que têm acesso. A listagem de pátios que o técnico seleciona no app (`GET field/parkings/`) passa a sair desse cadastro da empresa, e não mais de "todos os pátios" (decisão provisória, aguardando o QA).
- **Impacto previsto:**
  - novo vínculo empresa↔pátio (M2M) com migration;
  - a busca de pátios do técnico passa a filtrar pela empresa;
  - a abertura de ticket pelo botão deve conferir que o pátio pertence à empresa do técnico;
  - o app depende desse cadastro estar preenchido.
- **Em aberto (não decidido):**
  - regra para técnico sem empresa;
  - cadastro de pátios pelo admin do ERP;
  - comportamento na abertura pela visita;
  - migration inicial.
