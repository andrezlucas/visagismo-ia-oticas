# 04. Plano de Desenvolvimento e Quebra em Tasks

Para viabilizar a validação comercial e a **primeira venda piloto em 7 dias**, o desenvolvimento foi dividido em **5 Épicos de MVP** (foco em colocar o fluxo ponta a ponta no ar com catálogo real de uma primeira ótica) e um **Backlog V2** para escala pós-venda.

---

## Fase 1: MVP Relâmpago (7 Dias para Primeira Venda)

### Épico 1: Fundação, Repositório e Catálogo Piloto (Dia 1)
* **[TASK-01]** Estruturar repositório GitHub com `.gitignore`, variáveis de ambiente (`.env.example`) e organização `/backend` e `/frontend`.
* **[TASK-02]** Modelar e criar tabelas iniciais no PostgreSQL (ou SQLite/Supabase para deploy rápido do MVP): `lojas`, `armacoes` e `recomendacoes_metricas`.
* **[TASK-03]** Cadastrar catálogo piloto real com **20 a 30 armações** de uma ótica parceira (fotos, preços, SKU, formato, material, estilo e suporte a grau alto).

### Épico 2: Pipeline de Visão e Inteligência Artificial (Dias 2 e 3)
* **[TASK-04]** Implementar pré-validação de imagem com **MediaPipe Face Detection** para verificar presença de 1 rosto frontal e inclinação válida antes de acionar a API paga.
* **[TASK-05]** Configurar cliente da API Multimodal (**Gemini 2.5 Flash** ou **GPT-4o**) com `Structured Output` (Pydantic schema) para extrair formato de rosto, proporções e subtom de pele.
* **[TASK-06]** Garantir descarte imediato do buffer de imagem da memória RAM logo após a extração do JSON facial.

### Épico 3: Motor de Regras de Visagismo e API FastAPI (Dia 4)
* **[TASK-07]** Codificar a matriz de regras de visagismo (formato de rosto x formato de armação) e os filtros técnicos de grau de lente.
* **[TASK-08]** Implementar o algoritmo de pontuação (`Match %`) que filtra apenas itens com `estoque > 0` da loja e seleciona o Top 3.
* **[TASK-09]** Criar camada de geração da justificativa consultiva personalizada para cada uma das 3 armações retornadas.
* **[TASK-10]** Expor endpoints REST em FastAPI: `POST /api/v1/recomendar`, `GET /api/v1/catalogo/{loja_slug}` e `POST /api/v1/feedback`.

### Épico 4: Frontend Mobile-First (Dias 5 e 6)
* **[TASK-11]** Construir Tela 1 (Boas-vindas white-label da ótica + aceite explícito de privacidade LGPD).
* **[TASK-12]** Construir Tela 2 (Captura de selfie frontal com guia oval sobreposto + fallback para upload de foto da galeria).
* **[TASK-13]** Construir Tela 3 (Quiz rápido de 3 perguntas com chips táteis: estilo, tipo de lente/grau e material).
* **[TASK-14]** Construir Tela 4 (Exibição do perfil facial, cards das 3 armações com justificativa, botão "Reservar pelo WhatsApp" com mensagem pré-preenchida e botões `Gostei` / `Não curti`).

### Épico 5: Observabilidade Langfuse, Testes e Deploy (Dia 7)
* **[TASK-15]** Instrumentar o backend com **Langfuse SDK** (Traces, Spans de MediaPipe/Matching, Generations da VLM, custos por token e endpoint de score de feedback).
* **[TASK-16]** Executar bateria de testes com 15 fotos variadas (iluminações diferentes, formatos de rosto distintos e casos de erro como óculos escuros).
* **[TASK-17]** Realizar deploy do frontend (Vercel) e backend (Render/Railway), gerar QR Code de acesso e entregar o link piloto para a primeira ótica parceira.

---

## Fase 2: Backlog V2 (Evolução Pós-Primeira Venda)

* **[V2-01]** **Painel Administrativo da Ótica:** Interface web para o lojista cadastrar novas armações, atualizar estoque ou importar planilha CSV sem depender do desenvolvedor.
* **[V2-02]** **Bot Nativo no WhatsApp:** Integração com WhatsApp Cloud API para o cliente enviar a selfie diretamente na conversa da loja.
* **[V2-03]** **Avaliações Automatizadas (LLM-as-a-Judge):** Pipeline no Langfuse para auditar periodicamente se as justificativas geradas estão coerentes com os atributos faciais extraídos.
* **[V2-04]** **Provador Visual 2D (Overlay):** Sobreposição simples da imagem PNG transparente da armação sobre os pontos oculares detectados pelo MediaPipe.
