# react-native-signature-canvas — canvas 0x0 na 5.0.2

**Abrangência:** tecnologia (React Native, app ParkPro). **Status:** confirmado no aparelho (2026-11-03).

## Sintoma
Na tela de assinatura, o campo aparece em branco e não registra traço. No DOM da WebView, o canvas fica com `width="0" height="0"`, embora na tela ocupe o tamanho normal.

## Causa
Na 5.0.2, `resizeCanvas()` roda uma única vez, na inicialização. Se o layout ainda não tem tamanho nesse momento, o bitmap do canvas fica 0x0 para sempre.

## Correção
Atualizar para 5.1.1 (`^5.1.1`). A versão usa `ResizeObserver` para redimensionar o canvas quando o layout muda (#439 no changelog da biblioteca).

## Como verificar
- Pelo DevTools da WebView (`adb forward` com o socket `webview_devtools_remote_<pid>`): `canvas.width` e `canvas.height` devem ser maiores que 0.
- Desenhar um traço e confirmar que ele aparece.

Relacionado: [[ParkPro - Notas técnicas]]
