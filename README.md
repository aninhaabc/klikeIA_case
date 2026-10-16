# Klike.ai — Data Science Take-Home Challenge

Análise exploratória, previsão do **Klike Score** e protótipo de recomendações para criativos publicitários.

## Visão geral

O objetivo é entender quais características dos criativos se relacionam com performance, prever o `klike_score` (escala de 0 a 100) e transformar evidências em sugestões acionáveis para profissionais de marketing.

## Resultados principais

| Etapa | Resultado |
|---|---|
| EDA | 500 campanhas, 26 variáveis; análises de qualidade, atributos, plataformas e métricas |
| Modelagem | **Ridge Regression**, selecionada por validação cruzada |
| Teste da Ridge | **MAE 6,785**, **RMSE 8,749**, **R² 0,661** |
| Feature engineering | 13 features originais; interação hook × CTA testada e descartada por não trazer ganho consistente |
| Importância preditiva | `has_hook`, `has_face` e `text_density` se destacaram na Permutation Importance |
| Recommendations Engine | 392 de 500 campanhas (**78,4%**) com pelo menos três sugestões priorizadas, conforme a última execução registrada |

A cobertura inclui tanto **melhorias previstas** quanto **hipóteses para teste**.

## Estrutura do repositório

```text
klikeIA_case/
├── data/
│   └── klike_challenge_dataset.csv  
├── img/                              
├── models/
│   └── ridge_klike_score.joblib      # gerado pela Parte 2
├── notebooks/
│   ├── analise_exploratória.ipynb
│   ├── modelagem_preditiva.ipynb
│   └── recommendations_engine.ipynb
├── product_vision.md
├── requirements.txt
├── .gitignore
└── README.md
```

## Como executar

**Requisitos:** Python 3.12 ou versão compatível com as dependências, Jupyter e o dataset do desafio disponibilizado pela Klike.

Na raiz do projeto:

```bash
python -m venv .venv
```

Ative o ambiente virtual:

```powershell
# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Instale as dependências e inicie o Jupyter:

```bash
python -m pip install -r requirements.txt
jupyter notebook
```

Coloque `klike_challenge_dataset.csv` em `data/`. Execute os notebooks **nesta ordem**:

1. `notebooks/analise_exploratória.ipynb` — exploração e visualizações.
2. `notebooks/modelagem_preditiva.ipynb` — treinamento, comparação, avaliação e exportação do pipeline.
3. `notebooks/recommendations_engine.ipynb` — carregamento do artefato e geração das recomendações.

O terceiro notebook depende de `models/ridge_klike_score.joblib`, criado no segundo. Se o arquivo não existir, execute primeiro a célula de persistência do modelo. As figuras são gravadas em `img/`.

## Parte 1 — Análise Exploratória (EDA)

Foram investigados valores ausentes, duplicatas, distribuições, outliers e consistência de métricas, além de comparações entre características dos criativos e resultados de campanhas.

**Principais achados:** a presença de hook esteve associada a CTR maior; a relação entre hook e ROAS variou entre plataformas. Formato e densidade de texto também apresentaram padrões distintos conforme a métrica analisada.

As visualizações de marketing priorizam atributos versus CTR, hook por plataforma e densidade de texto por plataforma. Os gráficos são análises descritivas: não demonstram causalidade.

## Parte 2 — Modelagem Preditiva

O alvo é `klike_score`. O modelo principal usa características dos criativos e contexto conhecidos antes da veiculação, evitando métricas pós-publicação como entradas.

O pré-processamento é realizado em `Pipeline`/`ColumnTransformer`, com imputação, padronização e One-Hot Encoding ajustados somente no treinamento. A divisão é de 80% treino e 20% teste.

Foram comparados Dummy Regressor, Ridge Regression e Random Forest, com validação cruzada de cinco divisões no conjunto de treinamento. A **Ridge com as 13 features originais** apresentou o menor MAE médio (6,953) e foi selecionada para avaliação final.

A feature derivada `hook_cta_interaction` foi criada como hipótese, mas não trouxe ganho consistente: sua inclusão elevou o MAE da Ridge de 6,953 para 6,978. Esse resultado motivou a decisão de não mantê-la.

No teste final, a Ridge alcançou **MAE 6,785; RMSE 8,749; R² 0,661**. A Permutation Importance destacou `has_hook`, `has_face` e `text_density`. Importância preditiva e contrastes de coeficientes **não são efeitos causais**.

## Parte 3 — Recommendations Engine

O protótipo recebe uma campanha e:

1. Prepara as entradas no mesmo formato do treinamento.
2. Gera alternativas acionáveis para características do criativo.
3. Simula a mudança de uma característica por vez usando a Ridge.
4. Compara métricas históricas em contextos hierárquicos (plataforma + categoria + objetivo; plataforma + objetivo; plataforma).
5. Classifica e prioriza as alternativas, gerando mensagens compreensíveis para marketing.

As saídas distinguem **melhoria prevista**, **hipótese para teste** e **alteração não priorizada**. As diferenças de Klike Score são previsões do modelo; diferenças de CTR são comparações observacionais.

Na última execução registrada, **392/500 campanhas (78,4%)** receberam pelo menos três sugestões priorizadas. Em 108 campanhas o protótipo não alcançou esse mínimo.

## Parte 4 — Visão de Produto

As respostas sobre extração de features do vídeo original, arquitetura de produção e melhorias futuras estão em [product_vision.md](product_vision.md).

## Limitações

- Comparações históricas observacionais, sujeitas a diferenças de público, orçamento e objetivo.
- CTR usado como métrica contextual complementar mesmo quando o objetivo exige outros indicadores.
- A própria campanha é excluída do histórico comparativo, mas ainda não há corte temporal para impedir o uso de campanhas posteriores.
- O modelo não analisa o conteúdo audiovisual bruto e não representa todas as interações possíveis.
- A cobertura do engine não garante a eficácia das recomendações.

## Conclusão

Este desafio permitiu desenvolver um fluxo de análise de dados que conecta a exploração de campanhas publicitárias, a previsão do Klike Score e a geração de recomendações para otimização de criativos.

A análise exploratória identificou associações relevantes entre características dos criativos e métricas de performance, além de diferenças entre plataformas. A modelagem selecionou a Ridge Regression, que apresentou MAE de 6,785, RMSE de 8,749 e R² de 0,661 no conjunto de teste. Já o Recommendations Engine transformou as previsões e as evidências históricas em sugestões contextualizadas, alcançando cobertura de 78,4% para a geração de pelo menos três sugestões priorizadas na última execução registrada.

O principal aprendizado é que **uma característica associada a mais cliques não necessariamente está associada a maior retorno financeiro**. Por isso, recomendações eficazes precisam considerar a plataforma, o objetivo da campanha, a métrica de interesse e a qualidade das evidências disponíveis.

O protótipo é um ponto de partida para apoiar decisões de marketing, não um sistema que garante ganhos de performance. Como próximos passos, destacam-se a validação das recomendações por testes A/B, o uso de métricas alinhadas a cada objetivo de campanha e a extração de características diretamente dos vídeos.
