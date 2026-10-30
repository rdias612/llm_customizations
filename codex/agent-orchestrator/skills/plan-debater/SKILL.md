---
name: plan-debater
description: Submete um plano técnico existente a crítica Red Team, defesa Blue Team e arbitragem baseada no código quando o usuário pede debater, estressar ou validar o plano antes de implementar.
---

# Plan Debater

Leia [execução no Codex](../../references/runtime.md). O debate produz relatórios; não modifica código de produção. Alterar o plano depende do escopo solicitado pelo usuário.

1. Localize o plano indicado, confirme o escopo no código e escolha `discussao/slug/` como diretório de saída, respeitando destinos do usuário/ambiente e preservando execuções anteriores.
2. Delegue ao [Red Team](../../references/agents/plan-red-team.md) a leitura do plano e dos contratos reais. Escrita permitida apenas em `01-red-team.md` no diretório desta execução. Aguarde e leia o resultado. Se houver zero críticas fundamentadas, entregue esse resultado com link; não invente debate nem declare arbitragem que não ocorreu.
3. Havendo críticas, delegue ao [Blue Team](../../references/agents/plan-blue-team.md) a leitura do plano e do relatório Red. Escrita permitida apenas em `02-blue-team.md`. Aguarde sua conclusão.
4. Delegue ao [árbitro](../../references/agents/plan-referee.md) a leitura do plano e dos dois relatórios; ele verifica divergências no código e escreve apenas `03-arbitragem.md`.
5. Confira que todos os identificadores Red foram respondidos e julgados. Apresente no chat a contagem e uma síntese autossuficiente das críticas aceitas e ignoradas, com links para os três relatórios. Não crie um arquivo adicional apenas para a visualização da UI.
6. Se o usuário já pediu para aplicar os ajustes procedentes, aplique-os ao plano, preserve etapas concluídas e confira checklist, ordem, arquivos e verificações. Caso tenha pedido somente debate/validação, entregue a arbitragem e peça uma decisão apenas se houver ajustes a aplicar. Ofereça aplicar os aceitos, escolher itens pelos identificadores ou manter o plano, contextualizando etapa e efeito de cada ajuste. Aguarde resposta antes de modificar o plano.

Na retomada, use os relatórios desta execução, releia o plano e confirme que as evidências ainda se aplicam. Aplicar ajustes ao plano não inicia sua implementação. Não reutilize os agentes do debate para implementar código.
