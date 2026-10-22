# 2026-10-22 — Vídeo que não reproduz no chat (#723)

## Contexto
- Vídeo gravado no iPhone (HEVC, em MOV) toca só o áudio em alguns navegadores Linux. Acontece no chat de ticket e no checklist.
- O envio do ParkPro manda o arquivo original, sem conversão.

## Decisão
- **ERP (mitigação):** fallback com aviso abaixo do player e download pelo link `?download=1`.
- **Origem (correção):** o envio do ParkPro passa a gravar em H.264 720p. A conversão está em andamento no PR cloudpark-parkpro#161 (branch `cloudpark/cloudpark-erp/723-test`, base `test`), confirmada no código do PR.
- Notas técnicas: [[ParkPro - Notas técnicas]], [[CloudPark ERP - Notas técnicas]]. Projeto: [[723 - Vídeo que não reproduz no chat]].

## Justificativa
- Só o fallback não resolve: o vídeo HEVC continua sem reprodução no ERP. A conversão no envio evita o problema nos vídeos novos.
- Aviso e download não dependem do codec, então servem também para os casos que o fallback não cobre.

## Consequências
- Vídeos HEVC já enviados continuam sem reprodução no ERP. O fallback é mitigação, não correção.
- Vídeos novos só saem em H.264 depois do merge do PR #161 e de uma versão do app que o inclua.
- O aviso depende da detecção de falha ao carregar o vídeo, que ainda é hipótese (ver [[Vídeo HEVC em MOV - reprodução e download no navegador]]).
