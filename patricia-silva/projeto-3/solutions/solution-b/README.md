# Solution B — RAG leve com FAQ no Google Sheets

## Visão geral

A Solução B é a arquitetura escolhida para entrega final. Ela combina triagem por IA com consulta a base de conhecimento externa (FAQ em planilha), gerando resposta contextual e mantendo rastreabilidade completa.

Arquivo de referência:

- `workflows/solution-b-faq-sheets.json` (solução adotada no ADR-001)

## Como o fluxo funciona

1. **Webhook** recebe o chamado.
2. **Normalize Input** extrai e padroniza `message` e `email`.
3. **Input valido?** filtra entradas inválidas.
4. Em paralelo:
  - **Gemini Triagem** classifica o chamado em JSON estruturado.
  - **Ler FAQ Sheet** busca as linhas da aba `FAQ`.
5. **FAQ para Texto** converte a planilha em contexto textual.
6. **Merge Triagem e FAQ** combina os dois ramos.
7. **Juntar Campos** consolida os dados em um único objeto.
8. **Gemini Com FAQ** gera orientação contextual usando o FAQ.
9. **Parse Resposta Final** define rota operacional (`auto`, `revisao`, `escala`).
10. **Switch Rota** seleciona caminho de atendimento.
11. **Sheets Log Tickets** registra auditoria.
12. **Respond OK** devolve resposta HTTP.

## Por que esta solução é RAG leve

O mecanismo não usa vetor/embeddings, mas aplica o padrão de “recuperar contexto externo antes de gerar resposta”:

- Recuperação: leitura da base `FAQ` no Google Sheets.
- Augmentação: concatenação da FAQ com a classificação do chamado.
- Geração: segunda chamada do Gemini usando o contexto recuperado.

## Papel da IA no roteamento e no conteúdo

- **IA 1** impacta o fluxo de decisão (`categoria`, `urgencia`, `confianca`).
- **IA 2** impacta a qualidade da orientação final ao usuário.
- Em caso de baixa confiança ou falha, o fluxo cai em rota conservadora (`revisao`).

## Pontos fortes

- Atende ao requisito de base de conhecimento integrada.
- Melhor experiência de resposta que a solução puramente classificatória.
- Mantém rastreabilidade no `Tickets` para auditoria e análise posterior.

## Limitações

- Custo e latência maiores (duas chamadas de modelo por execução).
- Dependência da qualidade e atualização da aba `FAQ`.
- Maior exposição a rate limit quando testado em alta frequência.

## Configuração mínima no n8n

- Importar `solution-b-faq-sheets.json`.
- Configurar `x-goog-api-key` nos nós `Gemini Triagem` e `Gemini Com FAQ`.
- Configurar credencial Google Sheets nos nós `Ler FAQ Sheet` e `Sheets Log Tickets`.
- Substituir `SUBSTITUA_ID_PLANILHA` pelo ID da planilha real.
- Garantir:
  - aba `FAQ` com colunas `titulo` e `resposta`;
  - aba `Tickets` compatível com mapeamento de colunas.
- Validar o nó `Merge` em modo de combinação por posição após importação.

## Cenários de teste recomendados

- Chamado simples com FAQ conhecido (“esqueci a senha”).
- Chamado crítico (“sistema indisponível para todos”).
- Entrada ambígua para verificar `confianca` e rota `revisao`.
- Entrada inválida para validar bloqueio no `Input valido?`.

## Critérios de evidência

- Resposta HTTP com `ok: true` e `orientacao`.
- Linha registrada em `Tickets` com rota e resumo.
- Demonstração da leitura da `FAQ` influenciando a resposta final.

## Evidências desta solução

As evidências visuais desta solução estão em [../../docs/evidence/](../../docs/evidence/):

