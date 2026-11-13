# 2026-11-11 - Mensagem automática do aceite configurada no ERP (#881)

Abrangência: **projeto** (ParkPro + ERP). Status: **confirmado** (implementado; PR do app mesclada em `test`, PR para RC aberta).

Projeto: [[881 - Mensagem automática do aceite do plantão]].

## Contexto
A mensagem enviada ao cliente quando o plantonista aceita o chamado era fixa no app. O cliente pediu que ela fosse configurável.

## Decisão
- O texto fica em Constance > Config > Suporte no ERP, com o texto atual como padrão.
- O app busca o texto no endpoint `GET /v1/parkpro/support/on-call/accept-message/` ao abrir a tela de aceite e preenche o campo, que continua editável.
- Se a busca falhar, o campo fica vazio, aparece um aviso orientando a digitar a mensagem e o aceite segue liberado. Sem modal.
- Erro do backend é mostrado ao usuário com a mensagem que o backend devolve. Erro inesperado vira texto fixo em pt-BR, sem texto técnico.

## Justificativa
- Um texto fixo no app obrigava a nova versão para trocar uma frase.
- Bloquear o aceite quando a busca falha colocaria o alarme de plantão em risco; o plantonista precisa conseguir atender.

## Consequências
- O ERP precisa subir antes do app (cloudpark-erp#912). Sem o endpoint, o campo vem vazio e o plantonista digita a mensagem.
- Aparelhos com versão antiga do app seguem com o texto fixo até atualizarem.
- A publicação nas lojas só depois do deploy do #912 em produção.
