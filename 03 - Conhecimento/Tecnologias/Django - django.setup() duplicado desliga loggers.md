# Django - django.setup() duplicado desliga loggers

- Abrangência: tecnologia (Django 4.2, qualquer projeto com `LOGGING` que usa `disable_existing_loggers: True`).
- Status: **confirmado** (reproduzido em 2026-11-16 no ERP).

## Problema
- Um script de teste que chama `django.setup()` e depois `execute_from_command_line(...)` chama o setup duas vezes.
- Cada `setup()` reaplica o `LOGGING` com `dictConfig`. Com `disable_existing_loggers: True`, os loggers dos módulos já importados ficam desligados.
- Efeito: `assertLogs("app.modulo")` falha com "no logs ... triggered" mesmo quando o código loga. Nenhum erro aparece.

## Como evitar
- Chamar `django.setup()` uma vez só e, depois, `call_command("test", ...)` em vez de `execute_from_command_line`.
- O `manage.py test` normal não tem o problema: ele faz um setup só.
- Antes de culpar o código de log, confirmar com `logging.getLogger("app.modulo").disabled` depois do setup.
