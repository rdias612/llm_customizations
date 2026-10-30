---
name: plan-executor
description: Executa ou retoma um plano Markdown existente, coordenando implementação e revisão por etapa, atualizando seu checklist e criando commits locais por etapa quando o estado do Git permite.
---

# Plan Executor

Leia [execução no Codex](../../references/runtime.md), incluindo Git por etapa. O agente principal coordena, atualiza o plano e faz commits; a implementação e a revisão são delegadas a papéis distintos quando há subagentes disponíveis.

## Preparar e retomar

1. Leia o plano escolhido e as instruções locais. Identifique a primeira etapa pendente, critérios e comandos de verificação. Resolva contradições impeditivas antes de editar.
2. Registre o estado Git inicial e as alterações preexistentes. Não considere todos os arquivos sujos como alterações desta execução.
3. Atualize o status para `Em Execução`. Se todos os itens já estiverem marcados, confira o status e as evidências da verificação global: itens marcados não provam que a validação final passou. Se estiver ausente ou houver mudanças relevantes, execute a validação antes de declarar conclusão.

## Para cada etapa pendente

1. Registre o estado dos destinos antes da etapa. Delegue ao [implementador](../../references/agents/code-implementer.md) o caminho absoluto do plano, número da etapa, raiz do projeto e lista de escrita permitida. Ele lê a etapa, implementa e executa sua verificação. Guarde seu identificador e aguarde a conclusão.
2. Confira o diff real e o resultado da verificação. Delegue ao [revisor](../../references/agents/code-reviewer.md) plano, etapa, arquivos modificados, baseline e evidências de teste. Escrita de código não é permitida ao revisor. Ele emite `[VEREDITO: APROVADO]` ou `[VEREDITO: NECESSITA AJUSTES]` com evidências.
3. Se houver ajustes, retome o mesmo implementador desta etapa com os itens concretos. Aguarde a correção e validação; depois retome o mesmo revisor. Não solicite implementação e revisão ao mesmo tempo sobre arquivos que ainda estão mudando.
4. Limite a três ciclos de correção ou pare antes diante de impedimento estrutural. Preserve o estado e explique o bloqueio. Quando depender de escolha do usuário, apresente as opções concretas: manter o trabalho e informar a pendência, replanejar apenas etapas pendentes, ou reverter somente alterações da etapa com autorização. Não faça rollback automaticamente.
5. Após aprovação e verificação satisfatória, marque somente esta etapa com `[x]`. Crie o commit local com título semântico e resumo, incluindo a atualização do plano, conforme as condições de Git em runtime. Se não for possível comitar sem incluir trabalho alheio, registre a pendência separadamente e preserve o trabalho. Um checkbox significa implementação validada e revisada, não prova de commit.
6. Não use essa dupla na etapa seguinte; crie agentes novos. Encerre individualmente apenas se houver ferramenta correspondente. Continue até concluir as etapas autorizadas ou precisar de uma decisão indispensável.

## Validação final

- Execute a verificação global declarada no plano sobre o estado final. Reaproveite resultado da última etapa apenas se cobriu exatamente a mesma verificação e não houve alteração relevante posterior.
- Se falhar por regressão desta execução, corrija dentro do escopo autorizado com implementação, revisão e nova validação. Replanejamento que muda escopo exige decisão do usuário. Se faltar ambiente/dependência, registre a limitação real.
- Marque o status global `[x] Concluído` somente após verificação final satisfatória; caso contrário, use `[!] Falha na Verificação Global` ou `[!] Verificação Global Pendente`, conforme o resultado. Mantenha os checkboxes das etapas efetivamente aprovadas.
- Se os commits estiverem habilitados e seguros, inclua a atualização final do status em um commit local; não reescreva commits anteriores. Se houver pendência de commit, informe-a sem confundir com falha de validação do código.
- Entregue link do plano, etapas entregues, verificações, commits efetivos e pendências. Inclua título e descrição curta pronta para PR no próprio chat, sem publicar PR nem criar arquivos extras para isso.
