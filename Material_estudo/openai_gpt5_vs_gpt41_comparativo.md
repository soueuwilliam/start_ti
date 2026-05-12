# GPT-5 vs GPT-4.1: Análise de custo-benefício por tipo de tarefa

> Complemento ao guia principal. Dados de mai/2026 — preços e benchmarks verificados via OpenAI, OpenRouter e fontes independentes (Artificial Analysis, Vellum, Microsoft Learn).

---

## Sumário

1. [Visão geral da família GPT-5](#1-visão-geral-da-família-gpt-5)
2. [Tabela de preços comparada](#2-tabela-de-preços-comparada)
3. [Benchmarks: onde o GPT-5 realmente supera](#3-benchmarks-onde-o-gpt-5-realmente-supera)
4. [Análise do custo real por cenário](#4-análise-do-custo-real-por-cenário)
5. [O contexto como vantagem oculta do GPT-4.1](#5-o-contexto-como-vantagem-oculta-do-gpt-41)
6. [O cache como vantagem oculta do GPT-4.1](#6-o-cache-como-vantagem-oculta-do-gpt-41)
7. [Veredicto por tipo de tarefa](#7-veredicto-por-tipo-de-tarefa)
8. [Quando migrar para GPT-5: checklist de decisão](#8-quando-migrar-para-gpt-5-checklist-de-decisão)

---

## 1. Visão geral da família GPT-5

Lançada em agosto de 2025, a família GPT-5 representa uma mudança de arquitetura em relação ao GPT-4.1: é um **sistema unificado** com roteamento interno entre um modo rápido e um modo de raciocínio profundo. Na prática, isso significa que o modelo decide automaticamente quando "pensar mais" antes de responder, sem necessitar de chamadas separadas à série o-series.

**Modelos disponíveis (mai/2026):**

| Modelo | Lançamento | Contexto | Raciocínio interno |
|---|---|---|---|
| `gpt-5` | ago/2025 | 400K tokens | Sim (adaptativo) |
| `gpt-5-mini` | ago/2025 | 400K tokens | Sim (configurável) |
| `gpt-5-nano` | ago/2025 | 400K tokens | Limitado |
| `gpt-5.4` | mar/2026 | 400K tokens | Sim |
| `gpt-5.4-mini` | mar/2026 | 400K tokens | Sim |
| `gpt-5.5` | abr/2026 | 1M tokens | Sim (alto) |

> **Nota sobre `gpt-5.4` e gerações subsequentes:** as versões 5.4 e 5.5 representam iterações mais recentes da família GPT-5. O GPT-5.5 recupera a janela de contexto de 1M tokens, equiparando-se ao GPT-4.1 nesse aspecto, mas a preços mais elevados ($5/$30 MTok). Para fins práticos de custo-benefício, este guia foca nos modelos `gpt-5` e `gpt-5-mini` como representantes da família, pois são os que exibem a troca mais relevante com o GPT-4.1 e GPT-4.1 mini.

---

## 2. Tabela de preços comparada

Todos os preços em USD por 1 milhão de tokens (MTok). Cache hit = desconto sobre o input.

### Tier nano (mais barato)

| Modelo | Input | Output | Cache hit | Contexto |
|---|---|---|---|---|
| GPT-4.1 nano | $0.10 | $0.40 | **75% off → $0.025** | 1M tokens |
| GPT-5 nano | $0.05 | $0.40 | 50% off → $0.025 | 400K tokens |

O GPT-5 nano tem input mais barato, mas desconto de cache menor e contexto 2.5× menor. Para classificação de alto volume com prompts curtos, o GPT-5 nano é mais barato. Para RAG ou docs longos com cache, o GPT-4.1 nano é melhor.

### Tier mini (produção padrão)

| Modelo | Input | Output | Cache hit | Contexto |
|---|---|---|---|---|
| GPT-4.1 mini | $0.40 | $1.60 | **75% off → $0.10** | 1M tokens |
| GPT-5 mini | $0.25 | $2.00 | 50% off → $0.125 | 400K tokens |

Esta é a comparação mais relevante para a maioria das aplicações de produção. O GPT-5 mini tem input mais barato (37.5% menos), mas output 25% mais caro e desconto de cache inferior.

### Tier full (tarefas complexas)

| Modelo | Input | Output | Cache hit | Contexto |
|---|---|---|---|---|
| GPT-4.1 | $2.00 | $8.00 | **75% off → $0.50** | 1M tokens |
| GPT-5 | $1.25 | $10.00 | 50% off → $0.625 | 400K tokens |

O GPT-5 tem input 37.5% mais barato mas output 25% mais caro. O GPT-4.1 domina em custo total quando output e cache são relevantes.

---

## 3. Benchmarks: onde o GPT-5 realmente supera

### 3.1 Raciocínio matemático e científico

Esta é a vantagem mais clara e documentada do GPT-5 e GPT-5 mini.

| Benchmark | GPT-5 mini | GPT-4.1 mini | Delta |
|---|---|---|---|
| AIME 2025 (matemática competitiva) | **91.1%** | ~52% | +39pp |
| GPQA Diamond (ciências) | **82.3%** | ~55% | +27pp |
| HMMT 2025 (matemática avançada) | **87.8%** | significativamente menor | >20pp |
| Humanity's Last Exam | **16.7%** | menor | +pp relevante |

Para qualquer aplicação que precise de raciocínio matemático, químico, físico ou biológico avançado, o GPT-5 mini justifica seu custo diferenciado de forma clara.

### 3.2 Agentes e instrução multi-turn

A diferença no MultiChallenge benchmark é dramática:

| Benchmark | GPT-5 | GPT-4.1 | Delta |
|---|---|---|---|
| MultiChallenge (instrução multi-turn) | **69.6%** | 38.3% | +31pp — quase o dobro |
| SWE-bench Verified (código agentic) | **74.9%** | 55% | +20pp |

Segundo análise independente da Microsoft, o GPT-5 "planeja, chama APIs, interpreta resultados e decide o próximo passo sem perder o objetivo — com muito menos intervenções do usuário em comparação ao GPT-4.1".

### 3.3 Precisão factual e alucinações

Esta é a segunda grande vantagem do GPT-5 para aplicações de alto risco:

| Métrica | GPT-5 (thinking) | GPT-4o (referência) | Delta |
|---|---|---|---|
| HealthBench (erros factuais) | **1.6%** | 12.9% | -89% |
| Erros em tráfego real de produção | **4.8%** | 11.6% | -59% |
| Alucinação em benchmarks abertos | ~6× menos que o3 | — | referência |

> **Fonte:** OpenAI, "Introducing GPT-5" (agosto 2025): com web search ativo, as respostas do GPT-5 são 45% menos propensas a conter erros factuais que o GPT-4o; com thinking, são 80% menos propensas a errar do que o o3.

### 3.4 Onde o GPT-4.1 ainda se mantém

| Benchmark | GPT-4.1 | Observação |
|---|---|---|
| CharXiv-D (gráficos e diagramas) | 88.4% | GPT-4.1 mini lidera |
| MMLU (conhecimento geral) | 87.5% | ainda competitivo |
| IFEval (instruction following simples) | 84.1% | adequado para templates |

---

## 4. Análise do custo real por cenário

O preço por token é apenas parte da equação. O custo **real** depende do mix input/output e do uso de cache.

### Fórmula de custo por cenário

```
custo_total = (input_tokens × input_price)
            + (output_tokens × output_price)
            - (cached_tokens × cache_discount)
```

### Cálculo comparativo: GPT-4.1 mini vs GPT-5 mini

**Cenário A — Sumarização (input pesado, 80:20)**
```
1M tokens totais: 800K input + 200K output

GPT-4.1 mini: (800K × $0.40) + (200K × $1.60) = $0.32 + $0.32 = $0.64
GPT-5 mini:   (800K × $0.25) + (200K × $2.00) = $0.20 + $0.40 = $0.60

→ GPT-5 mini: 6% mais barato. Vantagem marginal.
```

**Cenário B — Geração de conteúdo (output pesado, 20:80)**
```
1M tokens totais: 200K input + 800K output

GPT-4.1 mini: (200K × $0.40) + (800K × $1.60) = $0.08 + $1.28 = $1.36
GPT-5 mini:   (200K × $0.25) + (800K × $2.00) = $0.05 + $1.60 = $1.65

→ GPT-4.1 mini: 21% mais barato. Vantagem clara.
```

**Cenário C — Chatbot com cache ativo**
```
1M tokens, 70% cache hit:

GPT-4.1 mini cached: 700K × $0.10 + 300K × $0.40 (novo) = $0.07 + $0.12 = $0.19
GPT-5 mini cached:   700K × $0.125 + 300K × $0.25 (novo) = $0.088 + $0.075 = $0.163

→ GPT-5 mini marginalmente mais barato com cache — mas desconto 75% vs 50% inverte isso
   se cache hit aumentar para 85%+:
GPT-4.1 mini: 850K × $0.10 + 150K × $0.40 = $0.085 + $0.06 = $0.145
GPT-5 mini:   850K × $0.125 + 150K × $0.25 = $0.106 + $0.0375 = $0.144

→ Praticamente empate. GPT-4.1 mini vence em alta taxa de cache.
```

**Cenário D — Classificação em escala (95:5)**
```
1M tokens totais: 950K input + 50K output

GPT-4.1 mini: (950K × $0.40) + (50K × $1.60) = $0.38 + $0.08 = $0.46
GPT-5 mini:   (950K × $0.25) + (50K × $2.00) = $0.24 + $0.10 = $0.34

→ GPT-5 mini: 26% mais barato. Vantagem real para classificação pura.
```

### Resumo da análise de custos

| Padrão de uso | Modelo mais barato | Margem |
|---|---|---|
| Input > 70% do total | GPT-5 mini | 10–30% |
| Output > 60% do total | GPT-4.1 mini | 15–25% |
| Cache hit > 70% | GPT-4.1 mini (leve vantagem) | 5–15% |
| Contexto > 400K tokens | GPT-4.1 mini (único disponível) | — |

---

## 5. O contexto como vantagem oculta do GPT-4.1

A janela de contexto é **a diferença mais prática e definitiva** entre as duas famílias para aplicações de produção:

| Família | Janela de contexto máxima |
|---|---|
| GPT-4.1 / mini / nano | **1.048.576 tokens (~750K palavras)** |
| GPT-5 / mini / nano | **400.000 tokens (~300K palavras)** |
| GPT-5.5 | 1M tokens (mas $5/$30 MTok) |

Isso significa que qualquer aplicação que processe:
- Contratos jurídicos completos
- Bases de código inteiras
- Relatórios financeiros extensos
- Múltiplos documentos em uma chamada
- Histórico de conversas muito longas

...precisa do GPT-4.1 ou de chunking forçado com GPT-5. O chunking adiciona latência, complexidade de engenharia e custo de múltiplas chamadas.

**Cálculo de exemplo — base de código de 600K tokens:**
```
GPT-4.1:  1 chamada   → $1.20 (input)
GPT-5 mini: mínimo 2 chamadas → $0.75 × 2 = $1.50 + overhead de engenharia
```

---

## 6. O cache como vantagem oculta do GPT-4.1

O desconto de cache da família GPT-4.1 (75% off) vs GPT-5 (50% off) não parece grande em papel, mas o impacto acumulado em produção é significativo.

**Exemplo: aplicação com 100K chamadas/mês, system prompt de 2.000 tokens (estático)**

```
Tokens por chamada: 2.000 (system, cacheado) + 500 (query variável) = 2.500 input

GPT-4.1 mini sem cache: 250M tokens × $0.40/MTok = $100/mês
GPT-4.1 mini com cache (80% hit):
  80M novos + 200M × $0.10 (cached) = $32 + $20 = $52/mês → 48% de desconto

GPT-5 mini sem cache: 250M tokens × $0.25/MTok = $62.50/mês
GPT-5 mini com cache (80% hit):
  80M novos + 200M × $0.125 (cached) = $20 + $25 = $45/mês → 28% de desconto

→ GPT-5 mini mais barato ($45 vs $52), mas a vantagem se reduz com taxa de cache mais alta.
  Com cache 95%+ (system prompts muito longos), GPT-4.1 mini passa a ser mais barato.
```

**Regra de bolso:** Se seu sistema tem system prompts maiores que 5.000 tokens e alta taxa de reutilização (chatbots, assistentes com docs fixos), o desconto de cache do GPT-4.1 frequentemente compensa o input aparentemente mais caro.

---

## 7. Veredicto por tipo de tarefa

### Tarefas onde GPT-5 (mini) justifica o custo

| Tarefa | Motivo técnico | Ganho esperado |
|---|---|---|
| Raciocínio matemático avançado | AIME 91.1% vs ~52% | +39pp de acurácia |
| Raciocínio científico/médico | GPQA 82.3%, HealthBench 1.6% erro | criticamente superior |
| Agentes multi-step autônomos | MultiChallenge 69.6% vs 38.3% | quase o dobro de confiabilidade |
| Perguntas de domínio especializado | hallucination 6× menor que o3 | confiabilidade produção |
| Análise jurídica/compliance/diagnóstico | precisão factual crítica | risco reduzido |
| Classificação em altíssimo volume (>100M tokens/mês) | input 37.5% mais barato | economia de escala |

### Tarefas onde GPT-4.1 (mini) é superior ou equivalente

| Tarefa | Motivo prático | Vantagem |
|---|---|---|
| Geração de código (não agentic) | GPT-4.1 já lidera SWE-bench em execução direta | custo de output menor |
| RAG com documentos longos | janela 1M vs 400K | único viável sem chunking |
| Sumarização de longa escala com cache | 75% off no cache | custo efetivo menor |
| Chatbots e assistentes com docs fixos | cache 75% off + contexto longo | custo/sessão menor |
| Geração de texto, e-mails, conteúdo | output é > 50% do custo — GPT-4.1 mini leva | 15–25% mais barato |
| Workloads estáveis em produção | sem ganho mensurável para trocar | zero risco de migração |
| Extração e formatação estruturada | quality parity com GPT-4.1 mini | GPT-4.1 mini mais barato |

### Onde o GPT-5 nano supera o GPT-4.1 nano

O GPT-5 nano ($0.05/$0.40) tem input metade do preço do GPT-4.1 nano ($0.10/$0.40), com mesmo preço de output. Para classificação pura (input > 90% dos tokens), o GPT-5 nano é claramente mais barato:

```python
# Exemplo: triagem de 10M mensagens/mês, média 50 tokens/mensagem, output 5 tokens
Total input:  500M tokens
Total output: 50M tokens

GPT-4.1 nano: 500M × $0.10 + 50M × $0.40 = $50 + $20 = $70/mês
GPT-5 nano:   500M × $0.05 + 50M × $0.40 = $25 + $20 = $45/mês

→ GPT-5 nano: 36% mais barato para classificação pura de alto volume.
```

A ressalva: GPT-5 nano está limitado a 400K tokens de contexto e tem desconto de cache menor (50% vs 75%). Para triagem simples sem contexto longo, é o modelo mais econômico disponível.

---

## 8. Quando migrar para GPT-5: checklist de decisão

Use este checklist para decidir se vale migrar workloads existentes de GPT-4.1 para GPT-5:

### Migração é recomendada se:

- [ ] Sua aplicação usa raciocínio matemático/científico e acurácia é crítica
- [ ] Você tem agentes autônomos com muitos passos dependentes
- [ ] Alucinações causam impacto real no negócio (médico, jurídico, financeiro)
- [ ] Seu padrão de uso é dominado por input (>70% tokens de entrada)
- [ ] O contexto necessário fica dentro de 400K tokens
- [ ] Você não depende de altas taxas de cache para manter custo

### Migração NÃO é recomendada se:

- [ ] Seu workload gera muito output (>50% dos tokens são saída)
- [ ] Você processa documentos maiores que 400K tokens por chamada
- [ ] Você tem cache ativo com >70% de hit rate em system prompts longos
- [ ] Sua tarefa é extração, formatação ou geração de conteúdo estruturada
- [ ] O GPT-4.1 mini já atende sua SLA de qualidade sem erros relevantes
- [ ] Você não tem orçamento para testes A/B rigorosos antes de migrar

### Estratégia de migração progressiva

```python
# Não migre tudo de uma vez. Use routing condicional para validar antes.

def route_task(task: dict) -> str:
    """Roteamento progressivo durante migração GPT-4.1 → GPT-5."""
    
    # Tarefas que claramente se beneficiam do GPT-5
    if task["type"] in ["math", "science", "multi_step_agent", "medical"]:
        return "gpt-5-mini"
    
    # Tarefas que se beneficiam da janela de contexto do GPT-4.1
    if task["context_length"] > 300_000:
        return "gpt-4.1-mini"
    
    # Tarefas com muito output (GPT-4.1 mini mais barato)
    if task["output_ratio"] > 0.6:
        return "gpt-4.1-mini"
    
    # Para o resto: A/B test para decidir com dados reais
    import random
    if random.random() < 0.1:  # 10% para validação
        return "gpt-5-mini"
    return "gpt-4.1-mini"  # 90% no modelo estável
```

---

## Resumo executivo

| Dimensão | GPT-5 mini vence | GPT-4.1 mini vence |
|---|---|---|
| Preço de input | $0.25 (melhor) | $0.40 |
| Preço de output | $2.00 | $1.60 (melhor) |
| Desconto de cache | 50% | **75%** (melhor) |
| Janela de contexto | 400K | **1M** (melhor) |
| Raciocínio matemático | **91.1% AIME** | ~52% |
| Agentes autônomos | **69.6% MultiChallenge** | 38.3% |
| Precisão factual | **1.6% HealthBench** | maior |
| Geração de conteúdo | — | **15–25% mais barato** |
| Aplicações com cache intenso | — | **melhor custo total** |

**Conclusão:** Para a maioria das aplicações de produção atuais (RAG, chatbots, geração de conteúdo, sumarização), o GPT-4.1 mini ainda oferece o melhor custo-benefício real quando consideradas as dimensões de cache e output. O GPT-5 mini se justifica claramente para tarefas de raciocínio avançado, agentes complexos e aplicações onde precisão factual é não-negociável. A estratégia ideal é manter GPT-4.1 mini como padrão e rotear para GPT-5 mini apenas as tarefas que comprovadamente se beneficiam das suas capacidades superiores.

---

## Referências

- [Introducing GPT-5 — OpenAI](https://openai.com/index/introducing-gpt-5/)
- [GPT-5 vs GPT-4.1 Guide — Microsoft Learn](https://learn.microsoft.com/en-us/azure/foundry/foundry-models/how-to/model-choice-guide)
- [GPT-5 Benchmarks — Vellum AI](https://www.vellum.ai/blog/gpt-5-benchmarks)
- [OpenAI API Pricing 2026 — PE Collective](https://pecollective.com/tools/openai-api-pricing/)
- [GPT-5 Technical Breakdown — Encord](https://encord.com/blog/gpt-5-a-technical-breakdown/)
- [LLM API Pricing Comparison — IntuitionLabs](https://intuitionlabs.ai/articles/llm-api-pricing-comparison-2025)
