# SERS — Checkpoint 02

Projeto de análise de dados e aprendizado de máquina aplicado ao setor de energia. O trabalho reúne duas tarefas complementares:

1. **Tarefa 1 — Classificação:** classificar a fonte de geração de empreendimentos cadastrados pela ANEEL a partir de potência outorgada e localização.
2. **Tarefa 2 — Regressão:** estimar a radiação solar horizontal média em Petrolina (PE) a partir de condições meteorológicas históricas e da hora local.

---

## Participantes

- **Enzo Ricardo Silva** — RM: 571333
- **Eric Hernandes Penhalbel** — RM: 570237
- **João Guilherme Figuereido** — RM: 572697
- **Ryan Luther Roque** — RM: 572993
- **Matheus Borges Soares** — RM: 574085

---

## Objetivos

### Tarefa 1 — Classificação da fonte de geração

Investigar se é possível prever a categoria de fonte de geração de um empreendimento usando somente:

- `potencia_kw`: potência outorgada, em kW;
- `latitude`: latitude aproximada;
- `longitude`: longitude aproximada.

A variável-alvo `fonte` possui três classes:

- **Solar**, a partir de `UFV`;
- **Eólica**, a partir de `EOL`;
- **Hidráulica**, agrupando `UHE`, `PCH` e `CGH`.

O objetivo é classificar a categoria cadastral da fonte de geração. Não se trata de estimar diretamente a energia produzida.

O conjunto registrado na execução do notebook contém **3.876 empreendimentos válidos** após a limpeza dos registros.

### Tarefa 2 — Regressão da radiação solar

Estimar a **radiação solar global horizontal média da hora anterior**, em W/m², para Petrolina (PE), usando:

- temperatura a 2 m;
- umidade relativa a 2 m;
- cobertura de nuvens;
- velocidade do vento a 10 m;
- hora local.

A coluna `data_hora` é usada para manter a ordem temporal e realizar a divisão entre treino e teste, mas não é utilizada como entrada do modelo.

Importante: radiação em W/m² é uma medida de potência solar por área. Ela não equivale diretamente à energia elétrica produzida por um sistema fotovoltaico em kWh.

---

## Fontes e período dos dados

### ANEEL — SIGA

**Fonte:** Sistema de Informações de Geração da ANEEL (SIGA).

- Recurso consultado: `11ec447d-698d-4ab8-977f-b424d5deee6a`
- API utilizada: CKAN/DataStore da ANEEL.
- Tipos consultados: `UFV`, `EOL`, `UHE`, `PCH` e `CGH`.
- Campos utilizados: potência outorgada, latitude e longitude.
- Alvo: `fonte`.

O notebook consulta o **cadastro disponível no momento da execução**. Não há um intervalo histórico de datas definido para a Tarefa 1. A consulta solicita até 1.200 registros por sigla, portanto os resultados dependem dos registros retornados pela API no momento da coleta.

Fontes oficiais:

- [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel)
- [ANEEL — recurso e campos usados](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel/resource/11ec447d-698d-4ab8-977f-b424d5deee6a)

### Open-Meteo — histórico meteorológico

A Tarefa 2 utiliza a Historical Weather API do Open-Meteo.

- Localidade: **Petrolina (PE)**
- Coordenadas aproximadas: `-9.39, -40.50`
- Período: **01/04/2025 a 30/06/2025**, inclusive
- Fuso horário: `America/Recife`
- Frequência: horária
- Recorte utilizado: horas entre **7h e 17h**, inclusive
- Registros válidos na execução registrada: **1.001 horas**

Variáveis:

| Campo no CSV | Variável | Unidade | Uso |
|---|---|---:|---|
| `data_hora` | `time` | data/hora local | ordenação e divisão temporal |
| `temperatura_c` | `temperature_2m` | °C | entrada |
| `umidade_pct` | `relative_humidity_2m` | % | entrada |
| `nuvens_pct` | `cloud_cover` | % | entrada |
| `vento_kmh` | `wind_speed_10m` | km/h | entrada |
| `hora` | hora extraída de `time` | 0–23 | entrada |
| `radiacao_w_m2` | `shortwave_radiation` | W/m² | alvo |

Os dados históricos do Open-Meteo são estimativas provenientes de modelos/reanálise, e não medições de um sensor específico.

Fonte oficial:

- [Open-Meteo — Historical Weather API e unidades](https://open-meteo.com/en/docs/historical-weather-api)

---

## Estrutura do projeto

Mantenha todos os arquivos na mesma pasta:

```text
.
├── README.md
├── Tarefa01_SERS_Checkpoint02.ipynb
├── Tarefa02_SERS_Checkpoint02.ipynb
├── aneel_classificacao_orange.csv
└── meteo_regressao_orange.csv
```

Os dois CSVs são gerados pelos próprios notebooks. Se eles não forem incluídos na entrega, podem ser reproduzidos pelas células de consulta às APIs.

Não são necessários tokens, senhas ou credenciais.

---

## Requisitos

Recomenda-se:

- Python 3.10 ou superior;
- Jupyter Notebook ou JupyterLab;
- acesso à internet para consultar as APIs públicas.

Instalação:

```bash
python -m pip install --upgrade pip
python -m pip install notebook ipykernel pandas numpy matplotlib seaborn scikit-learn
```

As consultas às APIs usam módulos da biblioteca padrão do Python, principalmente `json`, `urllib` e `csv`.

---

## Como executar

### 1. Preparar o ambiente

Opcionalmente, crie um ambiente virtual:

```bash
python -m venv .venv
```

No Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

No Linux/macOS:

```bash
source .venv/bin/activate
```

Depois instale as dependências:

```bash
python -m pip install --upgrade pip
python -m pip install notebook ipykernel pandas numpy matplotlib seaborn scikit-learn
```

### 2. Executar a Tarefa 1

Abra:

```text
Tarefa01_SERS_Checkpoint02.ipynb
```

Execute **todas as células de cima para baixo**, sem pular as células de consulta e preparação.

A sequência principal é:

1. importar as bibliotecas;
2. consultar a API da ANEEL;
3. filtrar e limpar os registros;
4. gerar `aneel_classificacao_orange.csv`;
5. carregar o CSV;
6. analisar tipos, classes e valores ausentes;
7. definir `X` e `y`;
8. fazer divisão estratificada 80%/20% com `random_state=42`;
9. ajustar `StandardScaler` somente no conjunto de treino;
10. treinar os três classificadores;
11. calcular as quatro métricas;
12. apresentar as matrizes de confusão;
13. registrar a interpretação final.

### 3. Executar a Tarefa 2

Abra:

```text
Tarefa02_SERS_Checkpoint02.ipynb
```

Execute **todas as células de cima para baixo**.

A sequência principal é:

1. importar os módulos da biblioteca padrão;
2. consultar o histórico do Open-Meteo;
3. filtrar as horas entre 7h e 17h;
4. remover registros com valores ausentes;
5. gerar `meteo_regressao_orange.csv`;
6. carregar e analisar o CSV;
7. definir `X` e `y`;
8. manter a ordem cronológica;
9. utilizar aproximadamente 80% das primeiras observações para treino e 20% finais para teste;
10. treinar os três regressores;
11. calcular MAE, MSE e R²;
12. apresentar gráfico de valores reais versus previstos;
13. registrar a interpretação final.

### 4. Executar pelo terminal

Na pasta do projeto, também é possível abrir o Jupyter com:

```bash
jupyter lab
```

ou:

```bash
jupyter notebook
```

---

## CSVs gerados

### `aneel_classificacao_orange.csv`

Gerado por `Tarefa01_SERS_Checkpoint02.ipynb`.

Colunas:

```text
potencia_kw, latitude, longitude, fonte
```

Cada linha representa um empreendimento válido.

### `meteo_regressao_orange.csv`

Gerado por `Tarefa02_SERS_Checkpoint02.ipynb`.

Colunas:

```text
data_hora, temperatura_c, umidade_pct, nuvens_pct, vento_kmh, hora, radiacao_w_m2
```

Cada linha representa uma hora válida entre 7h e 17h.

Se os CSVs não estiverem incluídos na entrega, execute o notebook correspondente desde a primeira célula para recriá-los. A reprodução exige conexão com as APIs públicas.

---

# Tarefa 1 — Classificação

Arquivo:

```text
Tarefa01_SERS_Checkpoint02.ipynb
```

## Preparação

As entradas são:

```text
X = potencia_kw, latitude, longitude
```

O alvo é:

```text
y = fonte
```

A divisão é estratificada:

- 80% treino;
- 20% teste;
- `random_state=42`.

O `StandardScaler` é ajustado somente no conjunto de treino e posteriormente aplicado ao teste, evitando vazamento de informação.

## Três classificadores

### 1. K-Nearest Neighbors (KNN)

Classifica uma observação considerando os vizinhos mais próximos. O escalonamento é importante porque o algoritmo utiliza distâncias.

### 2. Regressão Logística

Modelo linear de classificação multiclasse utilizado como referência. É simples, eficiente e interpretável.

### 3. Random Forest Classifier

Combina várias árvores de decisão e consegue representar relações não lineares e interações entre as variáveis.

## Comparação 1 — Métricas de classificação

As três alternativas são avaliadas sobre a mesma divisão de teste usando:

- Accuracy;
- Precision macro;
- Recall macro;
- F1-score macro.

Valores registrados na execução salva no notebook:

| Modelo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|---|---:|---:|---:|---:|
| KNN | 0,9665 | 0,9677 | 0,9649 | 0,9662 |
| Regressão Logística | 0,8235 | 0,8275 | 0,8200 | 0,8182 |
| Random Forest | 0,9755 | 0,9769 | 0,9741 | 0,9753 |

Esses valores correspondem à execução registrada no arquivo `.ipynb`. Como a consulta à ANEEL é dinâmica, uma nova coleta pode produzir dados e métricas diferentes.

## Comparação 2 — Matrizes de confusão

O notebook apresenta uma matriz de confusão para cada um dos três classificadores.

Na matriz registrada para o Random Forest, as classes são:

```text
Eólica
Hidráulica
Solar
```

e os valores são:

| Classe real / prevista | Eólica | Hidráulica | Solar |
|---|---:|---:|---:|
| Eólica | 235 | 4 | 1 |
| Hidráulica | 2 | 294 | 0 |
| Solar | 5 | 7 | 228 |

As matrizes permitem observar diretamente quais classes são confundidas pelo modelo.

## Resposta da Tarefa 1

A interpretação registrada no notebook aponta o **Random Forest Classifier** como o modelo com os maiores valores das métricas na execução documentada.

O notebook também destaca que potência e localização não são suficientes para representar todos os fatores que influenciam a escolha e a operação de uma fonte de geração. Entre as limitações discutidas estão clima, topografia, recursos naturais, infraestrutura, tecnologia, custos, incentivos e características específicas de cada empreendimento.

---

# Tarefa 2 — Regressão

Arquivo:

```text
Tarefa02_SERS_Checkpoint02.ipynb
```

## Preparação

As entradas são:

```text
X = temperatura_c, umidade_pct, nuvens_pct, vento_kmh, hora
```

O alvo é:

```text
y = radiacao_w_m2
```

A divisão preserva a ordem temporal:

- 800 primeiras observações para treino;
- 201 observações finais para teste;
- total de 1.001 observações.

Não há embaralhamento.

## Três regressores

### 1. Regressão Linear

Modelo simples e interpretável, utilizado como baseline para verificar quanto pode ser explicado por uma relação aproximadamente linear.

### 2. Random Forest Regressor

Combina árvores de decisão e consegue capturar relações não lineares e interações entre as variáveis.

### 3. Gradient Boosting Regressor

Constrói modelos de árvore sequencialmente, procurando corrigir os erros dos modelos anteriores.

## Comparação 3 — MAE

O MAE mede o erro absoluto médio das previsões, em W/m².

Na execução documentada:

| Modelo | MAE (W/m²) |
|---|---:|
| Regressão Linear | 145,204885 |
| Random Forest | 66,804179 |
| Gradient Boosting | 67,081644 |

## Comparação 4 — MSE

O MSE penaliza erros maiores de forma mais intensa.

| Modelo | MSE ((W/m²)²) |
|---|---:|
| Regressão Linear | 30034,201092 |
| Random Forest | 7307,417901 |
| Gradient Boosting | 7444,700189 |

## Comparação 5 — R²

O R² mede a proporção da variabilidade do alvo explicada pelo modelo.

| Modelo | R² |
|---|---:|
| Regressão Linear | 0,359845 |
| Random Forest | 0,844248 |
| Gradient Boosting | 0,841322 |

## Comparação 6 — Valores reais × previstos

O notebook também produz um gráfico de **valores reais versus previstos** para o Random Forest Regressor.

A visualização permite verificar graficamente a proximidade entre as previsões e os valores observados no conjunto de teste.

## Resposta da Tarefa 2

Na execução registrada, Random Forest e Gradient Boosting apresentaram erros menores e R² maiores que a Regressão Linear. Isso é compatível com a presença de relações não lineares entre as condições meteorológicas, a hora e a radiação solar.

A variável `hora` é importante porque a radiação acompanha o ciclo diário do Sol. O gráfico de radiação média por hora produzido no notebook permite visualizar essa variação.

A estimativa da radiação solar não determina automaticamente a energia elétrica produzida por um sistema fotovoltaico. Para estimar geração seriam necessários outros fatores, como área e potência dos módulos, orientação e inclinação, temperatura dos painéis, sombreamento, sujeira, perdas elétricas, eficiência do inversor e características específicas do sistema.

---

## Onde encontrar as seis comparações e as duas respostas

Para facilitar a avaliação, os **seis modelos comparados** estão identificados individualmente nos notebooks:

| Comparação | Tarefa | Modelo | Onde encontrar |
|---|---|---|---|
| 1 | Tarefa 1 — Classificação | K-Nearest Neighbors (KNN) | seção **3. Escolha e Treinamento de Três Classificadores** |
| 2 | Tarefa 1 — Classificação | Regressão Logística | seção **3. Escolha e Treinamento de Três Classificadores** |
| 3 | Tarefa 1 — Classificação | Random Forest Classifier | seção **3. Escolha e Treinamento de Três Classificadores** |
| 4 | Tarefa 2 — Regressão | Regressão Linear | seção **3. Escolha, justifique e treine três algoritmos de regressão diferentes** |
| 5 | Tarefa 2 — Regressão | Random Forest Regressor | seção **3. Escolha, justifique e treine três algoritmos de regressão diferentes** |
| 6 | Tarefa 2 — Regressão | Gradient Boosting Regressor | seção **3. Escolha, justifique e treine três algoritmos de regressão diferentes** |

### Avaliação dos seis modelos

**Tarefa 1:** os três classificadores são comparados pelas métricas Accuracy, Precision macro, Recall macro e F1-score macro. A seção **4. Comparação de Métricas de Avaliação e Matrizes de Confusão** também apresenta uma matriz de confusão para cada modelo.

**Tarefa 2:** os três regressores são comparados pelas métricas MAE, MSE e R². A seção **4. Compare MAE, MSE e R²** também apresenta o gráfico de valores reais versus previstos para o Random Forest Regressor.

### Duas respostas finais

- **Resposta da Tarefa 1:** seção **5. Escolha do Modelo, Classes Confusas e Limitações para Aplicações Reais**.
- **Resposta da Tarefa 2:** seção **5. Interpretação dos Erros e Discussão do Peso da Hora**.

Assim, a banca consegue localizar diretamente os três classificadores, os três regressores, suas métricas, as matrizes/gráficos e as duas conclusões, sem depender de procurar essas informações em outras partes do projeto.

## Figuras e visualizações

Não há dependência de arquivos de imagem externos para a análise.

As visualizações são produzidas diretamente pelos notebooks:

- matrizes de confusão dos três classificadores;
- gráfico da radiação solar média por hora do dia;
- gráfico de valores reais versus previstos para o Random Forest Regressor.

As bibliotecas `matplotlib` e `seaborn` são utilizadas para as visualizações.

---

## Verificação de execução

Foi feita uma inspeção da ordem das células dos dois notebooks e um teste sequencial em ambiente temporário, usando respostas simuladas para as APIs.

Foi verificado que:

- os imports necessários aparecem antes do uso das bibliotecas;
- a consulta à ANEEL ocorre antes da criação e leitura de `aneel_classificacao_orange.csv`;
- a consulta ao Open-Meteo ocorre antes da criação e leitura de `meteo_regressao_orange.csv`;
- a divisão treino/teste ocorre antes do treinamento dos modelos;
- o `StandardScaler` da Tarefa 1 é ajustado somente no treino;
- a Tarefa 2 mantém a ordem temporal e não embaralha os dados;
- os três classificadores estão implementados;
- os três regressores estão implementados;
- as métricas e conclusões aparecem depois do treinamento e das previsões;
- em teste local com respostas simuladas, os dois notebooks executaram as células de código em sequência, criaram os respectivos CSVs e chegaram às etapas de treinamento e avaliação sem erro.

### Limitação do teste das APIs reais

A execução neste ambiente não conseguiu acessar diretamente os servidores externos das APIs. Portanto, o teste realizado confirma a **lógica e a execução sequencial dos notebooks**, mas não substitui uma execução final com as APIs reais e conexão com a internet.

Como os CSVs não estão incluídos nesta pasta de trabalho, a entrega pode usar as instruções de reprodução deste README. Antes do envio definitivo, recomenda-se executar uma vez cada notebook em um ambiente com internet e confirmar que:

1. `aneel_classificacao_orange.csv` é criado pela Tarefa 1;
2. `meteo_regressao_orange.csv` é criado pela Tarefa 2;
3. os dois notebooks chegam à última célula sem erro.


---

## Reprodutibilidade

As consultas são feitas diretamente às APIs públicas e não utilizam credenciais.

Para reproduzir os resultados em um ambiente com internet:

1. coloque os dois notebooks na mesma pasta;
2. instale as dependências;
3. tenha acesso à internet;
4. execute `Tarefa01_SERS_Checkpoint02.ipynb` do início ao fim;
5. execute `Tarefa02_SERS_Checkpoint02.ipynb` do início ao fim;
6. confirme a criação dos arquivos:
   - `aneel_classificacao_orange.csv`
   - `meteo_regressao_orange.csv`.

A Tarefa 1 utiliza `random_state=42` na divisão e nos modelos em que esse parâmetro se aplica.

A Tarefa 2 mantém a ordem cronológica e utiliza as primeiras aproximadamente 80% das observações para treino e as 20% finais para teste.

Como a API da ANEEL consulta um cadastro que pode ser atualizado, uma nova execução pode gerar quantidade de registros, distribuição de classes e métricas diferentes das saídas atualmente salvas no notebook.

---

## Integridade da entrega

Não são necessários:

- tokens;
- senhas;
- chaves de API;
- arquivos pessoais;
- arquivos sem relação com a avaliação.

A entrega deve conter somente os arquivos relacionados ao projeto:

```text
README.md
Tarefa01_SERS_Checkpoint02.ipynb
Tarefa02_SERS_Checkpoint02.ipynb
aneel_classificacao_orange.csv
meteo_regressao_orange.csv
```

