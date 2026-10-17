# Beta_RollingBeta_Embraer
Cálculo do beta das ações da Embraer em relação ao Ibovespa e construção do gráfico do rolling beta, estimado com janela móvel, ao longo de um período determinado.
### baixando bibliotecas que vamos usar para obter o beta dos ativos
```python
import yfinance as yf
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import statsmodels.api as sm
from statsmodels.tsa.stattools import coint, adfuller
```
```python
tickers = ['^BVSP', 'EMBJ3.SA']
inicio = '2024-01-01'
fim = '2026-10-16'

ativos = pd.DataFrame()

# Popular o DataFrame com os dados

for i in tickers:

  df = yf.download(i, start = inicio, end = fim)['Close']
  df.rename(columns={'Close': i}, inplace=True)
  ativos = pd.concat([ativos,df], axis=1)
  ativos.index.name = 'Date'

ativos
```
### Calculando retornos e eliminando NaNs
```python
retorno = ativos.pct_change()
retorno.dropna(inplace=True)
```
```python
fig = go.Figure()

fig.add_trace(go.Scatter(x=retorno['^BVSP'], y=retorno['EMBJ3.SA'],
                         name= 'EMBJ3', mode="markers"))

fig.update_layout(title= "Distribuição de Retornos", template = 'simple_white', paper_bgcolor = '#FFFFFF', margin= dict(l=0, r=0, t=0, b=0), width = 500, height = 600)

fig.show()
```
### calculando regressão linear das duas variáveis
```python
Y = retorno['EMBJ3.SA']
X = retorno['^BVSP']

X = sm.add_constant(X)

modelo = sm.OLS(Y,X)
resultado = modelo.fit()
beta = resultado.params[1]
beta
```
# Rolling Beta
### baixando bibliotecas que vamos usar para obter o beta dos ativos
```python
import yfinance as yf
import pandas as pd
import numpy as np
import plotly.graph_objects as go
import statsmodels.api as sm
from statsmodels.tsa.stattools import coint, adfuller
```
```python
tickers = ['^BVSP', 'EMBJ3.SA']
inicio = '2025-01-01'
fim = '2026-10-16'

ativos = pd.DataFrame()

### Popular o DataFrame com os dados

for i in tickers:

  df = yf.download(i, start = inicio, end = fim)['Close']
  df.rename(columns={'Close': i}, inplace=True)
  ativos = pd.concat([ativos,df], axis=1)
  ativos.index.name = 'Date'

ativos
```
### Calculando retornos e eliminando NaNs
```python
retorno = ativos.pct_change()
retorno.dropna(inplace=True)
```
```python
from statsmodels.regression.rolling import RollingOLS

Y = retorno['EMBJ3.SA']
X = retorno['^BVSP']

X = sm.add_constant(X)
modelo = RollingOLS(Y, X, window=100)
resultado = modelo.fit()
params = resultado.params.copy()
params.index = np.arange(1, params.shape[0]+ 1)
```
```python
beta = pd.DataFrame(pd.to_numeric(params['^BVSP']))
beta['^BVSP'] =  beta['^BVSP'].astype(float)
```
### Construindo Gráfico do Rolling Beta
```python
fig = go.Figure()

fig.add_trace(go.Scatter(x = retorno.index, y = beta['^BVSP'], 
               name = 'EMBJ3'))
fig.update_layout(template = 'simple_white', height = 500, width = 900)

fig.show()
```
