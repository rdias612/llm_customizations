---
name: plan-debater
description: >-
  Use esta skill para submeter um plano de implementação a um debate rigoroso entre
  Red Team (críticas técnicas realistas) e Blue Team (defesa pragmática anti-overengineering),
  com julgamento final por um Árbitro Neutro que consolida as críticas aceitas e rejeitadas em 'discussao/<slug>/03-arbitragem.md'.
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
  1. Encerre o subagente chamando `manage_subagents(Action="kill", ConversationIds=["<red_conv_id>"])`.
  2. Atualize o cabeçalho de `<caminho_plano>` (`edit_file`) adicionando ou atualizando o metadado `- **Debate Técnico**: Aprovado sem ressalvas pelo Red Team (\`discussao/<slug>/01-red-team.md\`)`.
  3. Encerre o debate imediatamente e informe ao usuário que o plano foi aprovado sem ressalvas.

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

### Passo 5: Apresentação do Veredito, Consolidação e Rastreabilidade (`ask_question`)
1. Apresente no chat principal:
   - O resumo executivo do veredito gerado pelo Árbitro (balanço de aceitas vs. ignoradas).
   - Destaque objetivo dos itens que foram julgados **PROCEDENTES** (se houver), incluindo o **Tipo de Operação no Plano** indicado pelo Árbitro.
   - Links clicáveis com caminho absoluto (`file:///<workspace_path>/...`) para o plano e para os artefatos gerados:
     - [01-red-team.md](file:///<workspace_path>/discussao/<slug>/01-red-team.md)
     - [02-blue-team.md](file:///<workspace_path>/discussao/<slug>/02-blue-team.md)
     - [03-arbitragem.md](file:///<workspace_path>/discussao/<slug>/03-arbitragem.md)
2. **Resolução Pós-Arbitragem**:
   - **Se houver 0 críticas aceitas**: atualize o cabeçalho de `<caminho_plano>` (`edit_file`) adicionando `- **Debate Técnico**: Validado sem ajustes pendentes (\`discussao/<slug>/03-arbitragem.md\`)`.
   - **Se houver críticas aceitas na Seção 1 de `<referee_artifact>`**:
     1. Utilize `ask_question` para perguntar ao usuário como deseja prosseguir:
        - `(Recommended) Aplicar todas as críticas aceitas no plano original (<caminho_plano>)`
        - `Escolher manualmente quais críticas aceitas aplicar no plano`
        - `Manter o plano original sem alterações por enquanto`
     2. Caso o usuário escolha **`Escolher manualmente quais críticas aceitas aplicar no plano`**, faça uma segunda chamada a `ask_question` com `is_multi_select: true`, listando cada `[ITEM-XX] <Título>` aceito pelo Árbitro como uma opção selecionável.
     3. Caso o usuário opte por aplicar (todas ou as selecionadas), leia a Seção 1 de `<referee_artifact>` e edite `<caminho_plano>` (`edit_file`):
        - Execute as operações especificadas pelo Árbitro (`MODIFICAR ETAPA X`, `INSERIR NOVA ETAPA ENTRE X E Y` ou `REORDENAR ETAPAS`), sincronizando tanto a seção **Checklist de Execução de Etapas** quanto a **Especificação das Etapas** e preservando a invariante de build entre etapas.
        - Registre no cabeçalho do plano: `- **Debate Técnico**: Validado e revisado (\`discussao/<slug>/03-arbitragem.md\`)`.
3. **Limpeza da Sessão**:
   - Encerre a trinca de subagentes do debate chamando `manage_subagents(Action="kill", ConversationIds=["<red_conv_id>", "<blue_conv_id>", "<referee_conv_id>"])` para liberar memória e processos.
