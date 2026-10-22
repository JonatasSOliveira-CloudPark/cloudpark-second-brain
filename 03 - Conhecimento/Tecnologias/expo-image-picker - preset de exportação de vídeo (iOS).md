# expo-image-picker — preset de exportação de vídeo (iOS)

Abrangência: **tecnologia** (expo-image-picker v55, usado no ParkPro). Verificado no código da lib instalada (`ImagePicker.types.d.ts`, `ImagePickerModule.swift`, `MediaHandler.swift`, `VideoUtils.swift`), **não em aparelho**. Origem: [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]].

## Fatos confirmados no código
- `videoExportPreset` é do iOS (11+) e aparece como deprecated nos tipos, apontando para a doc da Apple. O padrão é `Passthrough`.
- O enum tem `H264_640x480`, `H264_960x540`, `H264_1280x720`, `H264_1920x1080`, `H264_3840x2160` e as variantes `HEVC_*`.
- Com `Passthrough` e um `PHAsset`, o picker copia o recurso original, sem recodificar (`MediaHandler.handleVideo`).
- Com outro preset, o `AVAssetExportSession` exporta **antes** de a promise resolver. Se o preset não for compatível, lança `UnsupportedVideoExportPresetException`.
- O caminho legado (`UIImagePickerController`, usado com `allowsEditing: true`) copia o arquivo para `.mov`.

## Armadilhas
- As mensagens de erro nativas vêm em inglês ("Failed to transcode picked video"). O `getUserFriendlyErrorMessage` do ParkPro mostra mensagens que não sejam técnicas, então passar o erro direto expõe o texto em inglês. Use texto fixo para o usuário.
- Pelo código não dá para saber se `asset.fileSize` é o tamanho do original ou do exportado. Teste antes de depender dele.
- A opção de preset é do iOS. O comportamento no Android não foi verificado.

## Não verificado
- Tempo e bateria da exportação em aparelho.
- Interação entre o preset e `allowsEditing`.

## Relacionados
- [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]]
- [[react-native-vision-camera - gravação em H.264 e formato 720p]]
