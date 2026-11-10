# 2026-11-09 - Mapa e cópia do endereço com mensagem fixa (#172)

**Abrangência:** projeto (ParkPro). **Status:** decidido e implementado na branch da #172 (PR #173, para `test`).

## Contexto
A tela de detalhes da visita ganhou "Abrir no Maps" e "Copiar" no cartão de endereço. Essas ações podem falhar por motivos do aparelho: sem app de mapas, ou falha na área de transferência.

## Decisão
- Erros dessas duas ações usam `Toast.show` com texto fixo em pt-BR. O detalhe técnico vai para o `Logger`.
- Não usar `showErrorToast` nelas.
- Os links de app de mapa (`geo:`, `comgooglemaps://`, `waze://`, `maps.apple.com`) são montados pelo app a partir do endereço do pátio e abertos com `Linking.openURL`, sem `safeOpenUrl`.

## Justificativa
- `showErrorToast` repassa a mensagem original quando ela não bate com a lista de padrões de rede. Uma mensagem nativa do sistema chegaria ao técnico.
- A regra de produto é nunca mostrar erro técnico ao usuário. Ver `CLAUDE.md`, seção "Estado de tela".
- Os links de mapa não vêm de fora nem são páginas web, então `safeOpenUrl` não se aplica. A exceção está no `CLAUDE.md`, seção "Push, plantão e MQTT".

## Consequências
- Pendente, fora do escopo da #172: `getUserFriendlyErrorMessage` (`src/presentation/utils/errorFeedback.utils.ts`) ainda repassa mensagem técnica em outros pontos do app.
- Novas ações de mapa ou cópia devem seguir o mesmo padrão.

Relacionado: [[ParkPro - Notas técnicas]]
