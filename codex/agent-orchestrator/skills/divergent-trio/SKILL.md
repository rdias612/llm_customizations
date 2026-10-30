---
name: divergent-trio
description: Gera três abordagens técnicas distintas em arquivos separados e compara seus trade-offs no chat quando o usuário pede três exemplos, soluções ou propostas alternativas para uma mesma demanda.
---

# Divergent Trio

Leia [execução no Codex](../../references/runtime.md). Gere três alternativas realmente diferentes, cada uma em seu arquivo; a síntese comparativa fica no chat.

1. Confirme pelo pedido a demanda, o formato esperado e o destino. Use o diretório informado; na ausência dele, `proposals/` para propostas ou `examples/` para exemplos, respeitando os destinos do ambiente. Escolha extensões adequadas ao conteúdo e caminhos que não sobrescrevam trabalho existente.
2. Use os três eixos pedidos pelo usuário. Se não os definiu, adote: **minimalista/pragmático**, **modular com responsabilidades explícitas** e **alternativa técnica pertinente ao problema**. Modularidade não justifica arquitetura empresarial especulativa; a alternativa precisa resolver o mesmo problema com restrições comparáveis.
3. Delegue três tarefas ao papel [gerador divergente](../../references/agents/divergent-generator.md), uma por eixo. Cada tarefa recebe demanda, eixo, formato, referências necessárias e seu próprio caminho absoluto de escrita. Escrita permitida somente nesse arquivo, sem modificar código compartilhado, configurações ou artefatos dos outros agentes.
4. Dispare as tarefas de forma concorrente quando houver três vagas. Se o ambiente limitar a concorrência, execute em lotes ou sequencialmente e informe essa limitação. Aguarde todos os resultados antes de comparar; não misture sucesso de duas opções com conclusão das três.
5. Confira a existência e o conteúdo de cada arquivo, sua aderência ao pedido, distinção entre as abordagens e evidências de execução, quando aplicáveis. Corrija uma opção incompleta retomando somente seu autor. Cada gerador retorna até oito linhas com caminho, vantagem, custos e verificações reais.
6. Apresente três links e uma tabela com abordagem, complexidade, pontos fortes, trade-offs e cenário indicado. Recomende uma opção com base nas restrições do usuário. Não crie arquivo adicional de consolidação, salvo solicitação.

Proposta conceitual não exige código executável. Exemplo anunciado como executável deve ter suas verificações realizadas quando o ambiente permitir; declare dependências e limitações. Não instale três soluções no projeto nem escolha uma para integrar sem pedido correspondente.
