# Hurst-Index
Indice de fractalidad desarrolado en Python 
from pandas_datareader import data
import math as math
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from sklearn.linear_model import LinearRegression

def expected(k):
  a=0
  l=k
  for i in range(1,l-1):
    a=a+math.sqrt((l-i)/i)
  if k > 340:
    e=(math.gamma((k-1)/2)/(math.sqrt(3.141516)*math.gamma(k/2)))*a
  else:
    e=(1/(math.sqrt(k*3.141516/2)))*a
  return e



def HurstEXP(ts,e,max_lag=20):                                         # Función para el calculo del exponente de Hurst con la correción Anis-Lloid
    time_series=ts.to_list()                                   # Esta probado con la caminata aleatoria, para la cual H \approx 0.5
    lags = range(2, max_lag)
    # Se implementa el algoritmo de acuerdo al algoritmo oficial de Python
    tau = [np.std(np.subtract(time_series[lag:], time_series[:-lag])) for lag in lags]
    # Se hace la correcion Anis-Lloid (reducir el sesgo)
    for lag in range(0,np.size(tau)-1):
      a=tau[lag]
      tau[lag]=0.5+(a-e)
      if tau[lag]<=0:
        tau[lag]=a
    # se calcula la pendiente de la recta, que corresponde al coeficiente de Hurst
    reg = np.polyfit(np.log(lags), np.log(tau), 1)
    return reg[0]                


start_date = '2010-01-04'
end_date = '2021-09-26'
p = data.DataReader('EURUSD=X', 'yahoo', start_date, end_date)['Adj Close']
#q = data.DataReader('GCARSOA1.MX', 'yahoo', start_date, end_date)['Volume']


# Dividir el periodo total de datos

N=p.size
n=50
e=expected(150)
H=[]

# Calculo el indice del pasado hasta hoy

for i in range(0,N-n):
  P1=p[i:n-1+i]
  H.append(HurstEXP(P1,e))
Hm=np.mean(H)
print(Hm)
x = np.linspace(0,np.size(H),np.size(H))
fig, ax=plt.subplots(2)
fig.set_size_inches(12.5, 12.5, forward=True)
ax[0].plot(p)
ax[1].plot(x,H,'-')
ax[1].axhline(y=0.5, color='g', linestyle='-')
ax[1].axhline(y=Hm, color='r', linestyle='-')
plt.show()
