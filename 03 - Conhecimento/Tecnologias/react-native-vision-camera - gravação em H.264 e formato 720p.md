# react-native-vision-camera — gravação em H.264 e formato 720p

Abrangência: **tecnologia** (react-native-vision-camera 4.7, ParkPro). Verificado nos tipos instalados (`src/types/VideoFile.ts`, `src/hooks/useCameraFormat.ts`). Origem: [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]].

## Fatos
- `startRecording({ videoCodec })` aceita `'h264' | 'h265'`. A doc diz que `h264` tem suporte amplo e `h265` gera arquivos até 50% menores.
- A resolução da gravação vem do **formato** da câmera, não de uma opção por gravação. `useCameraFormat(device, [{ videoResolution: { width, height } }])` escolhe o formato, e a prop `format` do `<Camera />` aplica.
- O formato também define a resolução das fotos. Um formato 720p no modo foto deixa a foto menor, então o formato só deve valer no modo vídeo.

## Boas práticas usadas
- Filtro do `useCameraFormat` em constante de módulo. Um array literal dentro do componente é recriado a cada render.
- Em teste, o `Camera` mockado expõe `startRecording` via `useImperativeHandle`, e `useCameraFormat` é mockado.

## Não verificado
- Default de `videoCodec` nesta versão da lib (não conferido).
- Qualidade e FPS no formato 720p em aparelho.
- Se trocar o `format` reinicia a sessão e causa atraso ao alternar entre foto e vídeo.

## Relacionados
- [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]]
- [[expo-image-picker - preset de exportação de vídeo (iOS)]]
