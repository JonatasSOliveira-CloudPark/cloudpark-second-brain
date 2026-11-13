# React Navigation - voltar de tela com estado de fluxo preso

Abrangência: **tecnologia** (React Navigation com `useFocusEffect`). Status: **confirmado** no ParkPro (#881) por teste de unidade; o comportamento do `reset` em `beforeRemove` segue a documentação do React Navigation e não foi verificado em aparelho.

## Problema
- A tela de alarme marca um flag de "fluxo encerrado" e navega para outra tela (ex.: aceite).
- Ao voltar, o `useFocusEffect` retorna cedo quando o flag está ligado.
- Qualquer estado de carregamento ligado antes da navegação (ex.: `isAccepting`) nunca é zerado, e os botões ficam desabilitados.

## Correção aplicada
- No foco, se o flag estiver ligado, zerar o flag e o estado de carregamento antes de reiniciar o fluxo normal (alarme e reconsulta do status).
- Não basta zerar só o carregamento: sem reiniciar o alarme e a reconsulta, o chamado pendente fica sem acompanhamento.

## Bloquear a volta durante um envio
- `navigation.addListener("beforeRemove", ...)` com `event.preventDefault()` impede a saída.
- Bloquear só quando `event.data.action.type` for `GO_BACK` ou `POP`. Se bloquear tudo, a navegação de sucesso (`reset`) também pode ser cancelada.
- Remover o listener no cleanup do effect e depender de `[navigation, isLoading]`.

## Testes
- O mock de `useNavigation` precisa ser estável; senão o `useFocusEffect` do mock roda a cada render e mascara o bug. Ver [[Jest - jest.mock com JSX e NativeWind]].
- Para provar o bug, simular o foco de novo no teste (contador no mock de `useFocusEffect`) e conferir `disabled` e `busy`.
- Ver [[Testes - provar que o teste reproduz o bug]].
