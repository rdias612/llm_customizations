---
name: find-code-slop
description: >-
  Use esta skill quando o usuário solicitar detectar, auditar ou encontrar
  "code slop" (código inchado, redundante, abstrações prematuras, comentários óbvios de IA,
  escape hatches de tipagem, test slop, dead code ou tratamento defensivo silencioso)
  no projeto, nas alterações atuais do Git ou em arquivos específicos.
---

# Detector e Auditor de Code Slop (`find-code-slop`)

Esta skill identifica cirurgicamente **code slop** — falhas silenciosas, *escape hatches* de tipagem (`any`/`# type: ignore`), over-engineering, *test slop*, código morto e comentários óbvios de IA — mantendo a janela de contexto principal limpa através do subagente `code-slop-auditor` e delegando a limpeza autorizada ao `code-implementer`.

---

## Fluxo de Execução

### Passo 1: Delimitar o Escopo e o Modo de Saída
Identifique o escopo solicitado pelo usuário:
1. **Arquivo único pequeno**: você pode inspecioná-lo diretamente ou delegar ao `code-slop-auditor`.
2. **Modo Delta (Alterações da branch / working tree atual)**: quando o usuário pedir para auditar o que foi alterado recentemente, defina o escopo como os arquivos modificados (*staged* e *unstaged*) e novos (*untracked*) obtidos via `git status --porcelain` e `git --no-pager diff HEAD --name-only`.
3. **Diretório amplo ou Repositório completo (Modo *Blackboard*)**:
   - Delegue obrigatoriamente ao subagente `code-slop-auditor`.
   - Defina `<caminho_relatorio>` como `docs/slop_audit_report.md` (se a pasta `docs/` existir) ou `slop_audit_report.md` na raiz, para que o subagente persista o laudo completo em disco sem estourar a janela de contexto principal.

### Passo 2: Executar Auditoria via `code-slop-auditor`
Invoque `invoke_subagent` (passando apenas a referência de escopo e saída, sem duplicar as regras já presentes no prompt de sistema do agente):
- `TypeName`: `"code-slop-auditor"`
- `Role`: `"Code Slop Auditor"`
- `Workspace`: `"inherit"`
- `Prompt`:
  - **Para escopo focado / Modo Delta**:
    > "Audite o escopo `<escopo>` aplicando sua estratégia Search-First & Tool-Assisted e as regras anti-falso-positivo. Respeite o `AGENTS.md` local (se existir) e retorne o relatório completo classificado por gravidade (`🔴 Crítico`, `🟡 Moderado`, `🟢 Cosmético`) com links clicáveis, linhas exatas e evidências."
  - **Para diretório amplo / Repositório completo**:
    > "Audite o escopo `<escopo>` aplicando sua estratégia Search-First & Tool-Assisted e as regras anti-falso-positivo. Respeite o `AGENTS.md` local (se existir), salve o relatório completo classificado por gravidade em `<caminho_relatorio>` e retorne no chat apenas o resumo executivo (máx. 15 linhas) com a contagem por severidade e os top 5 achados mais críticos."

### Passo 3: Apresentação e Decisão de Limpeza (`ask_question`)
1. Encerre imediatamente o subagente auditor com `manage_subagents(Action="kill")` após receber o relatório.
2. Apresente o resultado estruturado ao usuário (e o link clicável para `[slop_audit_report.md](file:///<caminho_relatorio>)`, caso tenha sido gerado em disco).
3. Se houver achados acionáveis, consulte o usuário com a ferramenta `ask_question` oferecendo:
   - *(Recommended) Limpar apenas os itens `🔴 Crítico` e `🟡 Moderado`*
   - *Limpar todos os itens (`🔴 Crítico`, `🟡 Moderado` e `🟢 Cosmético`)*
   - *Gerar um plano estruturado em `plans/cleanup-slop.md` (recomendado se a limpeza envolver refatorações em muitos arquivos)*
   - *Manter apenas o relatório (não alterar código)*

### Passo 4: Execução Delegada da Limpeza (Se Autorizada)
Nunca edite múltiplos arquivos de produção diretamente no agente principal:
- **Se o usuário escolher gerar um plano**: siga o fluxo da skill `plan-designer` criando `plans/cleanup-slop.md` a partir dos achados aprovados.
- **Se o usuário autorizar a limpeza imediata**:
  1. Invoque o subagente `code-implementer` (`Workspace: "inherit"`) passando a lista exata de arquivos/linhas (ou apontando para `<caminho_relatorio>`) e a severidade aprovada (`🔴 + 🟡` ou `Todos`), exigindo 100% de preservação dos contratos externos e execução do comando de build/testes/typecheck do projeto ao final.
  2. Encerre o `code-implementer` com `manage_subagents(Action="kill")` após a conclusão e apresente o resumo dos arquivos limpos e o status de verificação (`PASSED`).
