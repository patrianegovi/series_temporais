# Resumo teórico: `01creatingtimeseries.ipynb`

## Ideia central

O notebook transforma **dados transacionais** em uma **série temporal**. Os dados transacionais têm uma linha por evento, aqui cada título adicionado à Netflix. A série temporal tem um valor por período, em intervalos regulares, aqui o total de títulos adicionados por mês.

O processo tem quatro etapas:

1. Limpar os dados e converter as datas.
2. Agregar os eventos por período.
3. Tratar as lacunas na série.
4. Visualizar o resultado.

## Etapas e conceitos

### 1. Preparação dos dados

- **Tipo de data:** a coluna `date_added` vem como texto, como `"September 25, 2021"`, e precisa virar `datetime` para o pandas operar sobre o tempo.
- **Valores nulos em colunas de texto:** são trocados por rótulos como `'No Director'`, o que preserva as linhas.
- **Datas inválidas:** as linhas sem data são removidas, pois não há como situar o evento no tempo.

### 2. Agregação temporal

A frequência `'MS'` (*Month Start*) agrupa os eventos por mês e marca cada período com o dia 1. O resultado é uma contagem mensal de lançamentos.

### 3. Regularidade e lacunas

Muitos modelos de séries temporais exigem intervalos regulares, sem períodos faltando.

Para verificar isso, o notebook gera o calendário completo de meses e o compara com o índice da série. A lacuna encontrada, `2021-07`, foi **criada de propósito** com `drop(...)`. O `resample().size()` já preenche com 0 os meses sem eventos, então nenhuma lacuna surgiria sozinha.

### 4. Métodos de preenchimento (imputação)

| Método | Como funciona | Valor em 2021-07 | Quando faz sentido |
|---|---|---|---|
| Zero | Atribui 0 | 0 | A ausência significa que nada ocorreu |
| Forward fill | Repete o valor anterior | 207 | A série muda pouco de um período para o outro |
| Backward fill | Copia o valor seguinte | 178 | Reconstrução histórica, pois usa dados do futuro |
| Interpolação linear | Traça uma reta entre os vizinhos | 192,5 | A variação entre os pontos é suave |
| Interpolação quadrática | Ajusta uma curva de 2º grau | Não exibido | A série tem curvatura |

O método deve refletir o motivo da falta. Aqui, 0 seria enganoso, porque o dado existia e foi removido.

Em modelos de previsão, `bfill` e interpolação usam informação futura, o que pode causar **vazamento de dados** (*data leakage*).

### 5. Visualização

O gráfico de linha mostra a evolução no tempo. O notebook recorta a série a partir de 2016, pois os anos anteriores têm contagens muito baixas e dificultam a leitura do período mais recente.

## Principais funções

| Função | Para que serve |
|---|---|
| `pd.read_csv()` | Carrega o CSV em um DataFrame |
| `df.head()` | Mostra as primeiras linhas |
| `pd.to_datetime(format=..., errors='coerce')` | Converte texto em data; valores inválidos viram `NaT` |
| `df.isna().sum()` | Conta os nulos por coluna |
| `fillna()` | Substitui nulos por um valor |
| `set_index()` | Define a data como índice |
| `resample('MS').size()` | Agrupa por mês e conta os eventos |
| `drop()` | Remove um item pelo índice |
| `pd.date_range(start, end, freq='MS')` | Gera o calendário completo de meses |
| `Index.difference()` | Encontra os meses ausentes |
| `reindex(all_months, fill_value=0)` | Alinha a série ao calendário e preenche as lacunas com 0 |
| `ffill()` / `bfill()` | Propaga o valor anterior / seguinte |
| `interpolate(method=...)` | Estima os valores faltantes |
| `Series.plot()` | Gera um gráfico rápido |
| `plt.plot()`, `plt.title()`, `plt.xlabel()`, `plt.ylabel()`, `plt.xticks()`, `plt.grid()`, `plt.tight_layout()`, `plt.show()` | Montam e exibem o gráfico personalizado |
| `plot_series()` | Função criada no notebook que reúne esses comandos do matplotlib |

## Pontos de atenção no notebook

- **Saídas enganosas:** as células foram executadas fora de ordem, então o primeiro `isna().sum()` mostra 0 nulos, e o `head()` já mostra `'No Director'`. O CSV original tem campos vazios e datas em texto.
- **Código repetido:** `lancamentos_neighbors` e `lancamentos_linear` usam exatamente o mesmo método, portanto são iguais. Com apenas um mês faltando, a interpolação linear equivale à média dos dois vizinhos.
- **Cuidado ao reexecutar:** rodar de novo a célula do `drop(lancamentos.index[-3])` remove mais um mês.
- **Import sem uso:** o `seaborn` é importado, mas não é utilizado.
- **Comparação incompleta:** a última célula cria as seis séries, mas não as exibe nem as compara, então o valor da interpolação quadrática não aparece.
