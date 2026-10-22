# Vídeo HEVC em MOV — reprodução e download no navegador

Links: [[ParkPro - Notas técnicas]], [[CloudPark ERP - Notas técnicas]]. Decisão: [[2026-10-22 - Vídeo que não reproduz no chat (#723)]].

## Fato (especificação)
- O atributo `download` do `<a>` é ignorado quando a URL é de outra origem. Para forçar o download nesse caso, o servidor precisa enviar `Content-Disposition: attachment` com `filename`.

## Hipótese (não validada)
- Vídeo HEVC em MOV, sem decoder no Chromium Linux, toca só o áudio.
- Detecção possível: `videoWidth === 0` no evento `loadedmetadata` indica que não há imagem decodificada.
- Validação pendente: teste manual com vídeo de iPhone real em Chromium Linux. Até lá, é hipótese.
