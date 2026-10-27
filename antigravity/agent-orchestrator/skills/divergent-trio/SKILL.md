---
name: divergent-trio
description: >-
  Use esta skill quando o usuário solicitar 3 exemplos, abordagens, propostas ou
  soluções alternativas para um problema ou tarefa técnica. Dispara 3 subagentes em paralelo,
  onde cada um gera seu próprio arquivo de artefato no diretório solicitado e o agente principal
  apresenta exclusivamente no chat um comparativo consolidado com tabela de trade-offs e links clicáveis.
---

# Triplo Gerador Divergente (`divergent-trio`)

Esta skill orquestra **3 subagentes concorrentes** para explorar e implementar 3 perspectivas técnicas ortogonais e contrastantes para uma mesma demanda ou problema (ex.: Pragmática/KISS, Robusta/Modular, e Inovadora/Alternativa).

Cada subagente persiste sua própria solução completa em um arquivo de artefato dedicado dentro do diretório solicitado pelo usuário, e o agente principal sintetiza uma comparação executiva estruturada diretamente no chat (sem gerar arquivos extras de consolidação).

---

## Princípios de Execução

1. **Persistência Direta nos Artefatos (Zero Overhead no Chat Principal)**:
   - Os 3 subagentes geram o conteúdo volumoso (código-fonte completo, exemplos funcionais, diagramas e especificações técnicas) diretamente nos arquivos designados no diretório de destino.
   - Cada subagente retorna ao orquestrador apenas uma notificação enxuta (5 a 8 linhas) com o link clicável do arquivo gerado e seus trade-offs.
2. **Consolidação Exclusivamente no Chat**:
   - O agente principal **NÃO** cria um arquivo de artefato em disco para o consolidado (salvo se o usuário exigir explicitamente).
   - A síntese é apresentada diretamente no chat com links clicáveis (`file:///...`), tabela comparativa de critérios e recomendação técnica.
3. **Divergência Real Garantida**:
   - Os 3 subagentes recebem personas e critérios ortogonais deliberadamente distintos, evitando que gerem variações triviais da mesma implementação.
4. **Execução Concorrente e Limpeza Atômica**:
   - Os 3 subagentes são disparados em paralelo no mesmo turno através de chamadas a `invoke_subagent`.
   - Ao receber os 3 retornos, o orquestrador encerra imediatamente as 3 sessões chamando `manage_subagents(Action="kill", ConversationIds=[...])` para liberar memória e processos em background.

---

## Fluxo de Execução

### Passo 1: Delimitação de Escopo e Diretório de Saída

1. **Identifique a Demanda / Desafio Técnico**:
   - O que deve ser exemplificado, construído ou proposto (ex.: fluxo de autenticação, camada de repositório, estratégia de cache, componente de UI, etc.)?
2. **Defina o Diretório de Saída (`<target_dir>`)**:
   - Se o usuário especificou um diretório (ex.: `examples/`, `docs/proposals/`, `specs/`), utilize exatamente esse diretório.
   - Se o usuário não especificou um diretório, adote por padrão:
     - `proposals/` (se for proposta de arquitetura ou design).
     - `examples/` (se forem exemplos de código).
   - Certifique-se de que o diretório exista (crie se necessário antes do disparo).
3. **Defina os 3 Eixos de Divergência**:
   - Se o usuário especificou as 3 abordagens desejadas, adote rigorosamente as escolhas do usuário.
   - Se o usuário não especificou os ângulos, aplique os 3 eixos fundamentais:
     - **Opção 1 — Minimalista & Pragmática (KISS / YAGNI)**:
       - Foco: Menor atrito possível, zero dependências extras, implementação direta, legibilidade imediata e código conciso.
       - Arquivo alvo: `<target_dir>/opcao_1_minimalista.<ext>`
     - **Opção 2 — Robusta & Modular (Enterprise / Scalability)**:
       - Foco: Separação estrita de responsabilidades, alta testabilidade, tipagem forte, abstração e extensibilidade para o futuro.
       - Arquivo alvo: `<target_dir>/opcao_2_modular.<ext>`
     - **Opção 3 — Inovadora / Alternativa (Out-of-the-Box / High-Performance)**:
       - Foco: Padrão moderno ou alternativo (ex.: programação funcional, reatividade, concorrência orientada a eventos, metaprogramação ou APIs modernas da linguagem/runtime).
       - Arquivo alvo: `<target_dir>/opcao_3_inovadora.<ext>`

---

### Passo 2: Disparo Concorrente dos 3 Subagentes

Em uma única rodada de ferramentas, invoque simultaneamente os 3 subagentes via `invoke_subagent` (utilize `TypeName: "divergent-generator"` se disponível no ambiente, ou `TypeName: "self"` com os papéis abaixo):

#### Subagente 1
- `TypeName`: `"divergent-generator"` (ou `"self"`)
- `Role`: `"Divergent Generator 1 - Minimalista"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista técnico na **Opção 1 (Minimalista & Pragmática / KISS)** para a demanda: `<demanda>`.
  > Escreva a solução completa, autoexplicativa e pronta para uso diretamente no arquivo: `<caminho_absoluto_opcao_1>`.
  > Respeite o `AGENTS.md` do repositório se existir.
  > Não envie o código inteiro na resposta do chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável para o arquivo gerado: [Opção 1: Minimalista](file:///<caminho_absoluto_opcao_1>)
  > - Filosofia central e pontos fortes
  > - Principais trade-offs assumidos
  > - Cenário ideal para escolha desta opção."

#### Subagente 2
- `TypeName`: `"divergent-generator"` (ou `"self"`)
- `Role`: `"Divergent Generator 2 - Robusta"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista técnico na **Opção 2 (Robusta & Modular / Enterprise)** para a demanda: `<demanda>`.
  > Escreva a solução completa, autoexplicativa e pronta para uso diretamente no arquivo: `<caminho_absoluto_opcao_2>`.
  > Respeite o `AGENTS.md` do repositório se existir.
  > Não envie o código inteiro na resposta do chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável para o arquivo gerado: [Opção 2: Robusta](file:///<caminho_absoluto_opcao_2>)
  > - Filosofia central e pontos fortes
  > - Principais trade-offs assumidos
  > - Cenário ideal para escolha desta opção."

#### Subagente 3
- `TypeName`: `"divergent-generator"` (ou `"self"`)
- `Role`: `"Divergent Generator 3 - Inovadora"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Você é o especialista técnico na **Opção 3 (Inovadora / Alternativa / Alta Performance)** para a demanda: `<demanda>`.
  > Escreva a solução completa, autoexplicativa e pronta para uso diretamente no arquivo: `<caminho_absoluto_opcao_3>`.
  > Respeite o `AGENTS.md` do repositório se existir.
  > Não envie o código inteiro na resposta do chat. Retorne APENAS um resumo de 5 a 8 linhas contendo:
  > - Link clicável para o arquivo gerado: [Opção 3: Inovadora](file:///<caminho_absoluto_opcao_3>)
  > - Filosofia central e pontos fortes
  > - Principais trade-offs assumidos
  > - Cenário ideal para escolha desta opção."

Guarde os 3 `conversationId` retornados.

---

### Passo 3: Limpeza Atômica das Sessões

Assim que os 3 subagentes concluírem a execução e enviarem suas respostas com os resumos:
1. Encerre imediatamente as 3 instâncias de subagentes chamando:
   `manage_subagents(Action="kill", ConversationIds=["<conv_id_1>", "<conv_id_2>", "<conv_id_3>"])`.

---

### Passo 4: Consolidação no Chat (Sem Artefato Extra em Disco)

Apresente a síntese comparativa **diretamente no chat**, estruturada para facilitar a tomada de decisão do usuário:

1. **Apresentação dos Artefatos Gerados**:
   - `[Opção 1: Minimalista](file:///<caminho_absoluto_opcao_1>)`: resumo em 1 linha da abordagem.
   - `[Opção 2: Robusta & Modular](file:///<caminho_absoluto_opcao_2>)`: resumo em 1 linha da abordagem.
   - `[Opção 3: Inovadora & Alternativa](file:///<caminho_absoluto_opcao_3>)`: resumo em 1 linha da abordagem.

2. **Matriz Comparativa de Trade-offs**:

| Critério | Opção 1 (Minimalista) | Opção 2 (Robusta) | Opção 3 (Inovadora) |
| :--- | :--- | :--- | :--- |
| **Abordagem** | KISS, baixa sobrecarga, direto ao ponto | Separação estrita, modular e extensível | Padrão reativo/funcional ou otimizado |
| **Complexidade / Curva** | Muito baixa | Média (exige conhecer os contratos) | Média/Alta (requer domínio do paradigma) |
| **Pontos Fortes** | Rápida de implementar e fácil de ler | Preparada para evolução e alta testabilidade | Alta performance ou menor boilerplate |
| **Trade-offs / Limites** | Menos adaptável a mudanças estruturais | Maior número de arquivos e boilerplate | Curva de aprendizado da equipe |
| **Cenário Ideal** | Prototipagem, microserviço, escopo fechado | Core domain corporativo, times grandes | Gargalo de I/O, concorrência ou pipelines |

3. **Recomendação & Próximos Passos**:
   - Forneça uma recomendação curta com base no contexto do projeto.
   - Finalize convidando o usuário a decidir: *"Qual das 3 opções você prefere seguir (1, 2 ou 3), ou deseja combinar aspectos de mais de uma?"*
