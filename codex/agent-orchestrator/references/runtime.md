# Execução no Codex

Leia este arquivo ao iniciar qualquer skill deste plugin. Os papéis em `references/agents/` são instruções para subagentes genéricos, não agentes registrados automaticamente. Resolva os links em relação ao arquivo que os contém, nunca ao diretório do projeto.

## Escopo e ferramentas

- Siga as instruções do usuário, do ambiente e os `AGENTS.md` aplicáveis. Este plugin não amplia permissões. Use documentação adicional do projeto quando relevante.
- Busque primeiro com `rg` e `rg --files`; leia apenas os trechos necessários via ferramenta de shell disponível. Edite com `apply_patch` quando disponível; uma alternativa precisa preservar UTF-8 e as quebras de linha existentes.
- Use as ferramentas realmente expostas na sessão. Não execute nomes de ferramentas como comandos de shell. Não instale dependências nem use serviços externos apenas para satisfazer um exemplo deste plugin.
- Em um projeto, mantenha os destinos relativos à raiz do projeto; em chat sem projeto, respeite os diretórios de trabalho e entregáveis definidos pelo ambiente. Caminhos fornecidos pelo usuário prevalecem. Não sobrescreva relatórios existentes de outra execução; use um sufixo livre ou atualize somente a execução explicitamente retomada.
- Relatórios completos ficam em Markdown; apresente resumos e links com caminhos absolutos no chat, por exemplo `[relatório](C:/projeto/docs/relatorio.md)` e `[arquivo](C:/projeto/src/app.ts:42)`. Respeite eventuais restrições de links da interface. Não dependa de APIs de artefatos ou de visualização de outro aplicativo.

## Delegação por referência

1. Antes de delegar, resolva os caminhos absolutos da raiz do projeto, deste arquivo, do papel escolhido, do plano/relatório e dos arquivos de saída autorizados.
2. Crie um subagente genérico usando a ferramenta disponível: `collaboration.spawn_agent` no desktop quando exposta, ou `spawn_agent` em clientes que a disponibilizem. O nome do papel é uma descrição da tarefa, não um valor de tipo de agente registrado.
3. No desktop, use `task_name` único e `fork_turns: "none"` quando disponíveis. Em outros clientes, escolha contexto novo quando suportado. Não fixe modelo nem esforço sem solicitação ou configuração aplicável; mantenha o padrão do ambiente.
4. Envie um prompt curto com as referências: "Leia as instruções de execução em CAMINHO_RUNTIME e o papel em CAMINHO_PAPEL. Trabalhe no projeto RAIZ. Execute TAREFA/ETAPA lendo PLANO/RELATÓRIO. Escrita permitida apenas em DESTINOS. Retorne resumo, arquivos e evidências de verificação." Inclua restrições explícitas do usuário que não estejam nesses arquivos. O subagente deve ler o `AGENTS.md` aplicável antes de agir.
5. Guarde o identificador retornado. Aguarde a conclusão com a ferramenta de espera/status disponível (`collaboration.wait_agent` e notificações no desktop, ou `wait` de subagentes quando exposta); não confunda com espera de processos shell. Leia o resultado e confirme a existência do relatório ou diff antes de iniciar uma fase dependente.
6. Para ajustes no desktop, `collaboration.followup_task` retoma um agente ocioso; `collaboration.send_message` envia orientação a um agente em execução. Em outros clientes, use a ferramenta documentada de envio/retomada. Não presuma que uma mensagem para um agente ocioso inicia uma nova execução.
7. Ao terminar uma etapa, não reutilize seus agentes na etapa seguinte. Use encerramento individual se a sessão oferecer essa capacidade; caso contrário, deixe o agente concluído sem novas tarefas. Interrompa apenas agentes deste fluxo ainda em execução quando necessário. Não use cancelamento global nem afirme que agentes concluídos foram destruídos ou que a memória foi liberada.

Os subagentes compartilham arquivos e herdam permissões da sessão. Contexto novo e restrições no prompt não criam um sandbox independente. Um revisor é somente leitura por instrução, sem garantia de isolamento pelo sistema. Cada escritor recebe uma lista explícita de destinos; fases que escrevem no mesmo relatório são sequenciais.

Se não houver ferramentas de subagentes ou capacidade suficiente, informe a limitação e execute os papéis sequencialmente no agente principal, mantendo os mesmos critérios e artefatos. Não apresente essa execução como revisão independente ou concorrência real. Não crie chats do usuário nem processos externos de Codex para simular subagentes.

## Decisões e autorização

- Preserve autorizações já dadas. Planejar, debater ou auditar não autoriza implementar. Se o usuário também pediu aplicação/correção, conclua o escopo autorizado sem perguntar novamente.
- Para uma decisão necessária ainda não autorizada, use a ferramenta de perguntas disponível, como `request_user_input_async`; se indisponível, pergunte no chat. Perguntas assíncronas exigem resposta antes da ação dependente; silêncio não é aprovação. Não dependa de seleção múltipla ou de uma ferramenta restrita a Plan mode.
- Dentro da tarefa autorizada, prossiga nas verificações e correções necessárias; pergunte apenas quando uma escolha material do usuário for indispensável.

## Git por etapa

- Antes de editar, registre `git status --short`, HEAD (se existir), alterações staged/unstaged e arquivos novos nos destinos da etapa e no plano. Identifique também alterações staged fora do escopo. Em repositório sem commit inicial, não use comandos que exigem HEAD; examine o índice e os arquivos diretamente.
- Commits locais por etapa fazem parte do fluxo de `plan-executor`, salvo restrição do usuário/projeto. Não publique branches ou PRs. Fora de Git, opere só com checklist. Não invente autor: sem identidade Git configurada, preserve o trabalho e informe que os commits ficaram pendentes.
- Stage apenas alterações desta etapa. Se os destinos estavam limpos e não há alterações staged alheias, use `git add -- CAMINHOS_EXPLÍCITOS` e confira `git diff --cached` antes de `git commit -m TITULO -m RESUMO`. Escolha `feat`, `fix`, `refactor` ou `docs` conforme a mudança real. Preserve hooks e assinatura.
- Se houver alterações anteriores sobrepostas ou conteúdo staged alheio, não faça commit indiscriminado. Preserve índice e arquivos do usuário; só comite se conseguir isolar com segurança os próprios hunks. Caso contrário, mantenha as alterações sem commit e informe o motivo, continuando o trabalho autorizado quando possível. Nunca use `git add .`, desative hooks/assinatura, altere identidade/configuração Git ou apague locks para contornar um bloqueio.
- Um erro real de commit não equivale a sucesso. Inspecione o estado após hooks, pois eles podem modificar arquivos. Corrija apenas problemas introduzidos na etapa dentro do escopo; informe pendências que dependem do usuário.
- Rollback exige autorização correspondente. Inverta apenas os hunks produzidos na etapa, usando o estado registrado antes dela. Remova arquivos novos somente se criados por esta etapa e sem alterações posteriores de terceiros. Não restaure arquivos inteiros a HEAD nem faça limpeza global. Se não for possível separar alterações, preserve o estado e peça a decisão necessária.

## Verificação e conclusão

Use os comandos configurados pelo projeto e adequados à alteração. O resultado deve incluir comando e diretório, sucesso/falha real e limitações. Não reporte teste não executado como aprovado. Não repita uma verificação pesada sem mudança ou dúvida nova. Resultado de código revisado deixa de cobrir um diff que mudou depois; valide/revise novamente o delta relevante.
