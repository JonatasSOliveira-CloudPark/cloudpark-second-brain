# Notifee - cancelar notificação do FCM no Android

- Abrangência: tecnologia (`@notifee/react-native` com notificação publicada pelo FCM).
- Status: **hipótese forte**. Verificado na leitura do bytecode da biblioteca (`app.notifee.core`, método de cancelamento por lista), não testado no aparelho.

## Problema
- Uma notificação exibida pelo FCM (sem passar pelo notifee) fica com id `0` e a tag que o servidor mandou.
- `notifee.cancelNotification(id, id)` procura a tag com o hash do id, não com o `"0"`. A notificação não é removida.
- Resultado: a notificação continua na bandeja depois que a situação acabou.

## Como cancelar
- Usar `notifee.cancelDisplayedNotification("0", notificationId)`, com a tag que o servidor enviou no `android.notification.tag`.
- Validar no aparelho antes de dar como resolvido.

Contexto de uso: [[616 - Reforço de plantão]].
