# Riscos, ética, viés, segurança e LGPD — EngMan Copilot

O assistente apoia decisão sobre infraestrutura crítica (segurança viária, arrecadação e trabalho em campo). Um erro de IA aqui tem consequência física e financeira, por isso este mapeamento é um artefato próprio e **cada risco tem endereço**: um bloco do prompt, uma etapa do fluxograma, um caso do eval set ou uma regra da base.

Legenda de impacto: **S** segurança · **F** financeiro · **L** legal · **R** reputacional. Severidade: 🔴 alta · 🟠 média · 🟡 baixa.

Blocos do fluxograma (`docs/arquitetura.png`): 1 Entrada · 2 Guardrail de entrada · 3 Montagem do contexto · 4 Chamada ao LLM · 5 Decisão de tool · 6 Execução da tool · 7 Guardrail de saída · 8 Resposta + histórico.

## 1. Matriz de riscos (12 riscos)

| ID | Risco | Como se manifesta neste projeto | Impacto | Mitigação concreta | Onde aparece (rastreabilidade) |
|---|---|---|---|---|---|
| R01 | **Alucinação** | Inventar TAG, modelo, nº de OS, métrica, norma ou registro de histórico | S, F, R 🔴 | Regra de não inventar; tool devolve o dado real; citar fonte; admitir "não sei" com frase padrão | Prompt: regras 1, 2, 5 · Fluxo: blocos 5–7 (tool + checagem de TAG na saída) · Eval: C01, C03, C04, C05, C06, C13 |
| R02 | **Prompt injection** (direta e indireta) | "Ignore as instruções" ou comando escondido em laudo/log colado | S, L, R 🔴 | Instruções imutáveis; texto colado tratado como dado; guardrail de entrada; frase padrão de recusa | Prompt: regra 8 · Fluxo: bloco 2 · Eval: C09, C10 |
| R03 | **Risco operacional** | Orientar abertura de painel energizado, trabalho em altura ou na via sem POP | S 🔴 | Recusa obrigatória com frase padrão; remeter ao POP e a NR-10/NR-35; reforço de desenergização, EPI e sinalização; profissional habilitado | Prompt: regra 7 · Fluxo: blocos 2 e 7 · Eval: C10, C11, C12 |
| R04 | **Viés na priorização** | Priorizar sempre o mesmo tipo de ativo, rodovia ou praça; decidir por "sensação" | F, S, R 🟠 | Prioridade só pela matriz (4·S + 2·A + 1·D) com notas e conta visíveis; região/praça proibidas como critério; ranking só por criticidade e reincidência | Prompt: regra 11 + exemplos few-shot · Base: `matriz_criticidade.md` · Tool: `listar_ativos_por_criticidade` · Eval: C03 |
| R05 | **LGPD — dado pessoal** | Placa (OCR), imagem de ocupante, nome/matrícula de empregado entrarem na base ou no prompt | L, R 🔴 | Lista do que não entra na base; não repetir nem processar o dado; recusar identificação de pessoa; anonimizar laudos ("Técnico A/B"); modelo local | Prompt: regra 9 · Base: `inventario.md` §4 · Fluxo: blocos 2 e 7 · Eval: C14 |
| R06 | **Confidencialidade operacional** | Histórico de falhas e arrecadação indo para API pública de terceiros; vazamento do system prompt | F, R 🟠 | Modelo local (Ollama) como caminho padrão; dados 100% simulados no repositório; prompt não revelado | Decisão técnica: `decisoes_tecnicas.md` · Prompt: regra 8 · Eval: C09 |
| R07 | **Excesso de confiança** | Engenheiro trata hipótese do modelo como diagnóstico confirmado; modelo concorda com afirmação errada | S, F 🔴 | FATO × HIPÓTESE; "Nível de confiança" obrigatório; manter a fonte contra afirmação do usuário; validação humana antes da OS | Prompt: regras 3, 6, 12 + FORMATO · Fluxo: bloco 7 · Eval: C01, C02, C03, C04, C08 |
| R08 | **Contexto estourado** | Conversa longa passa da janela; assistente perde o TAG ou mistura ativos | S, F 🟠 | Política de histórico: resumo + 6 últimas mensagens; TAG reafirmado a cada resposta; `num_ctx` dimensionado | Prompt: regra 14 + BORDA · Fluxo: bloco 8 · Doc: `decisoes_tecnicas.md` §4 · Eval: C13 |
| R09 | **Fontes conflitantes ou desatualizadas** | Plano, POP e laudo divergem; versão antiga de documento é usada | S, F 🟠 | Hierarquia tool > POP vigente > plano > ficha > laudo; mostrar as duas fontes; versão nos metadados; confiança baixa | Prompt: regra 12 · Base: `inventario.md` §2–3 · Eval: C02 |
| R10 | **Responsabilidade técnica e validade legal** | Assistente emitir laudo/ART/parecer ou "liberar" equipamento | L, S 🔴 | Escopo recusa laudo/ART; assistente só recomenda; decisão e assinatura do engenheiro | Prompt: regras 4, 6 · README: escopo · Eval: C05, C07, C08 |
| R11 | **Agência excessiva (ação autônoma)** | Abrir OS sem confirmação, duplicar OS ou agir sobre texto injetado | F, S 🟠 | Contrato da tool exige `confirmado_pelo_engenheiro = true`; prompt exige rascunho + confirmação; tool simulada | Prompt: regra 10 · Tool: `abrir_ordem_servico` · Fluxo: blocos 5–6 · Eval: C03, C10 |
| R12 | **Falha de tool / indisponibilidade** | Tool com erro ou timeout leva o modelo a responder "de memória" | F, S 🟡 | Contrato de erro padronizado; 1 nova tentativa; orientar consulta direta ao FSM; nunca responder sem fonte | Prompt: regras 5, 13 · Tools: `formato_erro` · Eval: C06 |

Todos os 12 riscos têm pelo menos um endereço no prompt e um caso no eval set (R12 e R06 por C06 e C09; R10 por C05, C07, C08).

## 2. Mapeamento para o OWASP Top 10 para aplicações com LLM (edição 2025)

| Categoria OWASP | Risco do projeto | Tratamento |
|---|---|---|
| LLM01 Prompt Injection | R02 | Regra 8, guardrail de entrada, C09, C10 |
| LLM02 Divulgação de informação sensível | R05, R06 | Regra 9, base sem dado pessoal, modelo local |
| LLM04/LLM08 Envenenamento de dados e fragilidades de embeddings (RAG) | R02, R09 | Laudos tratados como dado; metadados de versão e origem; só documentos do inventário entram no índice |
| LLM05 Tratamento inadequado de saída | R01, R03 | Guardrail de saída (TAG, fonte, fato × hipótese) |
| LLM06 Agência excessiva | R11 | Confirmação humana + parâmetro `const: true` na tool de escrita |
| LLM07 Vazamento de system prompt | R06 | Regra 8 e C09 |
| LLM09 Desinformação | R01, R07, R09 | Citação de fonte, nível de confiança, hierarquia de fontes |

## 3. LGPD: o que entra e o que nunca entra

**Princípios aplicados** (Lei 13.709/2018): finalidade (apoio à manutenção), necessidade (só dado técnico de ativos), segurança e prevenção (modelo local, sem envio a terceiros) e responsabilização (este documento).

| Categoria | Entra na base? | Justificativa e tratamento |
|---|---|---|
| Dados técnicos de ativos (TAG, modelo, km, falhas) | Sim, **simulados** | Não são dados pessoais. No Challenge são 100% fictícios. |
| Procedimentos, plano, normas (resumo), glossário | Sim | Conteúdo técnico público ou criado pelo grupo. |
| Laudos de campo | Sim, **anonimizados** | Sem nome de técnico (usa "Técnico A/B"), sem placa, sem imagem. |
| Placas de veículos (OCR) e TAG de pedágio ligada a pessoa | **Nunca** | Dado pessoal de terceiro; fora do escopo. Se aparecer na conversa, o assistente não repete e orienta remover (regra 9, C14). |
| Imagens/frames de câmeras com rostos ou ocupantes | **Nunca** | Dado pessoal; o assistente não processa imagem. |
| Dados de empregados (nome, matrícula, CPF, contato, escala) | **Nunca** | Não há finalidade técnica que exija. |
| Dados contratuais sensíveis e arrecadação real | **Nunca** | Confidencialidade operacional (R06). |
| Credenciais e chaves de API | **Nunca** | `.gitignore` bloqueia `.env`, `*.key`, `*.pem`. |

Em uma versão de produção (fora do escopo acadêmico), ainda seria necessário: base legal documentada para tratamento, relatório de impacto (RIPD) caso haja dado pessoal, política de retenção de logs de conversa e definição do encarregado (DPO) como ponto de contato.

## 4. Viés e ética

- **Priorização**: a conta é pública e reproduzível (matriz com notas S, A, D). O assistente mostra a conta; o engenheiro audita. Revisão sugerida: comparar periodicamente a distribuição de prioridades por rodovia/praça para detectar viés sistemático (o critério nunca usa região, mas os dados de entrada podem carregar viés, por exemplo, ativos de trechos menos monitorados terem menos registros).
- **Viés de dados**: ativo com poucos registros gera conclusão frágil. O assistente declara o número de falhas e usa "confiança baixa/média" quando n é pequeno (ex.: métricas com n < 5 = tendência "indeterminada").
- **Automação e responsabilidade**: o assistente **recomenda**, o engenheiro **decide e assina**. Não há ação física nem liberação de equipamento.
- **Transparência**: dados simulados são identificados como simulados no inventário, nos arquivos e no README. Nenhum registro é apresentado como dado real da Motiva.
- **Responsabilidade do desenvolvedor** (Aula 04): o grupo responde por documentar limites conhecidos, testar casos de borda antes de qualquer uso (eval set) e não vender o assistente como substituto de procedimento de segurança.
- **Uso de IA na elaboração** (permitido pelo enunciado): rascunho de prompt, eval set e dados simulados foram feitos com apoio de IA e revisados pelo grupo; nenhum output de modelo é declarado como executado.
