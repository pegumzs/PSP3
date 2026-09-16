# 🛒 Online Shoppers — Previsão de Intenção de Compra

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![pandas](https://img.shields.io/badge/pandas-latest-150458)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-F7931E)
![License](https://img.shields.io/badge/license-MIT-green)

Análise de dados e modelo de *machine learning* para prever se uma sessão de navegação em um e-commerce termina em compra.

> Base: [Online Shoppers (Kaggle)](https://www.kaggle.com/datasets/billcampos/online-shoppers) — versão do *Online Shoppers Purchasing Intention Dataset* (UCI).

---

## 📌 Sobre o projeto

Cada linha da base é **uma sessão de navegação** em uma loja virtual, com páginas visitadas, tempo gasto e métricas de engajamento. O objetivo é prever a variável `Revenue` (a sessão virou compra ou não) — um problema de **classificação binária**.

Aplicação prática: identificar em tempo real quem tem intenção de compra permite personalizar ofertas, disparar cupons e priorizar retargeting sem gastar verba com quem não converteria.

| | |
|---|---|
| **Sessões** | 12.330 (12.205 após remover duplicatas) |
| **Variáveis** | 18 (10 numéricas + 8 categóricas) |
| **Alvo** | `Revenue` — booleano |
| **Taxa de conversão** | 15,6% (classes desbalanceadas) |
| **Período** | 1 ano |

---

## 📂 Estrutura

```
.
├── README.md
├── analise_online_shoppers_simples.ipynb   # notebook principal (recomendado)
├── analise_online_shoppers.ipynb           # versão estendida
└── apresentacao_online_shoppers.pptx       # slides do projeto
```

**Notebook simples** (30 células): EDA enxuta, pré-processamento, 2 modelos, avaliação.
**Notebook estendido** (56 células): inclui validação cruzada, ajuste de limiar, importância por permutação, segmentação com K-Means e teste de robustez sem `PageValues`.

---

## 🚀 Como executar

### Google Colab (recomendado)

Substitua `USUARIO/REPO` pelo caminho do seu repositório:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/USUARIO/REPO/blob/main/analise_online_shoppers_simples.ipynb)

Basta rodar as células na ordem. Todas as bibliotecas já vêm instaladas no Colab, exceto o `kagglehub`, instalado pela primeira célula.

### Local

```bash
git clone https://github.com/USUARIO/REPO.git
cd REPO
pip install -r requirements.txt
jupyter notebook analise_online_shoppers_simples.ipynb
```

**`requirements.txt`**

```
pandas
numpy
matplotlib
seaborn
scikit-learn
kagglehub
jupyter
```

### Sobre o acesso aos dados

O notebook baixa a base pelo `kagglehub`. Se o Kaggle pedir autenticação, gere um token em *Account → Create New Token* e defina `KAGGLE_USERNAME` / `KAGGLE_KEY` (nos **Secrets** do Colab ou como variáveis de ambiente).

A célula de carga tem um **fallback** que busca a mesma base em um espelho público, então o notebook roda de ponta a ponta mesmo sem credenciais.

---

## 🔬 Metodologia

| Etapa | O que foi feito |
|---|---|
| **1. Coleta** | Download via `kagglehub`, localização automática do `.csv`, fallback para espelho público |
| **2. Limpeza** | Checagem de nulos (nenhum), remoção de 125 linhas duplicadas, conferência de tipos |
| **3. Exploração** | Distribuição do alvo, conversão por mês e tipo de visitante, comparação de comportamento entre as classes |
| **4. Pré-processamento** | `StandardScaler` nas numéricas, `OneHotEncoder` nas categóricas, split 80/20 estratificado — tudo em `Pipeline` |
| **5. Modelagem** | Regressão Logística e Random Forest, ambos com `class_weight="balanced"` |
| **6. Avaliação** | Matriz de confusão, curva ROC, importância das variáveis |

Duas decisões que valem destaque:

- **`Pipeline` do scikit-learn** — o escalonamento e o encoding são aprendidos só no treino e depois aplicados ao teste. É o que evita **vazamento de dados** (*data leakage*).
- **`OperatingSystems`, `Browser`, `Region` e `TrafficType` são códigos**, não grandezas com ordem. Tratados como categóricos, não como numéricos.

---

## 📊 Resultados

| Modelo | Acurácia | Precisão | Recall | F1 | ROC AUC |
|---|---|---|---|---|---|
| Regressão Logística | 0,847 | 0,508 | 0,796 | 0,620 | 0,910 |
| **Random Forest** | **0,898** | **0,672** | 0,681 | **0,676** | **0,929** |

*Conjunto de teste (2.441 sessões), `random_state=42`.*

⚠️ **Acurácia é enganosa aqui**: como 84% das sessões não convertem, chutar "ninguém compra" já acerta 84%. As métricas que importam são **ROC AUC**, **recall** e **F1**.

### Principais insights

- **Sazonalidade forte**: novembro converte **25,5%** (Black Friday), fevereiro apenas **1,7%**.
- **Novos visitantes convertem mais** (24,9%) que os recorrentes (14,1%) — contraintuitivo, mas consistente com navegação exploratória e pesquisa de preço entre recorrentes.
- **`PageValues` domina** o poder preditivo (~38% da importância no Random Forest), seguida por `ExitRates` e pelas métricas de páginas de produto.
- **Fim de semana quase não muda nada**: 17,5% contra 15,1% nos dias de semana.
- A **origem do tráfego** varia muito: algumas fontes convertem o dobro da média.

---

## ⚠️ Limitações

- **Dependência de `PageValues`** — métrica do Google Analytics calculada *a partir* de transações. Em produção ela pode não estar disponível no início da sessão, o que caracteriza um vazamento parcial. No notebook estendido há um teste sem a variável: a ROC AUC cai de **0,929 para 0,783**.
- Dados de **um único site e um único ano** — não generalizam automaticamente para outros e-commerces.
- Métricas de um único split; a validação cruzada (notebook estendido) reduz o risco, mas não substitui teste em produção.

---

## 🔭 Próximos passos

- [ ] Otimizar hiperparâmetros com `GridSearchCV` / `RandomizedSearchCV`
- [ ] Testar XGBoost, LightGBM e CatBoost
- [ ] Comparar estratégias de desbalanceamento (SMOTE, undersampling, `scale_pos_weight`)
- [ ] *Feature engineering*: tempo médio por página, razão produto/administrativo, flag de alta temporada
- [ ] Interpretabilidade com SHAP
- [ ] Definir matriz de custo real (custo do cupom × margem da venda) e escolher o limiar que maximiza lucro em vez de F1

---

## 📚 Referências

- Sakar, C.O., Polat, S.O., Katircioglu, M., Kastro, Y. (2018). *Real-time prediction of online shoppers' purchasing intention using multilayer perceptron and LSTM recurrent neural networks*. **Neural Computing and Applications**.
- [UCI Machine Learning Repository — Online Shoppers Purchasing Intention Dataset](https://archive.ics.uci.edu/dataset/468/online+shoppers+purchasing+intention+dataset)
- [Dataset no Kaggle](https://www.kaggle.com/datasets/billcampos/online-shoppers)

---

## 📄 Licença

Código sob licença MIT. A base de dados segue os termos originais do UCI Machine Learning Repository.
