# 2026-11-13 — Assinatura do cliente com desenho nativo (#179)

**Abrangência:** projeto (ParkPro). **Status:** confirmado no Android (validação em aparelho com fluxo completo); iOS pendente (sem aparelho para validar).

## Contexto
No iPhone 16, a tela de assinatura do cliente não rolava até o botão Confirmar: o canvas da WebView (`react-native-signature-canvas`) capturava os toques e não havia área de rolagem. Ver [[react-native-signature-canvas - canvas 0x0 na 5.0.2]] para o histórico da lib.

## Decisão
- Trocar o canvas da WebView por desenho nativo: `react-native-svg` (traço em `Path`) e `react-native-view-shot` (`captureRef`, PNG em base64 para o envio).
- Remover `react-native-signature-canvas` do app.
- Reorganizar a tela: nome do assinante antes do quadro, botão Confirmar fixo no rodapé, Desfazer e Limpar (com confirmação), aviso ao sair com desenho.

## Justificativa
- O desenho nativo fica dentro da árvore de views do React Native, então o gesto de arraste não disputa com a rolagem da tela.
- Remove a WebView e a dependência com bugs de dimensão do canvas.

## Consequências
- Mudança de dependência nativa: exige build novo nas lojas (ver [[ParkPro - Notas técnicas]]).
- Pendente: validar no iPhone (rolagem, traço, envio). Em Android, fluxo completo validado com envio real em visita de teste.
- A PR para `test` é a #181; a PR para `release_candidate` é a #182, mesclada; a PR para `stable` é a #183, aberta.

Relacionado: [[ParkPro - Notas técnicas]]
