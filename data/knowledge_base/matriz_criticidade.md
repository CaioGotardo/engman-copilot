---
fonte: SIMULADO (critérios criados pelo grupo; não é a matriz real da Motiva)
documento: Matriz de criticidade e regra de prioridade
tipo_documento: matriz_criticidade
tipos_ativo: CFTV, PMV, CBX
---

# Matriz de criticidade e regra de prioridade — SIMULADO

Critérios **explícitos e auditáveis** (mitigação do risco de viés na priorização, R04). Região, praça ou rodovia **nunca** entram na conta.

## Notas de impacto (1 a 5) para a falha atual

| Nota | S — Segurança viária | A — Arrecadação | D — Disponibilidade da função do ativo |
|---|---|---|---|
| 1 | Sem efeito | Sem efeito | Função preservada |
| 2 | Efeito mínimo, outros meios cobrem | Efeito marginal | Degradação leve |
| 3 | Reduz monitoramento do trecho | Efeito moderado | Degradação relevante |
| 4 | Reduz resposta a incidente | Efeito alto | Função quase perdida |
| 5 | Impede socorro ou alerta ao usuário | Perda direta de receita | Função perdida |

## Score e faixas

**Score = 4·S + 2·A + 1·D** (mínimo 7, máximo 35)

| Prioridade | Faixa do score | Prazo de atendimento |
|---|---|---|
| P1 — Emergencial | ≥ 28 | até 4 h |
| P2 — Alta | 21 a 27 | até 24 h |
| P3 — Média | 14 a 20 | até 72 h |
| P4 — Baixa | ≤ 13 | próxima janela preventiva |

**Regra de segurança (sobrepõe o score):** se S = 5 e o ativo estiver sem a função (D = 5), a prioridade é **P1**, qualquer que seja o score.

## Criticidade do ativo (campo do catálogo)

| Criticidade | Critério |
|---|---|
| A | Ativo cuja perda impede socorro/alerta ou cobre ponto crítico de ocorrências (todo Callbox; CFTV e PMV em pontos críticos) |
| B | Ativo com cobertura redundante no trecho |
| C | Ativo de apoio, sem impacto direto em segurança ou arrecadação |

## Tipo de intervenção (enum)

| Valor | Quando |
|---|---|
| corretiva | Ativo falhou ou perdeu função |
| preventiva | Tarefa programada no plano de manutenção |
| inspecao | Suspeita de degradação sem falha confirmada |
| cbm | Intervenção disparada por condição medida (ex.: tensão da bateria abaixo do limite) |
