# Questionário indisponível: sem fallback offline nesta entrega (#166)

**Abrangência:** projeto (ParkPro). **Status:** confirmado pelo dev; o PO ainda não validou formalmente. **Data:** 2026-11-04. **Atividade:** parkpro#166.

## Contexto
O critério da task pede que o técnico não fique bloqueado quando o questionário não carrega. Um revisor perguntou qual fallback valeria: usar o questionário já baixado ou cair no campo de descrição.

## Decisão
- Quando o questionário não carrega, a tela mostra "Não foi possível carregar o questionário" com "Tentar novamente". Concluir não é possível sem conexão.
- Não há questionário guardado para uso offline. Trabalhar sem internet é atividade à parte, com sincronização.

## Justificativa
- Trocar status, concluir e assinar já dependem de internet. Usar questionário baixado não permite concluir sem conexão.
- Cair no campo de descrição, com a v1, permitiria concluir sem respostas obrigatórias, contra o critério de obrigatórias.

## Consequências
- Se o PO decidir outro fallback, a mudança vira PR separada ou ajuste da task.
- Um fluxo offline precisa de uma atividade própria, com definição de armazenamento local e sincronização.

Relacionado: [[2026-11-04 - Questionário de conclusão - rota v2 e convivência com o app antigo]], [[ParkPro - Notas técnicas]]
