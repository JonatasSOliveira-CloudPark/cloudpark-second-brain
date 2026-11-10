# 2026-11-10 — Reforço de plantão (#616)

Abrangência: **projeto** (CloudPark ERP + app ParkPro). Contexto: [[616 - Reforço de plantão]].

## Comportamento
- **Ignora Não perturbar:** o canal do reforço tem bypass de DND.
- **Tela informativa sem botão** de confirmação. O reforço não tem "Entendi" nem silencia por toque; o alarme só para com a primeira resposta, o encerramento, a reatribuição, novo aceite ou aguardando cliente.
- **Mensagem automática do aceite não conta como primeira resposta.** Só a mensagem enviada pelo plantonista conta.
- **Visita técnica não para o reforço.** Não tem relação com o plantão.
- **"Aguardando cliente" para o reforço.** Mantido por decisão de produto. Limitação conhecida: o status pode ser colocado sem mensagem ao cliente.
- **Com a tela de alarme ou aceite de outro chamado aberta, o reforço fica só como notificação.** Não tira o foco do alarme de outro chamado.
- **Abrir o chamado sem responder não pausa o ciclo.** A abertura pela web é registrada; a abertura pelo app não é registrada.
- **Erro na rodada:** após 3 erros seguidos, o reforço deixa de ser reagendado e o erro é logado.

## Técnica
- **Canal próprio** (`oncall_reinforcement_v1`), declarado no PR. Divergência da nota técnica da task: o som no Android fica no canal.
- **Registro das paradas no app em memória, por 2 h.** Aceito como limitação: se o app for encerrado entre a parada e um reforço atrasado, o reforço aparece.
- **Flag `push_enabled` desligada** até o app estar publicado e adotado. Depois, remover a flag.

## Consequências
- Ligar a flag antes do app publicado faz o reforço chegar para app antigo, sem tratamento.
- Testes no aparelho pendentes para os pontos que o jest não cobre (iOS, cancelamento nativo, toque com app fechado).
