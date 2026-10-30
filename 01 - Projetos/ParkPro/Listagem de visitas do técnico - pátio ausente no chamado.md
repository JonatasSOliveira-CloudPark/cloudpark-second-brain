# Listagem de visitas do técnico - pátio ausente no chamado

- **Data:** 2026-10-30
- **Abrangência:** projeto específico (cloudpark-erp, app ParkPro) com aprendizado reutilizável sobre encadeamento de FK nulável em serializer.
- **Status:** hipótese confirmada pelo código; ainda não confirmada nos dados de produção.

## Sintoma

Erro em produção na listagem de visitas do técnico:

```
AttributeError: 'NoneType' object has no attribute 'address_json'
```

Origem: `get_parking_address` em [parkpro/serializers.py](parkpro/serializers.py), que lê `obj.ticket.parking.address_json`.

## Causa

- O caminho é `FieldVisit → ticket → parking`. A visita não tem FK direta para o pátio.
- `Ticket.parking` é `null=True` ([support/models.py](support/models.py)). Chamado sem pátio faz `obj.ticket.parking` valer `None`.
- Com o pátio `None`, o `.address_json` estoura e derruba a listagem inteira, não só a visita afetada.
- Pátio existente com `address_json` vazio **não** causa o erro: o `if address:` retorna `""`.

## Correção proposta (ainda não aplicada)

- Guardar `parking is None` no serializer e retornar `""`.
- Adicionar `select_related("ticket__parking")` no queryset da listagem.
- Teste de regressão com chamado sem pátio (o teste deve falhar sem a correção).

## Como confirmar nos dados

Visitas cujo chamado não tem pátio, usando `ticket__parking__isnull=True`. Se houver linhas, a hipótese está confirmada.

## Relacionado

- [[Testes - provar que o teste reproduz o bug]]
- [[DRF - ModelSerializer com campo unique bloqueia upsert]]
