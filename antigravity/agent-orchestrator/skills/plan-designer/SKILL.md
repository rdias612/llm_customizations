---
name: plan-designer
description: >-
  Use esta skill quando o usuário pedir para planejar, projetar ou estruturar
  um plano técnico de desenvolvimento com checklist de etapas (em 'plans/' ou na raiz).
  Analisa o código real, decompõe a demanda em etapas atômicas sequenciais que preservam
  o build entre etapas e prepara o arquivo markdown para o plan-executor.
---

# Plan Designer (Arquiteto e Criador de Planos)

Esta skill conduz a **Fase de Concepção e Planejamento Arquitetural**, gerando um plano técnico determinístico em disco pronto para ser executado pelo `plan-executor`.

## Princípios de Operação
1. **Zero Alteração Direta em Código de Produção**: O arquiteto nunca edita código de produção diretamente; gera o arquivo markdown do plano (`plans/<slug>.md`) ou, em micro-tarefas elegíveis após confirmação do usuário, delega via *Fast-Track* para a dupla `code-implementer` + `code-reviewer`.
2. **Pre-Flight e Verificação de Contratos no Código Real (Anti-Alucinação)**: Antes de especificar qualquer etapa, use as ferramentas de busca e listagem para localizar os arquivos-alvo e leia os trechos relevantes para confirmar nomes reais de funções, tipos, rotas e imports. Nunca invente assinaturas não verificadas.
3. **Invariante de Build entre Etapas**: Ordene as etapas de modo que a etapa $N+1$ dependa apenas do que foi consolidado até a etapa $N$, mantendo o projeto compilável ao final de cada etapa.
4. **Verificação Polimórfica por Etapa**: Especifique em cada etapa o comando mais rápido e adequado de validação (ex.: `tsc --noEmit` para tipos, comando de teste unitário focado para regras de negócio, build para infra/config ou inspeção de diff para alterações puramente visuais/documentais), evitando forçar testes artificiais ("test slop") ou builds globais lentos em toda etapa.
5. **Aderência Estrita às Regras Locais**: Incorpore no plano as restrições do `AGENTS.md` (ou `GEMINI.md`) do repositório.

---

## Fluxo de Execução

### Passo 1: Inspeção Cirúrgica Focada e Triagem Adaptativa (Pre-Flight)
1. Leia o `AGENTS.md` (se existir) e use busca nativa para identificar rapidamente os arquivos diretamente relacionados à demanda.
2. Mapeie os símbolos exatos (componentes, funções, tipos, tabelas) que serão criados ou modificados, lendo apenas as linhas necessárias de cada arquivo.
3. **Triagem Adaptativa de Escopo (100% decidida no Pre-Flight)**:
   - **Standard Track ($\ge 3$ arquivos afetados, novos módulos ou múltiplos passos interdependentes)**: Prossiga direto para o **Passo 2** e gere o arquivo formal `plans/<slug>.md`.
   - **Fast-Track Elegível ($\le 2$ arquivos existentes, alteração localizada e sem novos módulos)**:
     - Se o usuário pediu apenas para *planejar*, consulte-o brevemente com `ask_question` oferecendo: **(Recomendado) Executar via Fast-Track agora** (sem gerar arquivo `plans/<slug>.md`, delegando direto para `code-implementer` + `code-reviewer` + commit atômico) ou **Gerar arquivo formal `plans/<slug>.md`**.
     - Se o usuário escolher o Fast-Track (ou se já tiver pedido implementação direta no prompt), acione a dupla `code-implementer` e `code-reviewer` em etapa única, execute o commit semântico atômico (`git add -- <arquivos>` e `git commit --no-verify --no-gpg-sign -m "feat(<slug>): <titulo>" -m "<resumo>"`), encerre os subagentes com `kill` e apresente o resumo final.

### Passo 2: Geração do Arquivo de Plano
1. **Caminho de Destino**:
   - Padrão: `plans/<slug_da_tarefa>.md` (ou salve na raiz como `plano_<slug>.md` se o projeto restringir novos diretórios).
2. **Estrutura do Arquivo**:

```markdown
# Plano de Desenvolvimento: [Título da Demanda]

- **Data**: AAAA-MM-DD
- **Status**: [ ] Planejado
- **Arquivo de Controle**: `plans/[slug].md`

## 1. Visão Geral e Regras Locais Aplicáveis
- **Objetivo**: [Resumo direto da solução]
- **Restrições do Projeto (`AGENTS.md`)**: [Regras críticas que o implementador e o revisor devem obedecer]
- **Comando de Build Global**: `[ex: npm run build / tsc --noEmit / cargo check]`

## 2. Checklist de Execução de Etapas
- [ ] Etapa 1: [Nome da Etapa 1]
- [ ] Etapa 2: [Nome da Etapa 2]

---

## 3. Especificação das Etapas

### Etapa 1: [Nome da Etapa 1]
- **Objetivo**: [O que deve ser entregue]
- **Arquivos e Símbolos Afetados**:
  - `caminho/arquivo.ext` (`funcaoX`, `TipoY`)
- **Instruções Técnicas**:
  - [Passos objetivos de implementação]
- **Verificação da Etapa**: `[ex: npx tsc --noEmit / pytest tests/unit/test_x.py / Inspeção de diff]`
- **Critérios de Aceitação (Code Review)**:
  - [Critério verificável 1]
  - [Critério verificável 2]
```

### Passo 3: Entrega no Chat
Apresente um resumo curto com link clicável para o arquivo gerado e informe que, após revisar o plano, o usuário pode pedir sua execução via `plan-executor` (ou, opcionalmente, submetê-lo a um debate adversário via `plan-debater` para estressar falhas e contratos antes de implementar).
