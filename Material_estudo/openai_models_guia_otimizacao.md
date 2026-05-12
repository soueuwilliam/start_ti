# Guia de Seleção e Otimização de Modelos OpenAI via API

> Dados verificados em maio de 2026. Preços em USD por milhão de tokens (MTok). Verifique sempre a [página oficial de preços](https://platform.openai.com/docs/pricing) para valores mais recentes.

---

## Sumário

1. [Mapa atual dos modelos OpenAI](#1-mapa-atual-dos-modelos-openai)
2. [Tabela de preços e capacidades](#2-tabela-de-preços-e-capacidades)
3. [Guia de escolha por tipo de tarefa](#3-guia-de-escolha-por-tipo-de-tarefa)
4. [O custo oculto dos modelos de raciocínio (o-series)](#4-o-custo-oculto-dos-modelos-de-raciocínio-o-series)
5. [Otimização específica por modelo](#5-otimização-específica-por-modelo)
6. [Descontos e mecanismos de economia](#6-descontos-e-mecanismos-de-economia)
7. [Arquitetura de cascade: roteamento inteligente](#7-arquitetura-de-cascade-roteamento-inteligente)
8. [Guia de decisão rápida](#8-guia-de-decisão-rápida)

---

## 1. Mapa atual dos modelos OpenAI

A linha de modelos OpenAI se divide em duas famílias com filosofias diferentes:

### Família GPT — "Os executores" (workhorses)
Modelos de baixa latência, sem etapa de raciocínio interno. Ideais para execução direta, geração de texto, extração, formatação e tarefas com padrão claro.

```
GPT-5.5          ← flagship atual (mai/2026), complexidade máxima
GPT-5 / GPT-5-mini / GPT-5-nano  ← família GPT-5, equilíbrio geral
GPT-4.1          ← workhorse de produção, 1M tokens de contexto
GPT-4.1 mini     ← versão econômica do GPT-4.1, supera GPT-4o em benchmarks
GPT-4.1 nano     ← mais rápido e barato, classificação e autocompleção
```

### Família o-series — "Os planejadores" (reasoners)
Modelos treinados com reinforcement learning para "pensar antes de responder". Geram uma cadeia de raciocínio interna (tokens invisíveis que você paga). Ideais para problemas com múltiplos passos, lógica complexa, matemática e ciências.

```
o3-pro   ← raciocínio máximo, pensa mais tempo antes de responder
o3       ← raciocínio avançado, 80% mais barato após corte de junho/2025
o4-mini  ← melhor custo-benefício em raciocínio, excelente em código/visual
```

---

## 2. Tabela de preços e capacidades

Todos os preços em USD por 1 milhão de tokens (MTok). Dados de abril/maio 2026.

### Família GPT

| Modelo | Input ($/MTok) | Output ($/MTok) | Cache hit | Contexto | Melhor para |
|---|---|---|---|---|---|
| GPT-5.5 | ~$2.50 | ~$15.00 | 50% off | 128K | Raciocínio complexo, agentes |
| GPT-5 | $1.25 | $10.00 | 50% off | 128K | Tarefas complexas gerais |
| GPT-5-mini | $0.25 | $2.00 | 50% off | 128K | Volume médio, texto geral |
| GPT-5-nano | $0.05 | $0.40 | 50% off | 128K | Classificação, extração simples |
| GPT-4.1 | $2.00 | $8.00 | **75% off** | **1M** | Código, documentos longos, agentes |
| GPT-4.1 mini | $0.40 | $1.60 | **75% off** | **1M** | Produção econômica, multimodal |
| GPT-4.1 nano | $0.10 | $0.40 | **75% off** | **1M** | Classificação, autocompleção |

### Família o-series (raciocínio)

| Modelo | Input ($/MTok) | Output ($/MTok) | Cache hit | Contexto | Observação |
|---|---|---|---|---|---|
| o3-pro | $20.00 | $80.00 | 50% off | 200K | Raciocínio máximo, uso cirúrgico |
| o3 | $2.00 | $8.00 | 50% off | 200K | Raciocínio avançado, 80% mais barato desde jun/25 |
| o4-mini | $1.10 | $4.40 | 50% off | 200K | Melhor custo/raciocínio, código e visão |

> **Nota crítica sobre o-series:** O preço de output inclui **reasoning tokens** — tokens de raciocínio interno que o modelo gera antes de responder. Eles são cobrados mas não aparecem no output visível. Uma resposta de 500 tokens pode ter custado 5.000–15.000 tokens de raciocínio internos.

---

## 3. Guia de escolha por tipo de tarefa

### Classificação e extração simples
**→ GPT-4.1 nano** (`$0.10/$0.40`)

Tarefas: análise de sentimento, classificação de categorias fixas, extração de entidades, moderação de conteúdo, detecção de intenção.

```python
# Ideal: classificação com output JSON curto e direto
response = client.chat.completions.create(
    model="gpt-4.1-nano",
    max_tokens=50,
    messages=[{
        "role": "user",
        "content": 'Classifique o sentimento. Responda apenas JSON: {"sentiment":"positive|negative|neutral"}\n\nTexto: "Adorei o produto!"'
    }]
)
```

---

### Sumarização, tradução e extração estruturada
**→ GPT-4.1 mini** (`$0.40/$1.60`) ou **GPT-5-mini** (`$0.25/$2.00`)

Tarefas: resumo de documentos, tradução profissional, extração de dados estruturados, geração de descrições de produto, e-mails automáticos.

O GPT-4.1 mini é preferível quando o contexto é longo (até 1M tokens) ou o custo de cache importa (desconto de 75% vs 50% do GPT-5-mini).

```python
# GPT-4.1 mini com contexto longo e cache
response = client.chat.completions.create(
    model="gpt-4.1-mini",
    max_tokens=300,
    messages=[
        {"role": "system", "content": system_prompt},  # cacheado automaticamente
        {"role": "user", "content": f"Resuma em 3 pontos:\n\n{documento}"}
    ]
)
```

---

### Geração de código e análise técnica
**→ GPT-4.1** (`$2.00/$8.00`) para geração, **o4-mini** (`$1.10/$4.40`) para debugging complexo

O GPT-4.1 é o modelo de referência para código, superando o GPT-4o em 21.4% no SWE-bench. Para debugging que exige raciocínio multi-passo, o o4-mini oferece melhor custo-benefício que o o3.

```python
# GPT-4.1 para geração de código
response = client.chat.completions.create(
    model="gpt-4.1",
    max_tokens=2000,
    messages=[
        {"role": "system", "content": "Você é um engenheiro sênior Python. Escreva código limpo, tipado e com docstrings."},
        {"role": "user", "content": task}
    ]
)

# o4-mini para debugging complexo
response = client.chat.completions.create(
    model="o4-mini",
    max_tokens=2000,
    reasoning_effort="medium",  # low | medium | high
    messages=[{"role": "user", "content": f"Encontre e corrija o bug:\n\n{code}"}]
)
```

---

### Análise de documentos longos (contratos, relatórios, bases de código)
**→ GPT-4.1** (`$2.00/$8.00`) com janela de 1M tokens + cache 75% off

O GPT-4.1 foi treinado especificamente para atenção confiável ao longo de toda a janela de 1M tokens. Para documentos longos com system prompt estável, o desconto de cache de 75% torna o custo real muito competitivo.

```python
# Custo real com cache ativo:
# Input cache miss: $2.00/MTok
# Input cache hit:  $0.50/MTok (75% off!)
# Output:           $8.00/MTok

# Estruture sempre: system prompt estável → documentos → query variável
messages = [
    {"role": "system", "content": instrucoes_fixas},   # vai para cache
    {"role": "user", "content": f"{documento_longo}\n\nPergunta: {query}"}
]
```

---

### Raciocínio matemático, científico e lógica complexa
**→ o4-mini** (`$1.10/$4.40`) como padrão, **o3** (`$2.00/$8.00`) para casos críticos

O o4-mini atingiu 99.5% no AIME 2025 com acesso a Python. O o3 é reservado para tarefas onde o o4-mini demonstra limitações mensuráveis, pois o custo por resposta pode ser 5–15× maior com reasoning tokens.

```python
# o4-mini com reasoning_effort controlado
response = client.chat.completions.create(
    model="o4-mini",
    max_tokens=1000,
    reasoning_effort="high",  # use "low" para reduzir reasoning tokens
    messages=[{"role": "user", "content": problema_matematico}]
)
```

---

### Agentes autônomos e orquestração multi-step
**→ GPT-4.1** como executor + **o4-mini** como planejador

O padrão "planner-doer" é recomendado pela própria OpenAI: o modelo de raciocínio define a estratégia e decompõe o problema; o GPT executa cada passo com baixa latência.

```python
# Passo 1: o4-mini planeja
plano = o4mini_call(f"Decomponha em subpassos esta tarefa: {tarefa}")

# Passo 2: GPT-4.1 executa cada passo
resultados = []
for passo in plano.steps:
    resultado = gpt41_call(passo)
    resultados.append(resultado)

# Passo 3: GPT-4.1 mini consolida
resposta_final = gpt41mini_call(f"Consolide: {resultados}")
```

---

### Tarefas em lote (batch, não tempo-real)
**→ Qualquer modelo via Batch API** com desconto automático de 50%

Se não há necessidade de resposta imediata (processamento noturno, análise de datasets, geração de conteúdo em volume), a Batch API entrega exatamente o mesmo resultado com 50% de desconto em todos os modelos.

```python
from openai import OpenAI
client = OpenAI()

# Criar batch com múltiplas requisições
batch = client.batches.create(
    input_file_id=file_id,
    endpoint="/v1/chat/completions",
    completion_window="24h"  # até 24h para processar
)
```

---

## 4. O custo oculto dos modelos de raciocínio (o-series)

Este é o ponto mais crítico para evitar surpresas na fatura.

### Como reasoning tokens funcionam

Quando você chama o3 ou o4-mini, o modelo gera uma cadeia de raciocínio interna antes de produzir o output visível. Essa cadeia é cobrada como output tokens, mas não aparece na resposta.

```
Sua chamada:   [input: 500 tokens]
Raciocínio:    [interno: 8.000 reasoning tokens] ← você paga isso
Sua resposta:  [output: 300 tokens visíveis]

Custo real (o4-mini):
  Input:     500  × $1.10/MTok = $0.00055
  Reasoning: 8000 × $4.40/MTok = $0.0352  ← domina o custo!
  Output:    300  × $4.40/MTok = $0.00132
  Total:                        = $0.0371
```

### Regra prática para detectar supergasto

```python
# Monitore sempre a razão: reasoning_tokens / output_tokens
# Se > 10×, a tarefa pode não precisar de um modelo de raciocínio

usage = response.usage
reasoning_ratio = usage.completion_tokens_details.reasoning_tokens / usage.completion_tokens

if reasoning_ratio > 10:
    print(f"⚠️  Alto reasoning ratio: {reasoning_ratio:.1f}x — considere GPT-4.1")
```

### Controle de reasoning_effort

O parâmetro `reasoning_effort` controla quanto o modelo "pensa" antes de responder:

| Nível | Reasoning tokens | Custo relativo | Usar quando |
|---|---|---|---|
| `low` | ~500–2.000 | 1× | Tarefas moderadamente complexas |
| `medium` | ~2.000–8.000 | 3–5× | Padrão recomendado |
| `high` | ~8.000–32.000 | 10–20× | Problemas críticos, alta precisão necessária |

```python
# Sempre defina reasoning_effort explicitamente
response = client.chat.completions.create(
    model="o4-mini",
    reasoning_effort="low",   # Economize quando possível
    messages=[...]
)
```

### Quando NÃO usar o-series

| Tarefa | Por quê evitar | Usar em vez disso |
|---|---|---|
| Extração de dados | Sem raciocínio necessário | GPT-4.1 nano |
| Tradução de texto | Padrão claro, sem lógica | GPT-4.1 mini |
| Formatação/templates | Nenhuma lógica multi-passo | GPT-4.1 mini |
| Sumarização simples | Texto → resumo, direto | GPT-4.1 mini |
| Classificação | Output fixo, categórico | GPT-4.1 nano |

---

## 5. Otimização específica por modelo

### GPT-4.1 — Maximizando o desconto de cache de 75%

O GPT-4.1 oferece o maior desconto de cache entre todos os modelos OpenAI. A estratégia é estruturar prompts para maximizar cache hits:

```python
# ✅ Estrutura para cache máximo
# REGRA: conteúdo estático primeiro, dinâmico por último

system_prompt = """
Você é um assistente especializado em análise jurídica.
Sua função é extrair cláusulas relevantes e identificar riscos.
[...500 tokens de instruções fixas...]
"""  # → vai para cache automaticamente após 1.024 tokens

# Documentos de referência também podem ser cacheados
contexto_fixo = """
[Lei X, Art. Y...]
[Jurisprudência Z...]
"""  # → concatene com o system para aumentar o bloco cacheável

messages = [
    {"role": "system", "content": system_prompt + contexto_fixo},
    {"role": "user", "content": f"Analise este contrato:\n\n{contrato}"}
    # ↑ única parte variável — tudo acima é cacheado
]

response = client.chat.completions.create(
    model="gpt-4.1",
    messages=messages
)

# Verificar cache hits
print(f"Cache hits: {response.usage.prompt_tokens_details.cached_tokens}")
```

**Cálculo de economia com cache:**
```
Sem cache: 50.000 tokens × $2.00/MTok = $0.10 por chamada
Com cache: 50.000 tokens × $0.50/MTok = $0.025 por chamada  (75% off!)
Em 1.000 chamadas/dia: economia de $75/dia → $2.250/mês
```

---

### GPT-4.1 mini — Volume alto com qualidade confiável

O GPT-4.1 mini supera o GPT-4o em benchmarks de inteligência com 83% menos custo e metade da latência. É o modelo de produção padrão para a maioria das aplicações.

```python
# Técnicas específicas para GPT-4.1 mini

# 1. Controle estrito de output
messages = [{
    "role": "user",
    "content": """Extraia os campos abaixo do e-mail e responda APENAS com JSON válido.
Não inclua explicação ou markdown.

Schema: {"remetente": str, "assunto": str, "acao_necessaria": bool, "prioridade": "alta|media|baixa"}

E-mail: {email_content}"""
}]

# 2. Para pipelines de alto volume: use Batch API (50% off automático)
# 1M tokens/dia × $0.40 = $400 → com Batch API = $200

# 3. Combine few-shot + output estruturado para zero retries
```

---

### GPT-4.1 nano — Classificação e autocompleção em escala

O modelo mais barato com contexto de 1M tokens. Perfeito para guardrails, pré-filtragem de conteúdo e classificação de alta frequência.

```python
# Use para pré-filtrar antes de chamar modelos mais caros
def pre_classificar(texto: str) -> dict:
    """Triagem barata antes de chamar GPT-4.1 para processamento completo."""
    response = client.chat.completions.create(
        model="gpt-4.1-nano",
        max_tokens=30,
        temperature=0,  # saída determinística para classificação
        messages=[{
            "role": "user",
            "content": f"""Classifique e responda só JSON:
{{"tipo": "suporte|vendas|financeiro|outro", "urgente": true|false}}

Mensagem: {texto}"""
        }]
    )
    return json.loads(response.choices[0].message.content)

# Pipeline otimizado:
classificacao = pre_classificar(mensagem)  # $0.10/MTok
if classificacao["urgente"]:
    resposta = gpt41_call(mensagem)        # $2.00/MTok — só quando necessário
else:
    resposta = gpt41mini_call(mensagem)    # $0.40/MTok
```

---

### o4-mini — Raciocínio eficiente com controle de esforço

```python
import openai

client = openai.OpenAI()

def raciocinio_calibrado(problema: str, complexidade: str = "medium") -> str:
    """
    Ajusta reasoning_effort baseado na complexidade estimada da tarefa.
    
    complexidade:
      "simples"  → reasoning_effort="low"   (500-2K reasoning tokens)
      "medio"    → reasoning_effort="medium" (2K-8K reasoning tokens)  
      "complexo" → reasoning_effort="high"   (8K-32K reasoning tokens)
    """
    effort_map = {
        "simples":  "low",
        "medio":    "medium",
        "complexo": "high"
    }
    
    response = client.chat.completions.create(
        model="o4-mini",
        max_tokens=2000,
        reasoning_effort=effort_map.get(complexidade, "medium"),
        messages=[{"role": "user", "content": problema}]
    )
    
    # Log para monitoramento de custo
    rt = response.usage.completion_tokens_details.reasoning_tokens
    ot = response.usage.completion_tokens
    print(f"Reasoning: {rt} tokens | Output: {ot} tokens | Ratio: {rt/ot:.1f}x")
    
    return response.choices[0].message.content

# Exemplos de uso calibrado
codigo_simples = raciocinio_calibrado("Explique o que faz esta função Python: ...", "simples")
bug_complexo = raciocinio_calibrado("Este algoritmo de grafos tem um bug sutil: ...", "complexo")
prova_matematica = raciocinio_calibrado("Prove que sqrt(2) é irracional.", "medio")
```

---

## 6. Descontos e mecanismos de economia

### Prompt Caching — Economia automática

O cache é ativado automaticamente quando o prefixo do prompt excede 1.024 tokens e é idêntico entre chamadas. Não requer configuração — apenas estruturar os prompts corretamente.

| Família | Desconto cache | Input cached $/MTok |
|---|---|---|
| GPT-4.1 / mini / nano | **75% off** | $0.50 / $0.10 / $0.025 |
| GPT-5.x / o-series | **50% off** | Varia por modelo |

**Regras de ouro para cache:**
1. System prompt sempre primeiro e imutável entre chamadas
2. Documentos de referência logo após o system prompt
3. Histórico da conversa no meio
4. Query do usuário sempre por último

```
[CACHEÁVEL — fixo entre chamadas]
  System prompt completo
  Base de conhecimento / documentos de referência
  Few-shot examples
──────────────────────────────────────
[NÃO CACHEADO — varia por chamada]
  Histórico recente da conversa
  Query atual do usuário
```

---

### Batch API — 50% off em todos os modelos

Para qualquer processamento não urgente (até 24h de SLA), a Batch API oferece exatamente o mesmo output com metade do custo.

**Casos de uso ideais:**
- Análise de datasets offline
- Geração de conteúdo em volume
- Extração de dados de documentos
- Avaliação e testes de modelos
- ETL com enriquecimento por LLM

```python
import json
from openai import OpenAI

client = OpenAI()

# 1. Montar arquivo de requisições
requests = []
for i, texto in enumerate(lista_de_textos):
    requests.append({
        "custom_id": f"req-{i}",
        "method": "POST",
        "url": "/v1/chat/completions",
        "body": {
            "model": "gpt-4.1-mini",
            "max_tokens": 200,
            "messages": [
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": texto}
            ]
        }
    })

# 2. Criar arquivo e submeter batch
with open("requests.jsonl", "w") as f:
    for req in requests:
        f.write(json.dumps(req) + "\n")

batch_file = client.files.create(
    file=open("requests.jsonl", "rb"),
    purpose="batch"
)

batch = client.batches.create(
    input_file_id=batch_file.id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)

print(f"Batch ID: {batch.id} | Status: {batch.status}")
# Economia: 50% vs. chamadas síncronas individuais
```

---

### Flex Processing — Desconto adicional para baixa prioridade

Para tarefas que toleram latência variável (horas ou mais), a Flex Processing oferece preços ainda menores que o Batch API em troca de SLA flexível. Ideal para tarefas de background sem prazo definido.

---

## 7. Arquitetura de cascade: roteamento inteligente

A maior alavanca de economia em produção é nunca usar um modelo caro para uma tarefa barata.

```python
import openai
import json
from enum import Enum

client = openai.OpenAI()

class TaskComplexity(Enum):
    TRIVIAL   = "trivial"    # classificação, extração simples
    SIMPLE    = "simple"     # sumarização, tradução, formatação
    MEDIUM    = "medium"     # análise, código simples, Q&A
    COMPLEX   = "complex"    # código avançado, documentos longos
    REASONING = "reasoning"  # matemática, lógica multi-step, debugging

# Mapeamento modelo → custo relativo
MODEL_MAP = {
    TaskComplexity.TRIVIAL:   ("gpt-4.1-nano",  "$0.10/$0.40"),
    TaskComplexity.SIMPLE:    ("gpt-4.1-mini",  "$0.40/$1.60"),
    TaskComplexity.MEDIUM:    ("gpt-4.1",       "$2.00/$8.00"),
    TaskComplexity.COMPLEX:   ("gpt-4.1",       "$2.00/$8.00"),
    TaskComplexity.REASONING: ("o4-mini",       "$1.10/$4.40"),
}

def classify_task(task: str) -> TaskComplexity:
    """Classifica complexidade da tarefa usando o modelo mais barato."""
    response = client.chat.completions.create(
        model="gpt-4.1-nano",  # classificador usa o modelo mais barato
        max_tokens=20,
        temperature=0,
        messages=[{
            "role": "user",
            "content": f"""Classifique a complexidade desta tarefa de IA.
Responda APENAS uma palavra: trivial, simple, medium, complex, ou reasoning.

Tarefa: {task[:200]}"""
        }]
    )
    
    result = response.choices[0].message.content.strip().lower()
    try:
        return TaskComplexity(result)
    except ValueError:
        return TaskComplexity.MEDIUM  # fallback seguro

def smart_call(task: str, force_complexity: TaskComplexity = None) -> str:
    """
    Roteia automaticamente para o modelo ótimo.
    force_complexity permite override manual quando a tarefa é conhecida.
    """
    complexity = force_complexity or classify_task(task)
    model, price = MODEL_MAP[complexity]
    
    # Configuração específica para reasoning models
    extra_kwargs = {}
    if model.startswith("o"):
        extra_kwargs["reasoning_effort"] = "medium"
    
    response = client.chat.completions.create(
        model=model,
        max_tokens=2000,
        messages=[{"role": "user", "content": task}],
        **extra_kwargs
    )
    
    print(f"Modelo: {model} ({price}) | Complexidade: {complexity.value}")
    return response.choices[0].message.content


# Exemplos de uso:
# Override manual para tarefas conhecidas (evita custo de classificação)
resumo = smart_call(
    "Resuma em 3 pontos: " + documento,
    force_complexity=TaskComplexity.SIMPLE  # sem overhead de classificação
)

# Classificação automática para tarefas variáveis
resposta = smart_call(query_do_usuario)
```

### Estimativa de economia do cascade

Assumindo uma distribuição típica de tarefas em produção:

| Distribuição de tarefas | Sem cascade (tudo GPT-4.1) | Com cascade |
|---|---|---|
| 40% triviais | $2.00/MTok × 40% | $0.10/MTok × 40% |
| 30% simples | $2.00/MTok × 30% | $0.40/MTok × 30% |
| 20% médias | $2.00/MTok × 20% | $2.00/MTok × 20% |
| 10% raciocínio | $2.00/MTok × 10% | $1.10/MTok × 10% |
| **Custo médio ponderado** | **$2.00/MTok** | **~$0.61/MTok** |
| **Economia** | — | **~70%** |

---

## 8. Guia de decisão rápida

### Árvore de decisão para escolha de modelo

```
Precisa de resposta em tempo real? (< 2s)
├── NÃO → Batch API (50% off em qualquer modelo abaixo)
└── SIM → continue

Envolve raciocínio multi-step, matemática ou lógica complexa?
├── SIM → o4-mini (reasoning_effort="medium")
│         └── Se não resolver → o3 → o3-pro
└── NÃO → continue

Contexto > 128K tokens ou documentos longos?
├── SIM → GPT-4.1 (1M context, 75% cache discount)
└── NÃO → continue

É geração de código, análise técnica ou instruction-following avançado?
├── SIM → GPT-4.1
└── NÃO → continue

É sumarização, tradução, extração estruturada ou escrita geral?
├── SIM → GPT-4.1 mini (ou GPT-5-mini para tasks menos formais)
└── NÃO → GPT-4.1 nano (classificação, extração simples, autocompleção)
```

---

### Tabela de referência rápida por caso de uso

| Caso de uso | Modelo recomendado | Alternativa econômica |
|---|---|---|
| Chatbot de suporte ao cliente | GPT-4.1 mini | GPT-4.1 nano (triagem) + mini (resposta) |
| Análise de contratos e docs legais | GPT-4.1 | GPT-4.1 + cache 75% |
| Geração de código | GPT-4.1 | o4-mini para debugging |
| Debug e revisão de código | o4-mini | GPT-4.1 (bugs simples) |
| Matemática e provas | o4-mini / o3 | o4-mini (reasoning_effort low) |
| Sumarização em volume | GPT-4.1 mini | Batch API (50% off) |
| Classificação / moderação | GPT-4.1 nano | Fine-tune do nano |
| Agentes autônomos | o4-mini + GPT-4.1 | Planner/doer pattern |
| RAG com docs longos | GPT-4.1 | GPT-4.1 + cache hit 75% |
| Tradução técnica | GPT-4.1 mini | Batch API |
| Análise de imagens e gráficos | GPT-4.1 mini / o4-mini | GPT-4.1 mini |
| Processamento offline em lote | Qualquer via Batch API | — |

---

### Checklist de otimização antes de ir para produção

- [ ] Mapeei cada endpoint da aplicação para o modelo mínimo necessário?
- [ ] System prompt tem mais de 1.024 tokens para ativar cache automaticamente?
- [ ] Conteúdo estático (system, docs de referência) está antes do conteúdo dinâmico?
- [ ] Todos os outputs têm `max_tokens` definido explicitamente?
- [ ] Tarefas não urgentes estão usando Batch API (50% off)?
- [ ] Modelos o-series têm `reasoning_effort` explícito (não default)?
- [ ] Estou monitorando `cached_tokens` no usage para confirmar cache hits?
- [ ] Implementei cascade de modelos para variação de complexidade?
- [ ] Tenho alertas de billing para detectar spikes de custo?
- [ ] Reasoning token ratio está sendo monitorado nos modelos o-series?

---

## Referências

- [OpenAI Models Documentation](https://developers.openai.com/api/docs/models)
- [OpenAI Pricing](https://platform.openai.com/docs/pricing)
- [Reasoning Best Practices — OpenAI](https://developers.openai.com/api/docs/guides/reasoning-best-practices)
- [Prompt Caching Guide — OpenAI](https://developers.openai.com/api/docs/guides/prompt-caching)
- [Batch API Guide — OpenAI](https://developers.openai.com/api/docs/guides/batch)
- [GPT-4.1 Launch Post — OpenAI](https://openai.com/index/gpt-4-1/)
- [o3 and o4-mini Launch Post — OpenAI](https://openai.com/index/introducing-o3-and-o4-mini/)
