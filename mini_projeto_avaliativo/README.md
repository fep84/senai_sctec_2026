# Festa em Cena: análise de pedidos de Halloween

Mini-Projeto Avaliativo do Módulo 1 (Semana 07) do curso Machine Learning e Visão Computacional, Turma 05, do SENAI SCTec.

## 1. Descrição do projeto

### O problema
A Festa em Cena é uma rede fictícia que vende fantasias e acessórios de Halloween por três canais: **Mercado** (caixas de supermercados parceiros), **Loja física** (lojas próprias) e **Web** (pedidos pelo site). A empresa quer entender como os pedidos variam por canal e por período. Antes disso, porém, os registros têm problemas que precisam ser tratados: categorias vazias, pontuação indevida, canais escritos de formas diferentes, quantidades inválidas, preços ausentes e datas inválidas.

### Objetivo do programa
O notebook `sctec_mini-projeto_avaliativo.ipynb` lê o arquivo `halloween_vendas.csv` com Pandas, identifica e marca os registros com problemas sem inventar valores, padroniza os textos, converte as datas, testa uma regra de negócio sobre pedidos sem entrega e produz contagens de pedidos por canal e por ano-mês.

### O que é uma linha
Cada linha é **um pedido** de **um único produto**. Um pedido pode ter várias unidades desse produto. A base tem 1.500 pedidos distintos (`pedido_id` único), feitos de agosto a novembro de 2024 e de 2025. Os dados são fictícios.

### Quatro colunas
| Coluna | Significado |
|---|---|
| `data_venda` | Data e hora do pedido, no formato `AAAA-MM-DD HH:MM:SS`, no horário local de São Paulo. |
| `canal_venda` | Canal em que o pedido foi feito: Mercado, Loja física ou Web. |
| `quantidade` | Número de unidades do produto no pedido. Deve ser um inteiro maior que zero. |
| `status_pedido` | Situação do pedido: Concluído, Cancelado ou Devolvido. |

### Regras adotadas

**Textos**
- `categoria`: aplico `.strip()` e `.lower()` e depois `re.sub(r"[^a-z\s]", "", texto)` para remover pontuação, mantendo letras e espaços (as categorias desta base não têm acentos). Se o resultado ficar vazio, ou se a categoria já estiver vazia, uso `"sem categoria"`.
- `canal_venda`: removo os espaços, comparo em minúsculas e mapeio para `Mercado`, `Loja física` ou `Web`. Um canal que não for reconhecido fica fora da contagem por canal e é informado como exclusão.
- `status_pedido`: comparo sem diferenciar maiúsculas e minúsculas, mas não altero o valor.

**Valores**
- `quantidade` é válida quando pode ser lida como número inteiro maior que zero.
- `preco_unitario` é válido quando está preenchido, pode ser lido como número e é maior ou igual a zero.
- Registros inválidos são **marcados** nas colunas `problema_quantidade`, `problema_preco_unitario` e `tem_problema_numerico`. Nenhum valor é inventado, substituído ou apagado, porque preencher um preço ou uma quantidade sem informação real distorceria as análises.
- O total de pedidos afetados conta cada pedido uma vez só, mesmo que tenha mais de um problema (uso `|`, o "ou" lógico, em vez de somar as contagens).
- Custos e descontos não foram usados.

**Datas**
- `data_venda` é convertida com `datetime.strptime(texto, "%Y-%m-%d %H:%M:%S")`. O texto original fica preservado em `data_venda_original`.
- Das datas válidas extraio ano, mês, dia e hora e crio a versão brasileira com `strftime("%d/%m/%Y")` (exemplo: `12/10/2025`).
- Datas vazias ou inválidas ficam marcadas (`data_venda_vazia`, `data_venda_invalida`), e o processamento continua. Elas **não** são substituídas pela data de hoje.
- `data_entrega` vazia não é preenchida, e o status do pedido não é alterado.

## 2. Guia de execução

### Arquivos necessários
```
mini_projeto_avaliativo/
├── sctec_mini-projeto_avaliativo.ipynb   # notebook com a solução
├── halloween_vendas.csv                  # base de dados (mesma pasta do notebook)
├── README.md                             # esta documentação
└── report.md                             # relatório de achados e insights
```

### Como executar
1. Instale o Python 3 e as bibliotecas: `pip install pandas notebook`.
2. Clone o repositório: `git clone https://github.com/fep84/senai_sctec_2026.git`.
3. Entre na pasta do projeto: `cd senai_sctec_2026/mini_projeto_avaliativo`.
4. Confirme que `halloween_vendas.csv` está na **mesma pasta** do notebook, pois ele é lido pelo caminho relativo `"halloween_vendas.csv"`.
5. Abra o notebook no Jupyter (`jupyter notebook`), no VS Code ou no Google Colab (no Colab, envie o CSV para a sessão).
6. Execute todas as células em ordem, do início ao fim (*Run All*).

O arquivo CSV original não é alterado. Todas as transformações acontecem no DataFrame, em memória.

## 3. Resultados

| Indicador | Valor |
|---|---|
| Linhas lidas | 1.500 |
| Categorias vazias preenchidas com "sem categoria" | 18 |
| Categorias não vazias alteradas pela padronização | 1.482 |
| Problemas em `quantidade` (valor -1) | 7 |
| Problemas em `preco_unitario` (vazio) | 12 |
| Pedidos afetados por problema numérico (sem sobreposição) | 19 |
| Datas de venda inválidas | 4 |
| Pedidos cancelados | 109 |
| Pedidos sem data de entrega e cancelados | 109 |
| Pedidos sem data de entrega com outro status | 34 (todos "Concluído") |

**Pedidos por canal** (1.500 incluídos, 0 excluídos, todos os status)

| Canal | Pedidos | Cancelados |
|---|---|---|
| Web | 666 (44,4%) | 70 |
| Loja física | 551 (36,7%) | 31 |
| Mercado | 283 (18,9%) | 8 |

**Pedidos por ano-mês** (1.496 incluídos, 4 excluídos por data inválida, todos os status)

| Mês | 2024 | 2025 |
|---|---|---|
| Agosto | 77 | 62 |
| Setembro | 104 | 94 |
| Outubro | 393 | 410 |
| Novembro | 174 | 182 |

### Conclusão sobre entrega sem data
A hipótese "todo pedido sem `data_entrega` foi cancelado" é **falsa** nesta base. Dos 143 pedidos sem data de entrega, 109 estão cancelados, mas 34 aparecem como "Concluído" (por exemplo, os pedidos 1170, 990 e 740). Basta um contraexemplo para derrubar a hipótese. Por outro lado, todos os 109 pedidos cancelados estão sem data de entrega. Os 34 casos concluídos sem data apontam uma falha de registro que deve ser investigada. Não alterei o status nem inventei datas.

### Descoberta 1: canal
- **Evidência:** a Web teve 666 dos 1.500 pedidos registrados (44,4%), à frente da Loja física (551) e do Mercado (283).
- **Interpretação:** nesta base, o site é o canal com mais pedidos. Também é o canal com mais cancelamentos (70 de 666, cerca de 10,5%, contra cerca de 5,6% na Loja física e 2,8% no Mercado).
- **Limitação:** são contagens de pedidos registrados, incluindo cancelados e devolvidos. Não representam receita, unidades vendidas nem vendas concluídas, e a base é fictícia, sem refletir o mercado real.

### Descoberta 2: período
- **Evidência:** outubro concentra a maior parte dos pedidos nos dois anos: 393 de 748 em 2024 (52,5%) e 410 de 748 em 2025 (54,8%). Outubro de 2025 é o ano-mês com mais pedidos (410, ou 27,4% dos 1.496 com data válida).
- **Interpretação:** os pedidos se concentram no mês do Halloween (31/10), e o padrão se repete de um ano para o outro. Agrupar por ano-mês evita misturar 2024 e 2025.
- **Limitação:** a base cobre só agosto a novembro, então não dá para comparar com o resto do ano. As 4 datas inválidas foram excluídas, e as contagens incluem todos os status.

## 4. Reflexão: qualidade dos dados e Machine Learning
Um modelo aprende com os dados que recebe. Categorias escritas de formas diferentes ("Fantasia", " FANTASIA!!! ") seriam tratadas como grupos distintos. Quantidades negativas e preços vazios distorceriam médias e totais, e datas inválidas quebrariam qualquer análise temporal. Por isso a limpeza vem antes da modelagem. Mesmo assim, limpar não basta para evitar **overfitting** (o modelo decora os dados de treino) ou **underfitting** (o modelo é simples demais para captar o padrão). Também importam a representatividade dos dados (esta base cobre só quatro meses por ano), a escolha do modelo e a forma de avaliá-lo com dados que ele não viu. Marcar os problemas em vez de inventar valores mantém a base honesta e deixa a decisão de excluir ou imputar para uma etapa posterior, com justificativa.

## Autor

**Estudante SCTec**
Fernando Eduardo Pollheim  
[GitHub: @fep84](https://github.com/fep84)
