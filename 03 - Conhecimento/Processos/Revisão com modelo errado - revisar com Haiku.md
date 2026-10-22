# Revisão com modelo errado — /revisar com Haiku

Tipo: recomendação. Origem: revisão da #723 em 2026-10-22.

## Situação
- O cabeçalho do skill `/revisar` pede Opus, mas a revisão rodou em outro modelo (Haiku).
- A marca de revisão do PR precisa refletir o modelo que de fato revisou, não o indicado no cabeçalho.

## Recomendação
- Quando o cabeçalho do skill `/revisar` pedir Opus e a revisão rodar em outro modelo, confirmar o modelo antes de publicar a marca no PR.
- A marca do PR deve registrar o modelo real que fez a revisão.

## Lição
- Lição sem culpa: a checagem do modelo deve acontecer antes da publicação da marca.
