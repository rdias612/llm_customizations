---
name: plan-debater
description: >-
  Use esta skill para submeter um plano de implementação a um debate rigoroso entre
  Red Team (críticas técnicas realistas) e Blue Team (defesa pragmática anti-overengineering),
  com julgamento final por um Árbitro Neutro que consolida as críticas aceitas e rejeitadas em 'discussao/<slug>/03-arbitragem.md'
  e expõe um artefato visual para decisão informada do usuário via ask_question.
---

# Orquestrador de Debate Adversário (`plan-debater`)

Esta skill coordena uma sessão de estresse e validação técnica em 3 fases (`plan-red-team` $\rightarrow$ `plan-blue-team` $\rightarrow$ `plan-referee`) sobre um plano de implementação em markdown, garantindo que o plano seja blindado contra falhas reais sem cair na armadilha da sobre-engenharia ou de cenários abstratos fora da escala do projeto.

---

## Princípios e Arquitetura *Blackboard*

1. **Diretório `discussao/<slug>/` Isolado por Plano**:
   - Todo o histórico e os artefatos do debate são gerados dentro do diretório `discussao/<slug>/` na raiz do workspace (onde `<slug>` é o nome base do arquivo do plano sem extensão), evitando sobrescrever debates de outros planos.
2. **Zero Inchaço de Contexto e Passagem por Referência**:
   - Cada subagente lê e escreve diretamente nos artefatos em disco (`write_file` com `Overwrite: true`) e **nunca** modifica código de produção nem o arquivo do plano original (cuja edição cabe exclusivamente ao orquestrador no Passo 5).
   - Não duplique nos prompts de invocação as regras extensas que já constam nos system prompts de cada subagente; passe apenas os caminhos de referência (`<caminho_plano>`, `<red_artifact>`, `<blue_artifact>`, `<referee_artifact>`) e o objetivo direto da fase.
   - Os subagentes retornam apenas resumos executivos curtos (até 10–12 linhas) para o agente orquestrador.
3. **Estrutura Canônica Unificada, Evidência Real e Teto de Ruído**:
   - Todos os pontos debatidos compartilham a mesma numeração sequencial (`[ITEM-01]`, `[ITEM-02]`, etc., limitados aos 5–7 mais críticos), exigem comprovação com caminho e linha/símbolo no código (`caminho/arquivo.ext:L123`) e mantêm contexto autossuficiente nos três arquivos gerados.
4. **Regra de Ouro da Realidade, Escala e Invariante de Build**:
   - Cenários abstratos, puramente teóricos ou de escala desproporcional à realidade atual do projeto são expressamente rejeitados.
   - O debate verifica se as etapas propostas preservam a compilação/build do projeto ao final de cada etapa individual ($1 \rightarrow N$).
5. **Transparência Visual Obrigatória (Artefato de UI & Links Clicáveis)**:
   - Ao concluir a arbitragem, o orquestrador gera obrigatoriamente um **Artefato Nativo do Antigravity** (`ArtifactMetadata: UserFacing=true`) para que a interface abra a visualização formatada do debate lado a lado com o chat.
   - Todas as perguntas interativas (`ask_question`) e mensagens de chat DEVEM conter links clicáveis com o protocolo `file:///` para os artefatos de `discussao/<slug>/` e contextualização rica de cada item (etapa, operação e arquivo:linha).

---

## Fluxo de Execução

### Passo 1: Localização do Plano e Preparação do Ambiente
1. **Identifique o plano-alvo**:
   - Use o caminho informado pelo usuário ou busque o plano mais recente no diretório `plans/` (ou arquivos `plano_*.md` na raiz).
   - Defina `<caminho_plano>` e extraia `<slug>` (ex.: `plans/autenticacao-jwt.md` $\rightarrow$ `autenticacao-jwt`).
2. **Defina os caminhos dos artefatos (em `discussao/<slug>/`)**:
   - `<red_artifact>`: `discussao/<slug>/01-red-team.md`
   - `<blue_artifact>`: `discussao/<slug>/02-blue-team.md`
   - `<referee_artifact>`: `discussao/<slug>/03-arbitragem.md`

---

### Passo 2: Fase 1 — Ataque Técnico (`plan-red-team`)
Invoque `invoke_subagent`:
- `TypeName`: `"plan-red-team"`
- `Role`: `"Plan Red Team"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Audite o plano de implementação em `<caminho_plano>` confrontando-o com o código real do repositório e o `AGENTS.md` (se existir). Siga rigorosamente suas diretrizes (filtro de escala real, verificação da invariante de build entre etapas $1 \rightarrow N$, evidência `arquivo:linha` e teto de 5 a 7 itens `[BLOQUEANTE]`/`[MODERADO]`, ou 0 críticas se impecável).
  > 
  > Persista o relatório completo em `<red_artifact>` usando `write_file` (`Overwrite: true`) e retorne apenas o sumário executivo com os títulos e severidades (máx. 10 linhas)."

- **Curto-Circuito (Zero Críticas)**: Se o `plan-red-team` reportar **0 críticas**:
  1. Crie o artefato de UI em `<appDataDir>\brain\<conversation-id>/debate_<slug>.md` usando `write_file` com `ArtifactMetadata` (`UserFacing: true`, `RequestFeedback: false`, `Summary: "Debate Técnico: Plano Aprovado sem Ressalvas"`), contendo link para [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md) e atestando que o plano foi auditado e aprovado sem ressalvas pelo Red Team.
  2. Encerre o subagente chamando `manage_subagents(Action="kill", ConversationIds=["<red_conv_id>"])`.
  3. Atualize o cabeçalho de `<caminho_plano>` (`edit_file`) adicionando `- **Debate Técnico**: Aprovado sem ressalvas pelo Red Team (\`discussao/<slug>/01-red-team.md\`)`.
  4. Apresente no chat a confirmação com links clicáveis para [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md) e para o plano.
  5. Encerre o debate imediatamente.

---

### Passo 3: Fase 2 — Defesa Pragmática (`plan-blue-team`)
Após a conclusão do `plan-red-team`, invoque `invoke_subagent`:
- `TypeName`: `"plan-blue-team"`
- `Role`: `"Plan Blue Team"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Leia o plano original em `<caminho_plano>` e as críticas do Red Team em `<red_artifact>`. Responda a cada `[ITEM-XX]` seguindo suas diretrizes de defesa pragmática (KISS/YAGNI, evidências concretas no código atual e concessão honesta com mitigação minimalista quando a falha for real).
  > 
  > Persista a defesa completa em `<blue_artifact>` usando `write_file` (`Overwrite: true`) e retorne apenas o sumário com o status de cada item (`REBATIDO`, `CONCEDIDO` ou `PARCIALMENTE CONCEDIDO`)."

---

### Passo 4: Fase 3 — Julgamento Neutro (`plan-referee`)
Após a conclusão do `plan-blue-team`, invoque `invoke_subagent`:
- `TypeName`: `"plan-referee"`
- `Role`: `"Technical Referee & Tech Lead"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Julgue o debate técnico lendo o plano em `<caminho_plano>`, o ataque em `<red_artifact>` e a defesa em `<blue_artifact>`. Inspecione o código real em caso de divergência factual e aplique suas diretrizes de julgamento (descarte de cenários abstratos/fora de escala e rejeição de falsas concessões que introduzam sobre-engenharia).
  > 
  > Persista o veredito final estruturado nas 2 seções obrigatórias (Críticas Aceitas vs. Críticas Ignoradas) em `<referee_artifact>` usando `write_file` (`Overwrite: true`). Retorne apenas o balanço final e o resumo dos pontos julgados procedentes."

---

### Passo 5: Geração do Artefato de UI, Apresentação e Decisão Interativa (`ask_question`)

#### 5.1. Geração Obrigatória do Artefato Nativo do Antigravity (Visualização na UI)
Antes de qualquer interação com o usuário:
1. Leia o conteúdo completo de `<referee_artifact>` (`discussao/<slug>/03-arbitragem.md`).
2. Crie o artefato de visualização em `<appDataDir>\brain\<conversation-id>/debate_<slug>.md` usando `write_file` com `ArtifactMetadata`:
   - `UserFacing: true`
   - `RequestFeedback: false`
   - `Summary: "Relatório de Arbitragem Técnica: Veredito Final do Debate sobre <slug>"`
3. Estruture o conteúdo do artefato com:
   - **Navegação de Arquivos no Repositório**:
     ```markdown
     > **Artefatos do Debate no Diretório `discussao/<slug>/`**:
     > - [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md) — Ataque técnico e riscos reais
     > - [02-blue-team.md](file:///<workspace_path>/discussao/<slug>/02-blue-team.md) — Defesa pragmática e contra-argumentações
     > - [03-arbitragem.md](file:///<workspace_path>/discussao/<slug>/03-arbitragem.md) — Veredito do Árbitro e plano de edição
     > - [Plano de Implementação](file:///<workspace_path>/<caminho_plano>) — Documento base em análise
     ```
   - **Tabela Sintética do Debate**:
     | Item | Alvo no Plano | Evidência | Veredito | Operação no Plano |
     | :--- | :--- | :--- | :--- | :--- |
     | `[ITEM-01]` ... | Etapa X | `arquivo:Lxx` | PROCEDENTE | MODIFICAR ETAPA X |
   - **Seção 1: Críticas Aceitas (Ajustes Mandatórios no Plano)** na íntegra.
   - **Seção 2: Críticas Ignoradas (Descartes Fundamentados)** na íntegra.
*(Isso aciona o visualizador lateral de artefatos do Antigravity, permitindo ao usuário ler o relatório completo antes de decidir no modal).*

#### 5.2. Apresentação no Chat Principal
Apresente no chat principal:
- Balanço executivo do Árbitro (`X críticas aceitas | Y críticas ignoradas`).
- Link clicável em destaque para o artefato gerado e para os 3 arquivos em `discussao/<slug>/`:
  - [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md)
  - [02-blue-team.md](file:///<workspace_path>/discussao/<slug>/02-blue-team.md)
  - [03-arbitragem.md](file:///<workspace_path>/discussao/<slug>/03-arbitragem.md)
  - [Plano Original](file:///<workspace_path>/<caminho_plano>)
- Resumo conciso dos itens **PROCEDENTES** (título, etapa e operação).

#### 5.3. Resolução Pós-Arbitragem Interativa (`ask_question`)
1. **Se houver 0 críticas aceitas**:
   - Atualize o cabeçalho de `<caminho_plano>` (`edit_file`) adicionando `- **Debate Técnico**: Validado sem ajustes pendentes (\`discussao/<slug>/03-arbitragem.md\`)`.
   - Informe no chat que o plano está aprovado e pronto para execução via `plan-executor`.
2. **Se houver críticas aceitas na Seção 1 de `<referee_artifact>`**:
   - **Primeira Pergunta**: Invoque `ask_question` incluindo **obrigatoriamente** links clicáveis no texto da pergunta:
     - `question`: `"O debate técnico identificou X ajustes procedentes no plano [<slug>.md](file:///<workspace_path>/<caminho_plano>). Consulte o relatório completo no painel de artefatos ao lado ou em [03-arbitragem.md](file:///<workspace_path>/discussao/<slug>/03-arbitragem.md) (Red: [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md) | Blue: [02-blue-team.md](file:///<workspace_path>/discussao/<slug>/02-blue-team.md)). Como deseja proceder com o plano?"`
     - `options`:
       - `"(Recommended) Aplicar todas as críticas aceitas no plano original"`
       - `"Escolher manualmente quais críticas aceitas aplicar no plano"`
       - `"Manter o plano original sem alterações por enquanto"`
   - **Segunda Pergunta (Escolha Manual)**: Se o usuário escolher `"Escolher manualmente quais críticas aceitas aplicar no plano"`:
     - Invoque uma segunda chamada a `ask_question` com `is_multi_select: true`.
     - `question`: `"Consulte os detalhes de cada item no artefato aberto ou em [03-arbitragem.md](file:///<workspace_path>/discussao/<slug>/03-arbitragem.md). Selecione quais críticas aceitas pelo Árbitro você deseja incorporar ao plano:"`
     - `options`: **NUNCA passe apenas o título cru**. Cada opção DEVE conter identificador, título, operação indicada e arquivo de evidência:
       - Formato: `"[ITEM-XX] <Título Curto> — [<Tipo de Operação> na <Etapa>] (<arquivo.ext:Lxx>)"`
       - *(Exemplo: `"(Recommended) [ITEM-01] Risco de HTML Inválido e Conflito de Eventos — [MODIFICAR ETAPA 2] (LinhaGestao.tsx:L45)"`)*
   - **Aplicação das Mudanças no Plano**:
     - Caso o usuário opte por aplicar (todas ou as selecionadas), leia a Seção 1 de `<referee_artifact>` e edite `<caminho_plano>` (`edit_file`):
       - Execute as operações especificadas pelo Árbitro (`MODIFICAR ETAPA X`, `INSERIR NOVA ETAPA ENTRE X E Y` ou `REORDENAR ETAPAS`), sincronizando tanto a seção **Checklist de Execução de Etapas** quanto a **Especificação das Etapas** e preservando a invariante de build entre etapas.
       - Registre no cabeçalho do plano: `- **Debate Técnico**: Validado e revisado (\`discussao/<slug>/03-arbitragem.md\`)`.
       - Apresente no chat a confirmação das alterações incorporadas.

#### 5.4. Limpeza da Sessão
- Encerre a trinca de subagentes do debate chamando `manage_subagents(Action="kill", ConversationIds=["<red_conv_id>", "<blue_conv_id>", "<referee_conv_id>"])` para liberar recursos e memória.
