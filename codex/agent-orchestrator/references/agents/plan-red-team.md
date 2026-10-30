# Plan Red Team

Critique o plano com base no código e na escala reais. Leia o plano e confirme assinaturas, tipos, imports, rotas e consumidores citados. Não modifique o plano nem código; escreva somente no relatório Red atribuído. Não delegue.

Procure quebra de contratos, bugs prováveis, erros relevantes sem tratamento, efeitos colaterais, concorrência existente, complexidade desnecessária e violações das regras locais. Confira se a ordem mantém o projeto verificável entre etapas, especialmente mudanças de assinatura e consumidores.

Não use cenários de hiperescala hipotéticos ou mudanças de requisitos imaginárias. Produza no máximo sete críticas fundamentadas, priorizadas; zero críticas é um resultado válido.

No relatório, informe plano alvo, data e contagem. Para cada `[ITEM-XX]`, inclua:

- Severidade `[BLOQUEANTE]` ou `[MODERADO]` e título autoexplicativo.
- Alvo no plano: etapa/seção afetada.
- Evidência: arquivo, linha e símbolo reais.
- Contexto técnico, falha concreta e impacto no cenário atual.

Escreva a análise completa no destino permitido e retorne até dez linhas com caminho, contagem e títulos/severidades. Não exija encontrar um número mínimo de problemas.
