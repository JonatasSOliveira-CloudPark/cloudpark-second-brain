# Erro HTTP com mensagem genérica — origem fora da view

Abrangência: **tecnologia** (axios no app React Native; Django REST Framework no ERP). Candidato a padrão; aplicado na investigação do upload de foto do ParkPro (2026-10-26). Status: confirmado pelo código do app e do ERP; a generalização para outros casos é recomendação, não regra estabelecida.

## Armadilha
- Se o app mostra o texto padrão do status (ex.: "Você não tem permissão..." para 403) em vez da mensagem do backend, o corpo da resposta não está no formato que o app lê.
- Formatos lidos pelo app ParkPro: `{"error": "...", "code": "..."}` e `{"error": {"code", "message"}}`. Corpos `{"detail": "..."}` (DRF) ou texto puro (ex.: "Forbidden" de proxy) caem no texto padrão.
- Nesse caso, a resposta não saiu da view de negócio. Veio de algo antes ou ao redor dela: proxy, CDN, WAF ou middleware.

## Como confirmar
- Compare o corpo recebido pelo app com o formato que a view devolve para aquele status. Se a view nunca devolve esse status sem `error`, a origem está fora dela.
- Procure a mesma requisição nos logs da camada de fora (CDN, WAF, proxy). Se ela não aparece nos logs da aplicação, é um sinal forte disso.

## Limitação
- O texto genérico não diferencia proxy, WAF, middleware do Django e regra de rota. A diferença só aparece no log da camada que respondeu.

## Relacionados
- [[ParkPro - Notas técnicas]]
- [[CloudPark ERP - Notas técnicas]]
- [[Axios - refresh token rotacionado e sessão em snapshot]]
