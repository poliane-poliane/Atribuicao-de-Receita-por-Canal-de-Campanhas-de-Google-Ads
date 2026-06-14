# Atribuição de Receita por Canal de Marketing Digital

Análise da contribuição de diferentes tipos de campanhas digitais sobre a receita de vendas, com aplicação de Regressão Linear e Ridge Regression.

---

## Contexto e Motivação

Durante o acompanhamento de performance de campanhas no Google Ads, surgiu a necessidade de mensurar a contribuição individual de cada tipo de campanha (Pesquisa, Display, Performance Max, Geração de Demanda, Vídeo e App) sobre a receita líquida diária.

A fonte de dados original era o GA4, mas inconsistências temporárias na plataforma tornaram os dados não confiáveis para esse período. Como alternativa, desenvolvi um modelo de regressão com os dados brutos de investimento diário por tipo de campanha e receita líquida, com o objetivo de estimar a contribuição de cada canal durante esse intervalo.

---

## Estrutura da Análise

### 1. Tratamento e Transformação dos Dados
- Carregamento de CSV com registros diários de investimento por canal e receita
- Conversão de colunas numéricas (padrão brasileiro com vírgula decimal) para `float`
- Conversão da coluna de data para `datetime` com ordenação cronológica

### 2. Análise Exploratória
- Estatística descritiva por canal
- Gráficos de espalhamento: investimento total e por canal vs. receita
- Análise de ROAS por nível de investimento (`relplot`)
- Matriz de correlação entre canais e receita

### 3. Investigação de Canais Específicos
- **Vídeo**: análise em 4 caminhos (linha de tendência, receita média com canal ativo vs. inativo, controle por faixa de investimento total, isolamento do efeito)
- **Display e Geração de Demanda**: investigação de efeito assistido sobre campanhas de Pesquisa
- Comparação de dois subperíodos: antes e depois de uma mudança de estratégia de campanhas

### 4. Modelagem

#### Regressão Linear — período completo
- Variáveis preditoras: investimento diário por canal
- Variável resposta: receita líquida diária
- Divisão cronológica treino/teste (80/20)
- Diagnóstico de multicolinearidade via **VIF (Variance Inflation Factor)**
- **Resultado**: R² = -0.42 nos dados de teste — modelo não confiável para tomada de decisão

#### Ridge Regression — período completo
- Aplicada como alternativa para tolerar a multicolinearidade sem excluir variáveis
- Padronização das variáveis via `StandardScaler`
- Teste de múltiplos valores de `alpha` (0.1, 1, 10, 100)
- **Resultado**: R² piorou progressivamente com o aumento do alpha — o problema central era a quebra de padrão entre treino e teste causada pela mudança de estratégia, não apenas a multicolinearidade

#### Modelo 1: Regressão Linear — período anterior (excluindo últimos 30 dias)
- Tentativa de isolar o padrão da estratégia anterior
- **Resultado**: coeficientes distorcidos (ex: Display = -9.69), modelo ainda não confiável

#### Modelo 2: Regressão Linear — últimos 30 dias (estratégia atual)
- Treinado apenas no período pós-mudança de estratégia
- R² = 0.58 (avaliado no próprio conjunto de treino por limitação de dados)
- Coeficientes mais interpretáveis; Vídeo consistente como canal de maior retorno estimado (ROAS implícito de ~2.69x)
- **Modelo mais aderente à realidade atual**, com limitações documentadas

### 5. Aplicações Práticas (baseadas no Modelo 2)
- **Estimativa de contribuição por canal**: investimento médio × coeficiente por canal
- **Simulador de orçamento**: entrada do usuário com canal e novo valor de investimento → receita estimada, incremento diário, projeções semanal e mensal

---

## Principais Achados

- **Vídeo** foi o canal com comportamento mais consistente em toda a análise: correlação de 0.69 com receita, coeficiente positivo nos dois modelos e maior contribuição estimada diária (R$ 5.427) mesmo com o segundo menor volume de investimento médio entre os canais ativos
- **Performance Max** apresentou comportamento inversamente correlacionado com receita nos dados completos — dias com maior investimento coincidiam com menor receita, levantando hipótese de problema de estratégia ou atribuição
- **Display e Geração de Demanda** apresentaram coeficientes negativos em todos os modelos, provavelmente por multicolinearidade residual — **não devem ser interpretados como evidência de que esses canais prejudicam receita**
- A mudança de estratégia de campanhas no período foi o principal fator de instabilidade dos modelos: o padrão aprendido no treino não se replicou no período de teste

---

## Limitações

- Modelo 2 treinado com apenas 30 dias (ideal estatístico: mínimo 60 observações para 6 variáveis)
- Ausência de divisão treino/teste no Modelo 2 — R² de 0.58 é otimista
- Multicolinearidade residual nos últimos 30 dias
- Linearidade assumida: sem modelagem de saturação de canal
- Sem controle de sazonalidade, datas comemorativas ou demanda orgânica

---

## Próximos Passos

- Retreinar o Modelo 2 com 60+ dias da estratégia atual, incluindo divisão cronológica treino/teste
- Incluir variáveis de controle (dia da semana, períodos promocionais)
- Explorar modelos não-lineares (Random Forest, Gradient Boosting) após consolidar a base de dados
- Evoluir para **Marketing Mix Modeling (MMM)** com efeitos de carryover, saturação e variáveis externas

---

## Tecnologias Utilizadas

- **Linguagem**: Python
- **Manipulação de dados**: Pandas, NumPy
- **Visualização**: Matplotlib, Seaborn
- **Modelagem**: scikit-learn (LinearRegression, Ridge, StandardScaler, train_test_split), statsmodels (VIF)

---

## Como Executar

1. Clone o repositório
2. Abra o arquivo `.ipynb` no Google Colab ou Jupyter Notebook
3. Substitua o arquivo CSV pelo seu próprio dataset (mesma estrutura de colunas)
4. Execute as células sequencialmente
5. Ao final, use o simulador interativo para explorar cenários de orçamento
