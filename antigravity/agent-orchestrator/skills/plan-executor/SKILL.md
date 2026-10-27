---
name: plan-executor
description: >-
  Use esta skill quando o usuário pedir para executar ou continuar a implementação
  de um arquivo de plano markdown (.md) existente (ex: em 'plans/').
  Orquestra os subagentes 'code-implementer' e 'code-reviewer' etapa por etapa,
  atualizando o checklist (- [x]) diretamente no arquivo a cada aprovação.
---

# Plan Executor (Orquestrador de Execução Sequencial)

Esta skill consome um arquivo de plano (`.md`) e coordena os subagentes `code-implementer` e `code-reviewer` (já registrados pelo plugin) até concluir todas as etapas pendentes.

## Princípios de Eficiência de Contexto e Qualidade
1. **Orquestrador Não Edita Código de Produção**: Você edita exclusivamente os checkboxes (`- [x]`), o status do arquivo de plano e comita as alterações aprovadas no Git.
2. **Passagem por Referência (Sem Copy-Paste)**: Não copie o texto inteiro da etapa nem o `AGENTS.md` dentro do prompt do subagente. Ambos operam com `Workspace: "inherit"`: passe apenas o caminho do arquivo de plano e o número/nome da etapa para que o próprio subagente leia a seção correspondente.
3. **Isolamento Estrito por Etapa e Reutilização no Loop (`send_message`)**:
   - Cada Etapa $K$ possui sua própria dupla dedicada (`code-implementer` + `code-reviewer`).
   - Se o `code-reviewer` pedir ajustes na Etapa $K$, **não crie um novo subagente** via `invoke_subagent`. Use `send_message` para o `conversationId` do `code-implementer` da Etapa $K$ atual e depois `send_message` para o `conversationId` do `code-reviewer` da Etapa $K$ atual, aproveitando o contexto em cache deles.
   - Assim que a Etapa $K$ receber `[VEREDITO: APROVADO]`, encerre os subagentes da Etapa $K$ com `manage_subagents(Action="kill")` para liberar recursos e inicie a Etapa $K+1$ com subagentes novos e contexto 100% limpo.
4. **Git Step Committer Atômico**: Cada etapa aprovada gera um commit imutável no Git com mensagem estruturada (título semântico + descrição breve do que foi feito), garantindo rastreabilidade e histórico limpo.

---

## Fluxo de Execução

### Passo 1: Leitura do Plano e Pre-flight
1. Leia o arquivo do plano e localize na seção de checklist a primeira etapa pendente (`- [ ] Etapa K`).
2. Se todas estiverem marcadas com `[x]`, avise que o plano já foi concluído.
3. Caso o cabeçalho ainda esteja como `- **Status**: [ ] Planejado`, atualize-o para `- **Status**: [ ] Em Execução`.
4. **Git Pre-Flight Check**:
   - Execute `git rev-parse --is-inside-work-tree` via `run_command`.
   - Se for repositório Git: confirme se `user.name` e `user.email` estão configurados (`git config user.name`). Se não estiverem, adicione argumentos transientes (`-c user.name="Agent Orchestrator" -c user.email="orchestrator@agent.local"`) nos comandos de commit subsequentes.
   - Se NÃO for repositório Git: desative os commits e opere graciosamente em modo *Checklist-Only*.

### Passo 2: Execução Sequencial da Etapa $K$

#### 2.1. Implementação (`code-implementer`)
Invoque `invoke_subagent`:
- `TypeName`: `"code-implementer"`
- `Role`: `"Implementer - Etapa K"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Implemente a **Etapa K: [Nome]** especificada no arquivo `<caminho_do_plano>`. Leia apenas a seção 1 e a Etapa K no plano e respeite estritamente o `AGENTS.md` do projeto. Execute a verificação da etapa (ou comando de build do projeto) antes de concluir e retorne a lista exata de arquivos criados/modificados, um resumo objetivo e conciso (1-3 frases) do que foi implementado e o resultado da verificação."

Guarde o `conversationId` retornado para o implementador desta etapa.

#### 2.2. Revisão de Código (`code-reviewer`)
Invoque `invoke_subagent`:
- `TypeName`: `"code-reviewer"`
- `Role`: `"Reviewer - Etapa K"`
- `Workspace`: `"inherit"`
- `Prompt`:
  > "Revise a implementação da **Etapa K: [Nome]** do plano `<caminho_do_plano>`. Arquivos tocados pelo implementador: `<lista_de_arquivos>` (Verificação reportada pelo implementador: `<resultado_verificacao>`). Inspecione o diff apenas desses arquivos (`git --no-pager diff -- <lista_de_arquivos>` e leitura direta para arquivos novos untracked), valide todos os critérios de aceitação da Etapa K e as regras do `AGENTS.md` sem re-executar builds pesados desnecessariamente, e emita `[VEREDITO: APROVADO]` ou `[VEREDITO: NECESSITA AJUSTES]`."

Guarde o `conversationId` do revisor desta etapa.

#### 2.3. Ciclo de Ajustes, Rollback Cirúrgico e Re-anchoring Assistido (Se `NECESSITA AJUSTES`)
1. Envie a lista de correções ao implementador da Etapa $K$ usando `send_message(Recipient=<implementer_conv_id>, Message=...)`.
2. Após a correção, peça nova verificação ao revisor da Etapa $K$ usando `send_message(Recipient=<reviewer_conv_id>, Message="Verifique os ajustes aplicados nos arquivos <arquivos_corrigidos> inspecionando apenas o diff atualizado e emita o novo veredito.")`.
3. **Limite de 3 iterações ou Impedimento Estrutural**: Se após 3 tentativas o revisor mantiver `NECESSITA AJUSTES` (ou se ambos reportarem uma incompatibilidade arquitetural real com as etapas seguintes), encerre os subagentes da Etapa $K$ e consulte o usuário com `ask_question` oferecendo:
   - **Rollback Cirúrgico da Etapa $K$**: reverta exclusivamente os arquivos modificados nesta etapa (`git checkout HEAD -- "<arquivo1>" "<arquivo2>"`, mantendo intactos todos os commits das etapas $1 \dots K-1$ e jamais executando `git clean` global sobre arquivos untracked do usuário).
   - **Re-anchoring Assistido (Replanejamento das Etapas Pendentes)**: com a confirmação do usuário, reescreva no arquivo de plano apenas a Etapa $K$ e as etapas pendentes (`- [ ]`), preservando 100% congeladas as etapas já concluídas (`- [x]`), e retome a execução.

#### 2.4. Persistência do Avanço, Commit Atômico e Limpeza da Sessão
Assim que receber `[VEREDITO: APROVADO]`:
1. Atualize o arquivo do plano para marcar `- [x] Etapa K: [Nome]` (e, se esta for a última etapa do plano, atualize também o cabeçalho para `- **Status**: [x] Concluído`).
2. **Git Step Committer (Commit Atômico com Título e Descrição)**:
   - Se em repositório Git:
     - Extraia o resumo objetivo reportado pelo `code-implementer` sobre o que foi feito na etapa.
     - Adicione os arquivos tocados na etapa e o arquivo do plano (com barras `/` e entre aspas duplas):
       `git add -- "<arquivo1>" "<arquivo2>" "<caminho_do_plano>"`
     - Execute o commit atômico contendo o título da etapa e a descrição breve e resumida do que foi feito nela (com bypass de pre-commit hooks e de assinatura interativa):
       `git commit --no-verify --no-gpg-sign -m "feat(<slug>): [Etapa K] <Nome da Etapa>" -m "<descricao_breve_e_resumida_do_que_foi_feito>"`
       *(Nota: Se houver bloqueio em `.git/index.lock`, aguarde 500ms e tente novamente até 2 vezes. Se retornar `nothing to commit`, continue normalmente).*
3. Encerre a dupla de subagentes da Etapa $K$ concluída chamando `manage_subagents(Action="kill", ConversationIds=["<implementer_conv_id>", "<reviewer_conv_id>"])`.
4. Prossiga imediatamente para a próxima etapa pendente (`Etapa K+1`).

### Passo 3: End-of-Plan Gatekeeper e Resumo de PR Inline
Quando todas as etapas estiverem com `[x]`:
1. **Gatekeeper de Build Global**:
   - Execute via `run_command` o **Comando de Build Global** declarado na seção 1 do plano (caso não tenha sido o mesmo comando já executado na última etapa).
   - Se o build global **falhar**: atualize o cabeçalho do plano para `- **Status**: [!] Falha no Build Global` e consulte o usuário via `ask_question` para executar uma etapa corretiva de hotfix.
2. **Resumo de PR Inline (Sem Arquivos `.pr.md` Extras)**:
   - Certifique-se de que o cabeçalho do plano esteja como `- **Status**: [x] Concluído`.
   - Apresente diretamente no chat um resumo executivo contendo:
     - Link clicável para o plano concluído e lista das etapas entregues.
     - Bloco pronto para Pull Request / Merge Request (Título semântico + Sumário de alterações por etapa), sem criar arquivos `.pr.md` soltos no repositório.
