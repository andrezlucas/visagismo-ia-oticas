# Visagismo IA: Consultor Óptico Inteligente (SaaS B2B)

Plataforma SaaS B2B de consultoria óptica automatizada que analisa a geometria facial do cliente por meio de uma selfie guiada, cruza os traços com restrições técnicas de lentes e recomenda até **3 armações reais do estoque da ótica** em menos de 8 segundos.

---

## 1. O Problema e a Proposta de Valor

* **A dor do mercado:** Clientes sentem insegurança ao escolher óculos sozinhos, o que gera abandono no e-commerce e atendimento consultivo demorado no balcão da loja física. Soluções tradicionais de provador virtual 3D possuem custo elevado e implementação complexa para óticas independentes e redes regionais.
* **Nossa solução:** Um fluxo web leve e mobile-first (acessível por QR Code na vitrine/balcão, link no WhatsApp ou widget no site) que atua como um consultor de visagismo digital. O sistema identifica o formato do rosto, aplica regras ópticas reais (como espessura de lente por grau) e apresenta opções disponíveis no estoque da loja com justificativa personalizada.

---

## 2. Modelo de Negócio (Lean Canvas)

| Bloco | Definição |
| :--- | :--- |
| **Segmento de Clientes** | Óticas independentes, redes regionais e e-commerces ópticos. |
| **Proposta de Valor** | Aumento da taxa de conversão e do ticket médio com recomendação assertiva baseada em visagismo e estoque real, sem necessidade de hardware dedicado. |
| **Canais** | QR Code na loja física, atendimento via WhatsApp Business e integração em e-commerce. |
| **Fontes de Receita** | Assinatura mensal SaaS por loja (planos por faixa de análises/mês) + taxa opcional de setup para catalogação inicial das armações. |
| **Estrutura de Custos** | Consumo de API de IA multimodal (tokens), hospedagem em nuvem, observabilidade (Langfuse) e aquisição comercial. |
| **Métricas-Chave (KPIs)** | Taxa de conclusão da captura de foto, taxa de clique nas armações sugeridas, conversão em venda/reserva e custo médio por análise. |

---

## 3. Decisões Estratégicas de Produto e Arquitetura

1. **UX sem atrito (1 foto + 3 perguntas):** Em vez de exigir fotos de múltiplos ângulos (o que eleva a desistência no funil), o fluxo utiliza **1 selfie frontal guiada** (com foto lateral opcional) combinada a **3 perguntas rápidas** (estilo, faixa de grau e material preferido).
2. **Arquitetura Híbrida (Anti-Alucinação):** Combina pré-validação facial determinística (**MediaPipe**), análise visual estruturada via GenAI Multimodal (**Gemini 2.5 Flash / GPT-4o com Structured Output**) e filtragem rígida em banco de dados (**PostgreSQL**). A IA nunca inventa modelos fora do estoque.
3. **Privacy by Design (LGPD):** A foto do cliente é processada exclusivamente em memória temporária (RAM) para extração dos metadados geométricos e descartada imediatamente após a resposta.
4. **Observabilidade com Langfuse:** Rastreamento completo de latência, custo por requisição, versionamento de prompts e coleta de feedback explícito do usuário (`Gostei` / `Não curti`).

---

## 4. Índice da Documentação Técnica (SDD)

Toda a especificação funcional, arquitetural e o plano de execução estão organizados na pasta `docs/`:

* **[01. Regras de Negócio e Visagismo](docs/01-regras-de-negocio.md):** Matriz de compatibilidade facial, restrições por grau de lente, regras de estoque e conformidade LGPD.
* **[02. Arquitetura do Sistema e Observabilidade](docs/02-arquitetura.md):** Diagrama técnico, fluxo de dados, contratos JSON, schema SQL, prompts e instrumentação com Langfuse.
* **[03. Conceito de Design e Wireframes](docs/03-conceito-design.md):** Princípios de UX, Design System (paleta, tipografia, componentes) e wireframes das 4 telas principais.
* **[04. Plano de Desenvolvimento e Tasks](docs/04-tasks-desenvolvimento.md):** Quebra granular em tarefas técnicas priorizadas para o **MVP de 7 dias** (foco em validação e primeira venda) e backlog da V2.

---

## 5. Stack Tecnológica

* **Frontend:** Next.js (React) + Tailwind CSS (interface responsiva mobile-first).
* **Backend:** Python + FastAPI + Pydantic (validação estrita de contratos).
* **Visão Computacional e IA:** MediaPipe Face Detection (pré-filtro local) + VLM Multimodal com JSON Schema (extração facial e justificativa consultiva).
* **Banco de Dados:** PostgreSQL (catálogo de armações, regras de compatibilidade e métricas anônimas).
* **Observabilidade e Evals:** Langfuse SDK (tracing, gestão de prompts, controle de custos e scores de aprovação).
