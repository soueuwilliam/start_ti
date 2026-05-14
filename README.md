# 🎯 Boas Práticas de IA: Gerenciamento de Tokens e Otimização de Custos

> Guia prático e estratégico para maximizar ROI em aplicações de LLM via API
>
> **Última atualização:** Maio de 2026 | **Público:** Engenheiros, Product Managers, Decision Makers

---

## 📑 Índice

- [Parte 1: Fundamentos de Gerenciamento de Tokens](#parte-1-fundamentos-de-gerenciamento-de-tokens)
- [Parte 2: Estratégias de Redução de Custos](#parte-2-estratégias-de-redução-de-custos)
- [Parte 3: Maximizando Qualidade por Token](#parte-3-maximizando-qualidade-por-token)
- [Parte 4: Seleção e Roteamento de Modelos](#parte-4-seleção-e-roteamento-de-modelos)
- [Parte 5: Arquitetura de Produção](#parte-5-arquitetura-de-produção)
- [Parte 6: Decisões Estratégicas de Modelo](#parte-6-decisões-estratégicas-de-modelo)
- [Referências e Próximos Passos](#referências-e-próximos-passos)

---

---

# PARTE 1: FUNDAMENTOS DE GERENCIAMENTO DE TOKENS

## 1.1 O Que São Tokens?

Um **token** é a unidade de cobrança básica de uma API de LLM. Representa fragmentos de texto:
- **Aproximadamente 4 caracteres** = 1 token em inglês
- **Aproximadamente 2 caracteres** = 1 token em português
- Números, pontuação e espaçamento também contam como tokens

### Anatomia do Custo de uma Chamada API

```
Custo Total = (Input Tokens × Input Price) + (Output Tokens × Output Price) 
              - (Cached Tokens × Cache Discount)
```

**Exemplo Real:**
```
Query: "Resuma este documento de 50K tokens em 500 palavras"

Input:  50.000 tokens @ $0.40/MTok = $20.00
Output: ~2.000 tokens @ $1.60/MTok = $3.20
Total:  $23.20 por chamada
```

### Por Que Gerenciar Tokens?

| Métrica | Impacto |
|---------|---------|
| 30% redução em tokens | -30% custo mensal |
| Melhoria de 20% em qualidade | +20% ROI |
| Latência reduzida em 40% | Melhor UX + margin de capacidade |

---

## 1.2 Dinâmica: Redução × Qualidade

A relação entre custos e qualidade **não é linear**. As melhores estratégias equilibram:

```
     │ Qualidade
     │
100% ├─────────────────── Ótimo (sweet spot)
     │    /              /
     │   / Incremento   /
     │  /   decrescente/
     │ /_______________/ 
   0 └──────────────────── Custo ($)
     
A qualidade melhora dramaticamente nos primeiros 30-40% do investimento.
Depois os ganhos desaceleram. Buscar o equilíbrio é crítico.
```

**Regra de Ouro:** Não economize tokens destruindo qualidade. Otimize para **qualidade por token gasto**.

---

---

# PARTE 2: ESTRATÉGIAS DE REDUÇÃO DE CUSTOS

## 2.1 Compressão Semântica: "Fale Menos, Diga Mais"

### Princípio

Os modelos interpretam **linguagem telegráfica** com a mesma eficácia de linguagem formal. Eliminar redundâncias, artigos e repetição reduz input em **30–50%** com impacto mínimo na qualidade.

### Antes vs. Depois

| Aspecto | Verboso | Comprimido | Tokens |
|---------|---------|-----------|--------|
| **Instruções** | "Você pode me ajudar a escrever um e-mail profissional para o meu cliente informando que o prazo do projeto será adiado em uma semana?" | "E-mail: cliente notificação atraso projeto +1 semana. Tom: profissional." | 35 → 12 |
| **Contexto** | "O documento trata de políticas de segurança da empresa, incluindo protocolos de acesso..." | "Doc: políticas segurança + protocolos acesso." | 20 → 8 |

### Técnicas Práticas

**1. Eliminar artigos desnecessários**
```
❌ "Analise as vantagens e desvantagens desta abordagem com cuidado."
✅ "Vantagens/desvantagens: máx. 3 itens cada, 1 linha por item."
```

**2. Usar listas em vez de prosa**
```
❌ "Gostaria de saber a capacidade de produção, a qualidade dos produtos
   e a reputação no mercado de cada fornecedor."
✅ "Compare fornecedores em: capacidade produção | qualidade | reputação"
```

**3. Pedir formato restrito desde o início**
```
❌ "Resuma este relatório."
✅ "Resuma em JSON: {"pontos_chave": [...], "riscos": [...], "ações": [...]}"
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução input | 30–50% |
| Impacto qualidade | Mínimo (~2–5% piora mensurável) |
| Risco | Baixo |
| Implementação | Imediata |

---

## 2.2 Injeção Mínima de Contexto (RAG Eficiente)

### Princípio

Não envie o documento inteiro. Recupere apenas os chunks relevantes via **Retrieval Augmented Generation (RAG)** e injete apenas o necessário.

### Pipeline Padrão

```
1. Documento grande (1M tokens)
   ↓
2. Criar embeddings (processado offline 1× por doc)
   ↓
3. Query do usuário → similarity search → top-k chunks (ex: top-3)
   ↓
4. API call: [system] + [chunks relevantes] + [query]
   ↓
5. ← Resposta precisa com 70-90% menos tokens
```

### Exemplo Prático

**Cenário:** Base de conhecimento de 500K tokens, query sobre "políticas de férias"

```
SEM RAG:
  Input: 500K tokens + query (2K)
  Custo: 502K × $0.40 = $200.80

COM RAG (top-3 chunks):
  Input: 3K tokens (chunks) + 2K (query) = 5K
  Custo: 5K × $0.40 = $2.00
  
Economia: ~99% em input, ~99% em custo total
```

### Implementação Mínima

```python
import anthropic

# 1. Simulando embedding + retrieval (em produção: usar Pinecone, Weaviate, etc.)
def retrieve_relevant_chunks(query: str, documents: list, top_k: int = 3) -> list:
    """Simples busca por similaridade (em produção usar semantic search)"""
    # Simplificado para demonstração
    return documents[:top_k]

# 2. Chamar API com contexto mínimo
client = anthropic.Anthropic()

query = "Qual é a política de férias?"
documents = [
    "Política de férias: 20 dias por ano, acumuláveis até 30 dias...",
    "Processo de aprovação de férias...",
    "Férias coletivas acontecem em dezembro..."
]

relevant_chunks = retrieve_relevant_chunks(query, documents, top_k=3)
context = "\n".join(relevant_chunks)

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=500,
    messages=[
        {"role": "user", "content": f"Base de conhecimento:\n{context}\n\nPergunta: {query}"}
    ]
)

print(response.content[0].text)
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução contexto | 70–90% |
| Impacto precisão | Positivo (menos ruído) |
| Custo implementação | Médio (requer vector DB) |
| Velocidade de implementação | 1–2 semanas |

---

## 2.3 Controle Explícito do Output: "Pedir Menos, Receber o Exato"

### Princípio

Modelos são prolixos por padrão. Especificar tamanho, formato e restrições no prompt elimina introduções, conclusões genéricas e reformulações desnecessárias: **20–40% menos tokens de output**.

### Antes × Depois

| Requisição | Output | Tokens |
|-----------|--------|--------|
| ❌ "Analise as vantagens desta abordagem." | "Existem várias vantagens interessantes a considerar... Primeira, em termos de eficiência... Segunda... Concluindo, podemos ver que..." | ~400 |
| ✅ "Vantagens (máx 3 itens, 1 frase cada, sem intro/conclusão):" | "1. Eficiência operacional reduzida em 30%\n2. Menor footprint ambiental\n3. Compatibilidade com legado" | ~30 |

### Técnicas de Restrição

```python
# Técnica 1: Especificar número máximo de itens
prompt = "Liste 3 riscos principais. Máx. 1 sentença por risco."

# Técnica 2: Pedir formato específico
prompt = "Responda em JSON: {\"sentimento\": \"positive|negative|neutral\", \"score\": 0-1}"

# Técnica 3: Proibir elementos explicitamente
prompt = "Resuma sem introdução, sem conclusão. Apenas os fatos."

# Técnica 4: Usar máx_tokens agressivamente
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=100,  # Força economia drástica
    messages=[...]
)

# Técnica 5: Pedir modo "direto"
prompt = "Responda diretamente, sem preâmbulo. Apenas a resposta."
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução output | 20–40% |
| Implementação | Imediata, baixo esforço |
| Risco | Muito baixo |

---

## 2.4 Prompt Caching: Reutilizar o Prefixo (Desconto até 90%)

### Princípio

Manter o início do prompt **estável entre chamadas** permite reutilizar o KV-cache do prefixo. Você paga apenas pelos tokens novos: até **90% de desconto** nos tokens cacheados.

### Estrutura Ótima para Cache

```
┌─────────────────────────────────────────────────┐
│  ESTÁTICO (nunca muda entre chamadas)           │ ← CACHEADO
│  • System prompt completo                       │   (desconto 75-90%)
│  • Documentos base, base de conhecimento        │
│  • Instruções globais, persona                  │
├─────────────────────────────────────────────────┤
│  DINÂMICO (muda por requisição)                 │ ← Pago preço cheio
│  • Histórico da conversa atual                  │
│  • Query/contexto do usuário                    │
│  • Dados específicos da requisição              │
└─────────────────────────────────────────────────┘
```

### Exemplo com Cache Ativo

```python
import anthropic

client = anthropic.Anthropic()

# System prompt longo (será cacheado)
SYSTEM_PROMPT_LONGO = """
Você é um especialista jurídico. Suas responsabilidades:
- Analisar documentos com precisão
- Identificar riscos legais
- Sugerir mitigações
[... 5000 tokens de instruções detalhadas, exemplos, etc ...]
"""

# Primeira chamada: cache miss (paga cheio)
response1 = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1000,
    system=[
        {
            "type": "text",
            "text": SYSTEM_PROMPT_LONGO,
            "cache_control": {"type": "ephemeral"}  # ← Ativar cache
        }
    ],
    messages=[
        {"role": "user", "content": "Analise este contrato: [contrato 1]"}
    ]
)
print(f"Tokens utilizados: {response1.usage.input_tokens}")
# Resultado: 5500 tokens × $0.50 = $2.75

# Segunda chamada: cache hit (desconto 75-90%)
response2 = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1000,
    system=[
        {
            "type": "text",
            "text": SYSTEM_PROMPT_LONGO,  # Mesmo prefixo
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {"role": "user", "content": "Analise este outro contrato: [contrato 2]"}
    ]
)
print(f"Cache hit tokens: {response2.usage.cache_read_input_tokens}")
# Resultado: 5000 tokens cacheados × $0.05 (90% off) = $0.25
#           + 500 novos tokens × $0.50 = $0.25
#           Total: $0.50 (vs. $2.75 sem cache = 82% de desconto!)
```

### Cálculo de ROI: Aplicação com 100K chamadas/mês

```
Cenário: System prompt de 2.000 tokens (estático) + 500 tokens de query

SEM CACHE:
  250M tokens × $0.50/MTok = $125/mês

COM CACHE (80% hit rate):
  Cacheado: 160M × $0.05/MTok = $8.00
  Novo:      90M × $0.50/MTok = $45.00
  Total: $53/mês

Economia: 57% ao mês = $72/mês = ~$864/ano
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Desconto no prefixo | 75–90% |
| Melhor para | System prompts > 2K tokens + alta reutilização |
| ROI em escala | Excelente (~50–60% redução de custo total) |
| Implementação | Baixa (1 parâmetro extra) |

---

---

# PARTE 3: MAXIMIZANDO QUALIDADE POR TOKEN

## 3.1 Chain-of-Thought Calibrado: Pensar Custa Menos que Errar

### Princípio

Para tarefas com **raciocínio encadeado**, pedir passos intermediários melhora acurácia em **30–70%**. O custo extra de tokens é recuperado em menos retries e menor taxa de erro em produção.

### Quando Vale a Pena (ROI positivo)

| Tarefa | Acurácia sem CoT | Acurácia com CoT | Ganho |
|--------|-----------------|-----------------|-------|
| Matemática | ~45% | 85% | **+40pp** |
| Lógica multi-step | ~50% | 78% | **+28pp** |
| Raciocínio científico | ~60% | 88% | **+28pp** |
| Debugging de código | ~52% | 75% | **+23pp** |

### Quando NÃO Usar (Desperdício)

```
❌ Classificação binária ("spam ou não spam") → CoT não ajuda
❌ Extração de dados estruturados → output já é formatado
❌ Reformatação de texto → não requer raciocínio
❌ Respostas factuais diretas ("Qual a capital da França?") → overhead
```

### Exemplo Prático

```python
import anthropic

client = anthropic.Anthropic()

# SEM Chain-of-Thought
response_fast = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=100,
    messages=[{
        "role": "user",
        "content": "Um trem viaja a 80km/h. Quantas horas leva para cobrir 320km?"
    }]
)
# Saída: "4 horas" (rápida, mas 15% chance de erro em problemas complexos)

# COM Chain-of-Thought
response_reasoning = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=500,
    messages=[{
        "role": "user",
        "content": """
        Resolva passo a passo, mostrando seu raciocínio:
        Um trem viaja a 80km/h. Quantas horas leva para cobrir 320km?
        
        Passos:
        1. Dados conhecidos:
        2. Fórmula necessária:
        3. Cálculo:
        4. Resposta final:
        """
    }]
)
# Saída: "1. Velocidade = 80km/h, Distância = 320km
#         2. Tempo = Distância / Velocidade
#         3. Tempo = 320 / 80 = 4
#         4. 4 horas"
# (Token extra vale a pena: ~99% de acurácia)
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Melhoria acurácia | 30–70pp |
| Token extra | +200–500 tokens por response |
| ROI | Positivo se taxa de erro > 10% |
| Implementação | Trivial (ajuste prompt) |

---

## 3.2 Few-Shot Examples: Ensinar Padrões Via Exemplos

### Princípio

2–4 exemplos bem escolhidos no system prompt eliminam descrições longas e reduzem erros de formato em **60–80%**. O custo é amortizado ao longo de todas as chamadas.

### Fórmula Ótima

```
SYSTEM PROMPT:
  [Papel + regra central em 1-2 frases]
  
  Exemplo 1 (caso comum):
    Input: ...
    Output: ...
    
  Exemplo 2 (caso edge case):
    Input: ...
    Output: ...

USER MESSAGE:
  [Tarefa real]
```

### Exemplo Real: Classificação de Sentimento

```python
import anthropic

client = anthropic.Anthropic()

system_prompt = """
Você classifica sentimentos em tweets. Responda APENAS JSON.

Exemplo 1:
Input: "Adorei o novo produto! 10/10"
Output: {"sentiment": "positive", "confidence": 0.95}

Exemplo 2:
Input: "Produto não funciona. Péssimo."
Output: {"sentiment": "negative", "confidence": 0.92}

Exemplo 3:
Input: "O produto é ok, nada de especial."
Output: {"sentiment": "neutral", "confidence": 0.78}
"""

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=100,
    system=system_prompt,
    messages=[{
        "role": "user",
        "content": "Estou feliz com a compra, mas o atendimento foi lento."
    }]
)

# Resultado esperado:
# {"sentiment": "mixed", "confidence": 0.85}
# (modelo seguirá padrão dos exemplos com precisão ~95%)
```

### Diretrizes para Escolher Exemplos

| Critério | Detalhe |
|----------|---------|
| **Quantidade** | 2–4 exemplos; mais que 4 é overhead |
| **Variedade** | Cobrir caso comum, edge case, ambíguo |
| **Tamanho** | Mantê-los curtos; o padrão importa, não o volume |
| **Ordem** | Começar com o mais comum, terminar com o mais complexo |

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução erros formato | 60–80% |
| Tokens no system (uma única vez) | +300–800 |
| ROI acumulado | Excelente (amortizado em 100+ chamadas) |

---

## 3.3 Persona de Especialista: Calibrar Tom e Profundidade

### Princípio

Atribuir uma persona técnica específica no system prompt elimina a necessidade de repetir instruções de estilo em cada mensagem. Define tom, vocabulário e nível de detalhe permanentemente.

### Exemplo Comparativo

**SEM Persona (instruções repetidas):**
```python
# Cada chamada:
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1000,
    messages=[{
        "role": "user",
        "content": """
        Responda de forma técnica, sem jargão desnecessário,
        com exemplos práticos de código Python.
        Seu tom deve ser direto e educado.
        
        Pergunta: Como usar decoradores em Python?
        """
    }]
)

# Próxima chamada, repetir tudo de novo...
```

**COM Persona (definida uma única vez):**
```python
system_prompt = """
Você é um engenheiro de software sênior especializado em Python e sistemas distribuídos.
Sua audiência: desenvolvedores intermediários a avançados.
Características:
- Seja direto e pragmático
- Use exemplos de código quando útil
- Evite explicações óbias
- Recomende padrões battle-tested
"""

# Chamadas subsequentes:
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1000,
    system=system_prompt,
    messages=[{
        "role": "user",
        "content": "Como usar decoradores em Python?"
    }]
)

# Tone e profundidade já estão calibrados.
# Próxima chamada, apenas trocar a pergunta.
```

### Tipos de Persona (Exemplos)

```
Persona: "Analista de Negócios"
→ Foco em ROI, métricas, stakeholder impact

Persona: "Engenheiro de Segurança"
→ Foco em threats, mitigação, compliance

Persona: "Jornalista Técnico"
→ Foco em narrativa, contexto, impacto social

Persona: "Assistente Jurídico"
→ Foco em precedentes, risco legal, conformidade
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução instruções repetidas | ~40% menos token por chamada |
| Consistência de tone | +85% |
| Eficiência | Pequena, mas significativa em escala |
| Implementação | Trivial (1 system prompt bem feito) |

---

## 3.4 JSON Mode + Schema Explícito: Estrutura Garante Qualidade

### Princípio

Forçar output estruturado (JSON, XML) reduz tokens de saída, elimina parsing defensivo e previne alucinações em tabelas/dados estruturados.

### Antes × Depois

**Antes (verbose, requer parsing):**
```
"Claro! Aqui está a análise que você pediu. O sentimento identificado 
é positivo, com um score de 0.87, e as categorias principais foram 
'elogio' e 'recomendação do produto'. Acredito que o cliente está 
bastante satisfeito..."
```
**Tokens:** ~120

**Depois (JSON direto):**
```json
{
  "sentiment": "positive",
  "score": 0.87,
  "categories": ["elogio", "recomendação"],
  "confidence": 0.94
}
```
**Tokens:** ~25 | **Redução:** 79%

### Implementação

```python
import anthropic
import json

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1000,
    messages=[{
        "role": "user",
        "content": """
        Analise o sentimento do texto abaixo.
        
        RESPONDA APENAS COM JSON VÁLIDO, SEM TEXTO ADICIONAL, no exato formato:
        {
          "sentiment": "positive" | "negative" | "neutral",
          "score": 0.0 to 1.0,
          "categories": ["categoria1", "categoria2"],
          "summary": "1 sentença"
        }
        
        Texto: "Adorei o produto, superou minhas expectativas!"
        """
    }]
)

# Parse direto (sem try/except defensivo necessário)
result = json.loads(response.content[0].text)
print(result["sentiment"])  # "positive"
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução output | 50–80% |
| Eliminação de parsing defensivo | Sim |
| Risco de erros estruturais | Reduzido em 95% |
| Implementação | Simples (1 linha no prompt) |

---

---

# PARTE 4: SELEÇÃO E ROTEAMENTO DE MODELOS

## 4.1 Panorama de Modelos (Mai/2026)

### Família GPT: "Os Executores" (respostas rápidas, sem raciocínio interno)

| Modelo | Input | Output | Cache | Contexto | Melhor para |
|--------|-------|--------|-------|----------|------------|
| **GPT-5.5** | $2.50 | $15.00 | 50% off | 128K | Raciocínio complexo, agentes |
| **GPT-5** | $1.25 | $10.00 | 50% off | 128K | Tarefas complexas gerais |
| **GPT-5-mini** | $0.25 | $2.00 | 50% off | 128K | Volume médio, texto |
| **GPT-5-nano** | $0.05 | $0.40 | 50% off | 128K | Classificação, extração |
| **GPT-4.1** | $2.00 | $8.00 | **75% off** | **1M** | Código, docs longos |
| **GPT-4.1-mini** | $0.40 | $1.60 | **75% off** | **1M** | Produção econômica |
| **GPT-4.1-nano** | $0.10 | $0.40 | **75% off** | **1M** | Classificação, autocompleção |

### Família o-series: "Os Planejadores" (raciocínio interno profundo)

| Modelo | Input | Output | Cache | Contexto | Observação |
|--------|-------|--------|-------|----------|-----------|
| **o3-pro** | $20.00 | $80.00 | 50% off | 200K | Raciocínio máximo, casos críticos |
| **o3** | $2.00 | $8.00 | 50% off | 200K | Raciocínio avançado |
| **o4-mini** | $1.10 | $4.40 | 50% off | 200K | Melhor custo/raciocínio |

**⚠️ Nota Crítica:** Modelos o-series geram **reasoning tokens invisíveis** que você paga. Uma resposta de 500 tokens pode ter custado 5.000–15.000 tokens de raciocínio interno.

---

## 4.2 Matriz de Decisão: Qual Modelo Escolher?

### Por Tipo de Tarefa

| Tarefa | Modelo | Preço | Razão |
|--------|--------|-------|-------|
| **Classificação simples** | GPT-4.1-nano | $0.10/$0.40 | Mais barato, suficiente |
| **Sumarização, tradução** | GPT-4.1-mini | $0.40/$1.60 | Equilíbrio custo/qualidade |
| **Geração de código** | GPT-4.1 | $2.00/$8.00 | Lidera em SWE-bench |
| **Análise docs longos** | GPT-4.1 + cache | $2.00/$8.00 + 75% desconto | 1M contexto + cache eficiente |
| **Matemática/ciência** | o4-mini | $1.10/$4.40 | 99.5% no AIME 2025 |
| **Agentes autônomos** | o4-mini + GPT-4.1 | $1.10 + $2.00 | Padrão planner-doer |

### Árvore de Decisão Rápida

```
Precisa de resposta em tempo real (<2s)?
├─ NÃO → Usar Batch API (50% desconto)
└─ SIM → continue

Envolve raciocínio multi-step, matemática ou lógica?
├─ SIM → o4-mini (reasoning_effort="medium")
│        Se não resolver → o3 → o3-pro
└─ NÃO → continue

Contexto >128K ou documentos longos?
├─ SIM → GPT-4.1 (1M context + 75% cache)
└─ NÃO → continue

É geração de código ou instruction-following avançado?
├─ SIM → GPT-4.1
└─ NÃO → continue

É classificação ou extração simples?
├─ SIM → GPT-4.1-nano
└─ NÃO → GPT-4.1-mini (padrão versátil)
```

---

## 4.3 Cascade de Modelos: Roteamento Automático por Complexidade

### Princípio

Nunca use um modelo caro para uma tarefa barata. **Classificar tarefas automaticamente** e rotear para o modelo mínimo necessário reduz custo total em **40–70%**.

### Implementação em Python

```python
import anthropic
from enum import Enum

client = anthropic.Anthropic()

class TaskComplexity(Enum):
    TRIVIAL = "trivial"        # classificação, extração simples
    SIMPLE = "simple"          # sumarização, tradução
    MEDIUM = "medium"          # análise, código simples
    COMPLEX = "complex"        # código avançado, docs longos
    REASONING = "reasoning"    # matemática, lógica, debugging

MODEL_MAP = {
    TaskComplexity.TRIVIAL:   ("gpt-4.1-nano", "$0.10/$0.40"),
    TaskComplexity.SIMPLE:    ("gpt-4.1-mini", "$0.40/$1.60"),
    TaskComplexity.MEDIUM:    ("gpt-4.1", "$2.00/$8.00"),
    TaskComplexity.COMPLEX:   ("gpt-4.1", "$2.00/$8.00"),
    TaskComplexity.REASONING: ("o4-mini", "$1.10/$4.40"),
}

def classify_task(task: str) -> TaskComplexity:
    """Classifica complexidade usando o modelo mais barato."""
    response = client.messages.create(
        model="gpt-4.1-nano",  # Classificador usa nano
        max_tokens=10,
        temperature=0,
        messages=[{
            "role": "user",
            "content": f"""Classifique a complexidade (APENAS uma palavra):
trivial, simple, medium, complex, ou reasoning.

Tarefa: {task[:300]}"""
        }]
    )
    
    try:
        classification = response.content[0].text.strip().lower()
        return TaskComplexity(classification)
    except ValueError:
        return TaskComplexity.MEDIUM  # Fallback seguro

def smart_call(task: str, force_complexity: TaskComplexity = None) -> str:
    """Roteia automaticamente para o modelo ótimo."""
    complexity = force_complexity or classify_task(task)
    model, price = MODEL_MAP[complexity]
    
    extra_kwargs = {}
    if model.startswith("o"):
        extra_kwargs["reasoning_effort"] = "medium"
    
    response = client.messages.create(
        model=model,
        max_tokens=2000,
        messages=[{"role": "user", "content": task}],
        **extra_kwargs
    )
    
    print(f"✓ Modelo: {model} ({price}) | Complexidade: {complexity.value}")
    return response.content[0].text

# Uso em produção:
# Override para tarefas conhecidas (evita overhead de classificação)
resumo = smart_call(
    "Resuma este documento...",
    force_complexity=TaskComplexity.SIMPLE
)

# Classificação automática para tarefas variáveis
resposta = smart_call("Pergunta do usuário aqui")
```

### Estimativa de Economia

Assumindo distribuição típica de tarefas em produção:

| Distribuição | Sem Cascade | Com Cascade |
|--------------|------------|------------|
| 40% triviais | $2.00/MTok | $0.10/MTok |
| 30% simples | $2.00/MTok | $0.40/MTok |
| 20% médias | $2.00/MTok | $2.00/MTok |
| 10% raciocínio | $2.00/MTok | $1.10/MTok |
| **Custo médio** | **$2.00/MTok** | **~$0.61/MTok** |
| **Economia** | — | **~70%** |

---

## 4.4 GPT-5 vs GPT-4.1: Quando Migrar?

### Comparação Direta (mai/2026)

| Dimensão | GPT-5-mini | GPT-4.1-mini | Vencedor |
|----------|-----------|-------------|---------|
| **Preço input** | $0.25 | $0.40 | GPT-5-mini ✓ |
| **Preço output** | $2.00 | $1.60 | GPT-4.1-mini ✓ |
| **Desconto cache** | 50% | **75%** | GPT-4.1-mini ✓ |
| **Contexto** | 400K | **1M** | GPT-4.1-mini ✓ |
| **Raciocínio matemático** | **91.1% AIME** | ~52% | GPT-5-mini ✓ |
| **Agentes autônomos** | **69.6% MultiChallenge** | 38.3% | GPT-5-mini ✓ |
| **Precisão factual** | **1.6% erro HealthBench** | ~8% | GPT-5-mini ✓ |

### Cálculo de ROI: Quando GPT-5-mini Vale a Pena

**Cenário A: Geração de Conteúdo (80% output)**
```
1M tokens totais: 200K input + 800K output

GPT-4.1-mini: $0.08 + $1.28 = $1.36
GPT-5-mini:   $0.05 + $1.60 = $1.65

→ GPT-4.1-mini é 21% mais barato. Não migre.
```

**Cenário B: Classificação em Volume (95% input)**
```
1M tokens totais: 950K input + 50K output

GPT-4.1-mini: $0.38 + $0.08 = $0.46
GPT-5-mini:   $0.24 + $0.10 = $0.34

→ GPT-5-mini é 26% mais barato. Considere migrar.
```

**Cenário C: Raciocínio Matemático**
```
Mesma complexidade com GPT-4.1-mini: 5 retries, 80% de acurácia
Com GPT-5-mini: 0 retries, 95% de acurácia

Custo real com retries: $0.34 × 1.8 = $0.61
Custo real com GPT-5: $0.34 (1 chamada, 95% acurácia)

→ GPT-5-mini é 43% mais barato quando fator de erro é considerado.
```

### Checklist de Migração

**Migrar para GPT-5-mini se:**
- ✓ Padrão de uso é dominado por input (>70% tokens de entrada)
- ✓ Raciocínio matemático/científico é crítico
- ✓ Alucinações causam impacto real no negócio
- ✓ Contexto necessário < 400K tokens
- ✓ Taxa de cache não é alta (>70%)

**NÃO migrar se:**
- ✗ Workload gera muito output (>50% tokens de saída)
- ✗ Processa documentos > 400K tokens
- ✗ Cache ativo com >70% hit rate em system prompts longos
- ✗ GPT-4.1-mini já atende qualidade necessária

---

---

# PARTE 5: ARQUITETURA DE PRODUÇÃO

## 5.1 Gestão de Histórico em Conversas Multi-turn

### Problema

Em conversas longas, o histórico acumula tokens exponencialmente:

```
Turnos: 1, 2, 3, 4, ...
Histórico: 500, 1000, 1500, 2000, ... (linear)
Custo: $0.50, $1.00, $1.50, $2.00, ...

Após 100 turnos: ~50K tokens no histórico!
```

### Solução 1: Summarization Progressiva

A cada N turnos, comprimir o histórico antigo em resumo:

```python
def compress_history(messages: list, keep_last_n: int = 5):
    """Comprime histórico antigo, mantém N turnos recentes."""
    if len(messages) <= keep_last_n:
        return messages
    
    old_messages = messages[:-keep_last_n]
    recent_messages = messages[-keep_last_n:]
    
    # Resumir via API (usa modelo barato)
    summary_response = client.messages.create(
        model="gpt-4.1-nano",
        max_tokens=200,
        messages=[{
            "role": "user",
            "content": f"Resuma brevemente esta conversa em 3 pontos-chave:\n{old_messages}"
        }]
    )
    
    summary = summary_response.content[0].text
    
    return [
        {"role": "system", "content": f"Resumo anterior: {summary}"}
    ] + recent_messages

# Uso:
if len(conversation_history) > 20:
    conversation_history = compress_history(conversation_history)
```

**Resultado:** 70–80% menos tokens, contexto semântico mantido.

### Solução 2: Poda de Turnos Irrelevantes

Remover mensagens que não agregam valor:

```python
def prune_irrelevant(messages: list) -> list:
    """Remove mensagens irrelevantes."""
    irrelevant_patterns = [
        "ok", "certo", "entendi", "obrigado",
        "pode repetir", "foi mal", "desculpa"
    ]
    
    pruned = []
    for msg in messages:
        content = msg["content"].lower()
        if not any(p in content for p in irrelevant_patterns):
            pruned.append(msg)
    
    return pruned
```

### Solução 3: State Externalizado

Guardar estado fora da janela de contexto:

```python
# Ao invés de incluir histórico inteiro no prompt:
# Armazenar em banco de dados e injetar apenas delta relevante

conversation_id = "conv_12345"
db.save_state(conversation_id, {
    "user_preferences": {...},
    "previous_decisions": [...],
    "context": {...}
})

# Por chamada, injetar apenas:
current_state = db.load_state(conversation_id)
response = client.messages.create(
    model="...",
    messages=[{
        "role": "user",
        "content": f"Contexto: {current_state}\n\nNova pergunta: {user_query}"
    }]
)
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Redução histórico | 60–80% |
| Latência | Reduzida |
| Custo por chamada | ~40% menor |

---

## 5.2 Decomposição Paralela de Tarefas

### Princípio

Dividir tarefas grandes em sub-tarefas independentes e processar em paralelo:
- Reduz latência total
- Permite usar modelos menores especializados por sub-tarefa

### Exemplo: Análise de Relatório

```python
import asyncio
import anthropic

client = anthropic.AsyncAnthropic()

async def extract_finances(doc: str) -> str:
    """Extrai dados financeiros."""
    response = await client.messages.create(
        model="gpt-4.1-nano",  # Modelo barato para extração
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"Extraia dados financeiros:\n{doc}"
        }]
    )
    return response.content[0].text

async def identify_risks(doc: str) -> str:
    """Identifica riscos."""
    response = await client.messages.create(
        model="gpt-4.1-mini",  # Modelo mais forte para análise
        max_tokens=400,
        messages=[{
            "role": "user",
            "content": f"Identifique riscos principais:\n{doc}"
        }]
    )
    return response.content[0].text

async def summarize_executive(doc: str) -> str:
    """Resuma executivamente."""
    response = await client.messages.create(
        model="gpt-4.1-mini",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"Resuma para executivos:\n{doc}"
        }]
    )
    return response.content[0].text

async def analyze_report(document: str) -> dict:
    """Analisa relatório em paralelo."""
    # Executar 3 tarefas simultaneamente
    finances, risks, summary = await asyncio.gather(
        extract_finances(document),
        identify_risks(document),
        summarize_executive(document)
    )
    
    # Consolidar resultado
    consolidation = await client.messages.create(
        model="gpt-4.1-mini",
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"""
            Consolide estes resultados em um único parecer:
            
            Financeiro: {finances}
            Riscos: {risks}
            Executivo: {summary}
            """
        }]
    )
    
    return {
        "finances": finances,
        "risks": risks,
        "summary": summary,
        "consolidated": consolidation.content[0].text
    }

# Uso:
result = asyncio.run(analyze_report(large_document))
```

### Ganho Estimado

| Métrica | Valor |
|---------|-------|
| Latência total | Reduzida em ~70% (3 tarefas em paralelo vs. sequencial) |
| Custo | Igual ou menor (uso de modelos menores por tarefa) |
| Implementação | Média (requer async/await) |

---

# PARTE 6: DECISÕES ESTRATÉGICAS DE MODELO

## 6.1 Matriz de Seleção Final

### Tabela de Referência Rápida

| Caso de Uso | Modelo | Razão | Economia |
|-------------|--------|-------|----------|
| Chatbot de suporte | GPT-4.1-mini | Equilíbrio qualidade/custo | +30% vs full |
| Análise contratual | GPT-4.1 + cache 75% | 1M contexto + desconto | +60% vs sem cache |
| Geração de código | GPT-4.1 | Lidera em SWE-bench | — |
| Debugging complexo | o4-mini | Raciocínio eficiente | -20% vs o3 |
| Matemática/ciência | o4-mini / o3 | Acurácia crítica | Depende rigor |
| Classificação volume | GPT-4.1-nano + batch | Barato + desconto 50% | +80% vs full |
| Sumarização lote | GPT-4.1-mini + batch | Padrão + desconto 50% | +75% vs full |
| Agentes autônomos | o4-mini (planner) + GPT-4.1 (executor) | Padrão recomendado | -40% vs o3 só |
| RAG com docs longos | GPT-4.1 + cache | Aproveita 1M context + cache | +65% vs GPT-5 |

---

## 6.2 Checklist de Otimização Pré-Produção

Antes de lançar qualquer aplicação com LLM:

- [ ] **Seleção de Modelo**
  - Mapeei cada endpoint para o modelo mínimo necessário?
  - Usei cascade de modelos para roteamento automático?

- [ ] **Gerenciamento de Tokens**
  - Implementei compressão semântica de prompts?
  - Sistema prompt tem >1K tokens para ativar cache?
  - Conteúdo estático está antes do dinâmico?

- [ ] **Controle de Output**
  - Todos os endpoints têm `max_tokens` explícito?
  - Forço JSON/estruturado quando aplicável?

- [ ] **Batch Processing**
  - Identifiquei tarefas não-tempo-real para Batch API (50% off)?

- [ ] **Monitoramento**
  - Tenho alertas de billing para spikes?
  - Estou rastreando cache hit rate?
  - Monitoro ratio de reasoning tokens em modelos o-series?

- [ ] **Testes**
  - Comparei qualidade antes/depois de otimizações?
  - Validei compressão semântica não degrada resultados?
  - Testei cache em produção com padrões de uso real?

---

---

# REFERÊNCIAS

## 📚 Referências Técnicas

### Documentação Oficial

- [Anthropic Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Anthropic Prompt Caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [OpenAI Pricing](https://platform.openai.com/docs/pricing)
- [OpenAI Reasoning Models](https://platform.openai.com/docs/guides/reasoning)

### Papers e Pesquisas

- Wei et al. (2022) — *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*
- Brown et al. (2020) — *Language Models are Few-Shot Learners*
- Anthropic (2024) — *Prompt Caching Reduces Latency and Cost*

### Ferramentas Recomendadas

- **Vector DBs para RAG:** Pinecone, Weaviate, Milvus, Qdrant
- **Monitoramento:** Datadog, New Relic, Langfuse
- **A/B Testing:** VWO, Optimizely, LaunchDarkly

---
