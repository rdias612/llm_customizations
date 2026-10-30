# Codex

Customizações para o Codex, organizadas em plugins. Cada plugin possui manifesto (`.codex-plugin/plugin.json`), skills, instruções de runtime e papéis de referência para subagentes.

| Plugin | Versão | Descrição | Referência / Guia |
| --- | --- | --- | --- |
| agent-orchestrator | 1.0.0 | Orquestração de subagentes para planejamento, debate técnico, execução por etapas com revisão e commit atômico, ideação, auditoria de code slop e soluções divergentes. | [references/runtime.md](agent-orchestrator/references/runtime.md) |

Estrutura comum de cada plugin: `.codex-plugin/plugin.json`, `references/runtime.md`, `references/agents/`, `skills/` e `assets/`.

## Plugins Disponíveis

### [agent-orchestrator](agent-orchestrator/)

Adaptação para o Codex do Agent Orchestrator:
- **Manifesto**: `.codex-plugin/plugin.json` (ID: `agent-orchestrator-codex`)
- **Guia de Execução**: [references/runtime.md](agent-orchestrator/references/runtime.md) — define escopo, delegação por referência (`collaboration.spawn_agent` / `spawn_agent`), ciclo de Git por etapa e verificações.
- **Skills (`skills/`)**:
  - `plan-designer`: Design de planos técnicos em Markdown com etapas sequenciais e critérios verificáveis (`plans/<slug>.md`).
  - `plan-debater`: Debate adversário entre Red Team, Blue Team e Árbitro Neutro em `discussao/<slug>/`.
  - `plan-executor`: Execução sequencial com isolamento por etapa, revisão estrita por subagente e commits semânticos atômicos.
  - `idea-orchestrator`: Pipeline de ideação em *Blackboard* (exploração, ideação e curadoria crítica em `docs/ideation_report.md`).
  - `find-code-slop`: Auditoria e limpeza de code slop, escape hatches de tipagem, test slop e dead code.
  - `divergent-trio`: Geração paralela de 3 abordagens técnicas ortogonais com síntese comparativa de trade-offs.
- **Papéis de Referência (`references/agents/`)**:
  - `code-implementer.md`: Implementação técnica focada na etapa autorizada.
  - `code-reviewer.md`: Revisão estrita de código e conformidade de diff.
  - `code-slop-auditor.md`: Auditoria search-first de código inflado ou redundante.
  - `divergent-generator.md`: Geração de propostas técnicas alternativas.
  - `feature-evaluator.md`: Avaliação crítica e viabilidade de novas features.
  - `feature-ideator.md`: Ideação divergente em 3 eixos ortogonais.
  - `plan-blue-team.md`: Defesa pragmática e anti-overengineering de planos.
  - `plan-red-team.md`: Análise adversária de falhas, riscos e furos de arquitetura.
  - `plan-referee.md`: Julgamento técnico neutro consolidando arbitragem de debate.
  - `project-explorer.md`: Exploração analítica de escopo e contratos do repositório.
