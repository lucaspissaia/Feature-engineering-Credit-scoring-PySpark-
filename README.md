# 🔧 Feature Engineering - Pipeline de Dados para Credit Scoring

Este repositório contém o Projeto Integrado de **Feature Engineering** do **MBA em Data Science & AI da FIAP (10DTSR)**.

O objetivo foi construir um modelo de classificação de risco de crédito (Maus vs. Bons pagadores) a partir de um *dataset* **grande, complexo e com dados brutos** (`credit.csv`). O foco principal do projeto não estava na performance final do modelo, mas sim no processo iterativo de **limpeza de dados, tratamento de nulos, transformação de features e seleção de features** em larga escala.

---

## ⚙️ O Desafio: Dados Brutos e Desbalanceados

O *dataset* original apresentava diversos desafios de engenharia de dados:
* Mais de 50 colunas com nomes não padronizados.
* Múltiplas colunas com alta cardinalidade (ex: Cidades, Bairros).
* Grande volume de dados nulos (Missing Values) em colunas críticas.
* Mistura de dados numéricos e categóricos (strings).
* Classes de alvo (`ROTULO_ALVO_MAU=1`) desbalanceadas (aprox. 27% "maus").

## 🔄 Pipeline de Engenharia de Features (Iterativo)

Foram realizadas múltiplas tentativas para processar os dados, comparando os resultados:

### Tentativa 1: Apenas Numéricas (Drop NA)
* **Estratégia:** Eliminar todas as colunas categóricas e todas as linhas com valores nulos.
* **Resultado:** Modelo com performance muito ruim (perda excessiva de dados).

### Tentativa 2: Apenas Numéricas (Imputação de Nulos)
* **Estratégia:** Manter apenas colunas numéricas, mas substituir nulos pela mediana (para `MESES_RESIDENCIA`) ou por uma nova categoria (ex: -1000).
* **Resultado:** Melhor que a Tentativa 1, mas ainda descartando o potencial de dados categóricos.

### Tentativa 3: One-Hot Encoding (Pandas)
* **Estratégia:** Selecionar colunas categóricas de baixa cardinalidade (ex: `SEXO`, `ESTADO_RESIDENCIAL`) e aplicar `pd.get_dummies` (One-Hot Encoding).
* **Resultado:** Modelo com `recall` muito baixo (0.09), indicando que o desbalanceamento da base estava levando o modelo a ignorar a classe minoritária (maus pagadores).

### Solução Final: Pipeline Robusto com PySpark e Embedded Selection

Para lidar com a escala e complexidade, o processo foi refeito utilizando **PySpark** para o pré-processamento (ETL) e **XGBoost** para a seleção de features.

1.  **ETL com PySpark:**
    * Leitura dos dados brutos no Spark.
    * Renomeação e seleção de colunas.
    * Mapeamento manual de colunas categóricas (ex: `SEXO` M/F para 1/0).
    * Imputação de nulos (`fillna`) com mediana (calculada via `approxQuantile`) e valores *placeholder*.
    * Conversão do DataFrame Spark de volta para Pandas para modelagem.

2.  **Modelagem e Embedded Feature Selection (XGBoost):**
    * Treinamento de um primeiro modelo `XGBClassifier`.
    * Extração da `feature_importance_` do modelo treinado.
    * **Seleção de Features:** Deleção automática de todas as colunas com importância < 1%.
    * Treinamento de um novo modelo **final**, utilizando apenas o *subset* de features mais importantes.
    * **Resultado:** Um modelo mais leve, rápido e com melhor capacidade de generalização, focado apenas nas variáveis que realmente impactam o resultado.

---

## 🛠️ Stack Tecnológica

* **Linguagem:** Python
* **Processamento de Dados (ETL):** PySpark, Pandas
* **Modelagem:** Scikit-learn (`train_test_split`, `classification_report`)
* **Algoritmo:** XGBoost (`XGBClassifier`)
* **Técnica de Seleção:** Embedded Feature Selection (baseada em importância)

## 👥 Autores

* *(Adicione os nomes/links dos seus colegas de grupo aqui)*
