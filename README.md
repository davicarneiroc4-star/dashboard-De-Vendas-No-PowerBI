# 📊 Dashboard de Análise de Vendas — Power BI

Dashboard interativo em Power BI para analisar as vendas de uma loja de autopeças e veículos na Região Metropolitana de Belém (PA): faturamento, ticket médio, desempenho por vendedor, marca e produto, e evolução mês a mês.

> ⚠️ Os dados são **fictícios**, gerados em Excel para fins de estudo.

## 🖼️ Prévia

### Página 1 — Visão Geral
<img width="1405" height="803" alt="Página 1 do dashboard: visão geral das vendas" src="https://github.com/user-attachments/assets/2a079f67-6e71-4f70-b17d-b19711abe09b" />

### Página 2 — Evolução e Categorias
<img width="1403" height="811" alt="Página 2 do dashboard: evolução e categorias" src="https://github.com/user-attachments/assets/aa59a35d-e3da-4cb6-900d-93437ee2415e" />

## 🎯 Objetivo

Consolidar os dados de vendas em um painel visual que responda perguntas do dia a dia comercial: quanto foi vendido, quais marcas, produtos e vendedores se destacam, onde estão os clientes e como o faturamento evolui de um mês para o outro.

## 📈 Indicadores apresentados

**Página 1 — Visão Geral**
- Cartões com total vendido, ticket médio e quantidade de clientes
- Total de vendas por marca
- Total de vendas por vendedor
- Quantidade de compras por produto
- Vendas por gênero do cliente
- Mapa das vendas por cidade
- Filtro de **Data / Mês / Ano**: todos os visuais recalculam para o período escolhido, permitindo ver o faturamento de um mês específico

**Página 2 — Evolução e Categorias**
- Faturamento do mês atual, do mês anterior e a variação percentual entre os dois
- Quantidade de clientes novos no mês
- Evolução do faturamento ao longo do tempo
- Faturamento por categoria de veículo
- Quantidade de vendas por categoria de veículo e vendedor
- Faturamento por forma de pagamento
- Faturamento por dia da semana
- Filtro de data

## 🛠️ Tecnologias utilizadas

- **Power BI** — Power Query, medidas e colunas calculadas em DAX, visuais interativos
- **Excel** — criação e estruturação das bases de dados

## 🧩 Modelo de dados

O modelo tem **duas tabelas**, uma para cada página, importadas de planilhas Excel:

| Tabela | Conteúdo | Principais colunas |
|---|---|---|
| `Vendas` | 400 vendas (página 1) | Data, Cliente, Gênero, Categoria, Produto, Marca, Valor (R$), Cidade, Vendedor |
| `pagina 2` | 1.200 vendas de 2025 (página 2) | data, id_cliente, genero_cliente, vendedor, categoria_veiculo, marca, produto, quantidade, valor_unitario, valor_total, forma_pagamento |

A tabela `pagina 2` tem um código de cliente (`id_cliente`), o que permite identificar clientes novos. A tabela `Vendas` não tem código, então os clientes da página 1 são contados pelo **nome**. Em uma base real, o ideal é contar por um código, porque duas pessoas podem ter o mesmo nome.

## 🧮 Medidas DAX

### Página 1 (tabela `Vendas`)

**Total de Vendas**: soma do valor de todas as vendas no período filtrado.
```DAX
Total de Vendas = SUM(Vendas[Valor (R$)])
```

**Quantidade de Vendas**: número de vendas (cada venda tem um ID único).
```DAX
Quantidade de Vendas = DISTINCTCOUNT(Vendas[ID Venda])
```

**Quantidade De Clientes**: quantos clientes diferentes compraram no período. Um cliente que comprou várias vezes conta uma vez só.
```DAX
Quantidade De Clientes = DISTINCTCOUNT(Vendas[Cliente])
```

**Ticket Médio**: valor médio de cada venda. O `DIVIDE` evita erro de divisão por zero quando o filtro não tem vendas.
```DAX
Ticket Medio = DIVIDE([Total de Vendas], [Quantidade de Vendas])
```

### Página 2 (tabela `pagina 2`)

**Faturamento do mês atual**: guarda a data mais recente do período filtrado e soma as vendas do mês inteiro daquela data.
```DAX
Fat Mes Atual =
VAR UltimaData = MAX('pagina 2'[data])
RETURN
CALCULATE(
    SUM('pagina 2'[valor_total]),
    ALLEXCEPT('pagina 2',
        'pagina 2'[vendedor], 'pagina 2'[categoria_veiculo], 'pagina 2'[forma_pagamento],
        'pagina 2'[marca], 'pagina 2'[produto], 'pagina 2'[genero_cliente],
        'pagina 2'[Cidade], 'pagina 2'[Dia Semana]),
    MONTH('pagina 2'[data]) = MONTH(UltimaData),
    YEAR('pagina 2'[data]) = YEAR(UltimaData)
)
```

**Faturamento do mês anterior**: mesma lógica, voltando um mês com `EDATE`. Usar `EDATE` em vez de subtrair 1 do mês resolve a virada de ano (janeiro volta para dezembro do ano anterior).
```DAX
Fat Mes Anterior =
VAR UltimaData = MAX('pagina 2'[data])
VAR MesAnterior = EDATE(UltimaData, -1)
RETURN
CALCULATE(
    SUM('pagina 2'[valor_total]),
    ALLEXCEPT('pagina 2',
        'pagina 2'[vendedor], 'pagina 2'[categoria_veiculo], 'pagina 2'[forma_pagamento],
        'pagina 2'[marca], 'pagina 2'[produto], 'pagina 2'[genero_cliente],
        'pagina 2'[Cidade], 'pagina 2'[Dia Semana]),
    MONTH('pagina 2'[data]) = MONTH(MesAnterior),
    YEAR('pagina 2'[data]) = YEAR(MesAnterior)
)
```

O `ALLEXCEPT` é essencial aqui. O filtro de Data/Mês/Ano age sobre uma tabela de datas que o Power BI cria automaticamente. Sem o `ALLEXCEPT`, ao escolher julho no filtro, a medida pedia vendas de junho dentro de julho e o resultado ficava em branco. Ele remove o filtro de data e mantém os demais (vendedor, categoria etc.), então clicar em um gráfico continua filtrando os cartões.

**Variação %**: crescimento do faturamento em relação ao mês anterior.
```DAX
Variacao % = DIVIDE([Fat Mes Atual] - [Fat Mes Anterior], [Fat Mes Anterior])
```

**Clientes Novos**: clientes cuja **primeira compra da história** aconteceu no mês analisado. O `ALLEXCEPT` faz a primeira compra ser procurada em todo o histórico do cliente, e não só no período do filtro.
```DAX
Clientes Novos =
VAR UltimaData = MAX('pagina 2'[data])
VAR PrimeiraCompraPorCliente =
    ADDCOLUMNS(
        VALUES('pagina 2'[id_cliente]),
        "PrimeiraCompra",
            CALCULATE(MIN('pagina 2'[data]), ALLEXCEPT('pagina 2', 'pagina 2'[id_cliente]))
    )
RETURN
COUNTROWS(
    FILTER(
        PrimeiraCompraPorCliente,
        MONTH([PrimeiraCompra]) = MONTH(UltimaData) &&
        YEAR([PrimeiraCompra]) = YEAR(UltimaData)
    )
)
```

**Ticket Médio por Vendedor**: valor médio por venda, para comparar vendedores.
```DAX
Ticket Medio Vendedor = DIVIDE(SUM('pagina 2'[valor_total]), DISTINCTCOUNT('pagina 2'[id_venda]))
```

### Coluna calculada

**Dia Semana**: nome do dia da semana de cada venda, usado no gráfico de faturamento por dia.
```DAX
Dia Semana = FORMAT('pagina 2'[data], "dddd")
```

## 💡 O que aprendi

- A diferença entre **contar vendas** e **contar clientes**: a primeira versão da medida de clientes contava IDs de venda e mostrava o número de vendas. Contar valores distintos da coluna de cliente resolveu
- Como o contexto de filtro afeta uma medida:
  - a primeira versão de "Clientes Novos" procurava a primeira compra só dentro do mês filtrado, e todo cliente parecia novo;
  - o "Faturamento do mês anterior" ficava em branco quando um mês era escolhido no filtro.
  
  Nos dois casos, o `ALLEXCEPT` resolveu, removendo o filtro de data e mantendo os demais
- Usar `VAR` para deixar medidas longas mais legíveis e `DIVIDE` para evitar divisão por zero

## 🚀 Próximos passos

- Montar um **modelo estrela**, com uma tabela fato de vendas e tabelas dimensão (calendário, cliente, produto, vendedor)
- Criar uma **tabela calendário** e usar funções de inteligência de tempo (`DATEADD`, `SAMEPERIODLASTYEAR`)
- Unificar as duas bases em uma só, para que as duas páginas conversem entre si

## 📂 Arquivos do projeto

- `dashboard_de_vendas.pbix` — arquivo do Power BI (abra no Power BI Desktop)

## 👤 Autor

**Davi Carneiro** — Estudante de Análise e Desenvolvimento de Sistemas (UNIP), em formação para Engenharia de Dados.

[GitHub](https://github.com/davicarneiroc4-star)
