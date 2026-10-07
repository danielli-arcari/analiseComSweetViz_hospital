# Análise Exploratória de Dados de Câncer de Mama com Python

## Sobre o projeto

Este projeto foi desenvolvido durante minha participação em uma formação em Engenharia e Análise de Dados promovida pela [MCIO Brasil](https://mciobrasil.org.br/) em parceria com a [Leega Academy](https://leega.com.br/).

A formação faz parte da minha jornada de desenvolvimento profissional na área de dados. No Módulo 4, voltado à visualização de dados com Python, foram apresentadas técnicas de análise exploratória utilizando as bibliotecas Pandas, Seaborn, Matplotlib e Sweetviz.

Durante a aula, reproduzi o exercício proposto. No caminho, precisei resolver problemas com os arquivos de entrada e com mudanças de versão das bibliotecas, o que acabou sendo uma parte importante do aprendizado. Este repositório registra o exercício, as correções aplicadas e a minha interpretação dos resultados.

Neste projeto:

- identifiquei que os arquivos baixados não continham os dados, e sim um relatório HTML;
- localizei e validei a base original;
- resolvi problemas de leitura do arquivo no Google Colab e no Google Drive;
- adaptei o código para as versões atuais do Pandas;
- tratei a coluna vazia que impedia a geração dos gráficos;
- gerei o relatório automático de análise exploratória com o Sweetviz;
- interpretei os resultados respondendo às perguntas do cliente, como preparação para um Data Storytelling.

---

## O desafio

O cenário proposto no exercício é o de um **hospital** que forneceu um conjunto de dados sobre diagnósticos de câncer de mama. O foco é extrair insights dos dados e se preparar para um Data Storytelling.

Em reunião, os responsáveis pelo projeto pediram respostas para cinco perguntas:

1. Como estão distribuídas as amostras entre câncer benigno e maligno?
2. Como está a distribuição dos dados das outras colunas numéricas?
3. Quais são os outliers, considerando que os dados serão usados como fonte para os algoritmos de IA do hospital?
4. Qual a matriz de correlação entre as variáveis, para entender quais delas são importantes para o algoritmo?
5. Criar um dashboard de análise exploratória para os médicos, usando conceitos de estatística descritiva.

As respostas estão na seção [Respostas às perguntas do cliente](#respostas-às-perguntas-do-cliente).

---

## Resumo da base

A base utilizada é a **Breast Cancer Wisconsin (Diagnostic)**, disponível no Kaggle e no repositório de Machine Learning da UCI.

Cada linha representa um exame de uma massa mamária. As medidas foram calculadas a partir de imagens digitalizadas de punção aspirativa por agulha fina (PAAF) e descrevem as características dos núcleos celulares.

- **569 registros** e **33 colunas**
- `id`: identificador do exame
- `diagnosis`: diagnóstico, sendo **B** (benigno) ou **M** (maligno)
- **30 variáveis numéricas**, formadas por 10 características medidas de três formas:
  - `_mean`: média
  - `_se`: erro padrão
  - `_worst`: pior valor (média dos três maiores valores)
- `Unnamed: 32`: coluna vazia, gerada por uma vírgula a mais no final de cada linha do CSV

As 10 características são: raio (`radius`), textura (`texture`), perímetro (`perimeter`), área (`area`), suavidade (`smoothness`), compacidade (`compactness`), concavidade (`concavity`), pontos côncavos (`concave points`), simetria (`symmetry`) e dimensão fractal (`fractal_dimension`).

---

## Problemas encontrados e correções aplicadas

### 1. Os arquivos baixados não continham os dados

Ao executar o primeiro gráfico, o Seaborn retornou o erro:

```
ValueError: Could not interpret value `diagnosis` for `x`. An entry with this name does not appear in `data`.
```

Investigando o arquivo, percebi que o `cancer_data.csv` começava com `<!doctype html>`. Ou seja, não era uma tabela, e sim uma página HTML. Comparando os arquivos baixados, o CSV, o notebook (`.ipynb`) e o relatório (`.html`) tinham exatamente o mesmo conteúdo: os três eram o relatório gerado pelo Sweetviz.

Por isso o Pandas lia o arquivo como uma única coluna chamada `<!doctype html>`, e a coluna `diagnosis` não existia.

**Correção:** pelo conteúdo do relatório, identifiquei que a base era a Breast Cancer Wisconsin (Diagnostic), publicada no Kaggle. Obtive o arquivo original e validei que ele batia com o relatório: 569 linhas, 33 colunas, 357 diagnósticos benignos e 212 malignos.

### 2. O Google Drive converteu o CSV em planilha

Ao subir o arquivo correto no Google Drive, ele foi convertido automaticamente em Google Sheets (`.gsheet`). Esse formato é apenas um atalho para a planilha online, e o Pandas não consegue lê-lo, o que gerou um `OSError`.

**Correção:** passei a carregar o arquivo diretamente do computador para a sessão do Colab:

```python
from google.colab import files

arquivo = files.upload()
nome = list(arquivo.keys())[0]

cancer_data = pd.read_csv(nome)
cancer_data.head()
```

As linhas com `nome` garantem que o código leia exatamente o arquivo enviado, mesmo que o Colab altere o nome dele (por exemplo, `cancer_data (1).csv`).

### 3. A coluna vazia impedia a geração dos boxplots

O loop de outliers gerou um gráfico vazio e depois o erro `ValueError`. A causa era a coluna `Unnamed: 32`, que não tem nenhum valor preenchido.

**Correção:** remoção da coluna antes dos gráficos:

```python
cancer_data = cancer_data.drop(columns=["Unnamed: 32"])
```

### 4. Mudança de comportamento do Pandas na correlação

No material original, o `.corr()` ignorava automaticamente a coluna de texto `diagnosis`, apenas exibindo um aviso (`FutureWarning`) de que isso mudaria. Nas versões atuais do Pandas o comportamento mudou e a coluna de texto passou a gerar erro.

**Correção:** indicar explicitamente que apenas as colunas numéricas devem entrar no cálculo:

```python
correlation_matrix = cancer_data.corr(numeric_only=True)
```

---

## Etapas da análise

O código completo está no arquivo [`datavizusandopython_mod4_leega.py`](datavizusandopython_mod4_leega.py).

### 1. Distribuição do diagnóstico

```python
sns.countplot(x="diagnosis", data = cancer_data)
```

![Distribuição do diagnóstico](imagens/01_countplot_diagnosis.png)

### 2. Gráficos de distribuição

Um histograma para cada variável numérica.

```python
for column in cancer_data:
    if column != 'id' and column != 'diagnosis' and column != ' ':
        sns.displot(x=column,data=cancer_data)
        plt.figure()
```

Exemplo, `radius_mean`:

![Distribuição de radius_mean](imagens/02_distribuicao_radius_mean.png)

### 3. Outliers

Um boxplot para cada variável numérica.

```python
for column in cancer_data:
    if column!='id' and column!='diagnosis':
       sns.boxplot(y=column,data=cancer_data)
       plt.figure()
```

Exemplo, `area_mean`:

![Boxplot de area_mean](imagens/03_boxplot_area_mean.png)

### 4. Matriz de correlação

```python
correlation_matrix = cancer_data.corr(numeric_only=True)
plt.figure(figsize=(20,20))
sns.heatmap(correlation_matrix,cbar=True, fmt = '.1f', annot=True, cmap = 'Blues')
plt.savefig('Correlation Heat Map')
```

![Matriz de correlação](imagens/04_correlation_heat_map.png)

### 5. Relatório automático com Sweetviz

O [Sweetviz](https://github.com/fbdesignpro/sweetviz) é uma biblioteca de Python que gera, com poucas linhas de código, um relatório completo de análise exploratória em HTML. Para cada coluna, o relatório mostra tipo de dado, valores ausentes, valores distintos, estatísticas descritivas e histograma, além de um painel de associações entre as variáveis.

Ele é útil como primeiro contato com uma base nova, porque mostra rapidamente onde estão os problemas (colunas vazias, valores ausentes, distribuições estranhas) antes de começar a análise detalhada.

```python
! pip install SweetViz

import sweetviz as sv
report_cancer = sv.analyze(cancer_data)
report_cancer.show_html('eda_cancer.html')
```

O relatório gerado está em [`eda_cancer.html`](eda_cancer.html). O GitHub não exibe arquivos HTML diretamente, então para visualizar é possível baixar o arquivo e abrir no navegador, ou acessar pela [visualização online](https://htmlpreview.github.io/?https://github.com/danielli-arcari/analise-exploratoria-cancer-mama-python/blob/main/eda_cancer.html).

---

## Respostas às perguntas do cliente

### 1. Como estão distribuídas as amostras entre câncer benigno e maligno?

A base tem **357 casos benignos (62,7%)** e **212 malignos (37,3%)**.

Os benignos são maioria, mas os malignos representam mais de um terço das amostras. Para o hospital, isso significa que há volume suficiente dos dois grupos para compará-los. Para o uso em IA, é um ponto de atenção: como as classes não estão equilibradas, um algoritmo que errasse sempre para "benigno" ainda acertaria 62,7% das vezes. Por isso, a avaliação de um modelo não deveria olhar só a taxa de acerto geral, e sim quantos casos malignos ele consegue identificar.

### 2. Como está a distribuição das colunas numéricas?

A maioria das variáveis tem **assimetria à direita**: muitos exames concentrados em valores baixos e uma cauda longa de valores altos. Isso é mais forte nas variáveis de erro padrão (`_se`), como `area_se` e `concavity_se`. Já as variáveis de suavidade (`smoothness`) e a textura no pior valor (`texture_worst`) têm distribuições mais próximas da simétrica.

As variáveis também estão em **escalas muito diferentes**: `area_mean` passa de 2.000, enquanto `fractal_dimension_mean` fica em torno de 0,06. Para o uso em algoritmos, normalmente é preciso padronizar as escalas antes, para que as variáveis de valores grandes não dominem o modelo.

### 3. Quais são os outliers?

Pelo critério do boxplot (valores além de 1,5 vez o intervalo interquartil), **29 das 30 variáveis numéricas apresentam outliers**. A única que não apresenta é `concave points_worst`. As que mais têm são:

| Variável | Quantidade de outliers |
|---|---:|
| `area_se` | 65 |
| `radius_se` | 38 |
| `perimeter_se` | 38 |
| `area_worst` | 35 |
| `smoothness_se` | 30 |

O insight mais importante para o hospital é que **esses outliers não parecem ser erros de registro**. No caso de `area_mean`, por exemplo, **todos os 25 outliers são tumores malignos**. Os valores extremos representam justamente os casos mais graves.

**Recomendação:** não remover os outliers de forma automática antes de alimentar os algoritmos de IA. Removê-los apagaria parte dos casos malignos mais característicos, que são exatamente os que o algoritmo precisa aprender a reconhecer. O ideal é validar com a equipe médica se esses valores são clinicamente plausíveis.

### 4. Matriz de correlação: quais variáveis são importantes para o algoritmo?

A matriz mostra dois achados principais.

**Existem grupos de variáveis redundantes**, que medem praticamente a mesma coisa:

- `radius_mean`, `perimeter_mean` e `area_mean` têm correlação entre 0,99 e 1,00 entre si. Faz sentido geométrico: raio, perímetro e área de uma mesma forma crescem juntos.
- O mesmo acontece com as versões `_worst` dessas medidas, e entre cada medida `_mean` e a sua versão `_worst`.
- `compactness`, `concavity` e `concave points` formam um segundo grupo bastante correlacionado (entre 0,7 e 0,9).
- `id` não tem correlação relevante com nada, como esperado, já que é só um identificador e não deve entrar no algoritmo.

**Algumas variáveis têm relação forte com o diagnóstico.** Como a matriz do exercício considera apenas colunas numéricas, o diagnóstico ficou de fora. Para responder diretamente à pergunta do cliente, converti o diagnóstico em número (B = 0, M = 1) com o `LabelEncoder` e calculei a correlação de cada variável com ele:

```python
from sklearn.preprocessing import LabelEncoder

diag = LabelEncoder().fit_transform(cancer_data['diagnosis'])  # B = 0, M = 1
num = cancer_data.drop(columns=['id', 'diagnosis'])
num.corrwith(pd.Series(diag)).sort_values(ascending=False)
```

| Mais relacionadas ao diagnóstico | Correlação |
|---|---:|
| `concave points_worst` | 0,79 |
| `perimeter_worst` | 0,78 |
| `concave points_mean` | 0,78 |
| `radius_worst` | 0,78 |
| `perimeter_mean` | 0,74 |

| Sem relação com o diagnóstico | Correlação |
|---|---:|
| `fractal_dimension_mean` | -0,01 |
| `texture_se` | -0,01 |
| `symmetry_se` | -0,01 |
| `smoothness_se` | -0,07 |

Comparando as médias dos dois grupos, a diferença fica concreta:

```python
cancer_data.groupby('diagnosis')[['radius_mean', 'area_mean', 'concavity_mean', 'concave points_mean', 'fractal_dimension_mean']].mean()
```

| Variável | Benigno (média) | Maligno (média) | Maligno / Benigno |
|---|---:|---:|---:|
| `radius_mean` | 12,15 | 17,46 | 1,4x |
| `area_mean` | 462,79 | 978,38 | 2,1x |
| `concavity_mean` | 0,046 | 0,161 | 3,5x |
| `concave points_mean` | 0,026 | 0,088 | 3,4x |
| `fractal_dimension_mean` | 0,063 | 0,063 | 1,0x |

Os tumores malignos têm, em média, células com área duas vezes maior e contornos muito mais irregulares (concavidade e pontos côncavos mais de três vezes maiores). Já a dimensão fractal é praticamente igual nos dois grupos.

**Recomendação:** priorizar as variáveis de **tamanho** (raio, perímetro, área) e de **irregularidade do contorno** (concavidade e pontos côncavos). Dentro de cada grupo redundante, escolher um representante, por exemplo `radius_worst` no lugar de raio, perímetro e área juntos. Variáveis com correlação próxima de zero com o diagnóstico têm pouco a acrescentar sozinhas, mas a decisão final de retirá-las deve ser testada no próprio algoritmo.

### 5. Dashboard de análise exploratória para os médicos

O dashboard foi gerado com o Sweetviz e está no arquivo [`eda_cancer.html`](eda_cancer.html) (detalhes na seção [Relatório automático com Sweetviz](#5-relatório-automático-com-sweetviz)).

Ele reúne, em uma única página navegável, os conceitos de estatística descritiva aplicados a cada uma das variáveis: valores ausentes, valores distintos, mínimo, máximo, média, mediana, quartis, desvio padrão, assimetria e histograma. O painel de associações resume visualmente as correlações entre as variáveis.

Para os médicos, a vantagem é poder consultar qualquer variável sem precisar executar código.

---

## Data Storytelling: a mensagem principal

Se eu tivesse que apresentar estes resultados ao hospital em poucos minutos, a história seria esta:

1. **Contexto:** a base tem 569 exames, sendo pouco mais de um terço malignos.
2. **O que diferencia os tumores:** tamanho e irregularidade do contorno das células. Nos malignos, a área média é duas vezes maior e a concavidade, mais de três vezes.
3. **Cuidado com os dados extremos:** os outliers não são erros, são os casos malignos mais característicos. Removê-los prejudicaria o algoritmo.
4. **Menos é mais:** várias colunas repetem a mesma informação. Um conjunto menor e bem escolhido de variáveis tende a ser suficiente.
5. **Próximo passo:** validar essas conclusões com a equipe médica e testá-las no algoritmo de IA do hospital.

> Este é um exercício de análise exploratória para fins de estudo. Os resultados não têm finalidade diagnóstica.

---

## Estrutura do repositório

```
├── README.md
├── datavizusandopython_mod4_leega.py   # código do exercício (exportado do Colab)
├── eda_cancer.html                     # relatório gerado pelo Sweetviz
├── dados/
│   └── cancer_data.csv                 # base original
└── imagens/                            # gráficos usados neste README
```

## Ferramentas utilizadas

- Python
- Google Colab
- Pandas
- NumPy
- Seaborn
- Matplotlib
- Scikit-learn (LabelEncoder)
- Sweetviz

---

## Sobre as instituições

### Leega Academy

A Leega é uma empresa especializada em tecnologia, Dados, Analytics, Cloud e Inteligência Artificial, atuando também na formação e desenvolvimento de profissionais da área.
Conheça a Leega: https://leega.com.br/

### MCIO Brasil

A MCIO Brasil é uma associação sem fins lucrativos que trabalha para ampliar a inclusão, o desenvolvimento e a ascensão de mulheres no mercado de tecnologia, promovendo iniciativas como capacitações, mentorias e conexão com oportunidades profissionais.
Conheça a MCIO Brasil: https://mciobrasil.org.br/

A formação da qual este projeto faz parte é uma iniciativa realizada em parceria entre a MCIO Brasil e a Leega Academy.
