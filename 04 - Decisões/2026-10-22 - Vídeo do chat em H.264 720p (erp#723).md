# 2026-10-22 — Vídeo do chat em H.264 720p (erp#723)

Abrangência: **projeto ParkPro** (chat de ticket do cliente). Contexto: vídeos de iPhone (HEVC) enviados pelo chat tocam só como áudio em navegadores Linux de atendentes (caso do ticket #2330, issue erp#723). Notas do projeto em [[ParkPro - Notas técnicas]].

## Decisão
- **iOS, galeria:** preset `H264_1280x720` no `launchImageLibraryAsync`, via `CHAT_MEDIA_PICKER_OPTIONS` (`chatMediaPicker.constants.ts`).
- **Câmera do chat:** `videoCodec: "h264"` e formato 720p só no modo vídeo.
- **Android:** sem mudança; o envio segue com o arquivo original.
- **Trava de 100 MB:** mantida em `TicketFile.service`. No iOS, passa a valer sobre o arquivo já convertido.
- **Falha na exportação:** aviso com mensagem fixa em pt-BR, sem o texto nativo do picker.

## Justificativa
- Pelo código, o HEVC chega ao ERP porque o envio é feito sem conversão. H.264 é o codec com suporte mais amplo.
- O preset do picker resolve sem lib nativa nova. Lib nativa exigiria build novo nas lojas (regra do `CLAUDE.md`).
- 720p reduz tamanho e processamento e ainda serve como evidência.

## Alternativas descartadas
- **Pular a conversão quando o vídeo já é compatível:** precisaria ler o codec no aparelho, o que pede lib nativa.
- **Teto de 500 MB no original:** o picker exporta antes de devolver o arquivo, então o tamanho do original não é checável antes da conversão.
- **Repetir com `passthrough` quando o preset falha:** reabriria o picker e enviaria o HEVC sem aviso.
- **Converter no servidor (ERP):** não avaliado a fundo, com custo de infraestrutura.

## Consequências
- Vídeo grande do iPhone é exportado antes da recusa, com custo de tempo e bateria.
- No Android, vídeos em HEVC (vários celulares gravam nesse formato) ainda passam.
- Navegador Linux sem suporte a H.264 (build sem codecs proprietários) continua sem tocar o vídeo.
- Apps antigos seguem enviando HEVC, então o fallback de download no ERP continua necessário. Ele está em outra frente, sem PR nesta issue.
- Vídeos já enviados não mudam.

## Estado (2026-10-22)
- PR [#161](https://github.com/cloudpark/cloudpark-parkpro/pull/161) (`cloudpark/cloudpark-erp/723-test` → `test`), com label `revisado`.
- Checklist de finalização postado na erp#723. Teste em aparelho pendente.
- **Causa não confirmada:** falta o `ffprobe` do ticket #2330. Se o vídeo for H.264, a hipótese de HEVC cai e o plano muda.

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[expo-image-picker - preset de exportação de vídeo (iOS)]]
- [[react-native-vision-camera - gravação em H.264 e formato 720p]]
- [[Jest - jest.mock com JSX e NativeWind]]
