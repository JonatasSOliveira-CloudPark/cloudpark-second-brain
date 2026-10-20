# react-native-gifted-chat — identidade do remetente

Abrangência: **tecnologia** (React Native + `react-native-gifted-chat`). Observado no ParkPro em 2026-10-15 ([[ParkPro - Notas técnicas]]).

## Como a lib funciona
- O `GiftedChat` recebe `user={chatUser}` e posiciona cada mensagem à direita ou à esquerda comparando `message.user._id` com `user._id`. Ajustar um `isMine` só dentro de componentes customizados não muda esse lado. O agrupamento de bolhas (`previousMessage.user._id`) usa a mesma chave.

## Candidato a padrão (ainda não é regra)
- Quando os remetentes vêm de **tabelas diferentes** (cliente, atendente, técnico, sistema), os ids podem coincidir. Nesse caso, a identidade usada na UI deve combinar tipo e id (ex.: `` `${sender_type}:${sender.id}` ``), tanto no mapper das mensagens quanto no usuário atual do chat.
- O id numérico do servidor continua separado para o que vai ao backend ou ao MQTT (presença, reply). As comparações de "sou eu" também conferem `sender_type`.
- Aplicar quando o backend não garante ids únicos entre os tipos de remetente. Origem: a regra de "mensagem minha" do contrato da [[732 - Falar com Suporte (técnico)]] (`sender_type` e `sender.id`).

## Tipagem: `getColorScheme` no `IGiftedChatContext`
- Observado em 2026-10-20 com o `pnpm typecheck` do ParkPro, em `node_modules/react-native-gifted-chat/src/GiftedChat/index.tsx` (linha 322): o `getColorScheme` da lib retorna `ColorSchemeName`, que inclui `"unspecified"`, e o tipo esperado aceita só `"dark" | "light" | null | undefined`.
- O erro não é de código do projeto. Ao atualizar a lib, conferir se ele some antes de mexer em tipos do projeto. Não editar `node_modules` para contornar.
