# Modelando um Dashboard de E-commerce com Power BI Utilizando Fórmulas DAX

## Descrição do Projeto
Este projeto tem como objetivo aplicar conceitos de modelagem dimensional (Star Schema) e utilização de fórmulas DAX no Power BI, transformando uma base de dados única — *Financial Sample* — em um modelo relacional otimizado para análise.

A partir dessa modelagem, foi possível criar uma estrutura de tabelas dimensão e fato, permitindo explorar métricas de vendas e rentabilidade de forma flexível e analítica, utilizando linguagem M e DAX.

**OBS: este projeto é resposta de um desafio proposto pelo Bootcamp de Analise de dados, parceria entre a plataforma Dio e Klabin.**

O repositório com o desafio original pode ser encontrado em: [Desafio Power BI Curso - Juliana Mascarenhas](https://github.com/julianazanelatto/power_bi_analyst/tree/main/M%C3%B3dulo%204/Desafios%20de%20Projeto)


---

## Estrutura do Modelo (Star Schema)

O modelo foi desenvolvido a partir da tabela original `financials_origem`, que serviu como base para criação das seguintes tabelas:

### Tabela Fato
**f_vendas**  
Contém as métricas principais de vendas, como quantidade vendida, preço de venda, descontos, lucro e data da transação.  

**Campos principais:**  
`Date`, `Discounts`, `Gross Sales`, `id_produto`, `Profit`, `Country`, `Segment`, `Product`

### Tabelas Dimensão
**d_produto**  
Informações sobre os produtos e métricas agregadas, como médias, medianas e valores máximos de venda.

**d_detalhes_prod**  
Detalhes complementares dos produtos, incluindo faixa de desconto, preço de fabricação e preço de venda.

**d_desconto**  
Estrutura com informações sobre faixas de desconto e respectivas bandas de desconto.

**d_categoria**  
Agrupa os produtos por segmento e país, auxiliando na análise comercial por região e mercado.

**d_calendario**  
Criada via DAX, responsável por fornecer a base temporal do modelo, permitindo análises por ano, mês, trimestre e semana.

---

## Etapas de Construção

1. **Cópia e preparação da base original:**  
   - Criação da tabela `financials_origem` em modo oculto como backup.  
   - Utilização de linguagem M no Power Query para:
     - Agrupar colunas
     - Criar colunas de índice
     - Filtrar e organizar campos por contexto de negócio

2. **Criação das tabelas dimensão e fato:**  
   - Seleção das colunas relevantes de acordo com a granularidade desejada.  
   - Relacionamento das tabelas através de chaves primárias e estrangeiras (`id_produto`, `Date`).

3. **Modelagem em Star Schema:**  
   - Estabelecimento das relações entre as dimensões e a tabela fato.  
   - Ajuste dos relacionamentos para garantir a integridade e a direção do filtro no modelo.

4. **Criação da tabela de calendário com DAX:**  
   Utilizando funções DAX para gerar colunas de tempo adicionais, possibilitando análises temporais avançadas:  

   ```DAX
   ADDCOLUMNS(
       CALENDARAUTO(),
           "Ano", YEAR([Date]),
           "Mês Num", MONTH([Date]),
           "Trim", FORMAT([Date], "q"),
           "Semana Ano", WEEKNUM([Date]),
           "Dia Semana", WEEKDAY([Date]),
           "Mês", FORMAT([Date], "mmm"),
           "Mês Completo", FORMAT([Date], "mmmm"),
           "Mês_Ano", FORMAT(MONTH([Date]), "00") & "/" & YEAR([Date]),
           "Mês-Ano", UPPER(FORMAT(MONTH([Date]), "mmm")) & "/" & YEAR([Date]),
           "Trim_Ano", FORMAT([Date], "q") & "T" & YEAR([Date]),
           "Dia_Semana", FORMAT([Date], "ddd")
   )


**Funções DAX utilizadas**

Tipo                 | Função                                  | Descrição
-------------------- | --------------------------------------- | -----------------------------------------------
Calendário           | CALENDARAUTO()                          | Criação automática da tabela de datas
Relacionamento       | RELATED()                               | Recupera informações de tabelas relacionadas
Tempo                | YEAR(), MONTH(), WEEKNUM(), WEEKDAY()   | Geração de colunas derivadas de data
Formatação           | FORMAT()                                | Formata os valores de data em diferentes níveis de agregação (mês, trimestre, ano)
Agregação (via M)    | GROUP BY, INDEX COLUMN                  | Criação de colunas de índice e agrupamento no Power Query


**Relacionamentos principais**

f_vendas[id_produto]        →  d_produto[índice] 

f_vendas[Date]              →  d_calendario[Date]

f_vendas[id_produto]     →  d_desconto[id_produto]

f_vendas[Segmentopais]    →  d_categoria[Segmento - País] 

f_vendas[id_produto]    →  d_detalhes_prod[id_produto]

