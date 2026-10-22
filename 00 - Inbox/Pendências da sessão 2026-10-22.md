# Pendências da sessão 2026-10-22

- **PO — faixa "Noite" (#736):** definir se vale para início depois das 18:00 ou antes das 06:00. A task não define. Ver [[2026-10-22 - Máscara de período do horário da visita (#736)]].
- **Ordem de deploy (#736):** ERP precisa estar em produção antes de o app ir para a loja.
- **Ferramenta `/revisar` (plugin cloudpark-review 0.7.0):** `find_task.py` e `task_repos.py` não reconhecem remote com alias SSH (`git@github-cloudpark:`). Resultado: "Não achei o repo" e nenhum repo da task encontrado. Corrigir no plugin.
- **UI — placeholder do modal de pátio do técnico** (`ParkingSearchModal`, na `test`): "Buscar por nome, cliente ou código" corta em telas pequenas. Sugestão: "Buscar pátio". Sem decisão ainda.
- **ERP — merge com a `master`:** conferir se a branch do ERP ficou com o histórico esperado antes do PR.
