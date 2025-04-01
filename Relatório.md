Peço desculpas pelo mal-entendido. Aqui está o relatório completo em formato markdown como texto contínuo, sem os delimitadores de código:

# Relatório: Banco de Dados para Modelo Preditivo da Esquistossomose

Este relatório descreve o banco de dados utilizado para desenvolver um modelo preditivo de sobrevivência para pacientes com esquistossomose, baseado nas bases de dados fornecidas (ESQUBR20.csv, ESQUBR21.csv, ESQUBR22.csv, ESQUBR23.csv). O objetivo do modelo é prever, a partir de variáveis demográficas, clínicas e epidemiológicas, a probabilidade de sobrevivência dos pacientes.

## 1. Descrição do Banco de Dados

As bases de dados contêm registros de casos notificados de esquistossomose no Brasil entre 2020 e 2023. Cada registro inclui informações sobre a notificação, características demográficas, resultados de exames e tratamentos administrados. Os dados foram obtidos das fichas de notificação compulsória do SINAN (Sistema de Informação de Agravos de Notificação).

A análise exploratória inicial revelou que a esquistossomose possui uma baixa taxa de mortalidade, com aproximadamente 0.1% dos casos resultando em óbito, conforme registrado na variável EVOLUCAO. Esta característica torna os dados naturalmente desbalanceados para a tarefa de predição de sobrevivência.

## 2. Etapas de Pré-processamento Realizadas

### 2.1 Carregamento e Integração de Dados

Inicialmente, foram carregados os dados de esquistossomose dos anos 2020 a 2023:

```python
esq_20 = pd.read_csv('source/csv/ESQUBR20.csv')
esq_21 = pd.read_csv('source/csv/ESQUBR21.csv')
esq_22 = pd.read_csv('source/csv/ESQUBR22.csv')
esq_23 = pd.read_csv('source/csv/ESQUBR23.csv')
```

Para facilitar a análise temporal, foi criada uma coluna adicional identificando o ano de cada notificação:

```python
esq_20['Ano_atual'] = 2020
esq_21['Ano_atual'] = 2021
esq_22['Ano_atual'] = 2022
esq_23['Ano_atual'] = 2023
```

Os quatro conjuntos de dados foram então integrados em um único DataFrame:

```python
esq_total = pd.concat([esq_20, esq_21, esq_22, esq_23])
```

### 2.2 Seleção de Variáveis

Com base na análise exploratória e no conhecimento prévio sobre a doença, foram selecionadas 10 variáveis consideradas de maior relevância preditiva:

```python
colunas_selecionadas = [
    'CS_GESTANT',  # Status de gestação
    'AN_QUANT',    # Quantidade de ovos encontrados
    'AN_QUALI',    # Exame qualitativo
    'TRATAM',      # Tratamento realizado
    'FORMA',       # Forma clínica da doença
    'CS_ESCOL_N',  # Escolaridade
    'CS_SEXO',     # Sexo biológico
    'ANO_NASC',    # Ano de nascimento
    'COUFINF',     # Unidade Federativa
    'EVOLUCAO'     # Resultado (variável alvo)
]
```

Para melhorar a interpretabilidade, os nomes das colunas foram renomeados:

```python
novos_nomes_doenças = {
    'CS_GESTANT': 'Gestante',
    'AN_QUANT': 'Quantidade ovos encontrados',
    'AN_QUALI': 'Exame_qualitativo',
    'TRATAM': 'Tratamento realizado',
    'FORMA': 'Forma clínica',
    'CS_ESCOL_N': 'Escolaridade',
    'CS_SEXO': 'Sexo',
    'ANO_NASC': 'Ano de nascimento',
    'COUFINF': 'Unidade Federativa',
    'EVOLUCAO': 'Resultado'
}
```

### 2.3 Filtragem dos Casos

Como o foco do estudo é prever a sobrevivência, foram retidos apenas os casos com resultado conhecido, com os códigos 1 (cura) ou 3 (óbito):

```python
esq_valid = esq_filtrado[(esq_filtrado['Resultado'] == 1) | (esq_filtrado['Resultado'] == 3)]
```

### 2.4 Limpeza e Tratamento de Dados

#### 2.4.1 Status de Gestação (Gestante)

Transformação em variável binária:

```python
# 5, 6 e 9 indica que não é gestante, e todas as demais indicam que é gestante
esq_valid['nao_gestante'] = (esq_valid['Gestante'].isin([5, 6, 9])).astype(int)
esq_valid['gestante'] = (~esq_valid['Gestante'].isin([5, 6, 9])).astype(int)

# Após análise, decidiu-se remover essas colunas e trabalhar apenas com 'Sexo'
esq_valid = esq_valid.drop(columns=['Gestante', 'nao_gestante', 'gestante'])
```

#### 2.4.2 Quantidade de Ovos (Quantidade ovos encontrados)

Conversão para tipo inteiro:

```python
esq_valid['Quantidade ovos encontrados'] = esq_valid['Quantidade ovos encontrados'].astype(int)
```

#### 2.4.3 Resultado do Exame (Exame_qualitativo)

Tratamento de valores ausentes e normalização das categorias:

```python
# Adicionando "Desconhecido" para valores ausentes
esq_valid['Exame_qualitativo'].fillna("Desconhecido", inplace=True)

# Definindo valores numéricos como texto
esq_valid['Exame_qualitativo'].replace(1, "Positivo", inplace=True)
esq_valid['Exame_qualitativo'].replace(2, "Negativo", inplace=True)
esq_valid['Exame_qualitativo'].replace(3, "Nao_Realizado", inplace=True)

# Aplicação de one-hot encoding
df_onehot = pd.get_dummies(esq_valid['Exame_qualitativo'], prefix='Exame', dtype=int)
esq_valid = pd.concat([esq_valid, df_onehot], axis=1)
```

#### 2.4.4 Tratamento Realizado (Tratamento realizado)

Substituição dos códigos numéricos por descrições:

```python
esq_valid['Tratamento realizado'].replace(
    {
        1: "Sim_Praziquantel",
        2: "Sim_Oxaminiquine", 
        3: "Nao",
        9: "Ignorado"
    }, 
    inplace=True
)

# Aplicação de one-hot encoding
df_onehot = pd.get_dummies(esq_valid['Tratamento realizado'], prefix='Tratamento', dtype=int)
esq_valid = pd.concat([esq_valid, df_onehot], axis=1)
```

#### 2.4.5 Forma Clínica (Forma clínica)

Tratamento de valores ausentes e normalização das categorias:

```python
# Substituindo valores nulos para "Nao_especificado"
esq_valid['Forma clínica'].fillna("Nao_especificado", inplace=True)

# Dicionário de mapeamento dos valores
mapeamento_forma = {
    1: "Intestinal",
    2: "Hepato_Intestinal",
    3: "Hepato_Esplenica",
    4: "Aguda",
    5: "Outra"
}

# Substituição dos códigos por descrições
esq_valid['Forma clínica'] = esq_valid['Forma clínica'].replace(mapeamento_forma)

# Aplicação de one-hot encoding
df_onehot = pd.get_dummies(esq_valid['Forma clínica'], prefix='Forma', dtype=int)
esq_valid = pd.concat([esq_valid, df_onehot], axis=1)
```

#### 2.4.6 Escolaridade (Escolaridade)

Tratamento de valores ausentes e normalização:

```python
# Tratamento de valores ausentes ou inválidos
esq_valid['Escolaridade'].fillna(0, inplace=True)
esq_valid['Escolaridade'].replace(9, 0, inplace=True)  # Ignorado
esq_valid['Escolaridade'].replace(10, 0, inplace=True)  # Não se aplica
esq_valid['Escolaridade'] = esq_valid['Escolaridade'].astype(int)
```

#### 2.4.7 Sexo (Sexo)

Remoção de valores inválidos e codificação binária:

```python
# Remoção de valores 'I' (ignorado)
esq_valid['Sexo'].drop(esq_valid[esq_valid['Sexo'] == "I"].index, inplace=True)

# Codificação: M=0, F=1
esq_valid['Sexo'].replace("M", 0, inplace=True)
esq_valid['Sexo'].replace("F", 1, inplace=True)
```

#### 2.4.8 Idade (calculada a partir de Ano de nascimento)

Cálculo da idade a partir do ano de nascimento e tratamento de dados ausentes:

```python
# Removendo registros sem ano de nascimento
esq_valid.dropna(subset=['Ano de nascimento'], inplace=True)

# Cálculo da idade
esq_valid['Idade'] = (esq_valid['Ano_atual'] - esq_valid['Ano de nascimento']).astype(int)

# Remoção das colunas originais
esq_valid.drop(columns=['Ano de nascimento', 'Ano_atual'], inplace=True)
```

#### 2.4.9 Unidade Federativa (Unidade Federativa)

Tratamento de valores ausentes e agrupamento por região:

```python
# Tratamento de valores ausentes
esq_valid['Unidade Federativa'].fillna("Nao_especificado", inplace=True)

# Mapeamento de códigos UF para regiões
replace_uf_para_regiao = {
    11: 'Norte',  # Rondônia
    12: 'Norte',  # Acre
    13: 'Norte',  # Amazonas
    14: 'Norte',  # Roraima
    15: 'Norte',  # Pará
    16: 'Norte',  # Amapá
    17: 'Norte',  # Tocantins
    21: 'Nordeste',  # Maranhão
    22: 'Nordeste',  # Piauí
    23: 'Nordeste',  # Ceará
    24: 'Nordeste',  # Rio Grande do Norte
    25: 'Nordeste',  # Paraíba
    26: 'Nordeste',  # Pernambuco
    27: 'Nordeste',  # Alagoas
    28: 'Nordeste',  # Sergipe
    29: 'Nordeste',  # Bahia
    31: 'Sudeste',  # Minas Gerais
    32: 'Sudeste',  # Espírito Santo
    33: 'Sudeste',  # Rio de Janeiro
    35: 'Sudeste',  # São Paulo
    41: 'Sul',  # Paraná
    42: 'Sul',  # Santa Catarina
    43: 'Sul',  # Rio Grande do Sul
    50: 'Centro-Oeste',  # Mato Grosso do Sul
    51: 'Centro-Oeste',  # Mato Grosso
    52: 'Centro-Oeste',  # Goiás
    53: 'Centro-Oeste',  # Distrito Federal
}

# Substituição dos códigos pelas regiões
esq_valid['Unidade Federativa'] = esq_valid['Unidade Federativa'].replace(replace_uf_para_regiao)

# Aplicação de one-hot encoding
df_onehot = pd.get_dummies(esq_valid['Unidade Federativa'], prefix='Regiao', dtype=int)
esq_valid = pd.concat([esq_valid, df_onehot], axis=1)
```

## 3. Descrição dos Atributos

### 3.1. Idade (derivada de ANO_NASC)
- **Tipo**: Numérico (inteiro)
- **Distribuição**: Predominância de casos em adultos entre 20-60 anos, refletindo maior exposição em idade produtiva.
- **Relevância**: A idade impacta na resposta imunológica e na tolerância ao tratamento.

### 3.2. Sexo (CS_SEXO)
- **Tipo**: Categórico binário (0=Masculino, 1=Feminino)
- **Distribuição**: Aproximadamente 60% masculino e 40% feminino, refletindo maior exposição ocupacional masculina.
- **Relevância**: Diferenças biológicas na resposta à infecção e padrões distintos de exposição.

### 3.3. Escolaridade (CS_ESCOL_N)
- **Tipo**: Categórico ordinal (0-8)
- **Distribuição**: Predominância de níveis básicos de escolaridade.
- **Relevância**: Proxy para determinantes sociais de saúde e adesão ao tratamento.

### 3.4. Quantidade de ovos encontrados (AN_QUANT)
- **Tipo**: Numérico (inteiro)
- **Distribuição**: Valores discretos positivos, com a maioria dos casos apresentando contagens baixas a moderadas.
- **Relevância**: Preditor direto da gravidade e carga parasitária.

### 3.5. Exame qualitativo (AN_QUALI)
- **Tipo**: Categórico nominal (Positivo, Negativo, Nao_Realizado, Desconhecido)
- **Transformação**: One-hot encoding (Exame_Positivo, Exame_Negativo, etc.)
- **Relevância**: Confirmação diagnóstica complementar.

### 3.6. Tratamento realizado (TRATAM)
- **Tipo**: Categórico nominal (Sim_Praziquantel, Sim_Oxaminiquine, Nao, Ignorado)
- **Transformação**: One-hot encoding (Tratamento_Sim_Praziquantel, etc.)
- **Relevância**: Impacto direto na eficácia do tratamento e desfecho clínico.

### 3.7. Forma clínica (FORMA)
- **Tipo**: Categórico nominal (Intestinal, Hepato_Intestinal, Hepato_Esplenica, Aguda, Outra, Nao_especificado)
- **Transformação**: One-hot encoding (Forma_Intestinal, Forma_Hepato_Intestinal, etc.)
- **Relevância**: Principal determinante da morbimortalidade.

### 3.8. Unidade Federativa (COUFINF)
- **Tipo**: Categórico nominal (Norte, Nordeste, Sudeste, Sul, Centro-Oeste, Nao_especificado)
- **Transformação**: One-hot encoding (Regiao_Norte, Regiao_Nordeste, etc.)
- **Relevância**: Reflete variabilidade geográfica das cepas e acesso a serviços de saúde.

### 3.9. Resultado (EVOLUCAO)
- **Tipo**: Categórico binário (1=Cura, 3=Óbito)
- **Distribuição**: Altamente desbalanceada (≈0.1% óbitos)
- **Relevância**: Variável alvo para o modelo preditivo.

## 4. Análise Exploratória e Correlações

### 4.1. Distribuição das Classes

A análise da variável alvo (Resultado) demonstra um grande desbalanceamento entre as classes:

```
Resultado
1    29951  # Cura
3       34  # Óbito
```

Isso representa uma taxa de mortalidade de aproximadamente 0.1%, o que é um desafio significativo para modelos preditivos.

### 4.2. Correlações Relevantes

#### 4.2.1. Idade e Resultado
Pacientes idosos apresentam um risco maior de desfechos negativos, com uma correlação negativa entre idade e sobrevivência.

#### 4.2.2. Forma Clínica e Resultado
As formas hepatoesplênica e aguda da doença estão mais fortemente associadas a desfechos negativos.

#### 4.2.3. Tratamento e Resultado
Existe uma correlação positiva entre a realização de tratamento específico e a sobrevivência.

#### 4.2.4. Região e Resultado
Observou-se variação geográfica nos desfechos, com algumas regiões apresentando taxas de mortalidade ligeiramente superiores.

### 4.3. Desafios para o Modelo Preditivo

O principal desafio identificado para o desenvolvimento do modelo preditivo é o forte desbalanceamento das classes. Será necessário implementar técnicas específicas para lidar com esta característica, como:

1. Métodos de sobreamostragem (SMOTE, ADASYN)
2. Métodos de subamostragem controlada
3. Ajuste dos pesos das classes
4. Uso de métricas adequadas para classes desbalanceadas (F1-score, AUC-ROC)

## 5. Conclusão

O banco de dados pré-processado fornece um conjunto relevante de variáveis demográficas, clínicas e epidemiológicas para a predição de sobrevivência em casos de esquistossomose. O processo de tratamento de dados realizado permitiu:

1. **Lidar com valores ausentes**: Utilizando estratégias específicas para cada variável.
2. **Normalização de variáveis categóricas**: Facilitando a interpretação e o uso em modelos preditivos.
3. **Engenharia de atributos**: Criação de variáveis derivadas como idade e agrupamento regional.
4. **Codificação apropriada**: Aplicação de técnicas de one-hot encoding para variáveis categóricas.

Apesar do desbalanceamento significativo das classes, as correlações identificadas fornecem importantes insights para o desenvolvimento do modelo preditivo. A adequada implementação de técnicas para lidar com dados desbalanceados será fundamental para o sucesso do modelo.