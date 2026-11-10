# Android - capturar telas e navegar com adb

**Status:** confirmado em 2026-11-09, com um Samsung SM_A013M conectado por USB e o app em depuração.

## Problema
Gerar capturas reais de um app Android para documentação, sem instalar nada além do adb.

## Solução
- Conferir o aparelho: `adb devices -l`, e a tela em primeiro plano com `adb shell dumpsys window | grep mCurrentFocus`.
- Resolução: `adb shell wm size`. As coordenadas de toque usam essa escala. No aparelho usado, 720x1480.
- Captura: `adb exec-out screencap -p > arquivo.png`.
- Toque: `adb shell input tap X Y`. Arraste: `adb shell input swipe x1 y1 x2 y2 400`.
- Espere de 1 a 3 segundos depois de cada toque antes de capturar.

## Observações
- O primeiro toque, às vezes, não registra. Recapture antes de repetir, porque um toque repetido pode disparar uma ação que muda dados.
- Um toque em botão pode abrir app externo (mapas, por exemplo). Confira o estado antes de seguir.
- Não há confirmação de que o `scrcpy` funciona neste ambiente. Esta nota cobre só o adb.

Relacionado: [[ParkPro - Notas técnicas]]
