# Diretrizes de Prototipagem Conceitual (`concept-prototyper`)

Quando o usuário solicitar alternativas visuais, mockups de frontend, abordagens de documentação ou estruturas de apresentação para avaliar ideias antes de decidir como seguir:

1. **Objetivo Conceitual (Não Código de Produção)**: O foco é a prototipagem rápida e visual para decisão humana, sem alterar ou acoplar o código-fonte principal do projeto.
2. **Geração Divergente em Trio**: Dispare concorrentemente 3 subagentes com prismas conceituais contrastantes (ex.: Minimalista/Clean, Dashboard/Denso, Inovador/Experiencial).
3. **Persistência Direta dos Artefatos**: Cada subagente salva seu arquivo completo e funcional (ex.: `.html` autocontido com CSS/JS para telas, `.md` para documentação, ou slides navegáveis) diretamente no diretório solicitado pelo usuário (ou padrão como `mockups/`, `docs/propostas/` ou `decks/`).
4. **Consolidação Exclusiva no Chat**: O agente principal nunca gera um arquivo extra de consolidação em disco; apresenta o comparativo com links clicáveis (`file:///...`), tabela de trade-offs e próximos passos diretamente na mensagem do chat.
5. **Limpeza Imediata**: Encerre os subagentes criados logo após receber as 3 notificações para liberar memória e processos.
