# Engenharia de Prompt: Otimização de Tokens para APIs de LLM

> Técnicas validadas para reduzir custos e maximizar qualidade por token em chamadas via API.

---

## Sumário

1. [Estratégias de Redução de Tokens](#1-estratégias-de-redução-de-tokens)
2. [Estratégias de Qualidade por Token](#2-estratégias-de-qualidade-por-token)
3. [Estrutura de Prompt Eficiente](#3-estrutura-de-prompt-eficiente)
4. [Técnicas Avançadas de Arquitetura](#4-técnicas-avançadas-de-arquitetura)
5. [Guia de Decisão Rápida](#5-guia-de-decisão-rápida)

---

## 1. Estratégias de Redução de Tokens

### 1.1 Compressão Semântica de Instrução

**Objetivo:** Reduzir tokens de input sem perda de qualidade no output.

O modelo interpreta linguagem telegráfica com a mesma eficácia de linguagem coloquial. Eliminar redundâncias, artigos desnecessários e repetição de contexto pode cortar **30–50% dos tokens de input**.

| Métrica | Valor |
|---|---|
| Redução de tokens | ~48% |
| Impacto na qualidade | Mínimo |
| Risco | Baixo |

**Antes (verboso — ~35 tokens):**
```
Você pode me ajudar a escrever um e-mail profissional para o meu cliente
informando que o prazo do projeto será adiado em uma semana?
```

**Depois (comprimido — ~18 tokens):**
```
E-mail profissional para cliente: prazo do projeto adiado 1 semana.
```

---

### 1.2 Injeção de Contexto Mínimo Necessário (RAG)

**Objetivo:** Enviar apenas o trecho relevante do documento, não o arquivo inteiro.

Usar chunk retrieval (RAG) para extrair parágrafos pertinentes antes de chamar a API reduz o contexto em **70–90%** e, em geral, melhora a precisão da resposta por eliminar ruído.

**Pipeline recomendado:**
```
1. Documento → embeddings (processado offline)
2. Query do usuário → similarity search → top-k chunks
3. API call: system prompt + chunks relevantes + query
4. ← resposta focada e precisa
```

| Métrica | Valor |
|---|---|
| Redução de contexto | 70–90% |
| Impacto na precisão | Positivo |
| Custo de implementação | Médio |

---

### 1.3 Controle Explícito do Output

**Objetivo:** Reduzir tokens de saída especificando formato, tamanho e estrutura.

O modelo tende a ser prolixo sem restrições. Especificar o output esperado elimina introduções, conclusões e reformulações desnecessárias.

**Antes (sem restrição):**
```
Analise as vantagens e desvantagens desta abordagem.
```

**Depois (com restrição):**
```
Liste prós/contras: máx. 3 itens cada, 1 frase por item. Sem introdução.
```

**Técnicas de controle:**
- Especificar número máximo de itens ou palavras
- Pedir formato específico (lista, tabela, JSON)
- Proibir explicitamente introduções/conclusões genéricas
- Usar frases como "Responda diretamente, sem preâmbulo"

---

## 2. Estratégias de Qualidade por Token

### 2.1 Chain-of-Thought Calibrado

**Objetivo:** Aumentar drasticamente a acurácia em tarefas complexas, gastando mais tokens de forma justificada.

Para tarefas que envolvem raciocínio encadeado, pedir passos intermediários melhora a acurácia em 30–70%. O custo extra de tokens é recuperado em retries e correções evitados.

**Quando usar:**
- Raciocínio lógico ou matemático
- Múltiplos passos dependentes entre si
- Diagnóstico e debugging
- Situações onde alta precisão é crítica

**Quando evitar (desperdício de tokens):**
- Extração simples de dados
- Classificação em categorias fixas
- Reformatação de texto
- Respostas factuais diretas

**Exemplo de ativação:**
```
Resolva passo a passo, mostrando seu raciocínio antes da resposta final.
```

---

### 2.2 Few-shot Exemplos Calibrados

**Objetivo:** Substituir instruções longas por exemplos que ensinam o padrão por indução.

2–4 exemplos bem escolhidos eliminam a necessidade de descrever o formato em texto e reduzem erros de output em **60–80%**. O custo dos exemplos é único no system prompt e amortizado ao longo das chamadas.

**Fórmula ótima:**
```
system: papel + regra central (1-2 frases)
---
user: [Exemplo de input 1]
assistant: [Output ideal 1]

user: [Exemplo de input 2]
assistant: [Output ideal 2]

user: [Tarefa real]
```

| Métrica | Valor |
|---|---|
| Redução de erros de formato | 60–80% |
| Número ideal de exemplos | 2–4 |
| Custo | +tokens no input (custo único no system) |

**Dicas:**
- Escolher exemplos que cobrem casos-borda, não apenas o caso ideal
- Variar os exemplos para evitar overfitting de estilo
- Manter exemplos curtos — o padrão importa, não o volume

---

### 2.3 Persona de Especialista

**Objetivo:** Calibrar vocabulário, nível de detalhe e tom sem instruções repetitivas por chamada.

Atribuir uma persona técnica específica no system prompt elimina a necessidade de repetir instruções de estilo em cada mensagem do usuário.

**Antes (instruções repetitivas em cada chamada):**
```
Responda de forma técnica, sem jargão desnecessário,
com exemplos práticos de código Python...
```

**Depois (persona definida uma vez no system):**
```
Você é um engenheiro de software sênior especializado em Python
e sistemas distribuídos. Seja direto e use exemplos de código quando útil.
```

---

## 3. Estrutura de Prompt Eficiente

### 3.1 Anatomia de um Prompt Otimizado

A ordem das seções impacta tanto o custo (tokens processados) quanto a qualidade de atenção do modelo. A estrutura recomendada:

```
┌─────────────────────────────────────────┐
│  1. SYSTEM PROMPT                       │
│     Papel + restrições globais          │
│     → Reutilizado em todas as chamadas  │
├─────────────────────────────────────────┤
│  2. CONTEXTO MÍNIMO                     │
│     Apenas dados que a tarefa precisa   │
│     → Purgar histórico irrelevante      │
├─────────────────────────────────────────┤
│  3. TAREFA ESPECÍFICA                   │
│     Verbo de ação + objeto + constraints│
│     → Sem ambiguidade = sem retries     │
├─────────────────────────────────────────┤
│  4. FORMATO DO OUTPUT (quando necessário)│
│     JSON schema, markdown, lista, etc.  │
│     → Evita pós-processamento           │
└─────────────────────────────────────────┘
```

---

### 3.2 Gestão de Histórico em Conversas Multi-turn

Em conversas longas, o histórico acumula tokens rapidamente. Três técnicas para controlar isso:

#### Summarization Progressiva
A cada N turnos, comprimir o histórico antigo em um resumo e substituir as mensagens originais. Mantém contexto semântico com **70–80% menos tokens**.

```python
def compress_history(messages, keep_last_n=4):
    if len(messages) <= keep_last_n:
        return messages
    
    old_messages = messages[:-keep_last_n]
    recent_messages = messages[-keep_last_n:]
    
    # Resumir mensagens antigas via API
    summary = summarize_via_api(old_messages)
    
    return [{"role": "system", "content": f"Resumo da conversa anterior: {summary}"}] + recent_messages
```

#### Poda de Turnos Irrelevantes
Remover mensagens que não carregam informação nova:
- Confirmações ("ok, entendi", "certo")
- Tentativas com erro já corrigido
- Ajustes menores de formatação

#### State Externalizado
Guardar o estado da aplicação fora da janela de contexto (banco de dados, variáveis de sessão) e injetar apenas o delta relevante por chamada.

---

## 4. Técnicas Avançadas de Arquitetura

### 4.1 Cascade de Modelos por Complexidade

**Objetivo:** Rotear cada tarefa para o modelo mais barato capaz de resolvê-la.

Classificar a tarefa antes de enviar para a API e usar modelos menores para tarefas simples pode reduzir o custo total em **40–70%** sem degradação perceptível de qualidade.

```
classify(task) → "simple" | "complex" | "reasoning"
  simple    → Haiku  (custo: 1×)
  complex   → Sonnet (custo: ~5×)
  reasoning → Opus   (custo: ~15×)
```

**Exemplo de implementação em Python:**
```python
import anthropic

client = anthropic.Anthropic()

def classify_complexity(task: str) -> str:
    response = client.messages.create(
        model="claude-haiku-4-5-20251001",  # classificador barato
        max_tokens=10,
        messages=[{
            "role": "user",
            "content": f"Classifique: '{task}'. Responda apenas: simple, complex ou reasoning."
        }]
    )
    return response.content[0].text.strip().lower()

def smart_call(task: str) -> str:
    complexity = classify_complexity(task)
    
    model_map = {
        "simple":    "claude-haiku-4-5-20251001",
        "complex":   "claude-sonnet-4-6",
        "reasoning": "claude-opus-4-6",
    }
    
    model = model_map.get(complexity, "claude-sonnet-4-6")
    
    response = client.messages.create(
        model=model,
        max_tokens=1000,
        messages=[{"role": "user", "content": task}]
    )
    return response.content[0].text
```

| Métrica | Valor |
|---|---|
| Redução de custo total | 40–70% |
| Impacto na qualidade percebida | Mínimo |
| Custo de implementação | Alto |

---

### 4.2 Prompt Caching (Prefix Cache)

**Objetivo:** Reutilizar o KV-cache do prefixo entre chamadas, pagando apenas pelos tokens novos.

Manter o início do prompt estável entre chamadas permite até **90% de desconto** no preço de tokens cacheados (conforme política da Anthropic para `cache_control`).

**Estrutura para cache máximo:**
```
[ESTÁTICO — nunca muda entre chamadas]
  System prompt completo
  Documentos base / base de conhecimento
  ↑ cache hit aqui = desconto significativo no preço
─────────────────────────────────────────────────
[DINÂMICO — muda por request]
  Histórico da conversa atual
  Query do usuário
```

**Exemplo com `cache_control` (API Anthropic):**
```python
import anthropic

client = anthropic.Anthropic()

SYSTEM_PROMPT_LONGO = """
[Seu system prompt extenso com documentos e instruções...]
"""

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1000,
    system=[
        {
            "type": "text",
            "text": SYSTEM_PROMPT_LONGO,
            "cache_control": {"type": "ephemeral"}  # cachear este prefixo
        }
    ],
    messages=[
        {"role": "user", "content": "Query dinâmica do usuário aqui"}
    ]
)
```

---

### 4.3 Decomposição Paralela de Tarefas

**Objetivo:** Dividir tarefas grandes em sub-tarefas independentes e processar em paralelo.

Reduz latência total e permite usar modelos menores especializados para cada sub-tarefa em vez de um modelo grande genérico para tudo.

```python
import asyncio
import anthropic

client = anthropic.AsyncAnthropic()

async def call_api(task: str, model: str = "claude-haiku-4-5-20251001") -> str:
    response = await client.messages.create(
        model=model,
        max_tokens=500,
        messages=[{"role": "user", "content": task}]
    )
    return response.content[0].text

async def process_parallel(subtasks: list[str]) -> list[str]:
    results = await asyncio.gather(*[call_api(task) for task in subtasks])
    return list(results)

# Exemplo de uso
subtasks = [
    "Extraia os dados financeiros deste relatório: [...]",
    "Identifique os riscos mencionados neste relatório: [...]",
    "Resuma as conclusões executivas deste relatório: [...]",
]

results = asyncio.run(process_parallel(subtasks))
final_output = merge_results(results)
```

---

### 4.4 JSON Mode + Schema Explícito

**Objetivo:** Eliminar texto explicativo no output e garantir parseabilidade sem retries.

Forçar output estruturado reduz tokens de saída e elimina a necessidade de parsing defensivo.

**Antes (output verboso):**
```
Claro! Aqui está a análise que você pediu. O sentimento identificado
é positivo, com score de 0.87, e as categorias identificadas foram
"elogio" e "produto"...
```

**Depois (JSON direto):**
```json
{"sentiment": "positive", "score": 0.87, "categories": ["elogio", "produto"]}
```

**Prompt para forçar JSON:**
```
Analise o sentimento do texto abaixo.
Responda APENAS com JSON válido, sem texto adicional, no formato:
{"sentiment": "positive|negative|neutral", "score": 0.0-1.0, "categories": [...]}

Texto: [...]
```

---

## 5. Guia de Decisão Rápida

### Quando o objetivo é reduzir custo direto:

| Técnica | Ganho estimado | Esforço | Melhor para |
|---|---|---|---|
| Compressão semântica | 30–50% input | Baixo | Qualquer prompt |
| RAG com chunk retrieval | 70–90% context | Médio | Apps com documentos |
| Prompt caching | até 90% prefix | Médio | System prompts longos |
| Controle de output | 20–40% output | Baixo | Respostas estruturadas |

### Quando o objetivo é maximizar qualidade por token:

| Técnica | Ganho de qualidade | Custo extra | Melhor para |
|---|---|---|---|
| Few-shot (2–4 exemplos) | 60–80% menos erros formato | +tokens no system | Formatação consistente |
| Chain-of-Thought | 30–70% mais acurácia | +tokens no output | Raciocínio complexo |
| Persona especialista | Tom/profundidade calibrados | Mínimo | Todas as aplicações |
| JSON mode | Eliminação de retries | Nenhum | Outputs estruturados |

### Estratégias de arquitetura (maior retorno, maior esforço):

| Técnica | Ganho estimado | Esforço | Melhor para |
|---|---|---|---|
| Cascade de modelos | 40–70% custo total | Alto | Aplicações em produção |
| Decomposição paralela | Redução de latência + custo | Alto | Tarefas decomponíveis |
| Summarization progressiva | 70–80% tokens de histórico | Médio | Chatbots e agentes |

---

## Referências

- [Anthropic Prompt Engineering Docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- Wei et al. (2022) — *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*
- Brown et al. (2020) — *Language Models are Few-Shot Learners*
