# 2026-10-22 — Máscara de período do horário da visita (#736)

Abrangência: **projeto CloudPark ERP + ParkPro** (chamado do cliente). Issue: cloudpark-erp#736. PR do app: cloudpark-parkpro#160 (para `test`).

## Contexto
- O cliente não deve ver o horário exato da visita técnica, só o período. Evita expectativa rígida e cobrança ao suporte e ao técnico quando há atraso de alguns minutos.
- A máscara é só de exibição: agendamento, SLA, rota e o horário exato para técnico e ERP continuam iguais.

## Decisões
- **ERP:** o detalhe do chamado do cliente tira `scheduled_time` e envia `visit_period_label` ("Manhã", "Tarde" ou "Noite"). `scheduled_time_label` continua com o período, por compatibilidade com apps antigos.
- **Faixas:** pelo horário de início. Manhã 06:00–11:59, Tarde 12:00–17:59, Noite 18:00–05:59.
- **App:** o card mostra "Período: …". O rótulo "Horário:" foi trocado porque sugere hora exata.
- **App, sem a chave (ERP antigo):** a linha não aparece. Não há fallback para `scheduled_time_label` (ver [[Máscara de dado para cliente - fallback vaza o valor exato]]).
- **App:** `scheduled_time` sai do DTO do cliente, porque nenhuma tela lia. `scheduled_time_label` fica no DTO, sem uso no código novo, até os apps antigos saírem.
- **Fora da máscara:** notificações ao técnico, PDF da OS e fluxo do técnico, que continuam com horário exato.

## Justificativa
- Mascarar só no front deixaria o horário exato no payload. Por isso o ERP faz a máscara, e o app só exibe o que recebe.
- Sem fallback, o app antigo do cliente nunca mostra o horário exato, mesmo quando o ERP é antigo.

## Consequências
- O ERP precisa estar em produção antes de o app ir para a loja. Sem isso, a linha de período não aparece.
- Faixa "Noite" decidida pelo usuário: 18:00–05:59 pelo horário de início, como nas faixas acima. Com o expediente de 08:00 às 18:00, o caso não aparece hoje, mas a função nunca falha. A pergunta foi removida da PR do app (cloudpark-parkpro#160).
- `scheduled_time_label` deve sair do ERP e do app quando as versões antigas saírem de uso (tarefa separada).

## Exceções e pontos mantidos
- **PDF da ordem de serviço:** mantém o horário exato. É gerado depois da visita e enviado por e-mail e pelo chat ao finalizar a OS. A issue exige máscara em e-mails, e a exceção foi registrada na #736.
- **Histórico de etapas** (em deslocamento, chegou no local): mostra o horário real. É registro, não expectativa de chegada.
- **Notificações já gravadas:** não são reescritas.
- **Endpoint do PDF sem autenticação:** achado da revisão, já existia antes da mudança. Tratado na issue #781, separada desta decisão.

Notas: [[736 - Máscara do horário da visita para o cliente]].
