---
name: plan-designer
description: Cria um plano técnico em Markdown com etapas sequenciais e critérios verificáveis quando o usuário pede planejamento de desenvolvimento ou preparação de um plano para execução.
---

# Plan Designer

Leia [execução no Codex](../../references/runtime.md). Produza um plano ancorado no código real, sem alterar código de produção quando o pedido for apenas planejamento.

1. Leia as regras locais e localize arquivos, funções, tipos, rotas e consumidores reais com busca focada. Declare premissas e resolva ambiguidades que mudariam a solução. Não invente assinaturas ou dependências.
2. Divida o trabalho no menor número de etapas úteis. Ordene mudanças de contratos e seus consumidores para manter o projeto verificável ao final de cada etapa. Não crie abstrações para uma necessidade futura hipotética.
3. Para mudança localizada em até dois arquivos existentes, use um plano curto. Se o usuário também pediu implementação, a execução pode ser uma única etapa com os papéis [implementador](../../references/agents/code-implementer.md) e [revisor](../../references/agents/code-reviewer.md). Um pedido só de planejamento continua sendo planejamento, sem oferecer implementação como pré-condição para entregar o plano.
4. Salve em `plans/slug.md`, salvo destino solicitado ou restrição do ambiente. Preserve um plano existente que não tenha sido explicitamente escolhido para atualização.

Use esta estrutura, preenchendo somente informações relevantes:

```markdown
# Plano de Desenvolvimento: título

- **Data**: AAAA-MM-DD
- **Status**: [ ] Planejado

## 1. Contexto
- Objetivo e limites do escopo.
- Regras locais aplicáveis e contratos confirmados: arquivo, símbolo e linha.
- Premissas ou decisões pendentes, se houver.
- Comando de verificação global e diretório de execução; se não se aplica, justificar.

## 2. Checklist
- [ ] Etapa 1: nome

## 3. Etapas
### Etapa 1: nome
- Objetivo:
- Arquivos e símbolos afetados:
- Alterações necessárias:
- Dependências de etapas anteriores:
- Verificação: comando, diretório e resultado esperado.
- Critérios de aceitação observáveis:
```

Escolha verificações proporcionais: teste de comportamento para lógica, typecheck para contratos, build para configuração ou inspeção/renderização para documentos e aparência. Evite testes que apenas espelhem a implementação. Distinga verificação já executada da que está prevista no plano.

Entregue resumo e link para o plano. Quando útil, indique que ele pode ser debatido ou executado com as outras skills; não inicie essas fases sem solicitação correspondente.
