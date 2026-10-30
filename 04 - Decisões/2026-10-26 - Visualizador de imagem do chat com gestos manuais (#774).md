# 2026-10-26 - Visualizador de imagem do chat com gestos manuais (#774)

**Abrangência:** projeto (ParkPro). **Status:** decidido e implementado na branch `cloudpark/cloudpark-erp/774-zoom-imagens-chat`. Promovido para `test` (PR #165) e para `release_candidate` (PR #167).

## Contexto
A imagem do chat não podia ser ampliada (issue erp#774). Havia um `Modal` do react-native-paper com a mesma imagem, sem zoom.

## Decisão
Implementar o visualizador manualmente, com `react-native-gesture-handler` (pinça, arraste e duplo toque) e `react-native-reanimated`, sem biblioteca de zoom.

## Justificativa
- `react-native-zoom-toolkit` 5.1.1 era a única candidata ativa. Tinha issues abertas de peer faltando para Reanimated 4 e de crash nativo por callback síncrono chamado de worklet.
- As outras candidatas (`react-native-reanimated-zoom`, `react-native-zoomable-view`, `react-native-image-zoom-viewer`) estavam arquivadas ou sem atividade desde 2024.
- A implementação manual ficou em cerca de 120 a 180 linhas, sem dependência nova.

## Consequências
- Os limites ficam em `imageZoom.utils.ts`: escala de 1x a 4x, duplo toque para 2.5x, fechar por arraste com 120 px ou 800 px/s, e só em escala 1.
- Gestos não rodam no jest. A validação é feita no aparelho.
- Não validado ainda: iOS, aparelhos de entrada, e se a URL do anexo é o arquivo original.

Relacionado: [[ParkPro - Notas técnicas]]
