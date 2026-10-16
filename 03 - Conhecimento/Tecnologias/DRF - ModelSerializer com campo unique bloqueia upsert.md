# DRF — ModelSerializer com campo `unique` bloqueia upsert

Abrangência: **tecnologia** (Django REST Framework). Confirmado em 2026-10-14 no CloudPark ERP ([[739 - Alarme de plantão em celular compartilhado]]).

## Problema
Em `ModelSerializer`, um campo `unique=True` no modelo ganha automaticamente um `UniqueValidator`. Se o endpoint pretende fazer upsert (`update_or_create` pelo próprio campo), o `serializer.is_valid()` rejeita com **400** qualquer valor já existente, e o upsert nunca roda. Passa despercebido porque o teste do "primeiro cadastro" funciona.

## Como identificar
```python
[type(v).__name__ for v in MySerializer().fields["campo"].validators]
# contém 'UniqueValidator'
```

## Correção
```python
class Meta:
    extra_kwargs = {"campo": {"validators": []}}
```
- A unicidade continua garantida pelo banco.
- O `MaxLengthValidator` continua (o DRF o adiciona à parte da lista passada).
- O teste de regressão deve enviar um valor já cadastrado e esperar sucesso, de preferência por outro dono (reatribuição).
