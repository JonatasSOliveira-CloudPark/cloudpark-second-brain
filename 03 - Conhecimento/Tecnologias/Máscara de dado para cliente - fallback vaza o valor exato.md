# Máscara de dado para cliente — fallback vaza o valor exato

Abrangência: **tecnologia** (contrato entre backend e app mobile). Status: **confirmado em um caso** (#736, ParkPro/ERP). Candidato a padrão; não promover a regra geral sem confirmação.

## Problema
- Ao mascarar um dado para o cliente (ex.: horário exato → período), a chave antiga pode continuar carregando o valor exato enquanto apps antigos existirem.
- Se o app novo fizer fallback para essa chave quando a chave nova estiver ausente, ele mostra o valor exato.

## Regra aplicada
- O app mostra a informação mascarada só quando a chave nova vier preenchida.
- Sem a chave nova, a linha não aparece. Não há fallback para a chave antiga.
- Teste que cobre: payload sem a chave nova e com a chave antiga contendo o valor exato → o valor não aparece na tela.

## Onde aplicar a máscara
- Preferir o backend. O app só exibe o que recebe, então a API não envia o valor exato para o cliente.
- Máscara só no front deixa o valor exato no payload, visível para quem inspeciona a resposta.

## Relacionado
- Decisão do caso: [[2026-10-22 - Máscara de período do horário da visita (#736)]]
