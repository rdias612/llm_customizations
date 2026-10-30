---
name: idea-orchestrator
description: Explora um projeto, gera propostas de funcionalidades em três eixos e avalia sua viabilidade em relatório compartilhado quando o usuário pede ideação técnica ou um roadmap fundamentado no repositório.
---

# Idea Orchestrator

Leia [execução no Codex](../../references/runtime.md). Use um relatório compartilhado no disco para evitar retransmitir análises completas entre agentes. Este fluxo propõe funcionalidades; não as implementa.

1. Defina o relatório: `docs/ideation_report.md` se `docs/` existir, senão `ideation_report.md`, respeitando destinos solicitados e regras do ambiente. Preserve relatório anterior não escolhido para atualização. Crie cabeçalho com título, data e escopo.
2. Delegue ao [explorador](../../references/agents/project-explorer.md) a análise do projeto e a escrita da **Seção 1: Diagnóstico Técnico e de Escopo**, em duas camadas e no máximo 80 linhas: arquitetura/contratos/restrições e estado atual/dores/lacunas. Aguarde sua conclusão.
3. Delegue ao [ideador](../../references/agents/feature-ideator.md) a leitura da seção 1 e a escrita da **Seção 2: Propostas de Novas Funcionalidades**. Gere de seis a oito propostas concretas distribuídas entre domínio/valor de produto, performance/resiliência/segurança e operabilidade/DX/extensibilidade. Se o escopo não sustentar essa quantidade, prefira menos ideias fundamentadas.
4. Depois da seção 2 concluída, delegue ao [avaliador](../../references/agents/feature-evaluator.md) a leitura do relatório e a escrita das **Seções 3: Avaliação Crítica**, **4: Matriz de Priorização** e **5: Roadmap Sugerido**. Ele confirma o encaixe no código e justifica descartes, refinamentos e prioridades.
5. Confira a integridade das cinco seções e entregue o link do relatório com resumo de prioridades, esforço/impacto e limitações de evidência.

Todos os agentes leem o relatório diretamente e só podem escrever nele, preservando as seções anteriores. Execute as fases sequencialmente para evitar disputa de escrita. Cada agente retorna até 10–15 linhas. Não prometa redução percentual de contexto sem medição.
