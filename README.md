# Documentação do Projeto de Análise de Preços de Combustíveis

## 1. Introdução

Este projeto realiza análise estratégica dos preços de combustíveis no Brasil utilizando dados semestrais da Agência Nacional do Petróleo, Gás Natural e Biocombustíveis (ANP). O estudo abrange o período de 2020 a 2024 e foca em quatro dimensões principais:

1. **Evolução temporal** dos preços de gasolina e etanol
2. **Distribuição geográfica** da infraestrutura de postos
3. **Padrões sazonais** de variação de preços
4. **Variação regional** e volatilidade do mercado de etanol

A implementação utilizou tecnologias modernas de processamento distribuído, incluindo:

- Ambiente Databricks para execução e gerenciamento
- PySpark para manipulação de dados em escala
- Delta Lake para armazenamento otimizado
- SQL para análises estratégicas

## 2. Objetivos

### 2.1 Objetivo Principal
Fornecer insights acionáveis sobre o mercado de combustíveis brasileiro através de análise de dados oficiais.

### 2.2 Objetivos Específicos

| Área de Análise | Métrica | Aplicação |
|-----------------|---------|-----------|
| Evolução de Preços | Variação média anual por estado | Estratégias de precificação regional |
| Distribuição Geográfica | Top 10 municípios por número de postos | Planejamento de expansão de rede |
| Sazonalidade | Comparativo semestral de preços | Previsão de demanda sazonal |
| Volatilidade do Etanol | Desvio padrão por estado | Gestão de risco de preços |

## 3. Arquitetura e Ferramentas

### 3.1 Stack Tecnológico

**Camada** | **Tecnologia** | **Função**
-----------|----------------|-----------
Processamento | Databricks Community Edition | Ambiente de execução
| | PySpark | Transformação de dados em escala
Armazenamento | Delta Lake | Tabelas otimizadas
| | DBFS | Armazenamento temporário
Desenvolvimento | Python | Scripts de apoio
| | SQL | Análises estratégicas

### 3.2 Bibliotecas Principais

- **Processamento de dados**: PySpark SQL
- **Download de arquivos**: requests, zipfile, io
- **Manipulação de dados**: datetime, decimal

## 4. Modelo de Dados

### 4.1 Fonte de Dados
Dados abertos da ANP (https://www.gov.br/anp/pt-br/centrais-de-conteudo/dados-abertos/arquivos/shpc/dsas/ca/)

**Período**: 2020-2024 (10 semestres)  
**Formato**: 
- 2020-2021: CSVs diretos (ca-AAAA-SS.csv)
- 2022-2024: Arquivos ZIP contendo CSVs

### 4.2 Esquema Final

**Coluna Original** | **Coluna Transformada** | **Tipo** | **Descrição**
-------------------|------------------------|----------|--------------
Regiao - Sigla | regiao | string | Sigla da região
Estado - Sigla | estado | string | UF do posto
Valor de Venda | valorVenda | decimal(10,2) | Preço de venda
Data da Coleta | dataColeta | date | Data da coleta
CNPJ da Revenda | cnpj | string | Identificador único
Numero Rua | numeroRua | integer | Número do endereço

**Campos adicionados**:
- data_carga: date (data do processamento)
- data_hora_carga: timestamp (UTC-3)

## 5. Fluxo de Processamento

### 5.1 Pipeline Principal

1. **Extração**
   - Download automático de arquivos
   - Tratamento de formatos distintos (CSV/ZIP)
   - Padronização de nomes de arquivos

2. **Transformação**
   - Consolidação de semestres
   - Conversão de tipos de dados
   - Normalização de nomes de colunas
   - Adição de metadados de carga

3. **Carga**
   - Armazenamento em tabela Delta
   - Otimização com OPTIMIZE
   - Criação de tabelas analíticas

4. **Análise**
   - Execução de consultas estratégicas
   - Geração de visualizações
   - Exportação de resultados

### 5.2 Tratamento de Dados

**Desafio** | **Solução Implementada** | **Impacto**
------------|--------------------------|-----------
Arquivo fora do padrão | Validação estrutural | Garantia de consistência
Restrições de permissão | Gravação direta no DBFS | Continuidade do processamento
Tipagem inadequada | Conversão explícita de tipos | Qualidade dos dados analíticos

## 6. Resultados e Aplicações

### 6.1 Principais Saídas

- Tabelas Delta analíticas:
  - `silver.combustivel` (dados consolidados)
  - `gold.preco_combustivel_ano` (resultados agregados)

- Visualizações interativas:
  - Séries temporais de preços
  - Mapas de distribuição geográfica
  - Comparativos semestrais

### 6.2 Aplicações Práticas

**Área** | **Aplicação** | **Benefício**
---------|---------------|-------------
Gestão Comercial | Estratégias de precificação regional | Competitividade de mercado
Logística | Planejamento de distribuição | Redução de custos operacionais
Políticas Públicas | Monitoramento de mercado | Regulação eficiente

## 7. Próximas Etapas

### 7.1 Melhorias Técnicas

- Implementação de pipeline agendado
- Particionamento por região/temporalidade
- Documentação de metadados completa

### 7.2 Expansão Analítica

- Análise comparativa por bandeiras
- Correlação com indicadores econômicos
- Modelos preditivos de preços

### 7.3 Governança de Dados

- Controles de qualidade automatizados
- Monitoramento de atualizações da fonte
- Versionamento de resultados

## Perguntas de Negócio Respondidas

### 1. Evolução do Preço Médio por Ano e Estado
**Pergunta:** Como os preços médios de combustível variaram por ano e estado?

**Consulta:**
```sql
SELECT 
    YEAR(dataColeta) AS ano, 
    estado, 
    Produto, 
    AVG(valorVenda) AS preco_medio
FROM silver.combustivel
WHERE Produto
GROUP BY ano, estado, Produto
ORDER BY ano, estado;
```

**Insight:** Identifica tendências regionais de preços ao longo do tempo, sendo útil para estratégias de mercado e precificação.

---

### 2. Municípios com Maior Número de Postos
**Pergunta:** Quais são os 10 municípios com maior número de postos?

**Consulta:**
```sql
SELECT 
    Municipio, 
    estado, 
    COUNT(DISTINCT cnpj) AS num_postos
FROM silver.combustivel
GROUP BY Municipio, estado
ORDER BY num_postos DESC
LIMIT 10;
```

**Insight:** Revela áreas com alta densidade de postos, apoiando decisões de planejamento logístico e expansão.

---

### 3. Sazonalidade dos Preços
**Pergunta:** Como os preços de combustível variam entre o 1º e 2º semestre?

**Consulta:**
```sql
SELECT 
    YEAR(dataColeta) AS ano,
    CASE 
        WHEN MONTH(dataColeta) <= 6 THEN '1º Semestre' 
        ELSE '2º Semestre' 
    END AS semestre,
    Produto,
    AVG(valorVenda) AS preco_medio
FROM silver.combustivel
WHERE Produto
GROUP BY ano, semestre, Produto
ORDER BY ano, semestre;
```

**Insight:** Identifica padrões sazonais nos preços, como possíveis picos em períodos específicos do ano.

---

### 4. Variação do Preço do Etanol por Estado
**Pergunta:** Como o preço do etanol varia entre os estados, incluindo sua volatilidade?

**Consulta:**
```sql
SELECT 
    estado, 
    AVG(valorVenda) AS preco_medio_etanol, 
    STDDEV(valorVenda) AS desvio_padrao
FROM silver.combustivel
WHERE Produto = 'ETANOL'
GROUP BY estado
ORDER BY preco_medio_etanol DESC;
```

**Insight:** Destaca estados com preços de etanol mais altos ou voláteis, indicando oportunidades ou riscos de mercado.

# Resultados do Projeto de Análise de Combustíveis

## Dados e Análises Realizadas

**Base de Dados Processada:**  
`silver.combustivel` (formato Delta Lake)  
- Estrutura: 19 colunas (16 originais + campos de carga: `dataColeta`, `data_carga`, `data_hora_carga`)  

**Principais Análises Concluídas:**  
- Evolução temporal de preços por estado e ano  
- Ranking dos 10 municípios com maior concentração de postos  
- Padrões sazonais de preços (análise semestral)  
- Volatilidade de preços do etanol por unidade federativa  

**Saídas Geradas:**  
- Tabela de resultados: `transform.preco_combustivel_ano`  
- Visualizações analíticas: gráficos de séries temporais de preços médios  

## Desafios Técnicos e Soluções

**Processamento de Dados Brutos:**
- Extração e padronização de arquivos ZIP (2022-2024)  
- Tratamento especial para arquivo atípico (`precos-semestrais-ca.zip`) com validação de estrutura  

**Gerenciamento de Arquivos:**
- Restrições de permissão no Databricks Community Edition  
- Solução: Gravação direta de CSVs com nomenclatura padronizada (`ca-AAAA-SS.csv`)  

**Modelagem de Dados:**
- Adequação de tipos de dados (ex.: conversão de `numeroRua` para inteiro)  
- Normalização de nomes de colunas  

**Otimização de Performance:**
- Implementação de técnicas PySpark para processamento distribuído  
- Aplicação de recursos Delta Lake (OPTIMIZE)  

## Conclusões e Insights

O projeto consolidou dados oficiais da ANP, gerando análises estratégicas sobre:  
- Comportamento geotemporal de preços de combustíveis  
- Distribuição espacial da infraestrutura de postos  
- Perfil de volatilidade do mercado de etanol  

**Aplicações Práticas:**  
- Formulação de estratégias regionais de precificação  
- Planejamento de expansão de rede de postos  
- Monitoramento de risco de preços do etanol  

## Próximas Etapas

**Melhorias Operacionais:**  
- Implementação de pipeline automatizado no Databricks  
- Particionamento da tabela Delta por estado/região e período  

**Expansão Analítica:**  
- Análise comparativa por bandeiras de postos  
- Correlação com indicadores macroeconômicos (inflação, commodities)  

**Governança de Dados:**  
- Documentação de metadados da tabela Delta  
- Implementação de controles de qualidade de dados  
