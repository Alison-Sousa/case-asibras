# 📊 Repositório de Análises e Modelagem de Dados

Este projeto consolida dois pipelines completos de ciência de dados, demonstrando metodologias de classificação e previsão de séries temporais. Cada análise é autocontida e inclui desde o pré-processamento dos dados até a interpretação dos resultados do modelo.

## 📂 Estrutura do Projeto

A organização deste repositório foi pensada para separar os dados, o código-fonte e os relatórios de análise.

├── 📄 README.md # (Este arquivo)

├── 📦 data/

│   ├── ✈️ AirPassengers.csv # Dados brutos da série temporal

│   └── 🚶‍♂️ Churn.csv  # Dados brutos de churn de clientes

│

├── 📑 reports/

│   ├── ✈️ AirPassengers_Report.pdf # (Relatório técnico em LaTeX/PDF da análise 2)

│   └── 🚶‍♂️ Churn_Report.pdf # (Relatório técnico em LaTeX/PDF da análise 1)

│

├── 💻 src/ case.ipynb

│   ├── ✈️ 02_Pipeline_Series_Temporais.ipynb # Notebook com o código e saídas da Análise 2

│   └── 🚶‍♂️ 01_Pipeline_Churn.ipynb # Notebook com o código e saídas da Análise 1

├── 🔧 requirements.txt  # Lista de bibliotecas


* `data/`: Contém os conjuntos de dados brutos em formato `.csv` que servem de entrada para os pipelines.
* `reports/`: Contém os relatórios técnicos formais (gerados em LaTeX e compilados em PDF) que detalham e interpretam os resultados encontrados nos *notebooks*.
* `src/`: Contém os *notebooks* Jupyter (ou scripts Python) com todo o processo de análise, incluindo código, visualizações e saídas de execução.
* `requirements.txt`: Arquivo que especifica as bibliotecas e suas versões para garantir a reprodutibilidade das análises.

---

## 1. 🚶‍♂️ Análise de Churn de Clientes (Telco)

Esta análise aborda um problema de **classificação binária** para prever se um cliente irá ou não cancelar seu contrato (`Churn: Yes/No`).

### 🎯 O Problema
O objetivo é construir um modelo que não apenas preveja o *churn*, mas que também nos ajude a entender **quais fatores** mais influenciam essa decisão. Isso permite que a empresa atue de forma proativa para reter clientes em risco.

### ⚙️ Metodologia de Execução
O pipeline (`src/01_Pipeline_Churn.ipynb`) executa as seguintes etapas:

1.  **Pré-processamento:**
    * A coluna `TotalCharges` é convertida de `object` para `numeric`, e os 11 valores ausentes (de clientes novos) são preenchidos com `0`.
    * As variáveis numéricas (`tenure`, `MonthlyCharges`, `TotalCharges`) são padronizadas (com `StandardScaler`).
    * As variáveis categóricas são transformadas em colunas binárias (com `OneHotEncoder`).
    * Os dados são divididos em treino (80%) e teste (20%) de forma **estratificada**, garantindo que a proporção de *churn* (26.54%) seja idêntica em ambos os conjuntos.

2.  **Modelagem Comparativa:**
    * **Regressão Logística:** Um modelo linear, interpretável por padrão.
    * **XGBoost:** Um modelo baseado em *gradient boosting*, conhecido por seu alto desempenho.

3.  **Explicabilidade (XAI) com SHAP:**
    * O modelo XGBoost é analisado usando valores SHAP para entender o impacto de cada variável.
    * O código detectou uma incompatibilidade de versão entre o `xgboost` (v3.1.1) e o `shap` (v0.49.1), que impedia o uso do `TreeExplainer` (devido ao formato do `base_score`).
    * O pipeline ativou com sucesso um mecanismo de *fallback*, utilizando o `PermutationExplainer`, que é agnóstico ao modelo e permitiu o cálculo das contribuições de cada *feature*.

### 📊 Resultados Principais
* **Desempenho:** No conjunto de teste, o modelo de **Regressão Logística (AUC: 0.8421)** apresentou um poder de discriminação ligeiramente superior ao **XGBoost (AUC: 0.8152)**. Em termos de F1-Score para a classe "Churn", a Regressão Logística (0.61) também superou o XGBoost (0.54).
* **Importância (SHAP):** A análise de explicabilidade do XGBoost identificou os principais fatores de influência:
    1.  `tenure`: O fator mais importante. Clientes com baixo tempo de contrato têm uma propensão muito maior ao *churn*.
    2.  `Contract_Month-to-month`: Ter um contrato "mês a mês" (sem fidelidade) é o segundo maior indicador de risco de *churn*.
    3.  `MonthlyCharges`: Clientes com cobranças mensais mais altas tendem a cancelar mais.

---

## 2. ✈️ Previsão de Séries Temporais (AirPassengers)

Esta análise aborda um problema de **regressão para previsão de série temporal univariada**.

### 🎯 O Problema
O objetivo é prever o número de passageiros de companhias aéreas para os próximos meses, com base em 144 observações mensais (12 anos, de 1949 a 1960).

### ⚙️ Metodologia de Execução
O pipeline (`src/02_Pipeline_Series_Temporais.ipynb`) segue um fluxo rigoroso para modelagem temporal:

1.  **Análise Exploratória (EDA):**
    * A série temporal (Figura `serie_temporal.png`) mostra uma clara **Tendência** de crescimento e uma forte **Sazonalidade** de 12 meses.
    * A variância da sazonalidade aumenta junto com o nível da série, indicando uma natureza **multiplicativa**.
    * A decomposição multiplicativa (Figura `decomposicao.png`) isola com sucesso os três componentes (Tendência, Sazonalidade e Resíduos).

2.  **Testes de Estacionariedade:**
    * Os testes **ADF (p=0.99)** e **KPSS (p=0.01)** confirmam inequivocamente que a série original não é estacionária, necessitando de diferenciação e/ou transformação.

3.  **Modelagem Comparativa:**
    * **SARIMA (SARIMAX):** Modelo estatístico clássico. Foi aplicada uma transformação de `log` para estabilizar a variância. Uma busca em grade (grid search) por AIC encontrou a ordem ótima `(1, 0, 0)x(1, 0, 1, 12)`.
    * **Random Forest (com Lags):** Modelo de *machine learning*. Para que o RF "entendesse" o tempo, foram criadas *features* como: lags (1 a 12), médias móveis (janela 12) e o mês do ano.
    * **Naive Sazonal:** Modelo de *baseline* que prevê o valor do mesmo mês do ano anterior.

4.  **Validação (Walk-Forward):**
    * Foi utilizada a metodologia de **validação *walk-forward*** (ou *expanding window*) para os 24 meses do período de teste (1959-1960).
    * Para prever *cada* ponto de teste, os modelos foram **completamente re-treinados** com todos os dados históricos disponíveis até aquele ponto, simulando um ambiente de produção real e evitando qualquer vazamento de dados (*data leakage*).

### 📊 Resultados Principais

* **Desempenho (Métricas):** O **SARIMA foi o vencedor indiscutível**, com um Erro Percentual Médio (MAPE) de apenas **2.44%**.

    Abaixo está a comparação de desempenho, ordenada do melhor (menor erro) para o pior:

    **🥇 1. SARIMA (SARIMAX)**
    * **MAE:** 11.15
    * **RMSE:** 15.31
    * **MAPE:** 2.44%

    **🥈 2. RandomForest (lags)**
    * MAE: 23.84
    * RMSE: 32.76
    * MAPE: 5.02%

    **🥉 3. Naive Sazonal (Baseline)**
    * MAE: 47.58
    * RMSE: 49.99
    * MAPE: 10.52%

* **Análise Gráfica:** O gráfico de previsões (`forecast_vs_real.png`) mostra que o SARIMA (azul) capturou perfeitamente tanto a tendência crescente quanto a sazonalidade. O RandomForest (verde) aprendeu a sazonalidade, mas falhou em extrapolar a tendência (suas previsões ficaram "achatadas").

* **Diagnóstico de Resíduos:** A análise final (`diagnostico_residuos.png`) confirmou a superioridade do SARIMA.
    * **SARIMA:** Os resíduos não apresentaram autocorrelação (Teste Ljung-Box p=0.10), comportando-se como **ruído branco**. Isso indica que o modelo capturou toda a informação previsível dos dados.
    * **Random Forest:** Os resíduos mostraram padrões claros de autocorrelação, indicando que o modelo estava **subajustado (*underfit*)** e deixou informação por capturar.

---

## 🚀 Como Reproduzir os Resultados

### 1. 🔧 Configuração do Ambiente
Clone este repositório e instale as dependências listadas no arquivo `requirements.txt`.

```bash
git clone https://[https://github.com/Alison-Sousa/case-asibras.git]
cd [case-asibras]
pip install -r requirements.txt
2. 💻 Execução
Abra os notebooks na pasta src/ usando o Jupyter Notebook ou Google Colab.

Para 01_Pipeline_Churn.ipynb, você precisará fazer o upload do arquivo data/Churn.csv quando o script solicitar.

Para 02_Pipeline_Series_Temporais.ipynb, o pipeline também permite o upload do data/AirPassengers.csv, embora o script tenha um fallback para carregar o dataset padrão do statsmodels.

3. 📦 Bibliotecas Principais (requirements.txt)
pandas
numpy
scikit-learn
xgboost==1.7.6  # Versão específica usada no pipeline de Churn
shap==0.44.1    # Versão específica usada no pipeline de Churn
statsmodels
matplotlib
seaborn