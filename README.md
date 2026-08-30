📊 Dashboard de Análise de Vendas — Power BI

Dashboard interativo desenvolvido em Power BI para análise de performance de vendas, com foco em indicadores estratégicos, segmentação de clientes e apoio à tomada de decisão comercial.

🎯 Objetivo

Consolidar dados de vendas em um painel visual, permitindo acompanhar receita, comportamento de clientes e desempenho por categoria, região e marca de produto.

📈 Indicadores apresentados

Página 1 — Visão Geral

Receita total
Ticket médio
Quantidade de clientes
Total de vendas
Faturamento mensal
Segmentação de vendas por gênero
Faturamento por região
Faturamento por marca
Mapeamento geográfico de clientes
Ranking de vendedores

Página 2 — Evolução e Categorias

Evolução temporal das vendas
Segmentação por categoria de produto
🛠️ Tecnologias utilizadas
Power BI — modelagem de dados, relacionamentos e medidas DAX
Excel — tratamento e estruturação inicial dos dados
🧩 Modelagem de dados
## 🧮 Principais medidas DAX

**Ticket Médio**
```DAX
Ticket Médio = DIVIDE(SUM(Vendas[ValorVenda]), COUNTROWS(Vendas))
```
Calcula a receita média por venda, dividindo o valor total pela quantidade de vendas. Usa `DIVIDE` para evitar erros quando não há vendas no período filtrado.

**Vendedor Destaque**
```DAX
Vendedor Top = 
CALCULATE(
    SELECTEDVALUE(Vendas[Vendedor]),
    TOPN(1, VALUES(Vendas[Vendedor]), CALCULATE(SUM(Vendas[ValorVenda])), DESC)
)
```
## 🧮 Outras medidas DAX

**Receita Total**
```DAX
Receita Total = SUM(Vendas[ValorVenda])
```
Soma o valor de todas as vendas no período filtrado.

**Faturamento Mensal**
```DAX
Faturamento Mensal = 
CALCULATE(
    SUM(Vendas[ValorVenda]),
    ALLEXCEPT(Vendas, Vendas[Mes])
)
```
Calcula o total de vendas agrupado por mês, ignorando outros filtros ativos (como vendedor ou região).

**Quantidade de Clientes**
```DAX
Qtd Clientes = DISTINCTCOUNT(Vendas[ClienteID])
```
Conta o número de clientes únicos, evitando duplicidade quando um cliente aparece em várias vendas.

**Total de Vendas**
```DAX
Total de Vendas = COUNTROWS(Vendas)
```
Conta o número de transações realizadas no período filtrado.

**Faturamento por Marca (% do total)**
```DAX
% Faturamento por Marca = 
DIVIDE(
    SUM(Vendas[ValorVenda]),
    CALCULATE(SUM(Vendas[ValorVenda]), ALL(Vendas[Marca]))
)
````
Mostra a participação percentual de cada marca no faturamento total, comparando o valor filtrado com o total geral (sem filtro de marca).

**Crescimento Mês a Mês (MoM)**
```DAX
Crescimento MoM = 
VAR MesAtual = SUM(Vendas[ValorVenda])
VAR MesAnterior = 
    CALCULATE(SUM(Vendas[ValorVenda]), DATEADD(Vendas[Data], -1, MONTH))
RETURN
    DIVIDE(MesAtual - MesAnterior, MesAnterior)
```
Calcula a variação percentual de faturamento em relação ao mês anterior, útil para mostrar tendência de crescimento.
Identifica o vendedor com maior receita no período, comparando o total vendido por cada um e retornando o primeiro colocado.

O projeto utiliza modelagem dimensional, com estruturação de tabelas fato e dimensão para otimizar o desempenho das consultas e a criação das medidas em DAX.
