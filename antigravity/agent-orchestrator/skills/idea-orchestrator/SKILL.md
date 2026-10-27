---
name: idea-orchestrator
description: >-
  Use esta skill quando o usuário solicitar explorar um projeto, analisar o
  escopo existente e propor novas ideias de funcionalidades. Coordena os subagentes
  'project-explorer', 'feature-ideator' e 'feature-evaluator' usando escrita direta em
  arquivo (Blackboard) para economizar janela de contexto, salvando o relatório final em 'docs/ideation_report.md' ou na raiz.
---

# Orquestrador de Exploração e Ideação (`idea-orchestrator`)

Esta skill coordena um pipeline sequencial de 3 subagentes (`project-explorer` $\rightarrow$ `feature-ideator` $\rightarrow$ `feature-evaluator`) para diagnosticar o repositório, gerar propostas de funcionalidades e filtrá-las criticamente.

## Arquitetura *Blackboard* (Zero Retransmissão de Contexto)
Para evitar estourar a janela de contexto do agente principal copiando relatórios inteiros de um subagente para outro:
- O arquivo de destino (`docs/ideation_report.md` se a pasta `docs/` existir, ou `ideation_report.md` na raiz) funciona como **quadro compartilhado em disco (*Blackboard*)**.
- Cada subagente lê as seções anteriores diretamente do arquivo no disco, escreve a sua própria seção nele e retorna ao Orquestrador apenas um resumo curto (10–15 linhas).

---

## Fluxo de Execução

### Passo 1: Inicializar o Arquivo de Relatório
1. Verifique se o diretório `docs/` existe no projeto atual:
   - Se existir: defina `<caminho_relatorio>` como `docs/ideation_report.md`.
   - Se não existir: defina `<caminho_relatorio>` como `ideation_report.md` na raiz.
2. Crie o cabeçalho inicial do arquivo (`Overwrite: true`):
   ```markdown
   # Relatório de Diagnóstico e Ideação de Funcionalidades
   - **Data**: AAAA-MM-DD
   ```

### Passo 2: Fase 1 — Diagnóstico (`project-explorer`)
Invoque `invoke_subagent`:
- `TypeName`: `"project-explorer"`
- `Role`: `"Project Explorer"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Explore este repositório (ou o escopo específico solicitado pelo usuário) usando as ferramentas nativas de busca (`list_dir`, `find_by_name`, `grep_search`) e leitura focada (`README.md`, `AGENTS.md`, configurações e módulos principais). Adicione a **Seção 1: Diagnóstico Técnico e de Escopo** diretamente ao final do arquivo `<caminho_relatorio>` estruturada em 2 camadas (1. Ficha Técnica Arquitetural e Restrições do `AGENTS.md`; 2. Estado Atual, Dores e Lacunas) com no máximo 80 linhas. Retorne apenas um resumo de até 10 linhas."

### Passo 3: Fase 2 — Geração de Propostas (`feature-ideator`)
Após a conclusão do `project-explorer`, invoque `invoke_subagent`:
- `TypeName`: `"feature-ideator"`
- `Role`: `"Feature Ideator"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Leia a Seção 1 do arquivo `<caminho_relatorio>`. Gere de 6 a 8 propostas técnicas concretas e específicas para este projeto, distribuídas obrigatoriamente em 3 eixos ortogonais (1. Core Domain & Valor de Produto; 2. Performance, Resiliência & Segurança; 3. Operabilidade, DX & Extensibilidade), detalhando problema resolvido, funcionamento e encaixe nos módulos existentes. Adicione a **Seção 2: Propostas de Novas Funcionalidades** diretamente ao final de `<caminho_relatorio>` e retorne apenas os títulos das propostas."

### Passo 4: Fase 3 — Avaliação Crítica e Priorização (`feature-evaluator`)
Após a conclusão do `feature-ideator`, invoque `invoke_subagent`:
- `TypeName`: `"feature-evaluator"`
- `Role`: `"Feature Evaluator & Tech Lead"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Leia as Seções 1 e 2 do arquivo `<caminho_relatorio>`. Avalie criticamente cada proposta contra a arquitetura real e o `AGENTS.md`. Adicione ao final de `<caminho_relatorio>`: **Seção 3: Avaliação Crítica (Descartes Justificados, Ideias Aprovadas Refinadas e Ideias Sintéticas)**, **Seção 4: Matriz de Priorização (Quick Wins, Médio Prazo, Estratégicas)** e **Seção 5: Roadmap Sugerido**. Encerre a sessão e retorne no chat apenas o resumo executivo final."

### Passo 5: Limpeza e Apresentação ao Usuário
1. Encerre os subagentes finalizados com `manage_subagents(Action="kill_all")`.
2. Apresente no chat o resumo executivo retornado pelo avaliador e o link clicável para `[ideation_report.md](file:///<caminho_relatorio>)`.
