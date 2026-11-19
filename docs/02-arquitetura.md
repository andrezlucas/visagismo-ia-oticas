# 02. Arquitetura do Sistema, Contratos e Observabilidade

Este documento detalha a arquitetura técnica híbrida, o fluxo de processamento, os contratos de dados e a instrumentação de observabilidade com **Langfuse**.

---

## 1. Visão Geral da Arquitetura Híbrida

Para equilibrar custo operacional, velocidade de resposta (meta `< 8s` no p95) e qualidade consultiva, o sistema adota uma arquitetura em quatro estágios:

```text
+-----------------------------------------------------------------------+
|                     FRONTEND (Next.js / Mobile-First)                 |
|        [1. Consentimento] -> [2. Selfie Frontal] -> [3. Quiz]         |
+-----------------------------------+-----------------------------------+
                                    | HTTPS (Multipart / Base64 em RAM)
                                    v
+-----------------------------------------------------------------------+
|                     BACKEND API (Python + FastAPI)                    |
|                                                                       |
|  +-----------------------------------------------------------------+  |
|  | Estágio 1: Pré-Validação Local (MediaPipe Face Detection)       |  |
|  | - Verifica presença de 1 rosto, ângulo e iluminação             |  |
|  | - Rejeita fotos inválidas sem gastar tokens de IA               |  |
|  +--------------------------------+--------------------------------+  |
|                                   | Imagem Válida                     |
|                                   v                                   |
|  +-----------------------------------------------------------------+  |
|  | Estágio 2: Extração Facial Estruturada (VLM Multimodal)         |  |
|  | - Gemini 2.5 Flash / GPT-4o com Structured Output (JSON Schema) |  |
|  | - Descarta a imagem da memória RAM imediatamente após leitura   |  |
|  +--------------------------------+--------------------------------+  |
|                                   | JSON de Atributos Faciais         |
|                                   v                                   |
|  +-----------------------------------------------------------------+  |
|  | Estágio 3: Motor de Matching Determinístico (PostgreSQL)        |  |
|  | - Filtra estoque ativo da ótica por regras ópticas e visagismo  |  |
|  | - Ranqueia e seleciona os Top 3 SKUs                            |  |
|  +--------------------------------+--------------------------------+  |
|                                   | Top 3 Armações Reais              |
|                                   v                                   |
|  +-----------------------------------------------------------------+  |
|  | Estágio 4: Gerador de Argumento Consultivo (LLM)                |  |
|  | - Redige explicação personalizada conectando rosto + armação    |  |
|  +-----------------------------------------------------------------+  |
+-----------------------------------+-----------------------------------+
                                    |
          +-------------------------+-------------------------+
          |                                                   |
          v                                                   v
+-------------------+                               +-------------------+
|    PostgreSQL     |                               |     Langfuse      |
| (Catálogo & Logs) |                               | (Traces & Costs)  |
+-------------------+                               +-------------------+
```

---

## 2. Decisão Arquitetural: GenAI vs. Modelo de Classificação

| Critério | Modelo de Classificação Dedicado | GenAI Multimodal (Structured Output) | Decisão Adotada |
| :--- | :--- | :--- | :--- |
| **Tempo para MVP** | Alto (exige coleta, rotulagem de milhares de fotos e treinamento). | Imediato (via engenharia de prompt e validação de esquema). | **GenAI Multimodal** para análise facial e consultoria. |
| **Custo por Requisição** | Baixo em escala massiva, mas exige servidor com GPU/CPU dedicada. | Pay-per-use (fração de centavo por imagem analisada). | **MediaPipe local** na entrada para bloquear fotos inválidas a custo zero. |
| **Explicabilidade** | Entrega apenas rótulo seco (ex.: `rosto_redondo`). | Explica a harmonia entre sobrancelhas, ponte nasal e maxilar. | **Arquitetura Híbrida** (IA interpreta + Banco filtra estoque real). |

---

## 3. Contratos de Dados (JSON Schemas)

### 3.1. Saída Estruturada da Análise Facial (Estágio 2)
O modelo multimodal é forçado a responder estritamente neste formato via `response_schema` (Pydantic):

```json
{
  "formato_rosto": "quadrado",
  "largura_aparente": "media",
  "altura_ponte_nasal": "media",
  "subtom_pele": "quente",
  "usa_oculos_escuros": false,
  "observacoes_visagismo": "Linha do maxilar bem definida e testa proporcional, favorecendo armações com curvas suaves."
}
```

### 3.2. Resposta Final da API (`POST /api/v1/recomendar`)

```json
{
  "trace_id": "lf-trace-98a7bc65",
  "perfil_facial": {
    "formato_rosto": "quadrado",
    "resumo_consultoria": "Traços marcantes que ganham equilíbrio e leveza com linhas arredondadas ou ovais."
  },
  "recomendacoes": [
    {
      "sku": "OTICA-042",
      "nome": "Urban Round Acetato Tartaruga",
      "marca": "Essence Eyewear",
      "preco": 489.90,
      "imagem_url": "https://cdn.loja.com/armacao-042.jpg",
      "match_percentual": 96,
      "justificativa": "O aro arredondado suaviza os ângulos do seu maxilar, enquanto o tom tartaruga harmoniza com seu subtom de pele quente."
    }
  ]
}
```

---

## 4. Modelagem Inicial do Banco de Dados (PostgreSQL)

```sql
CREATE TABLE lojas (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nome VARCHAR(120) NOT NULL,
    slug VARCHAR(80) UNIQUE NOT NULL,
    ativo BOOLEAN DEFAULT TRUE,
    criado_em TIMESTAMP DEFAULT NOW()
);

CREATE TABLE armacoes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    loja_id UUID REFERENCES lojas(id) ON DELETE CASCADE,
    sku VARCHAR(60) NOT NULL,
    nome VARCHAR(150) NOT NULL,
    marca VARCHAR(80),
    categoria VARCHAR(20) CHECK (categoria IN ('grau', 'solar')),
    formato_armacao VARCHAR(30) NOT NULL, -- redonda, quadrada, retangular, aviador, gatinho, oval
    material VARCHAR(30) NOT NULL,        -- acetato, metal, titânio, misto
    estilo VARCHAR(30) NOT NULL,          -- classico, moderno, discreto, ousado
    altura_aro_mm INTEGER,
    suporta_grau_alto BOOLEAN DEFAULT TRUE,
    preco NUMERIC(10, 2) NOT NULL,
    estoque INTEGER NOT NULL DEFAULT 0,
    imagem_url TEXT NOT NULL,
    ativo BOOLEAN DEFAULT TRUE
);

CREATE TABLE recomendacoes_metricas (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    loja_id UUID REFERENCES lojas(id),
    trace_id VARCHAR(100) UNIQUE NOT NULL,
    formato_rosto_detectado VARCHAR(30),
    faixa_grau VARCHAR(30),
    feedback_usuario SMALLINT, -- 1 (Gostei), 0 (Não curti), NULL (Sem voto)
    criado_em TIMESTAMP DEFAULT NOW()
);
```

---

## 5. Observabilidade e Gestão de Prompts com Langfuse

Toda requisição ao endpoint de recomendação abre um **Trace** no Langfuse para garantir visibilidade técnica e financeira:

1. **Trace Principal (`consultoria_visagismo`):** Identificado por `loja_id` e `session_id`.
2. **Span `validacao_mediapipe`:** Mede latência da pré-checagem e contabiliza taxa de rejeição por qualidade de imagem.
3. **Generation `extracao_facial_vlm`:** Registra versão do prompt (gerenciado via **Langfuse Prompt Management**), tokens de entrada/saída, custo estimado em USD e latência do provedor.
4. **Span `matching_estoque`:** Registra quantos itens do catálogo passaram pelos filtros técnicos e quais SKUs foram selecionados.
5. **Generation `geracao_justificativa`:** Registra o texto consultivo final gerado para o cliente.
6. **Coleta de Feedback (`POST /api/v1/feedback`):** Quando o cliente clica em "Gostei" ou "Não curti" na interface, o backend envia `langfuse.score(trace_id=trace_id, name="user_satisfaction", value=1)` para correlacionar prompts e modelos com a aprovação real do usuário.

---

## 6. System Prompt Base (Extração Facial)

```text
Você é um especialista sênior em visagismo óptico e consultoria de estilo facial.
Analise a imagem frontal enviada pelo cliente e extraia EXCLUSIVAMENTE os atributos geométricos solicitados no esquema JSON.

Diretrizes obrigatórias:
1. Baseie-se apenas nas proporções visíveis na imagem (largura da testa, maçãs do rosto, linha do maxilar e altura da ponte nasal).
2. Não faça inferências sobre idade, gênero, etnia ou dados pessoais sensíveis.
3. Caso a iluminação dificulte a leitura do subtom de pele, retorne "neutro".
4. Se a pessoa estiver usando óculos escuros que cubram os olhos, marque "usa_oculos_escuros": true.
5. Mantenha o campo "observacoes_visagismo" técnico, acolhedor e objetivo, em no máximo 2 frases curtas.
```
