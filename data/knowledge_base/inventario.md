# Inventário da base de conhecimento — EngMan Copilot

Recorte A (Diagnóstico e Confiabilidade) · domínio: **CFTV (câmera PTZ), PMV e Callbox**.

> **Aviso de origem.** A Motiva não libera base real para trabalho acadêmico. Tudo marcado como **simulado** é fictício, criado pelo grupo (com apoio de IA, conforme permitido no enunciado) e está identificado como tal no cabeçalho de cada arquivo. Nenhum registro deve ser apresentado como dado real da Motiva.

## 1. Documentos (11 itens)

| # | Documento | Conteúdo e campos | Origem | Status | Como o assistente acessa |
|---|---|---|---|---|---|
| 1 | Catálogo de ativos | TAG, tipo, fabricante, modelo, km, localização, instalação, vida útil, criticidade — **32 linhas** | simulado | **pronto** (`catalogo_ativos.md`) | Tool `consultar_ativo` |
| 2 | Plano de manutenção preventiva | Ativo, tarefa, periodicidade, responsável, tempo, recursos, condição de execução | simulado | **pronto** (`plano_manutencao_preventiva.md`) | RAG |
| 3 | Histórico de falhas do FSM | TAG, data, sintoma, causa, tempo de reparo, peça, reincidência — **52 registros** | simulado | **pronto** (`historico_falhas_fsm.md`) | Tool `consultar_historico_falhas` |
| 4 | POPs (mín. 3) | POP-CFTV-01 limpeza de lente; POP-PMV-01 inspeção de módulos/fontes; POP-CBX-01 teste de botão/áudio — passo a passo + pré-requisitos de segurança | simulado | a escrever (Sprint 02+) | Tool `consultar_procedimento` + RAG |
| 5 | Fichas técnicas | Especificações de PTZ, PMV e Callbox (parte pública de catálogos de fabricantes como referência; o restante fictício) | real + simulado | a escrever | RAG |
| 6 | Matriz de criticidade | Notas S/A/D, score, faixas P1–P4, regra de segurança | simulado | **pronto** (`matriz_criticidade.md`) | RAG |
| 7 | Glossário de siglas | CFTV, PMV, SAT, SCA, CCO, CBM, RCM, WCM, FSM, RGA, LDA, λ, MTBF, MTTR, WIM, OCR | real (briefing) | a escrever | RAG |
| 8 | Normas e requisitos de segurança | Resumo de NR-10 e NR-35 e regras de trabalho em via (apenas resumo; o texto oficial prevalece) | real (resumo) | a escrever | RAG |
| 9 | Documentos do briefing Motiva | Contexto, ondas de inovação, Temas 1, 2 e 2.1 | real | a extrair do briefing | RAG |
| 10 | Laudos de campo (10+) | Relato textual do técnico, incluindo condição ambiental (chuva, neblina, glare) | simulado | a escrever | RAG |
| 11 | Ativos sintéticos e métricas (bônus) | 4 ativos com histórico completo + MTBF, λ, MTTR e R(t) — **ver `ativos_sinteticos.md`** | simulado | **pronto** | Tool `calcular_metricas_confiabilidade` |

## 2. Fonte única de verdade por dado

| Dado | Fonte única | Observação |
|---|---|---|
| Ficha e criticidade de um TAG | Tool `consultar_ativo` (catálogo) | O catálogo **não** é indexado no vector store, para não existir cópia divergente. |
| Falhas passadas de um TAG | Tool `consultar_historico_falhas` (FSM) | Idem. O laudo de campo (RAG) é texto narrativo; se divergir do FSM, vale o FSM e a divergência é sinalizada. |
| Periodicidade e tarefas | Plano preventivo (RAG) | — |
| Passo a passo e segurança | POP (tool + RAG) | A tool devolve o POP inteiro; o RAG só indexa os POPs para perguntas conceituais. Versão vigente nos metadados. |
| Métricas de confiabilidade | Tool `calcular_metricas_confiabilidade` | O modelo **nunca** calcula de cabeça. |

Hierarquia quando duas fontes discordam: **tool (FSM/catálogo) > POP vigente > plano preventivo > ficha técnica > laudo de campo**. O assistente expõe a divergência, cita as duas fontes, não escolhe em silêncio e marca confiança baixa.

## 3. Decisões de chunking e metadados

**Tamanho do chunk: 300 a 500 tokens, com sobreposição de ~50 tokens (10–15%).**
Justificativa (Aula 02, janela de contexto): o modelo roda com `num_ctx` de 16.384 tokens (ver `docs/decisoes_tecnicas.md`) e o contexto precisa comportar prompt de sistema (~2,5 mil tokens), definições das tools (~2,2 mil), histórico e retornos de tools. Recuperando 4 chunks de até 500 tokens gastamos ≤ 2 mil tokens do contexto. Chunks maiores diluem a similaridade; menores quebram um passo de POP do seu pré-requisito de segurança. Regras extras: **nunca separar uma tabela do cabeçalho** e **dividir POP por seção** (pré-requisitos sempre junto com os passos).

**Metadados guardados em cada trecho** (permitem busca filtrada antes da similaridade):

| Campo | Exemplo | Uso |
|---|---|---|
| `tipo_documento` | `plano_preventivo`, `pop`, `ficha_tecnica`, `norma`, `laudo`, `glossario`, `matriz_criticidade`, `briefing` | Filtrar por tipo de fonte |
| `tipo_ativo` | `CFTV`, `PMV`, `CBX` | Evitar misturar ativos |
| `tag` (quando aplicável) | `CFTV-R01-K412-01` | Laudos de um ativo |
| `versao` / `data` | `1.4` / `2026-09-01` | Priorizar versão vigente |
| `fonte` | `POP-CFTV-01 §3` | Citação obrigatória na resposta |
| `origem` | `real` ou `simulado` | Transparência |

## 4. O que NÃO entra na base (LGPD)

- Dados pessoais de empregados (nome, matrícula, CPF, telefone, e-mail, escala).
- Imagens ou frames de câmeras com rostos ou ocupantes de veículos.
- Placas de veículos capturadas por OCR e dados de TAG de pedágio vinculados a pessoa.
- Dados contratuais sensíveis e valores de arrecadação reais.
- Qualquer dado real da Motiva (a base é 100% simulada ou pública).
- Chaves de API e credenciais.

Laudos simulados são escritos sem nomes de técnicos (usa-se "Técnico A/B") e sem placas.
