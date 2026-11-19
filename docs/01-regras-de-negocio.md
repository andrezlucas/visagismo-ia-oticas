# 01. Regras de Negócio, Visagismo e Governança

Este documento define as regras determinísticas, critérios ópticos e diretrizes de privacidade que governam o motor de recomendação do sistema.

---

## 1. Matriz de Compatibilidade de Visagismo

O motor de recomendação aplica o princípio de **harmonia por contraste ou equilíbrio proporcional** entre o formato predominante do rosto identificado pela IA e a geometria das armações cadastradas no estoque da ótica.

| Formato do Rosto | Características Principais | Armações Priorizadas | Armações Evitadas | Objetivo Estético |
| :--- | :--- | :--- | :--- | :--- |
| **Redondo** | Largura e altura semelhantes, bochechas preenchidas e maxilar suave. | Retangular, Quadrada, Geométrica, Wayfarer. | Redonda pequena, Oval muito curva. | Criar linhas angulares que alongam e definem os traços. |
| **Quadrado** | Testa larga, maçãs do rosto alinhadas e mandíbula bem marcada. | Redonda, Oval, Aviador, Gatinho suave. | Quadrada rígida, Retangular de cantos vivos. | Suavizar os ângulos faciais e equilibrar a linha do maxilar. |
| **Oval** | Proporções equilibradas, queixo levemente arredondado. | Quase todos os formatos (piloto, retangular, redonda, geométrica). | Armações excessivamente largas que ultrapassem a linha das têmporas. | Manter a proporção natural sem distorcer o equilíbrio facial. |
| **Coração / Triângulo Invertido** | Testa mais larga e queixo estreito ou fino. | Aviador, Redonda, Oval, Aro fino ou meio-aro. | Gatinho muito exagerado ou hastes superiores pesadas. | Reduzir o peso visual na parte superior e equilibrar o queixo. |
| **Diamante** | Maçãs do rosto proeminentes, testa e maxilar mais estreitos. | Oval, Gatinho, Retangular com bordas suavizadas. | Armações muito estreitas que evidenciem demais as maçãs. | Valorizar a região dos olhos e suavizar a largura central. |
| **Retangular / Oblongo** | Rosto mais alongado verticalmente com linhas retas. | Quadrada alta, Aviador, Redonda de aro maior. | Retangular muito estreita e achatada. | Adicionar amplitude horizontal e equilibrar o comprimento vertical. |

---

## 2. Regras Técnicas Ópticas (Restrições por Tipo de Lente)

Para evitar que o sistema recomende uma armação esteticamente bonita, mas inviável tecnicamente para a receita do cliente, o questionário coleta a faixa de uso/grau e aplica filtros obrigatórios:

1. **Grau Alto (Miopia ou Hipermetropia elevada):**
   * **Regra:** Bloquear armações de aro muito grande, fio de nylon ou parafusadas (balgriff), pois evidenciam a espessura da borda da lente.
   * **Prioridade:** Armações de acetato fechado e aros menores/médios que disfarçam a espessura da lente.
2. **Lentes Multifocais / Progressivas:**
   * **Regra:** Exigir altura mínima de aro (`altura_aro_mm >= 30`) para comportar os campos de visão de longe, intermediário e perto sem cortes.
3. **Grau Leve / Descanso:**
   * **Regra:** Sem restrição estrutural de material; libera tanto metal fino quanto acetato e fio de nylon.
4. **Óculos de Sol:**
   * **Regra:** Filtra exclusivamente produtos da categoria `solar` no estoque da loja.

---

## 3. Regras de Catálogo, Estoque e Ranqueamento

* **RN01 - Estoque Ativo Obrigatório:** Somente armações com `ativo = true` e `quantidade_estoque > 0` na loja consultada podem entrar no pipeline de recomendação.
* **RN02 - Zero Alucinação de Produtos:** O modelo de linguagem (LLM) não tem autonomia para sugerir marcas ou modelos externos. Ele recebe apenas os itens previamente filtrados pelo banco de dados da ótica.
* **RN03 - Critério de Pontuação (Score de Match):**
  * Compatibilidade com formato do rosto: peso `50%`.
  * Compatibilidade com estilo declarado pelo cliente (clássico, moderno, discreto, ousado): peso `30%`.
  * Preferência de material (acetato, metal ou indiferente): peso `20%`.
* **RN04 - Limite de Exibição:** Retornar exatamente **3 recomendações** ordenadas pelo maior score final (ou até o máximo disponível caso o filtro restrito retorne menos de 3 itens, acionando relaxamento controlado da preferência de material).

---

## 4. Regras de Validação de Entrada (Qualidade de Imagem)

Antes de processar a análise com IA generativa, a imagem enviada passa por validação automática:

* Deve conter **exatamente 1 rosto humano** visível.
* Inclinação lateral ou vertical do rosto inferior a **20 graus**.
* Iluminação mínima adequada para distinguir contornos faciais e sobrancelhas.
* **Bloqueio de óculos escuros:** Caso o cliente envie foto usando óculos escuros ou com o rosto parcialmente coberto, a análise é interrompida com instrução clara para refazer a captura.

---

## 5. Governança de Dados Biométricos e Conformidade (LGPD)

* **Consentimento Granular:** O fluxo só é iniciado após aceite explícito do usuário na tela inicial informando a finalidade exclusiva da análise estética.
* **Processamento Efêmero (Zero Persistência de Imagem):** A selfie enviada trafega via HTTPS, é carregada apenas em memória volátil (RAM) durante a requisição e é **descartada imediatamente** após a extração do JSON de atributos faciais. Nenhuma foto é gravada em disco, bucket S3 ou banco de dados.
* **Anonimização de Métricas:** Os registros salvos para análise de desempenho (no PostgreSQL e no Langfuse) armazenam apenas identificadores de sessão (`session_id`), metadados geométricos agregados (ex.: `"formato_rosto": "oval"`) e o feedback de aprovação.
