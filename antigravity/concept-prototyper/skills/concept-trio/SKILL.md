---
name: concept-trio
description: >-
  Use esta skill quando o usuário quiser gerar 3 ideias, conceitos, mockups ou propostas
  alternativas que NÃO sejam alteração direta do código do projeto (ex.: 3 mockups HTML de
  frontend para visualizar uma nova tela, 3 formatos diferentes de documentação, ou 3 estruturas
  de apresentações/slides). Dispara 3 subagentes em paralelo para persistir os artefatos no
  diretório solicitado e resume o comparativo executivo diretamente no chat com links clicáveis.
---

# Triplo Prototipador Conceitual (`concept-trio`)

Esta skill coordena **3 subagentes concorrentes** para idealizar, prototipar e documentar 3 soluções conceituais alternativas para validação humana prévia. O objetivo **não é alterar código de produção do projeto**, mas sim materializar ideias para que o usuário decida com segurança qual caminho seguir.

A skill realiza **roteamento inteligente de subagentes especializados** para garantir a máxima qualidade em cada use case, além de dispor de um agente genérico para qualquer outra demanda conceitual.

---

## Roteamento Inteligente de Subagentes Especializados

| Use Case / Domínio | Subagente Especializado | Formato de Artefato | Foco Técnico |
| :--- | :--- | :--- | :--- |
| **Frontend / Telas de UI** | `ui-prototyper` (3x) | `.html` autocontido | Tailwind CDN, dados realistas, micro-interações em JS vanilla |
| **Apresentações / Decks / PPTs** | `slide-designer` (3x) | `.html` interativo ou `.md` | Decks navegáveis por setas `←` `→`, narrativa, speaker notes |
| **Documentações / Guias** | `doc-prototyper` (3x) | `.md` estruturado | Tutorials, referências formais, cookbooks e diagramas Mermaid |
| **Outros Conceitos / Genérico** | `concept-generator` (3x) | `.md`, `.json`, `.yaml`, etc. | Modelagem de dados, schemas de API, fluxos de esteira, etc. |

*(Nota: Caso algum subagente específico não esteja disponível no ambiente atual, utilize `TypeName: "self"` com os papéis descritos abaixo).*

---

## Princípios de Execução

1. **Materialização Rápida e Independente de Código**:
   - Cada subagente produz um arquivo de artefato completo e autossuficiente diretamente na pasta designada.
   - O projeto de produção existente permanece 100% intocado.
2. **Persistência Direta dos Artefatos**:
   - Os subagentes escrevem seus artefatos diretamente na pasta indicada pelo usuário (ou padrão).
   - Cada subagente retorna ao orquestrador apenas uma notificação concisa (5 a 8 linhas) com trade-offs e o link do arquivo gerado.
3. **Consolidação Exclusiva no Chat**:
   - O agente principal **não cria nenhum arquivo extra de consolidação em disco**.
   - A síntese comparativa é apresentada inteiramente no chat com links clicáveis (`file:///...`), tabela de trade-offs estéticos/estruturais e comandos rápidos para abrir/testar.
4. **Execução Concorrente e Descarte Imediato**:
   - Os 3 subagentes são disparados simultaneamente no mesmo turno.
   - Assim que as 3 entregas forem confirmadas, os subagentes são encerrados com `manage_subagents(Action="kill")`.

---

## Fluxo de Execução

### Passo 1: Triagem do Use Case, Diretório e Subagente

1. **Classifique a Demanda**:
   - **Frontend / Telas (`.html`)**: Selecione o subagente `ui-prototyper`.
   - **Slides / Apresentações (`.html` ou `.md`)**: Selecione o subagente `slide-designer`.
   - **Documentações / Manuais (`.md`)**: Selecione o subagente `doc-prototyper`.
   - **Outros Protótipos Conceituais (Dados, Schemas, Fluxos)**: Selecione o subagente `concept-generator`.

2. **Defina o Diretório de Destino (`<target_dir>`)**:
   - Se o usuário especificou um caminho (ex.: `mockups/nova-tela/`, `docs/propostas/`, `decks/`), use exatamente esse diretório.
   - Se não especificou, adote um diretório padrão:
     - Frontend: `mockups/<nome-da-tela>/`
     - Slides: `decks/<tema>/`
     - Documentação: `docs/propostas/<tema>/`
     - Genérico: `concepts/<tema>/`
   - Garanta que o diretório exista antes de acionar os subagentes.

3. **Configure os 3 Eixos de Divergência**:
   - **Se Frontend (`ui-prototyper`)**:
     - *Opção 1 — Minimalista & Clean*: Layout linear, foco em legibilidade e alto respiro. (`opcao_1_minimalista.html`)
     - *Opção 2 — Dashboard Produtivo & Denso*: Múltiplos painéis, alta densidade, tabelas e métricas. (`opcao_2_dashboard.html`)
     - *Opção 3 — Experiencial & Moderno*: Bento grid, glassmorphism e micro-transições visuais. (`opcao_3_experiencial.html`)
   - **Se Slides (`slide-designer`)**:
     - *Opção 1 — Pitch Executivo / Decisores*: Foco em métricas, ROI e síntese executiva de alto impacto. (`opcao_1_pitch_executivo.html`)
     - *Opção 2 — Deep-Dive Técnico / Arquitetura*: Foco em diagramas conceituais, topologia e robustez. (`opcao_2_tecnica.html`)
     - *Opção 3 — Storytelling & Narrativa de Usuário*: Foco na jornada do usuário e visão "Antes vs. Depois". (`opcao_3_storytelling.html`)
   - **Se Documentação (`doc-prototyper`)**:
     - *Opção 1 — Tutorial Hands-On (Quickstart)*: Focado em velocidade de adoção prática. (`opcao_1_tutorial.md`)
     - *Opção 2 — Referência Formal (Especificação)*: Focado em contratos de API e restrições. (`opcao_2_referencia.md`)
     - *Opção 3 — Cookbook / Casos de Uso*: Focado em receitas práticas e troubleshooting. (`opcao_3_cookbook.md`)
   - **Se Conceito Genérico (`concept-generator`)**:
     - *Opção 1 — Enxuta / Pragmática*: Solução direta com o menor número de componentes.
     - *Opção 2 — Enterprise / Modular*: Altamente desacoplada, configurável e expansível.
     - *Opção 3 — Orientada a Eventos / Reativa*: Baseada em publicação/assinatura ou streaming.

---

### Passo 2: Disparo Concorrente dos 3 Subagentes Especializados

Em uma única rodada de ferramentas, invoque simultaneamente os 3 subagentes selecionados:

#### Subagente 1
- `TypeName`: `<nome_do_agente_especifico>` (ex: `"ui-prototyper"`, `"slide-designer"`, `"doc-prototyper"` ou `"concept-generator"`)
- `Role`: `"Specialist 1 - Minimalista/Direto"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista responsável pela **Opção 1** para o conceito: `<demanda>`.
  > Escreva a proposta completa e autossuficiente diretamente no arquivo: `<caminho_opcao_1>`.
  > Não envie o código/conteúdo inteiro no chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável: [Opção 1: Nome](file:///<caminho_opcao_1>)
  > - Filosofia de design e proposta central
  > - Principais trade-offs
  > - Perfil ou caso de uso ideal desta opção."

#### Subagente 2
- `TypeName`: `<nome_do_agente_especifico>`
- `Role`: `"Specialist 2 - Estruturado/Denso"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista responsável pela **Opção 2** para o conceito: `<demanda>`.
  > Escreva a proposta completa e autossuficiente diretamente no arquivo: `<caminho_opcao_2>`.
  > Não envie o código/conteúdo inteiro no chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável: [Opção 2: Nome](file:///<caminho_opcao_2>)
  > - Filosofia de design e proposta central
  > - Principais trade-offs
  > - Perfil ou caso de uso ideal desta opção."

#### Subagente 3
- `TypeName`: `<nome_do_agente_especifico>`
- `Role`: `"Specialist 3 - Inovador/Experiencial"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista responsável pela **Opção 3** para o conceito: `<demanda>`.
  > Escreva a proposta completa e autossuficiente diretamente no arquivo: `<caminho_opcao_3>`.
  > Não envie o código/conteúdo inteiro no chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável: [Opção 3: Nome](file:///<caminho_opcao_3>)
  > - Filosofia de design e proposta central
  > - Principais trade-offs
  > - Perfil ou caso de uso ideal desta opção."

---

### Passo 3: Limpeza de Subagentes

Após receber as notificações com os resumos dos 3 subagentes:
1. Encerre imediatamente as 3 instâncias com `manage_subagents(Action="kill", ConversationIds=[...])`.

---

### Passo 4: Apresentação Comparativa no Chat

Formate o resultado **exclusivamente na resposta do chat**, sem gerar arquivos de consolidação em disco:

1. **Apresentação e Links de Acesso Rápido**:
   - `[Opção 1: Nome](file:///<caminho_opcao_1>)` — Descrição em 1 linha.
   - `[Opção 2: Nome](file:///<caminho_opcao_2>)` — Descrição em 1 linha.
   - `[Opção 3: Nome](file:///<caminho_opcao_3>)` — Descrição em 1 linha.

2. **Matriz Comparativa de Decisão**:
   - Exiba a tabela comparando Abordagem, Densidade/Profundidade, Pontos Fortes, Trade-offs e Público Recomendado.

3. **Chamada para Ação**:
   - Convide o usuário a abrir os arquivos e indicar qual caminho prefere seguir antes de avançar para a fase definitiva.
