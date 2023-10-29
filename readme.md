# Práctica I - Series de tiempo 

Utilizaremos el Indice de Precios de USA (Consumer Price Index for All Urban Consumers ),vale destacar que trabajaremos con la inflación mensual.

Para estimar y hacer infereencia sobre la serie necestiamos quee la serie de tiempo sea estacionaria. Ante un analisis grafico se puede observar una intuicion acerca de la estacionariedad en sentido debil.
Las condiciones estadisticas que buscamos es que $E[Y_t]$ y $VAR(y_t)$ sean constantes a lo largo del tiempo.

Empezamos linkeando la base

```
clear all

import excel "/Users/lucasordonez/Desktop/clase econ/cpi_usa.xlsx", sheet("Hoja1") cellrange(A1:F242) firstrow
set more off

*Seteo la serie en meses*
tsset Fecha,monthly
```

# Análisis de estacionariedad

Es recomendable trabajar con series logaritmicas, nos permite reducir la heterocedasticidad y suavizar la serie


```
gen logIPC=ln(IPC)
```
Graficando nuestra serie en niveles y la transformación logarítmica

```
*Gráficos*

tsline IPC, name(IPC)

tsline logIPC, name(logIPC)

graph combine IPC logIPC , col(1) iscale(1)
```

<img width="1353" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/bcf72913-a5b2-4ce7-8cb5-6154d58587e5">


Realizando la diferencia logaritmica del IPC aplicando el operador diferencia $\Delta y_t=y_t - y_{t-1}$. El comando es el siguiente en STATA:

```
d.logIPC
```
Al aplicar diferencias sobre la serie de IPC

$$ \Delta ln(IPC) = ln(IPC_t) - ln(IPC_{t-1})$$


Generamos una variable que representa la aproximación lineal de la *inflación mensual*. 

```
gen dlogIPC = d.logIPC
label variable dlogIPC "Inflación mensual"
```

Graficamos con el objetivo de intuir si existe tendencia (crece a lo largo de $t$) o tiene picos (comportamiento estacional).

```
tsline dlogIPC
```

<img width="1325" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/42b108a0-0bd0-41fe-bda0-e67aa964dc25">


## Análisis estadístico de la serie

### 1) Aplicamos filtro de Hodrick - Prescott, con el objetivo de conocer la descomposicion de la serie, la tendencia y el ciclo


```

**TENDENCIA HP


hprescott dlogIPC, stub(hplipc)

*Genera las siguientes variables, por un lado ciclo y por otro tendencia

* hplipc_dlogIPC_1 // ciclo
* tsline hplipc_dlogIPC_sm_1  // tendencia

rename hplipc_dlogIPC_1 ciclo_hp
rename hplipc_dlogIPC_sm_1 tendencia_hp

* Grafico HP
tsline ciclo_hp tendencia_hp

```
Gráficamente:

<img width="1354" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/0d9a2b51-fe2b-4038-9ad2-4f37bc5f37de">

### 2) Aproximacion de observacion de tendencia y estacionalidad

Buscamos un polinomio de grado $n$, que se asemeje mas a nuestro modelo

```
* Primera aproximacion de observacion de tendencia y estacionalidad

*** Tendencia

gen Tiempo2 = Tiempo ^2
gen Tiempo3 = Tiempo^3
gen Tiempo4 = Tiempo^4
******observacion de tendencia  y desestacionalización
preserve
reg dlogIPC Tiempo
predict Dlogipc_hat

reg dlogIPC Tiempo Tiempo2
predict Dlogipc_hat2


reg dlogIPC Tiempo Tiempo2 Tiempo3
predict Dlogipc_hat3

reg dlogIPC Tiempo Tiempo2 Tiempo3 Tiempo4
predict Dlogipc_hat4

*********************************************************************************

tsline  Dlogipc_hat tendencia_hp, name(tendencia1)
tsline  Dlogipc_hat2 tendencia_hp, name(tendencia2)
tsline  Dlogipc_hat3 tendencia_hp, name(tendencia3)
graph combine tendencia1 tendencia2 tendencia3
tsline ciclo_hp tendencia_hp 
*********************************************************************************
*observar la tendencia es simil a traves del filtro de hp y por otro lado con el ajuste polinomico

<img width="1355" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/844f4bd6-6621-4444-a38a-3691d7a48265">


```

*Podemos conjeturar que un polinomio de grado 2 ajusta correctamente*


<img width="640" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/b3525bf0-45ca-48dc-b4ab-752f1bea7565">

Realizamos los valores predichos para obtener **tendencia** y por otro lado a traves de los residuales de la regresion obtenemos la serie sin tendencia.

```
reg dlogIPC Tiempo Tiempo2 
predict trend
predict CLEAN_INF, resid
label variable CLEAN_INF "Inflación sin tendencia"

```

Para el analisis de estacionalidad creamos $Q-1$ Dummy con los meses respectivos para descomponener el efecto mensual. El analisis se realiza sobre la serie sin tendencia

```
** Ahora tiramos una regresión de la serie limpia sin tendencia vs i.Mes para chequear estacionalidad
reg CLEAN_INF b(1)i.Mes    
```

<img width="660" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/6c5bd192-60b1-4e10-978d-7825d7b90ef5">


Se puede **observar estacionalidad en un par de meses**

```
reg CLEAN_INF b(1)i.Mes    // Se puede ver estacionalidad en un par de meses
predict estacio
predict CLEAN_INF2, resid
label variable CLEAN_INF2 "Inflación sin tendencia y estacionalidad"

```

<img width="1341" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/48d90874-572e-4828-8a4b-0a900312b25c">


Finalmente obtenemos nuestra serie estacionaria

```
tsline CLEAN_INF2 dlogIPC
```


<img width="1331" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/dfd475dc-3755-457a-ae41-4a13be29c251">


### 3) Contraste de Dickey Fuller
Anteriormente observamos que la serie con la que estamos trabajando presenta un **componente tendencial** por ende la serie **no puede ser estacionaria**.

Aplicamos este test con el objetivo de buscar **Raices unitarias**


#### 1° Test simple de Dickey-Fuller
Analizamos si la serie presenta estacionariedad a traves del test de Dickey - Fuller, el objetivo de este test es testear si se presenta raíz unitaria. 

Ejemplo AR(1):

$$ Y_T = \rho Y_{T-1} + e_t$$


$$\Delta  Y_T = \theta Y_{T-1} + e_t$$

Donde $\theta = \rho - 1$

Si $H_0: \theta = 0 \Rightarrow$ se presenta raíz unitaria

Si $H_0: \theta = 0 <0 \Rightarrow$ no se presenta raíz unitaria

```
dfuller logIPC

```

<img width="661" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/c0711394-6d32-422e-81ef-628214635292">

Se presenta Raiz unitaria, ya que no rechazamos la hipotesis nula. Aplicando el operador diferencia nos permite trabajar con un orden de integracion 0 I(0).

```
dfuller dlogIPC
```

<img width="637" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/392a8551-3ac3-4995-97c6-947086fcaa3d">



Rechazamos $H_0$, no se presenta raiz unitaria

#### 2° Test de Dickey-Fuller ampliado

Con este comando vemos la cantidad de rezagos optimos en nuestro modelo, optamos por un rezago por criterio Bayesiano

```
1) busco cantidad de lags */
varsoc dlogIPC if Fecha<=tm(2023m9),maxlag (36)
```

<img width="626" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/affca1b4-ccdd-4820-9fdd-78f61aa9967c">


Luego Testeo a traves de estos rezagos en el test:

```
/* 2) Testeo */

dfuller dlogIPC if Fecha<=tm(2023m9),lag(1)

```

<img width="640" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/50dfe3c9-3745-4c74-beee-ffb915e5fc97">

Tambien a traves del Test puedo observar la significancia de la tendencia

```
*Si quiero identificar si la serie tiene una tendencia deterministica (si por el mero paso del tiempo se modifica la Y_t)

dfuller dlogIPC if Fecha<=tm(2023m9),lag(1) trend reg  // P value = 0,000 Entonces parece que no hay tendencia deterministica
```
<img width="649" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/ee13924f-d4c8-4aa0-a105-7ad19752765b">


En este caso no es significativo




