# App mobile — fila de envio com reenvio automático

Aprendizado da revisão da PR cloudpark/cloudpark-parkpro#151 (chat do ticket, 2026-10-15). Vale para qualquer envio feito pelo app que é enfileirado e reenviado sozinho, como mensagens e anexos.

**Status:** recomendação. Os três defeitos abaixo foram confirmados no código e corrigidos, mas isto ainda não é uma regra da equipe.

## O que deu errado
- **"Pode tentar de novo" foi tratado como "segura a fila".** Um erro local permanente do anexo (arquivo acima de 100MB) ficou como retentável e travou todas as mensagens seguintes. Um teste de unidade confirmava esse comportamento como esperado.
- **O retry foi adicionado depois da regra de supressão do erro global, e a regra não foi revista.** Com o modal global ativo em 5xx, cada tentativa automática reabria o modal depois que o usuário o fechava.
- **"Sem resposta" foi tratado como "o tempo real vai reconectar".** A tela prometia "Tentando novamente…", mas só o MQTT, a volta ao app ou um push disparavam a tentativa. O teste no aparelho foi só em modo avião, em que o MQTT também cai, e por isso o caso sempre funcionava.

## Recomendações
- **Separe três classes de falha:**
  - **de conexão** (sem rede, timeout, 5xx, 408, 429): segura a fila para manter a ordem, porque a próxima mensagem também falharia;
  - **definitiva** (4xx, arquivo recusado no aparelho): sai da fila e mostra o motivo;
  - **desconhecida**: não segura a fila e só é reenviada quando o usuário toca.
- **Lance erros de validação local com um tipo próprio,** não com `Error` genérico, para que a classificação consiga distinguir.
- **Timeout não é queda de rede.** Use temporizador para timeout. Para erro de rede, consulte o estado da rede do aparelho: conectado, use temporizador; offline, espere a rede voltar (ex.: NetInfo) sem gastar tentativas.
- **Todo texto do tipo "tentando novamente" precisa de um gatilho garantido.**
- **Ao adicionar retry, revise o erro global de cada chamada envolvida,** inclusive as que não passam pelo interceptor (ex.: upload direto para o storage).
- **Teste os cenários que quebram:** falha permanente seguida de outra mensagem, falha com a rede conectada e falha que se repete em todas as tentativas. Não basta a falha que se resolve na tentativa seguinte.
- **No aparelho, teste além do modo avião:** servidor lento, servidor em 5xx e arquivo inválido.

## Relacionados
- [[ParkPro - Notas técnicas]]
