# Guia do usuário - um documento por atividade

Abrangência: **processo** (documentação de uso para técnico e cliente). Status: **confirmado** na #881, que teve guia de ERP e app.

## Regra
- Uma atividade gera um só guia, com uma seção por alvo (ERP e/ou app), na ordem em que o usuário faz as coisas.
- Quando o ERP configura algo que o app usa, a seção do ERP vem primeiro.
- Não criar um guia separado por alvo para a mesma atividade.

## Como é capturado
- App Android: capturas por adb, sem tocar em botões que mudam estado.
- ERP: capturas pelo navegador, em ambiente de teste ou local, sem salvar configuração e sem login feito pelo agente.
- Imagens numeradas no documento inteiro, com prefixo do alvo (`01-erp-...`, `02-app-...`).

## Onde fica
- Pasta `guia-<slug-da-atividade>` com o `.md` e `imagens/`. Publicação no Outline só com pedido explícito.
- Skill: `guia-usuario-app` (estendida em 2026-11-11 para aceitar os alvos app e erp).
