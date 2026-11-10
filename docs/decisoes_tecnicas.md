# Decisões técnicas — EngMan Copilot

Regra da disciplina: **caminho de execução 100% gratuito**. Modelo pago é opção, nunca requisito.

> Limites e termos de planos gratuitos mudam com frequência. Os itens marcados com † devem ser reconferidos na documentação oficial na data da implementação.

## 1. Modelos (LLM)

| Critério | **Llama 3.1 8B Instruct** (Ollama local) | Qwen 2.5 7B Instruct (Ollama local) | Gemini Flash (Google AI Studio, free tier) † |
|---|---|---|---|
| Custo de uso | Zero (roda na máquina) | Zero | Zero dentro da cota gratuita |
| Licença / termos | Llama Community License | Apache 2.0 (7B) | Termos do serviço; no plano gratuito os dados enviados podem ser usados para melhoria do produto † |
| Qualidade em pt-BR | Português entre os idiomas oficialmente suportados | Multilíngue, inclui português | Alta |
| Janela de contexto | 128 mil tokens (usamos 16 mil por limite de RAM) | 32 mil por padrão | Muito grande (≥ 1 milhão) † |
| Privacidade | **Local**: nada sai da máquina | **Local** | Dado vai para a nuvem de terceiro |
| Integração com Python | Biblioteca `ollama` e API compatível; suporta tool calling | Idem | SDK oficial; tool calling |
| Maturidade / documentação | Muito ampla | Ampla | Ampla, mas limites mudam † |

**Escolha: Llama 3.1 8B Instruct local.** Pesa a favor a privacidade (R06: histórico de falhas não sai da máquina), custo zero sem cota, suporte a tool calling e documentação abundante.
**Trade-off aceito:** modelo de 8B é menos capaz em raciocínio longo e segue instruções com menos rigor que modelos grandes; exige ~6 GB de RAM livre (quantização de 4 bits) e responde mais devagar em CPU. Mitigação: temperatura baixa, regras curtas e verificáveis, guardrail de saída e eval set. **Caminho de contingência gratuito:** Gemini Flash no free tier, **apenas com dados simulados**.

**Parâmetros (Aula 02 e 08):** `temperature 0.2` (diagnóstico reprodutível; criatividade não é desejável), `top_p 0.9`, `max_tokens 600` (≈ 4 parágrafos, conforme o FORMATO), `num_ctx 16384`, `seed 42` nas avaliações.

## 2. Orquestração

**Escolha: chamada direta via SDK** (sem LangChain). Justificativa: o fluxo tem poucos passos (contexto → LLM → tool → LLM), o grupo está no 1º ano e as três peças do Módulo 3 (`chamar_llm()`, histórico, `montar_prompt()`) já cobrem o necessário; menos camadas facilitam depurar e auditar. **Trade-off:** perdemos abstrações prontas de cadeia/retriever; se o fluxo crescer, migrar para LCEL.

## 3. Vector store e embeddings

| Critério | **ChromaDB** | FAISS |
|---|---|---|
| Custo / licença | Gratuito, Apache 2.0 | Gratuito, MIT |
| Filtro por metadados | **Nativo** (`tipo_ativo`, `tipo_documento`, `tag`) | Não nativo; exige índice auxiliar próprio |
| Persistência | Embutida em disco | Salvar/carregar índice manualmente |
| Privacidade | Local | Local |
| Facilidade em Python | Alta (API simples) | Média (biblioteca de baixo nível) |
| Desempenho em grande escala | Suficiente para a base (centenas de trechos) | Superior em milhões de vetores |
| Documentação | Boa | Boa |

**Escolha: ChromaDB.** A base é pequena e o filtro por metadados é essencial (evita misturar CFTV com PMV). **Trade-off:** FAISS seria mais rápido em escala que não temos.

| Embeddings | **BAAI/bge-m3** (Sentence Transformers, local) | nomic-embed-text (Ollama) |
|---|---|---|
| pt-BR | Multilíngue | Foco maior em inglês † |
| Contexto do modelo | Até 8.192 tokens (cabe nossos chunks de 300–500) | Até 8.192 tokens |
| Custo / privacidade | Gratuito, local | Gratuito, local |
| Peso | Maior (~2 GB) e mais lento em CPU | Leve |

**Escolha: bge-m3**, por causa do português. **Trade-off:** mais pesado. A escolha será validada com um teste de recuperação nas Sprints seguintes (perguntas do eval set contra os chunks); se falhar, trocar para nomic-embed-text.

## 4. Política de contexto quantificada (bônus)

Estimativas por contagem de caracteres (≈ 3,6 caracteres/token em português; ≈ 3 em JSON). A confirmar com o tokenizador do modelo.

| Componente | Estimativa | Observação |
|---|---|---|
| System prompt (9 blocos) | ~2,5 mil tokens | 8,9 mil caracteres |
| Definições das 6 tools | ~2,2 mil | 6,7 mil caracteres de JSON |
| Trechos RAG (top-4 × até 500) | ≤ 2,0 mil | — |
| Histórico (resumo + 6 últimas mensagens) | ≤ 2,5 mil | Política de recorte abaixo |
| Retorno de tools (histórico de 12 meses de um TAG) | ~0,8 a 1,5 mil | 12 registros ≈ 60 tokens cada |
| Resposta (`max_tokens`) | 0,6 mil | — |
| **Total no pior caso** | **≈ 11,3 mil** | Cabe em `num_ctx` 16.384 com folga de ~5 mil |

Por que 16.384 e não 8.192: o total acima (≈ 11,3 mil) estoura 8 mil. O custo é RAM extra de cache de contexto (cerca de 2 GB para 16 mil tokens, estimativa a confirmar).

**Base inteira cabe no contexto?** Não precisa: catálogo (~1,2 mil tokens) e histórico (~1,9 mil) ficam fora do RAG e entram só via tool, por TAG. O restante da base indexada (plano, matriz, POPs, normas, glossário, laudos) é recuperado por trechos.

**Estratégia de recorte:** (1) a cada 6 turnos o assistente oferece um resumo (TAG, hipótese, decisão pendente); (2) acima de ~2,5 mil tokens de histórico, mantém-se o resumo + as 6 últimas mensagens; (3) o TAG em análise é reafirmado em toda resposta, para que nunca se perca no recorte (R08).

## 5. Exemplo de métricas de confiabilidade (bônus)

Entradas vêm do histórico simulado de `CFTV-R01-K412-01` (janela de 12 meses, 8.760 h): n = 12 falhas, soma dos reparos = 23,2 h, logo T_op = 8.736,8 h.

- MTBF = T_op / n = 8.736,8 / 12 ≈ **728 h**
- λ = 1 / MTBF ≈ **1,37 × 10⁻³ falhas/h**
- MTTR = 23,2 / 12 ≈ **1,9 h**
- R(720 h) = e^(−λ·720) ≈ **37,2%** (probabilidade de operar 30 dias sem falha); R(2.160 h) ≈ 5,1%

Premissa explícita: λ constante (modelo exponencial do briefing, Tema 2). Com n = 12, o intervalo de confiança é largo. Padrões como "5 das 12 falhas são teia de aranha" mostram que o modelo exponencial puro esconde causas sazonais e recorrentes: a recomendação é tratar a **causa dominante** (CBM ou limpeza mais frequente) e usar RGA/LDA para tendência quando houver mais dados. Valores calculados pela tool `calcular_metricas_confiabilidade`; o modelo não calcula de cabeça (regra 2).

## 6. Protótipo visual da interface (bônus, wireframe)

```text
+--------------------------------------------------------------------+
| EngMan Copilot                          Persona: Eng. de Manutenção|
+---------------------------+----------------------------------------+
| TAG em análise            |  Conversa                              |
| CFTV-R01-K412-01          |  Você: câmera perde imagem à noite     |
| Criticidade: A            |  Copilot:                              |
| Falhas 12 m: 12           |   Raciocínio: ...                      |
| Reincidência 90 d: 4      |   Fato: 5 de 12 por teia (FSM)         |
|                           |   Hipótese: teia refletindo IR         |
| Prioridade sugerida       |   Recomendação: corretiva              |
| P3 (S3·4 + A1·2 + D3 = 17)|   Prioridade: P3 (até 72 h)            |
|                           |   Nível de confiança: médio            |
| [Ver histórico]           |                                        |
| [Rascunho da OS]          |  [ Digite sua pergunta...        ][>]  |
| [Confirmar abertura] *    |                                        |
+---------------------------+----------------------------------------+
* só habilita após o engenheiro revisar o rascunho (regra 10)
```
