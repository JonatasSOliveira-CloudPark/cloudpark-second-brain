# FCM + sessão em app mobile — logout e aparelho compartilhado

Abrangência: **candidato a padrão** para apps mobile com push (FCM) e sessão em store reativo (ex.: Zustand) com token injetado por interceptor do axios. Até agora só foi aplicado no ParkPro (#739); não é regra global. Origem: [[739 - Alarme de plantão em celular compartilhado]] (2026-10-14).

## Armadilha: desvincular o aparelho depois de limpar a sessão
- **Sintoma:** o logout "funciona" no app, mas o servidor continua mandando push para o usuário anterior. No ParkPro também aparecia um diálogo de "sessão expirada" logo depois de sair.
- **Causa:** a chamada de desvínculo roda num efeito que reage a `tokens = null`, ou depois de `store.logout()`. O interceptor lê o token do store, encontra vazio e manda a requisição sem `Authorization`, que o servidor recusa. Um `finally` limpa o estado local e esconde a falha.
- **Solução:**
  - Fazer o logout como um caso de uso explícito: chamar o servidor com o token ainda válido e só depois limpar o store.
  - Efeitos reativos ao fim da sessão fazem só limpeza local.
  - Caminhos em que o token já não vale (expirou, refresh inválido) limpam só o local e deixam o próximo login reatribuir o aparelho.

## Aparelho compartilhado
- O token FCM é por **instalação**, não por usuário. Por isso:
  - o registro do aparelho é reenviado a todo login, mesmo que o token não tenha mudado;
  - o servidor precisa **reatribuir** um token já cadastrado ao usuário que está logando, em vez de recusar. No DRF, ver [[DRF - ModelSerializer com campo unique bloqueia upsert]].
- Mantenha um **UUID de instalação estável** entre logins. Ele identifica o aparelho no logout mesmo se o token mudou e permite ao servidor limpar registros antigos do mesmo aparelho. Dado ligado ao aparelho deve sempre ser filtrado também pelo usuário logado.
- Falha de registro do push não pode ficar só no log quando o push é crítico (ex.: alarme de plantão): mostre o estado ao usuário e tente de novo.
