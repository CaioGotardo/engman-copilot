---
fonte: SIMULADO (ativos sintéticos para o dataset; dados fictícios)
documento: Perfis de ativos sintéticos e métricas de confiabilidade
acesso: derivado do histórico via tool calcular_metricas_confiabilidade (diferencial)
---

# Ativos sintéticos e métricas de confiabilidade — SIMULADO

Janela: 2025-11-01 a 2026-10-31 (365 dias = 8.760 h). Tempo de operação = 8.760 h − soma dos tempos de reparo.
Fórmulas (modelo exponencial, taxa de falha constante): **MTBF = T_op / n**, **λ = 1 / MTBF**, **MTTR = Σ reparo / n**, **R(t) = e^(−λ·t)**.

| TAG | n falhas | T_op (h) | MTBF (h) | λ (falhas/h) | MTTR (h) | R(720 h ≈ 30 d) | R(2.160 h ≈ 90 d) |
|---|---|---|---|---|---|---|---|
| CFTV-R01-K412-01 | 12 | 8737 | 728 | 1.37e-03 | 1,9 | 37,2% | 5,1% |
| PMV-R01-K233-01 | 10 | 8728 | 873 | 1.15e-03 | 3,2 | 43,8% | 8,4% |
| CBX-R01-K150-02 | 9 | 8736 | 971 | 1.03e-03 | 2,6 | 47,6% | 10,8% |
| CFTV-R02-K088-03 | 9 | 8742 | 971 | 1.03e-03 | 2,0 | 47,7% | 10,8% |

> Premissa explícita: λ constante (fase de vida útil). Com n pequeno, o intervalo de confiança é largo — o assistente deve declarar isso (Nível de confiança: baixo/médio).

## CFTV-R01-K412-01 — Câmera PTZ em ponto crítico de ocorrências

Falha dominante: teia de aranha na lente refletindo infravermelho à noite, recorrente; segunda frente: conector/cabo de rede.

- Causas mais frequentes: teia na lente ×5, lente suja ×2, conector oxidado ×2.
- Registros com reincidência (≤ 90 dias): 4 de 12.
- Histórico completo: `historico_falhas_fsm.md` (filtrar por TAG).

## PMV-R01-K233-01 — PMV de grande porte em trecho de neblina

Falhas alternadas entre fonte de módulo, controlador travado e umidade nos módulos LED.

- Causas mais frequentes: fonte do módulo ×3, travamento do controlador ×3, comunicação ×2.
- Registros com reincidência (≤ 90 dias): 0 de 10.
- Histórico completo: `historico_falhas_fsm.md` (filtrar por TAG).

## CBX-R01-K150-02 — Callbox com alimentação solar + bateria de backup

Falha dominante: bateria de backup (botão sem resposta), o que é segurança viária.

- Causas mais frequentes: bateria ×4, áudio ×2, GPS ×1.
- Registros com reincidência (≤ 90 dias): 1 de 9.
- Histórico completo: `historico_falhas_fsm.md` (filtrar por TAG).

## CFTV-R02-K088-03 — Câmera PTZ em poste sujeito a vibração

Falha dominante: desfoque por vibração do poste, com reincidência apesar de reaperto de parafusos.

- Causas mais frequentes: vibração ×5, lente suja ×2, teia na lente ×1.
- Registros com reincidência (≤ 90 dias): 3 de 9.
- Histórico completo: `historico_falhas_fsm.md` (filtrar por TAG).

