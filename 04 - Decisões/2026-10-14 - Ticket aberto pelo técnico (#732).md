# 2026-10-14 — Ticket aberto pelo técnico (#732)

Abrangência: **projeto CloudPark ERP** (suporte, fieldcontrol, ParkPro). Contexto em [[732 - Falar com Suporte (técnico)]].

## Modelo e identificação
- **Decisão:** novo `Ticket.Kind.TECHNICIAN` ("Técnico") + FK `opened_by_technician`; `reference_ticket` quando aberto a partir de uma visita.
- **Justificativa:** campo estruturado e filtrável; `reported_source` tem outro significado e não foi reaproveitado.

## Abertura
- Um único `POST field/tickets/` com `parking_id` **ou** `visit_id`; `description` obrigatória (vai em `Ticket.description` e vira a 1ª mensagem do chat).
- Sem tipo de requisição (`request_type = None`) e **sem SLA** por enquanto.
- Time: `config.SUPPORT_MAIN_TEAM`; Implantação (`SUPPORT_FIRST_RESPONSE_TEAM`) se `parking.is_first_response_team`; vale o redirecionamento ao N1 fora do horário/feriado.
- Plantão: o plantonista é acionado (`notify_round`), mas **sem** aceite/cobrança de plantão.
- Título `Técnico: <nome>` (a IA sobrescreve no fechamento).
- Duplicidade: 1 ticket não `ENDED` por técnico + pátio (botão) ou técnico + chamado da visita; se existir → **409** no formato `DomainException` já tratado pelo app (`ACTIVE_TICKET_EXISTS`, `meta.ticket_id`). Ticket `ENDED` → abre um novo (não reabre).
  - Justificativa do 409: a descrição nova é descartada; resposta de sucesso esconderia isso.
- Rota `tickets/` (não `support-tickets/`): o namespace `field/` já indica que o recurso é do técnico.

## Chat
- Remetente novo `TicketMessage.SenderType.TECHNICIAN` + FK `technician_sender` (técnico terceiro não tem `ERPUser`).
- Técnico **não** reabre nem avalia; push via `TechnicianDevice`, sem histórico de notificações.
- Lado do atendente: técnico tratado como o cliente (atribui agente, `last_response_at`, contadores da fila).

## Exibição no ERP
- Mostrar só nome do técnico e empresa (`TechnicianCompany`); **sem telefone** (não existe e não é necessário).
- "Escalar para Campo" escondido no ticket do técnico (a visita sai do ticket do cliente).
- Busca/filtro por técnico como tipo próprio no `search_cache`.

## Segurança tratada junto (mesmo contexto)
- Chat ParkPro (`messages`, `reopen`, upload e confirmação de mídia) restrito aos tickets de cliente dos pátios do usuário — antes qualquer usuário acessava qualquer ticket.
- Mensagem só aceita mídia cuja `storage_key` seja do próprio ticket.

## Consequências
- App ParkPro precisa de tela própria de abertura/"meus chamados" e reaproveitar o chat parametrizando rotas, `sender_type`, tópicos MQTT por ticket e a regra de "mensagem minha" (`sender_type` + `sender.id`).

## 2026-10-15 — Ajustes de exibição no ERP
- **Descrição no chat:** a descrição continua sendo a 1ª mensagem do técnico, e o bloco fixo "Descrição" do chat foi escondido no ticket do técnico.
  - Justificativa: o ticket do técnico segue o padrão do ticket de cliente (solicitante de fora, pelo app, conversando no chat). O bloco fixo é o padrão do ticket interno. Sem a mensagem, a fila não marcaria "↙ Técnico aguardando", o app teria de exibir a `description` à parte, e o bloco já aparecia como "Agente - Atendente ERP".
- **Identificação:** badge "Técnico" (ícone de ferramenta) no card e no cabeçalho; badge "N/A" de artigo escondido no ticket do técnico.
- **Painéis sem artigo:** Soluções mostra "Nenhum artigo vinculado"; SLA mostra "Não se aplica a ticket de técnico" enquanto não há prazos. Se o atendente classificar o ticket, os prazos aparecem normalmente.

## 2026-10-20 — Revisão da PR #768
- **Tipo de requisição:** o chamado do técnico não tem tipo de requisição (confirmado pelo PO/QA). Relatórios por tipo não se aplicam a ele.
- **Plantão:** o plantonista é acionado na janela de plantão, mas o chamado não vira plantão: sem marcação `is_on_call` e sem aceite de cobrança.
- **Técnico sem empresa:** login e renovação de token ficam liberados. O bloqueio (`TECHNICIAN_WITHOUT_COMPANY`) vale só nas rotas de chamado; visitas seguem liberadas.
  - Revisão pediu bloqueio global; a decisão foi limitar às rotas de chamado (resposta publicada na PR).
  - Bloqueio global fica com TODO, para ativar depois da publicação da versão nova do app (pendência da #739).
- **Dependência do app:** o app (cloudpark-parkpro#156) precisa ler `refresh_token` na renovação; enquanto não sair, o ERP só devolve o campo.
