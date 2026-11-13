# 881 - Mensagem automática do aceite do plantão

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/881
- ERP: https://github.com/cloudpark/cloudpark-erp/pull/912 (dependência de deploy)
- App para `test`: https://github.com/cloudpark/cloudpark-parkpro/pull/178 (mesclada em 2026-11-12)
- App para `release_candidate`: https://github.com/cloudpark/cloudpark-parkpro/pull/180 (aberta em 2026-11-12, sem revisor)
- Guia do administrador e do plantonista (Outline): https://docs.cloudpark.com.br/doc/mensagem-automatica-do-aceite-do-plantao-guia-do-administrador-e-do-plantonista-m18G67fNSk
- Decisão: [[2026-11-11 - Mensagem automática do aceite configurada no ERP (#881)]]
- Notas técnicas: [[ParkPro - Notas técnicas]]

## Situação em 2026-11-12
- Ajustes da revisão do rkoball: aviso orienta a digitar a mensagem; voltar da tela de aceite fica bloqueado durante o envio.
- Bug corrigido: ao voltar do aceite, Atender e Recusar ficavam desabilitados e o botão carregava sem parar (ver [[React Navigation - voltar de tela com estado de fluxo preso]]).
- Revisão do rkoball aprovada e PR #178 mesclada em `test` em 2026-11-12.
- Falta validar o deploy do ERP #912 antes de publicar o app nas lojas.
