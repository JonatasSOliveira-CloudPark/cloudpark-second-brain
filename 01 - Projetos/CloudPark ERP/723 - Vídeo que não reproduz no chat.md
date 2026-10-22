# #723 — Vídeo que não reproduz no chat

- Issue: https://github.com/cloudpark/cloudpark-erp/issues/723
- PR ERP: https://github.com/cloudpark/cloudpark-erp/pull/783 (branch `cloudpark/cloudpark-erp/723`, base `master`)
- PR app: https://github.com/cloudpark/cloudpark-parkpro/pull/161 (branch `cloudpark/cloudpark-erp/723-test`, base `test`)
- Decisão: [[2026-10-22 - Vídeo que não reproduz no chat (#723)]]
- Notas técnicas: [[CloudPark ERP - Notas técnicas]], [[ParkPro - Notas técnicas]]
- Conhecimento: [[Vídeo HEVC em MOV - reprodução e download no navegador]]

## Problema
Vídeo de iPhone (HEVC, em MOV) no chat de ticket e no checklist toca só o áudio em alguns navegadores Linux, sem aviso e sem forma de baixar.

## Status (2026-10-22)
- PR #783 aberto, base `master`, label `revisado`.
- `/revisar` no commit `52cdd8fb` sem bloqueantes.
- Conversão para H.264 720p em andamento no PR cloudpark-parkpro#161.

## Pendências
- Rodar os testes do backend com Postgres.
- Validar HEVC manualmente em Chromium Linux e em iPhone real.
- Confirmar a reprodução no modal de checklist.
- Merge do PR cloudpark-parkpro#161.
- Follow-up: trocar a checagem do `MediaView` por truthiness.
- Remover a key redundante em `Message.tsx`.
- Revisar o texto de erro do picker no ParkPro.
