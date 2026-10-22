# Testes — provar que o teste reproduz o bug

Abrangência: **candidato a global** (processo de teste, não específico de uma stack). Status: **confirmado** em um caso (ParkPro, #736). Não tratar como regra geral sem confirmação.

## Problema
- Um teste escrito junto com a correção pode passar mesmo sem ela, e a suíte fica verde sem provar nada.
- A regra do time pede teste que falhe sem a correção.

## Como verificar
1. Restaurar, do commit anterior à correção, só os arquivos de código alterados. Os testes novos continuam.
   - `git checkout <commit-anterior> -- <arquivos de código>`
2. Rodar os testes novos. Eles precisam falhar pelo motivo do bug.
3. Restaurar o estado commitado: `git checkout HEAD -- <arquivos de código>`.
4. Conferir `git status` antes de seguir.

## Cuidados
- Só restaurar arquivos que já estão commitados, para não perder trabalho em andamento.
- Ver que a falha é a esperada (ex.: texto antigo na tela), não um erro de compilação ou de fixture.

## Relacionado
- Caso: [[2026-10-22 - Máscara de período do horário da visita (#736)]]
