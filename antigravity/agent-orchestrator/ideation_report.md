# Relatório de Diagnóstico e Proposta de Evolução do Plugin
- **Data**: 2026-10-20
- **Alvo**: `agent-orchestrator` (Plugin de Orquestração de Agentes)
- **Status da Proposta**: Implementada na `v1.7.0` após refinamento arquitetural e eliminação de over-engineering (Git Step Committer, End-of-Plan Gatekeeper & PR Summary inline, Rollback Cirúrgico Simplificado por Whitelist, Triagem Adaptativa no Pre-Flight com soberania do usuário e Re-anchoring Assistido).

---

## Seção 1: Diagnóstico Técnico e de Escopo

### Camada 1 — Ficha Técnica Arquitetural e Restrições do AGENTS.md
- **Propósito**: Suíte enxuta e de alta performance de orquestração de subagentes para o ecossistema Antigravity, cobrindo o ciclo ponta a ponta de engenharia: design de planos atômicos (com triagem adaptativa), execução sequencial isolada com code review, ideação via *Blackboard* e auditoria search-first de code slop.
- **Stack & Tecnologia**: Especificações declarativas Markdown/YAML ([plugin.json](file:///C:/Users/319029/.gemini/config/plugins/agent-orchestrator/plugin.json) v1.7.0, `SKILL.md`, `agents/*.md`), regras de orquestração ([rules/AGENTS.md](file:///C:/Users/319029/.gemini/config/plugins/agent-orchestrator/rules/AGENTS.md)) e ferramentas nativas (`grep_search`, `find_by_name`, `list_dir`, `edit_file`, `git`).
- **Módulos Principais e Responsabilidades**:
  - `skills/plan-designer`: Concepção de planos atômicos sequenciais (`plans/<slug>.md`) com verificação prévia de contratos no código real, validação polimórfica por etapa e triagem adaptativa no Pre-Flight para micro-tarefas ($\le 2$ arquivos).
  - `skills/plan-executor` + `code-implementer` + `code-reviewer`: Execução sequencial por referência de arquivo; isolamento estrito de duplas por etapa, loop via `send_message`, encerramento com `kill` ao aprovar, Git Step Committer atômico, Rollback Cirúrgico por whitelist da etapa, Re-anchoring Assistido e End-of-Plan Gatekeeper.
  - `skills/idea-orchestrator` + `project-explorer` + `feature-ideator` + `feature-evaluator`: Pipeline de exploração, geração ortogonal (3 eixos) e refinamento crítico em disco (*Blackboard*), poupando >90% de contexto.
  - `skills/find-code-slop` + `code-slop-auditor`: Varredura cirúrgica search-first (modo delta ou full) contra dead code, antipadrões e capturas silenciosas.
- **Restrições Inegociáveis (`AGENTS.md` e Agentes)**:
  - **Zero Poluição de Contexto**: Subagentes configurados com `inheritCustomizations: false` e `inheritMcp: false`; passagem estrita por referência de arquivo sem copy-paste de código.
  - **Precedência Local**: Diretrizes locais (`AGENTS.md` / `GEMINI.md`) têm prioridade absoluta sobre convenções gerais.
  - **Avanço Condicionado & Commit Atômico**: Nenhuma etapa avança sem `[VEREDITO: APROVADO]` do `code-reviewer`, atualização imediata do checkbox (`- [x]`) e commit semântico atômico com título e descrição do que foi feito (em repositórios Git).
  - **Integridade de I/O e Sem Rebuild Redundante**: Edição estritamente por ferramentas nativas (sem scripts de shell gravando arquivos para evitar corrupção de encoding no Windows); revisor inspeciona apenas diff isolado (`git --no-pager diff -- <arquivos>`) sem re-executar builds já aprovados.

### Camada 2 — Dores Identificadas e Resolvidas
1. **Falta de Atomicidade no Git**: Resolvida via **Git Step Committer** com título semântico e descrição resumida do que foi feito por etapa.
2. **Ausência de Rollback Seguro**: Resolvida via **Rollback Cirúrgico Simplificado** baseado exclusivamente na whitelist dos arquivos tocados na etapa atual, sem comandos globais destrutivos nem pastas temporárias extras.
3. **Falta de Fechamento de Ciclo (End-of-Plan)**: Resolvida via **End-of-Plan Gatekeeper**, que executa o comando de build global ao final do plano e entrega o resumo de PR pronto diretamente no chat/plano, sem criar arquivos `.pr.md` órfãos.
4. **Custo Alto para Fixes Pequenos**: Resolvida via **Triagem Adaptativa no Pre-Flight** do `plan-designer`, que identifica micro-tarefas ($\le 2$ arquivos) antes de qualquer edição e oferece o Fast-Track respeitando a soberania do desenvolvedor.
5. **Inflexibilidade em Voo**: Resolvida via **Re-anchoring Assistido**, que pausa e consulta o usuário via `ask_question` caso um impedimento arquitetural real exija reescrever as etapas pendentes (`- [ ]`), preservando as concluídas (`- [x]`).

---

## Seção 2: Propostas Consolidadas e Refinadas (Implementadas na v1.7.0)

### 1. Git Step Committer (Commits Semânticos Atômicos por Etapa com Título e Descrição)
- **Status**: **Implementado**.
- **Objetivo**: Registrar imutavelmente no Git cada etapa validada e aprovada, com título padronizado e descrição concisa do que foi alterado.
- **Funcionamento**: No [skills/plan-executor/SKILL.md](file:///C:/Users/319029/.gemini/config/plugins/agent-orchestrator/skills/plan-executor/SKILL.md), ao receber `[VEREDITO: APROVADO]` do revisor:
  1. Executa pre-flight check de Git (`git rev-parse --is-inside-work-tree`).
  2. Extrai do relatório do `code-implementer` o resumo objetivo do que foi feito (1-3 frases sucintas).
  3. Atualiza o `- [x] Etapa K: [Nome]` no plano markdown.
  4. Executa o commit atômico contendo título semântico e descrição resumida:
     ```bash
     git add -- "<arquivos_tocados_na_etapa>" "<caminho_do_plano>"
     git commit --no-verify --no-gpg-sign -m "feat(<slug>): [Etapa K] <nome da etapa>" -m "<descricao_breve_e_resumida_do_que_foi_feito>"
     ```

### 2. Rollback Cirúrgico Simplificado por Whitelist da Etapa
- **Status**: **Implementado (Refinado sem `.agent/quarantine/` nem snapshots por etapa)**.
- **Objetivo**: Restaurar a integridade do repositório caso uma etapa esgote as 3 tentativas de ajuste e o usuário opte por reverter, sem afetar etapas anteriores (`1..K-1`) nem tocar em arquivos untracked preexistentes do desenvolvedor.
- **Funcionamento**:
  1. Como as etapas $1 \dots K-1$ já estão commitadas no `HEAD` e o `code-implementer` reporta a lista exata de arquivos criados/modificados na Etapa $K$, não há necessidade de snapshots prévios ou pastas de quarentena.
  2. Se o usuário escolher reverter a Etapa $K$ no `ask_question`:
     - Reverte exclusivamente os arquivos modificados da etapa (`git checkout HEAD -- "<arquivos_modificados_da_etapa>"`).
     - Trata apenas os arquivos novos listados pelo implementador naquela etapa conforme confirmação do usuário, mantendo 100% intactos quaisquer rascunhos untracked preexistentes.

### 3. End-of-Plan Gatekeeper & Resumo de PR Inline
- **Status**: **Implementado (Refinado sem geração de arquivo `.pr.md` solto)**.
- **Objetivo**: Fechamento formal do ciclo de entrega com validação de build global e sumário de PR pronto para uso sem sujar o working tree após o último commit.
- **Funcionamento**: Após a conclusão e commit de todas as etapas pendentes no [skills/plan-executor/SKILL.md](file:///C:/Users/319029/.gemini/config/plugins/agent-orchestrator/skills/plan-executor/SKILL.md):
  1. O orquestrador executa o **Comando de Build Global** declarado na seção 1 do plano.
  2. Se o build global falhar, atualiza o cabeçalho para `- **Status**: [!] Falha no Build Global` e consulta o usuário para abrir uma etapa corretiva de hotfix.
  3. Se aprovado, atualiza o status para `- **Status**: [x] Concluído` (incluindo essa atualização no commit final ou mantendo a árvore limpa) e apresenta diretamente no chat o **Resumo de PR** estruturado (título, sumário executivo das etapas e diff stat), sem criar arquivos `.pr.md` extras no repositório.

### 4. Triagem Adaptativa no Pre-Flight & Fast-Track com Soberania do Desenvolvedor
- **Status**: **Implementado (Refinado sem Circuit Breaker em voo)**.
- **Objetivo**: Evitar a burocracia de criar e revisar um arquivo `plans/<slug>.md` completo para micro-tarefas ($\le 2$ arquivos), preservando a soberania do desenvolvedor.
- **Funcionamento**: No Passo 1 (*Inspeção Cirúrgica Focada*) do [skills/plan-designer/SKILL.md](file:///C:/Users/319029/.gemini/config/plugins/agent-orchestrator/skills/plan-designer/SKILL.md):
  1. A decisão entre **Fast-Track** e **Standard Track** é tomada **100% no Pre-Flight** pelo arquiteto.
  2. **Micro-tarefa ($\le 2$ arquivos existentes, sem novos módulos arquiteturais)**:
     - Se o usuário pediu apenas para *planejar*, o arquiteto oferece via `ask_question` executar diretamente via Fast-Track (1 etapa atômica com `code-implementer` + `code-reviewer` + commit atômico) ou gerar o arquivo formal `plans/<slug>.md`.
     - Se o usuário já autorizou a execução direta, aciona imediatamente a dupla `code-implementer` + `code-reviewer` e finaliza com o Git Step Committer.
  3. **Demanda padrão ou complexa ($\ge 3$ arquivos ou múltiplos passos interdependentes)**:
     - Segue o **Standard Track**, gerando `plans/<slug>.md` com checklist sequencial para o `plan-executor`.

### 5. Re-anchoring Assistido (Replanejamento de Etapas Pendentes sob Confirmação)
- **Status**: **Implementado (Refinado com aprovação obrigatória do usuário)**.
- **Objetivo**: Permitir readequação das etapas pendentes (`- [ ]`) caso surja uma incompatibilidade técnica real durante a execução, sem permitir que subagentes reescrevam o plano de forma autônoma/silenciosa.
- **Funcionamento**:
  1. Caso o `code-implementer` e o `code-reviewer` identifiquem um impedimento estrutural real que inviabilize a etapa atual ou as etapas seguintes, o `plan-executor` pausa o ciclo e consulta o usuário via `ask_question` apresentando o diagnóstico.
  2. Apenas se o usuário aprovar o replanejamento, o orquestrador atualiza as etapas pendentes (`- [ ]`) no arquivo `plans/<slug>.md` (mantendo todas as etapas `- [x]` já concluídas e commitadas intactas) e retoma a execução.

---

## Seção 3: Matriz Consolidada (Esforço vs. Impacto)

| ID | Funcionalidade | Status | Impacto | Módulos Afetados | Complexidade Adicionada |
|---|---|---|---|---|---|
| **1** | **Git Step Committer** | Concluído | **Alto** | `skills/plan-executor/SKILL.md`, `rules/AGENTS.md` | Mínima |
| **3** | **End-of-Plan Gatekeeper & PR Inline** | Concluído | **Muito Alto** | `skills/plan-executor/SKILL.md` | Mínima (sem `.pr.md` extra) |
| **2** | **Rollback Cirúrgico Simplificado** | Concluído | **Alto** | `skills/plan-executor/SKILL.md` | Mínima (via whitelist da etapa) |
| **4** | **Triagem Adaptativa (Pre-Flight Fast-Track)** | Concluído | **Muito Alto** | `skills/plan-designer/SKILL.md`, `rules/AGENTS.md` | Baixa (decisão no Pre-Flight) |
| **5** | **Re-anchoring Assistido** | Concluído | **Alto** | `skills/plan-executor/SKILL.md` | Baixa (sob confirmação humana) |

---

## Seção 4: Diretrizes Inegociáveis de Engenharia
1. **Regra da Não-Destrutividade**: Nenhuma automação executa `git clean` global; reversões atuam estritamente sobre a whitelist de arquivos tocados na etapa atual.
2. **Isolamento de Erro do Git**: Comandos Git com falha degradam graciosamente para modo *Checklist-Only* sem abortar o fluxo do desenvolvedor.
3. **Soberania do Desenvolvedor**: Decisões de Fast-Track (quando solicitado planejamento) e replanejamento de etapas pendentes (*Re-anchoring*) passam sempre pela confirmação do usuário.
