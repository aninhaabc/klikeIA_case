# Parte 4 — Visão de Produto

## 1. Se tivéssemos acesso ao vídeo original, quais features adicionais extrairíamos?

A solução atual utiliza metadados do criativo e do contexto da campanha. Com acesso ao vídeo, eu priorizaria características que fossem **mensuráveis, interpretáveis e acionáveis** para o profissional de marketing.

| Grupo | Features possíveis | Utilidade para recomendações |
|---|---|---|
| Abertura e narrativa | Tempo até a primeira mensagem relevante; mudança de cena nos três primeiros segundos | Avaliar se o vídeo comunica rapidamente sua proposta |
| Texto na tela | Texto extraído por OCR, densidade por segundo, legibilidade, posição e tempo de exposição | Recomendar simplificação e melhor posicionamento |
| Elementos visuais | Presença e duração de rostos, produto e marca; ritmo de cortes | Avaliar clareza visual e exposição da marca |
| Áudio e fala | Transcrição, tempo até a primeira fala, silêncio, proporção de fala e música | Identificar problemas de comunicação e acessibilidade |
| CTA | Momento, duração e clareza da chamada para ação | Recomendar posicionamento ou reformulação do CTA |
| Qualidade técnica | Resolução, proporção de tela, contraste e volume de áudio | Detectar problemas objetivos de produção |

**Estratégia incremental.** Começaria por OCR, transcrição e detecção de cenas, usando modelos existentes. Depois avaliaria se as features extraídas melhoram a previsão do `klike_score` em validação fora da amostra. Não usaria reconhecimento de identidade de pessoas: detectar presença de rosto é suficiente para a hipótese estudada e reduz riscos de privacidade.

**Cuidados.** A qualidade da extração varia com idioma, iluminação e formato. Também seria necessário avaliar direitos de uso dos vídeos, retenção dos arquivos e possíveis vieses entre públicos e plataformas.

## 2. Como colocar o Recommendations Engine em produção?

Eu começaria com uma arquitetura pequena, separando **treinamento offline** de **inferência online**. Não treinaria o modelo a cada solicitação.

```text
Usuário / painel Klike
        |
        v
API Python (validação de entrada)
        |
        +--> Dados estruturados da campanha
        |         |
        |         v
        |    Pipeline Ridge versionado
        |         |
        |         v
        |    Geração de alternativas
        |         |
        |         v
        |    Evidências históricas agregadas
        |         |
        |         v
        +----> Priorização e explicações
                     |
                     v
          Recomendações no painel
```

**Componentes mínimos.** Uma API com FastAPI, um artefato de modelo versionado, um banco de dados para campanhas e métricas históricas e logs de requisições e resultados. O frontend mostraria ações, diferenças previstas, evidências, tamanho das amostras e limitações. O treinamento e a atualização das estatísticas seriam tarefas offline.

**Quando houver vídeo bruto.** O upload seria enviado a armazenamento de objetos e processado de modo assíncrono. Uma fila de tarefas evitaria bloquear a API durante OCR, transcrição e extração de frames. Essa infraestrutura só seria necessária quando o processamento audiovisual fosse incorporado.

**Qualidade e segurança.** Validaria schema, valores permitidos e campos obrigatórios; limitaria acessos por cliente; evitaria registrar conteúdo sensível nos logs; controlaria versões de modelo, dados e features. Arquivos `joblib` seriam carregados apenas de origem confiável, com versões de dependências fixadas.

**Monitoramento.** Acompanharia latência, erros, cobertura de recomendações, proporção de hipóteses versus melhorias previstas, mudança na distribuição das features e desempenho do score quando os resultados reais estivessem disponíveis. Mudanças de modelo exigiriam avaliação antes da publicação e possibilidade de rollback.

**Experiência do usuário.** Recomendações deveriam explicar o *porquê* e o *quanto*; diferenças observacionais seriam apresentadas como evidências para testes, não como ganhos garantidos. Quando não houver três oportunidades sustentadas, o sistema deve informar essa limitação em vez de fabricar melhorias.

## 3. O que faria com mais tempo?

Eu priorizaria as melhorias por impacto esperado e risco de implementação:

1. **Validação causal das sugestões.** Testes A/B com versões de um mesmo criativo, mantendo público, objetivo e orçamento tão comparáveis quanto possível. É a etapa mais importante para transformar associações em decisões confiáveis.
2. **Métrica alinhada ao objetivo.** Substituir o uso universal de CTR por métricas adequadas a tráfego, engajamento, conversão e awareness, após verificar a disponibilidade e a definição de cada indicador.
3. **Histórico sem vazamento temporal.** Para uma campanha analisada em determinada data, usar apenas campanhas anteriores, excluir a própria campanha e documentar os critérios de comparação.
4. **Cobertura e qualidade das recomendações.** Investigar campanhas sem três sugestões priorizadas, ampliar alternativas somente quando houver evidência e medir utilidade com profissionais de marketing.
5. **Validação temporal e incerteza.** Comparar treino/teste cronológicos, examinar desempenho por plataforma e categoria e apresentar incerteza ou confiabilidade das estimativas.
6. **Features audiovisuais.** Testar OCR, transcrição e ritmo de edição, mantendo apenas as variáveis que melhorem o desempenho fora da amostra.

## Consideração final

A versão entregue é um **protótipo de apoio à decisão**. Sua principal contribuição é conectar exploração de dados, previsão e recomendações explicáveis. O próximo salto de qualidade não seria um modelo necessariamente mais complexo, mas evidência mais confiável de que as alterações sugeridas ajudam campanhas reais.
