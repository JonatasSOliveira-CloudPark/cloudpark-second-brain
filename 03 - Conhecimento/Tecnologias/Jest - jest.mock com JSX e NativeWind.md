# Jest — jest.mock com JSX e NativeWind

Abrangência: **tecnologia** (Jest 29 com jest-expo, ParkPro, que usa NativeWind e react-native-css-interop). Origem: teste de `Chat/components/Actions.test.tsx`, 2026-10-22 (ParkPro).

## Armadilhas
- `jest.mock` é içado para o topo do arquivo, e a factory não pode referenciar variáveis de fora. Variáveis com prefixo `mock` são aceitas, mas isso não cobre bindings que o próprio transform injeta.
- Com NativeWind, o transform injeta `_ReactNativeCSSInterop` quando há JSX no arquivo. O erro aparece como "Invalid variable access: _ReactNativeCSSInterop" e a suíte não roda.

## O que funcionou
- JSX dentro da factory, com os componentes importados por `require` dentro dela. É o mesmo padrão do mock do `Footer` em `CameraOverlay/index.test.tsx`.

## O que não funcionou
- Trocar o JSX do corpo do teste por `createElement` não resolveu.
- Importar `react` e `react-native` no topo do arquivo não basta para a factory.

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[2026-10-22 - Vídeo do chat em H.264 720p (erp#723)]]
