---
name: find-code-slop
description: Audita code slop, código morto, abstrações prematuras, supressões de tipos e testes sem valor em arquivos, diretórios ou alterações Git quando o usuário pede essa análise ou limpeza.
---

# Find Code Slop

Leia [execução no Codex](../../references/runtime.md). Separe achados comprovados de preferências de estilo e preserve código de produção durante uma auditoria sem pedido de correção.

1. Delimite o escopo solicitado: arquivo/diretório, working tree ou diferença de branch. Para working tree, examine `git status --porcelain`, diffs staged e unstaged e arquivos novos. Com HEAD existente, `git diff HEAD` reúne mudanças rastreadas; em repositório sem HEAD, examine índice/arquivos diretamente. Para branch, use a base solicitada ou confirme a base real; não substitua uma auditoria de branch pela do working tree.
2. Delegue ao [auditor](../../references/agents/code-slop-auditor.md) o escopo e a saída. Um arquivo pequeno pode ser inspecionado diretamente. Para escopo amplo, use `docs/slop_audit_report.md` se `docs/` existir, senão `slop_audit_report.md`, respeitando regras do ambiente e preservando execuções anteriores.
3. O auditor busca candidatos, verifica consumidores e usa apenas analisadores já disponíveis no projeto, sem instalar ferramentas. Não deve executar comandos de autofix ou geradores durante a auditoria.
4. Aguarde o resultado, confira evidências e apresente os achados por severidade com arquivo, linha, impacto e correção proposta. Para relatório amplo, entregue link, contagens e até cinco destaques. Permita zero achados.
5. Se o pedido já inclui correção, delegue ao [implementador](../../references/agents/code-implementer.md) apenas os itens comprovados dentro do escopo autorizado. Se o pedido era somente auditoria, entregue o resultado e obtenha autorização antes de limpar; quando necessário, ofereça itens críticos/moderados, itens selecionados ou plano de limpeza. Não exija nova autorização para correções já solicitadas.
6. Para limpeza autorizada, limite a escrita aos arquivos dos achados aprovados e preserve contratos públicos. Execute a verificação adequada e delegue revisão ao [revisor](../../references/agents/code-reviewer.md) quando a alteração justificar revisão independente. Mudança ampla pode ser organizada em plano antes de continuar, sem ampliar escopo.

Entregue o que foi encontrado e, quando aplicável, corrigido e verificado. Não remova código preexistente fora do pedido nem crie testes triviais apenas para satisfazer um checklist.
