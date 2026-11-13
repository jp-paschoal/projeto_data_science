# Previsão de atrasos de entrega (Olist)

Projeto do desafio técnico de Data Science. O objetivo é prever, na hora da compra, se um pedido vai ser entregue depois da data prometida ao cliente, usando o dataset público da Olist (cerca de 100 mil pedidos entre 2016 e 2018).

## Resumo

- Base final: 95.699 pedidos entregues, de janeiro de 2017 a agosto de 2018.
- Taxa de atraso: 6,8% (comparando só o dia da entrega com o dia estimado).
- Modelo final: LightGBM sem as colunas de estado, com PR-AUC de 0,071 no teste (um chute aleatório teria 0,035).
- Uso prático: marcando os 10% de pedidos com maior risco, o modelo pega 20,6% dos atrasos reais, mais que o dobro de um sorteio.
- Principal limite: o padrão de atraso por estado mudou entre o período de treino e o de teste.

## Como baixar os dados

Os arquivos CSV brutos não estão no repositório.

1. Baixe o dataset no Kaggle: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
2. Extraia os 9 arquivos `.csv` numa pasta chamada `data/` na raiz do projeto.

O notebook usa a pasta do Kaggle quando roda lá e a pasta `data/` quando roda no computador.

## Como executar

Rodei o projeto no Kaggle. Para rodar localmente, use Python 3.10 ou mais novo:

```
python -m venv venv
venv\Scripts\activate        # Windows (no Linux ou macOS: source venv/bin/activate)
pip install -r requirements.txt
jupyter notebook projeto_olist.ipynb
```

O cálculo do histórico do vendedor tem um laço e pode levar alguns minutos.

## Estrutura do repositório

```
.
├── data/                  # CSVs do Kaggle (fora do Git)
├── projeto_olist.ipynb    # notebook com todo o projeto
├── requirements.txt
├── README.md
└── .gitignore
```

## Roteiro do notebook

1. Os dados: carga das 9 tabelas e definição de uma linha por pedido.
2. Qualidade dos dados e criação do target.
3. Exploração dos dados: 9 hipóteses sobre o que causa atraso (prazo prometido, estado, distância, prazo dentro de cada faixa de distância, folga do prazo, época do ano, peso, frete em relação ao preço e frete em reais), com a limpeza de coordenadas erradas e de 2016.
4. Preparação da base: escolha das variáveis conhecidas na compra e divisão em treino e teste no tempo.
5. Modelagem: regressão logística, LightGBM e variações, mais a hipótese 10 (histórico do vendedor) e o ajuste de parâmetros.
6. Avaliação do modelo final: ganho prático por faixa de risco, importância das variáveis e análise de erros por estado.
7. Conclusão e uso na prática.

## Principais decisões

- Só entram pedidos entregues. Os outros 2.963 (enviados, cancelados etc.) não têm data de entrega, então não dá para saber se atrasaram. Isso pode fazer a taxa de atraso parecer menor do que é, já que pedidos ainda a caminho podem estar atrasados.
- Target por dia. O enunciado define atraso pela diferença entre as datas completas, mas a data estimada vem sempre às 00:00:00. Com a hora, 8,11% dos pedidos seriam atrasados, e 1.292 deles chegaram no próprio dia prometido. Comparando só o dia, são 6,77%.
- Sem vazamento de dados. Só entram variáveis conhecidas na compra. Datas de aprovação, envio e entrega e avaliações ficam de fora, e o histórico do vendedor usa só entregas já concluídas antes de cada compra.
- Divisão no tempo. Os 80% dos pedidos mais antigos (até 27/05/2018) são o treino e os 20% mais recentes são o teste.
- Métricas. Como poucos pedidos atrasam, uso PR-AUC (principal) e ROC-AUC, além da tabela de ganho por faixa de risco.

Limpeza da base:

| Etapa | Removidos | Base |
|---|---|---|
| Todos os pedidos | | 99.441 |
| Só status entregue | 2.963 | 96.478 |
| Entregue sem data de entrega | 8 | 96.470 |
| CEP sem coordenadas | 478 | 95.992 |
| Distâncias impossíveis (acima de 4.500 km) | 4 | 95.988 |
| Ano de 2016 (poucos pedidos) | 267 | 95.721 |
| Peso ou volume igual a zero | 22 | 95.699 |

## Resultados

| Modelo | PR-AUC | ROC-AUC |
|---|---|---|
| Chute aleatório | 0,035 | 0,500 |
| Regressão logística | 0,069 | 0,678 |
| LightGBM | 0,067 | 0,702 |
| LightGBM sem o mês | 0,055 | 0,606 |
| LightGBM mais simples, sem o mês | 0,061 | 0,643 |
| LightGBM com histórico do vendedor | 0,065 | 0,669 |
| LightGBM sem os estados (modelo final) | 0,071 | 0,699 |
| LightGBM sem os estados, com ajuste de parâmetros | 0,073 | 0,690 |

O ajuste de parâmetros (com validação que respeita o tempo) não trouxe ganho relevante, então mantive o modelo sem ajuste como final.

Ganho prático no teste (19.140 pedidos, 665 atrasos):

| Pedidos marcados (maior risco) | Atrasos pegos | Vezes melhor que sortear |
|---|---|---|
| 5% | 12,3% | 2,47 |
| 10% | 20,6% | 2,06 |
| 20% | 38,9% | 1,95 |
| 30% | 56,5% | 1,88 |

## Limitações

- O padrão de atraso mudou entre treino e teste (7,6% contra 3,5%), e estados como RJ e BA caíram bastante.
- São só 20 meses de dados, pouco para separar o que se repete todo ano do que foi pontual.
- O modelo só enxerga o que existe na hora da compra. Problemas depois disso (demora do vendedor, transportadora) ficam de fora.
- Os pedidos do fim do treino entram com o resultado final, mesmo quando chegaram depois do corte.
- Fiz uma única divisão de treino e teste.

## Uso de ferramentas e IA

Usei o Claude (Anthropic) e o Gemini (Google) como apoio durante o projeto, para tirar dúvidas, revisar código e organizar os textos. Também consultei outra assistente de IA para revisar partes do código. As hipóteses, as decisões e as conclusões foram revisadas por mim.

Bibliotecas: Python, pandas, NumPy, Matplotlib, Seaborn, scikit-learn e LightGBM. Ambiente: Kaggle Notebooks.

## Referências

- Olist. Brazilian E-Commerce Public Dataset by Olist. Kaggle. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
- CRISP-DM, roteiro de projetos de dados, como guia da organização do notebook.
