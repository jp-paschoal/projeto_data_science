# Previsão de Atrasos de Entrega (Olist)

Desafio técnico de Ciência de Dados aplicado ao e-commerce brasileiro.

## Sobre o projeto

O objetivo deste projeto é prever, **no exato momento da compra (checkout)**, a probabilidade de um pedido ser entregue após a data estimada informada ao cliente, utilizando o dataset público da Olist (\~100 mil pedidos realizados entre 2016 e 2018).

Antecipar pedidos com alto risco de atraso permite que as equipes de logística, atendimento e os próprios lojistas (_sellers_) atuem preventivamente — adicionando folga dinâmica no prazo exibido no checkout, priorizando a expedição do pacote ou enviando comunicações proativas ao consumidor.

Destaques metodológicos do projeto:

- **Prevenção estrita de vazamento de dados (_Data Leakage_):** utilização exclusiva de variáveis conhecidas até o segundo da compra, descartando datas posteriores (aprovação, postagem na transportadora, entrega real e avaliações).

- **Engenharia de atributos (_Feature Engineering_):** cálculo da distância física entre CEPs via fórmula de Haversine (`distancia_km`), criação do índice de folga logística (`dias_por_100km`) e construção do histórico temporal de atrasos de cada vendedor (`vend_hist_taxa`) com suavização bayesiana e busca binária estritamente no passado.

- **Validação temporal e desbalanceamento:** divisão cronológica (80% passado para treino, 20% futuro para teste) para respeitar a sazonalidade e mudanças de regime da operação, avaliando os modelos por **PR-AUC** e **ROC-AUC** devido à raridade da classe positiva (6,77% geral e 3,5% no período de teste).

## Estrutura do repositório

```
.
├── data/                        # Arquivos CSV do Kaggle (ignorados no Git via .gitignore)
├── projeto_olist.ipynb
├── requirements.txt
├── README.md
└── .gitignore

```

## Roteiro do notebook

1. **Os Dados:** carregamento das 9 tabelas relacionais da Olist e definição da granularidade por pedido.

2. **Qualidade dos Dados e Criação do Target:** filtro de pedidos entregues (`delivered`), tratamento de nulos e construção do target `atrasou` comparando apenas o dia (`.dt.normalize()`) para corrigir a distorção do horário `00:00:00` na data estimada.

3. **Exploração dos Dados (EDA):** investigação de 9 hipóteses de negócio (prazo prometido, UF do cliente/vendedor, distância física, cruzamento prazo vs. distância, folga em dias por 100 km, sazonalidade mensal, peso/volume e frete), incluindo remoção dos dados esparsos de 2016 e limpeza de outliers geográficos (> 4.500 km) e cadastrais.

4. **Preparação da Base para o Modelo:** seleção de features livres de vazamento e split temporal (corte em `27/05/2018`).

5. **Modelagem:** comparação entre Baseline Aleatório, Regressão Logística e variações do LightGBM (testando impacto da sazonalidade, poda de complexidade das árvores, adição do histórico do vendedor e remoção das siglas de UF para ganho de estabilidade).

6. **Avaliação do Modelo Final:** análise prática de negócio via Tabela de _Lift_ / Curva de Ganho, importância das variáveis (_Feature Importance_ por ganho) e diagnóstico de erros frente à mudança de regime logístico entre treino e teste (_Concept Drift_).

7. **Conclusão e Aplicação Prática:** plano de ação no checkout, limitações do estudo e recomendações de próximos passos.

## Modelos utilizados e resultados

| Modelo                                  | PR-AUC    | ROC-AUC   | Observação                                               |
| --------------------------------------- | --------- | --------- | -------------------------------------------------------- |
| **Chute aleatório (Baseline)**          | 0,035     | 0,500     | Taxa base de atrasos no período de teste (3,5%)          |
| **Regressão Logística**                 | 0,069     | 0,678     | Baseline linear com `StandardScaler` e pesos balanceados |
| **LightGBM (padrão)**                   | 0,067     | 0,702     | Numéricas + UF do cliente e do vendedor                  |
| **LightGBM (sem mês)**                  | 0,055     | 0,606     | Teste de remoção da sazonalidade                         |
| **LightGBM (simples, sem mês)**         | 0,061     | 0,643     | Árvores rasas (`num_leaves=8`, `min_child_samples=100`)  |
| **LightGBM + histórico do vendedor**    | 0,065     | 0,669     | Inclusão de `vend_hist_taxa` e `vend_hist_n`             |
| **LightGBM sem estados (Modelo Final)** | **0,071** | **0,699** | **Melhor PR-AUC e maior estabilidade regional**          |

- **Ganho operacional (_Lift_):** ao priorizar os **10% de pedidos com maior score de risco**, o modelo final captura **20,6% de todos os atrasos reais** (**2,06x melhor** que uma seleção aleatória). Expandindo para o **top 20%**, capturamos **38,9%** dos atrasos; no **top 30%**, antecipamos **56,5%** de todos os problemas de entrega.

## Como executar o projeto localmente

### 1. Obtenção dos dados (Kaggle)

Por boas práticas de versionamento, os arquivos CSV brutos não estão incluídos neste repositório.

1. Baixe o dataset oficial no Kaggle: [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).

2. Extraia os 9 arquivos `.csv` dentro de uma pasta chamada `data/` na raiz do projeto (o notebook detecta automaticamente tanto o ambiente do Kaggle quanto a pasta local `data/`).

### 2. Configuração do ambiente e instalação de dependências

No terminal, crie e ative um ambiente virtual (`venv`) e instale os pacotes necessários:

- **Windows (Cmd):**

  ```
  python -m venv venv
  venv\Scripts\activate
  pip install -r requirements.txt

  ```

- **Windows (Git Bash):**

  ```
  python -m venv venv
  source venv/Scripts/activate
  pip install -r requirements.txt

  ```

- **Linux / macOS:**

  ```
  python3 -m venv venv
  source venv/bin/activate
  pip install -r requirements.txt

  ```

### 3. Execução do Notebook

Abra o arquivo `projeto_olist.ipynb` no VS Code, JupyterLab ou Jupyter Notebook:

```
jupyter notebook projeto_olist.ipynb

```

## Tecnologias utilizadas

- **Python 3**

- **Pandas / NumPy** — manipulação de dados, agregações e cálculos vetoriais

- **Matplotlib / Seaborn** — visualização de dados, mapas de calor e curva de ganho

- **Scikit-learn** — pré-processamento, baseline linear e métricas de avaliação (`PR-AUC`, `ROC-AUC`)

- **LightGBM** — modelo principal de _Gradient Boosting_
