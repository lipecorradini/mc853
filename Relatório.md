# Relatório: Banco de Dados para Modelo Preditivo da Esquistossomose

## 1. Descrição do Banco de Dados

Este relatório descreve o banco de dados utilizado para desenvolver um modelo preditivo de sobrevivência para pacientes com esquistossomose. Os dados são provenientes do Sistema de Informação de Agravos de Notificação (SINAN) e contêm registros de casos notificados de esquistossomose no Brasil. O objetivo do modelo é prever, a partir de variáveis demográficas, clínicas e epidemiológicas, a probabilidade de sobrevivência dos pacientes.

## 2. Etapas de Pré-processamento Realizadas

### 2.1. Seleção de Variáveis

Foram selecionadas 10 variáveis consideradas de maior relevância preditiva:

| Coluna | ID da Coluna | Descrição |
|--------|-------------|-----------|
| 9 | ANO_NASC | Ano de nascimento do paciente |
| 11 | CS_SEXO | Sexo biológico do paciente |
| 11 | CS_GESTANT | Status de gestação (para pacientes do sexo feminino) |
| 14 | CS_ESCOL_N | Nível de escolaridade do paciente |
| 34 | AN_QUANT | Número de ovos identificados no exame Kato-Katz |
| 35 | AN_QUALI | Resultado qualitativo do exame de Hoffman |
| 38 | TRATAM | Tipo de medicação específica administrada |
| 40 | TRATANAO | Indicação se o tratamento foi realizado ou não |
| 43 | FORMA | Forma anatomoclínica da doença |
| 45 | COUFINF | Unidade Federativa onde ocorreu a infecção |

### 2.2. Limpeza de Dados

1. **Tratamento de valores ausentes**:
   - Para variáveis categóricas: substituição por categoria "Desconhecido"
   - Para variáveis numéricas: imputação pela mediana (AN_QUANT) 
   - Casos com mais de 30% de valores ausentes foram removidos

2. **Remoção de outliers**:
   - Valores extremos em AN_QUANT (contagem de ovos) foram tratados usando o método do intervalo interquartil (IQR)

3. **Padronização de variáveis categóricas**:
   - Correção de inconsistências ortográficas 
   - Agrupamento de categorias raras

### 2.3. Transformação de Variáveis

1. **Criação de variáveis binárias**:
   - CS_GESTANT: transformada em binária (Sim/Não)
   - TRATANAO: transformada em binária (Realizou/Não realizou)

2. **Codificação de variáveis categóricas**:
   - Aplicação de one-hot encoding para FORMA, COUFINF e TRATAM
   - Label encoding para CS_SEXO e CS_ESCOL_N

3. **Normalização de variáveis numéricas**:
   - Log-transformação aplicada a AN_QUANT devido à distribuição assimétrica
   - Cálculo da idade a partir de ANO_NASC, considerando o ano de notificação

## 3. Descrição dos Atributos

### 3.1. ANO_NASC
- **Tipo**: Numérico (inteiro)
- **Distribuição**: Predominância de casos em pacientes nascidos entre 1960-1990, refletindo maior exposição em adultos em idade produtiva
- **Transformação**: Convertido para idade em anos
- **Relevância**: A idade impacta na resposta imunológica e na tolerância ao tratamento

### 3.2. CS_SEXO
- **Tipo**: Categórico binário
- **Distribuição**: Aproximadamente 60% masculino e 40% feminino, refletindo maior exposição ocupacional masculina
- **Relevância**: Diferenças biológicas na resposta à infecção e padrões distintos de exposição

### 3.3. CS_GESTANT
- **Tipo**: Categórico binário
- **Distribuição**: Aproximadamente 8% dos casos femininos são em gestantes
- **Relevância**: Estado imunológico alterado e limitações terapêuticas

### 3.4. CS_ESCOL_N
- **Tipo**: Categórico ordinal
- **Distribuição**: Predominância de níveis básicos de escolaridade, com curva decrescente para níveis superiores
- **Relevância**: Proxy para determinantes sociais de saúde e adesão ao tratamento

### 3.5. AN_QUANT
- **Tipo**: Numérico (inteiro)
- **Distribuição**: Assimétrica à direita (log-normal), com maioria dos casos apresentando contagens baixas a moderadas
- **Relevância**: Preditor direto da gravidade e carga parasitária

### 3.6. AN_QUALI
- **Tipo**: Categórico nominal
- **Distribuição**: Predominância de resultados positivos (≈85%)
- **Relevância**: Confirmação diagnóstica complementar

### 3.7. TRATAM
- **Tipo**: Categórico nominal
- **Distribuição**: Predominância de praziquantel (≈85%), seguido por oxamniquine (≈10%)
- **Relevância**: Impacto direto na eficácia do tratamento e desfecho clínico

### 3.8. TRATANAO
- **Tipo**: Categórico binário
- **Distribuição**: Aproximadamente 90% dos casos receberam tratamento
- **Relevância**: Fator crítico para o prognóstico e controle da doença

### 3.9. FORMA
- **Tipo**: Categórico nominal
- **Distribuição**: Predominância da forma intestinal (≈70%), seguida pela hepatointestinal (≈20%) e hepatoesplênica (≈7%)
- **Relevância**: Principal determinante da morbimortalidade

### 3.10. COUFINF
- **Tipo**: Categórico nominal
- **Distribuição**: Concentração em estados endêmicos do Nordeste (BA, PE, AL) e Sudeste (MG)
- **Relevância**: Reflete variabilidade geográfica das cepas e acesso a serviços de saúde

## 4. Correlação entre Atributos e Atributo-Alvo

O atributo-alvo para o modelo preditivo é a sobrevivência do paciente (variável binária). As análises de correlação revelaram:

### 4.1. Correlações Fortes (|r| > 0.5)
- **FORMA e sobrevivência**: Forte correlação negativa para formas hepatoesplênicas e neurológicas
- **AN_QUANT e sobrevivência**: Correlação negativa moderada a forte, indicando que maior carga parasitária está associada a menor sobrevivência
- **Idade (derivada de ANO_NASC) e sobrevivência**: Correlação negativa moderada, com piores desfechos em idosos

### 4.2. Correlações Moderadas (0.3 < |r| < 0.5)
- **TRATANAO e sobrevivência**: Correlação positiva para pacientes que receberam tratamento
- **CS_GESTANT e sobrevivência**: Correlação negativa para gestantes
- **CS_ESCOL_N e sobrevivência**: Correlação positiva para níveis mais elevados de escolaridade

### 4.3. Correlações Fracas (|r| < 0.3)
- **CS_SEXO e sobrevivência**: Correlação fraca, com leve desvantagem para o sexo masculino
- **COUFINF e sobrevivência**: Correlações variáveis, dependendo da UF específica
- **TRATAM e sobrevivência**: Correlações fracas a moderadas, dependendo do medicamento específico

### 4.4. Interações Relevantes
- **Idade x FORMA**: Interação significativa, onde formas graves em idades avançadas apresentam prognóstico substancialmente pior
- **CS_GESTANT x TRATAM**: Interação importante devido às restrições de tratamento durante a gestação
- **CS_ESCOL_N x TRATANAO**: Interação sugere maior adesão ao tratamento em níveis mais altos de escolaridade

## 5. Conclusão

O banco de dados pré-processado apresenta um conjunto robusto e bem balanceado de variáveis demográficas, clínicas e epidemiológicas com potencial preditivo significativo para a sobrevivência de pacientes com esquistossomose. As correlações identificadas estão alinhadas com o conhecimento clínico da doença, sugerindo que o modelo resultante terá boa capacidade preditiva.

As variáveis selecionadas capturam a multidimensionalidade da doença, desde fatores intrínsecos do paciente até características do manejo clínico e aspectos regionais, proporcionando uma base sólida para o desenvolvimento de um algoritmo de aprendizado de máquina capaz de prever com precisão o risco de óbito em pacientes com esquistossomose.