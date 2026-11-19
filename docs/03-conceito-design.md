# 03. Conceito de Design, UX e Wireframes

Este documento descreve a experiência do usuário (UX), o Design System e o fluxo de telas da aplicação web mobile-first.

---

## 1. Princípios de Experiência do Usuário (UX)

* **Zero Instalação (Web / QR Code):** O cliente acessa diretamente pelo navegador do celular ao escanear um QR Code no balcão da ótica ou clicar em um link enviado no WhatsApp.
* **Redução Máxima de Atrito:** Em vez de exigir fotos de múltiplos ângulos (perfil direito, perfil esquerdo, frontal), o fluxo solicita **apenas 1 selfie frontal guiada** com feedback visual imediato, mantendo a foto lateral apenas como campo opcional para clientes com grau muito alto.
* **Transparência e Confiança:** O aviso de descarte imediato da foto aparece logo no primeiro botão, eliminando o receio de privacidade antes mesmo de abrir a câmera.
* **Consultoria Explicável:** Cada armação recomendada exibe um selo de compatibilidade e uma justificativa clara em linguagem acessível, ajudando tanto o cliente quanto o vendedor da loja física.

---

## 2. Design System (Identidade Visual)

### 2.1. Paleta de Cores
A paleta foi projetada com fundo neutro claro para evitar reflexos ou distorções na percepção das cores das armações exibidas na tela.

| Token | Hex | Uso |
| :--- | :--- | :--- |
| `--bg-primary` | `#F8F9FA` | Fundo principal da aplicação (Off-white suave). |
| `--bg-surface` | `#FFFFFF` | Cards de produto, modais e containers de formulário. |
| `--border-subtle` | `#E2E8F0` | Bordas de cards e divisores leves. |
| `--brand-primary` | `#0F172A` | Botões principais, títulos e elementos de autoridade (Azul-ardósia escuro). |
| `--brand-accent` | `#0D9488` | Selos de IA, indicadores de match e destaques consultivos (Verde-petróleo). |
| `--text-primary` | `#1E293B` | Textos principais e nomes de produtos. |
| `--text-secondary` | `#64748B` | Descrições técnicas e instruções auxiliares. |
| `--status-success` | `#16A34A` | Confirmação de enquadramento e feedback positivo. |
| `--status-error` | `#DC2626` | Alertas de foto escura, rosto inclinado ou óculos escuros. |

### 2.2. Tipografia
* **Títulos e Destaques:** `Plus Jakarta Sans` (SemiBold / Bold) para transmitir precisão técnica e modernidade.
* **Corpo e Especificações:** `Inter` (Regular / Medium) para alta legibilidade em telas pequenas.

### 2.3. Componentes Principais
* **Máscara Guiada de Câmera:** Overlay oval tracejado centralizado na viewport com checklist dinâmico de iluminação e posicionamento.
* **Chips de Seleção Única:** Botões táteis largos (`min-height: 48px`) para respostas rápidas com uma mão no celular.
* **Card de Armação Recomendada:** Foto do produto em proporção `4:3`, badge de `% Match`, preço, acordeão/bloco "Por que combina com você" e CTA direto ("Reservar na Loja" ou "Falar com Consultor").

---

## 3. Fluxo de Telas e Wireframes Estruturais

```text
+-----------------------------------+   +-----------------------------------+
|  TELA 1: BOAS-VINDAS E LGPD       |   |  TELA 2: CAPTURA GUIADA           |
|                                   |   |                                   |
|  [Logo da Ótica Parceira]         |   |  < Voltar              Passo 1/2  |
|                                   |   |                                   |
|  Descubra a armação ideal         |   |  +-----------------------------+  |
|  para o formato do seu rosto.     |   |  |      . - ~ ~ ~ - .          |  |
|                                   |   |  |    /   (o)   (o)   \        |  |
|  Como funciona:                   |   |  |   |       _         |       |  |
|  1. Tire uma selfie frontal       |   |  |    \    \___/      /        |  |
|  2. Responda 3 perguntas rápidas  |   |  |      ' - . _ . - '          |  |
|  3. Veja 3 opções do nosso estoque|   |  +-----------------------------+  |
|                                   |   |  [v] Boa luz   [v] Sem óculos    |
|  [x] Concordo com a análise da    |   |                                   |
|      imagem (foto descartada      |   |       (  Tirar Foto  )            |
|      logo após o resultado).      |   |       [Enviar da Galeria]         |
|                                   |   |                                   |
|       [ Começar Análise ]         |   |  + Foto lateral (opcional)        |
+-----------------------------------+   +-----------------------------------+

+-----------------------------------+   +-----------------------------------+
|  TELA 3: QUIZ RÁPIDO (3 ETAPAS)   |   |  TELA 4: RESULTADO E SUGESTÕES    |
|                                   |   |                                   |
|  < Voltar              Passo 2/2  |   |  Seu Perfil: Rosto Quadrado       |
|                                   |   |  "Traços marcantes que ganham     |
|  1. Qual seu estilo principal?    |   |   harmonia com linhas curvas."    |
|  (Clássico) [Moderno] (Discreto)  |   |                                   |
|                                   |   |  +-----------------------------+  |
|  2. Tipo de lente / grau:         |   |  | [Foto Armação 1]  96% Match |  |
|  [Grau leve/médio] (Grau alto)    |   |  | Urban Round Acetato         |  |
|  (Multifocal) (Apenas sol)        |   |  | R$ 489,90                   |  |
|                                   |   |  | Por que combina: Suaviza os |  |
|  3. Material preferido:           |   |  | ângulos do maxilar...       |  |
|  [Tanto faz] (Acetato) (Metal)    |   |  | [Reservar na Loja]          |  |
|                                   |   |  +-----------------------------+  |
|       [ Ver Minhas Armações ]     |   |  [👍 Gostei]   [👎 Não curti]     |
+-----------------------------------+   +-----------------------------------+
```

---

## 4. Detalhamento Funcional das Telas

1. **Tela 1 (Início e Privacidade):** Exibe a marca da ótica (white-label leve), explica o processo em 3 passos e bloqueia o botão principal até o aceite do termo de uso efêmero da imagem.
2. **Tela 2 (Captura Guiada):** Abre a câmera frontal via `navigator.mediaDevices.getUserMedia` com guia visual de enquadramento ou permite upload direto da galeria.
3. **Tela 3 (Quiz de Contexto):** Coleta estilo, necessidade óptica (grau) e preferência de material em uma única tela rolável, sem recarregamentos.
4. **Tela 4 (Vitrine Consultiva):** Apresenta o diagnóstico do formato de rosto, os 3 cards ranqueados com justificativa personalizada, botão de conversão (envio direto para o WhatsApp do vendedor com o SKU já preenchido) e botões de avaliação (`Gostei` / `Não curti`) conectados ao Langfuse.
