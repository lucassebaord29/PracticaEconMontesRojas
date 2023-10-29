# Práctica I - Series de tiempo 

Utilizaremos el índice de Precios de USA (Consumer Price Index for All Urban Consumers ), vale destacar que trabajaremos con la inflación mensual



Para estimar y hacer infereencia sobre la serie necestiamos quee la serie de tiempo sea estacionaria. Ante un análisis grafico se puede observar con intuición acerca de la estacionariedad en sentido débil


Las condiciones estadíssticas que buscamos es que $E[Y_t]$ y $VAR(y_t)$ sean constantes a lo largo del tiempo

Empezamos linkeando la base

```
clear all

import excel "/Users/lucasordonez/Desktop/clase econ/cpi_usa.xlsx", sheet("Hoja1") cellrange(A1:F242) firstrow
set more off

*Seteo la serie en meses*
tsset Fecha,monthly
```

# Análisis de estacionariedad


Es recomendable trabajar con series logarítmicas, nos permite reducir la heterocedasticidad y suavizar la serie


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


Realizando la diferencia logarítmica del IPC aplicando el operador diferencia $\Delta y_t=y_t - y_{t-1}$. El comando es el siguiente en STATA:

```
d.logIPC
```

Al aplicar diferencias sobre la serie de IPC


$$ \Delta ln(IPC) = ln(IPC_t) - ln(IPC_{t-1})$$


Generamos una variable que representa la aproximación lineal de la **inflación mensual**. 

```
gen dlogIPC = d.logIPC
label variable dlogIPC "Inflación mensual"
```

Graficamos con el objetivo de intuir si existe tendencia (crece a lo largo de $t$) o tiene picos (comportamiento estacional)

```
tsline dlogIPC
```

<img width="1325" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/42b108a0-0bd0-41fe-bda0-e67aa964dc25">


## Análisis estadístico de la serie

### 1) Aplicamos filtro de Hodrick - Prescott, con el objetivo de conocer la descomposición de la serie, la tendencia y el ciclo


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

### 2) Aproximación de observación de tendencia y estacionalidad

Buscamos un polinomio de grado $n$, que se asemeje mas a nuestro modelo

```
* Primera aproximación de observación de tendencia y estacionalidad

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



```
<img width="1355" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/844f4bd6-6621-4444-a38a-3691d7a48265">

\
**Podemos conjeturar que un polinomio de grado 2 ajusta correctamente**

\
Nuestra salida en STATA es la siguiente:


<img width="640" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/b3525bf0-45ca-48dc-b4ab-752f1bea7565">

\
Realizamos los valores predichos para obtener **tendencia** y por otro lado a través de los residuales de la regresión obtenemos la serie sin tendencia.

```
reg dlogIPC Tiempo Tiempo2 
predict trend
predict CLEAN_INF, resid
label variable CLEAN_INF "Inflación sin tendencia"

```

Para el análisis de estacionalidad creamos $Q-1$ Dummy con los meses respectivos para descomponener el efecto mensual. El análisis se realiza sobre la serie sin tendencia

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

\



<img width="1341" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/48d90874-572e-4828-8a4b-0a900312b25c">


\
Finalmente obtenemos nuestra serie estacionaria

```
tsline CLEAN_INF2 dlogIPC
```


<img width="1331" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/dfd475dc-3755-457a-ae41-4a13be29c251">


## 3) Contraste de Dickey Fuller
Anteriormente observamos que la serie con la que estamos trabajando presenta un **componente tendencial** por ende la serie **no puede ser estacionaria**

Aplicamos este test con el objetivo de buscar **Raices unitarias**


### 1° Test simple de Dickey-Fuller
Analizamos si la serie presenta estacionariedad a través del test de Dickey - Fuller, el objetivo de este test es testear si se presenta raíz unitaria 

Ejemplo AR(1):

$$ Y_T = \rho Y_{T-1} + e_t$$


$$\Delta  Y_T = \theta Y_{T-1} + e_t$$

Donde $\theta = \rho - 1$

Si $H_0: \theta = 0 \Rightarrow$ se presenta raíz unitaria

Si $H_0: \theta = 0 <0 \Rightarrow$ no se presenta raíz unitaria


```
dfuller logIPC

```
\
Nuestra salida en STATA es la siguiente:

<img width="661" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/c0711394-6d32-422e-81ef-628214635292">


\
Se presenta Raíz unitaria, ya que no rechazamos la hipótesis nula. Aplicando el operador diferencia nos permite trabajar con un orden de integracion 0 I(0)

```
dfuller dlogIPC
```

\
Nuestra salida en STATA es la siguiente:


<img width="637" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/392a8551-3ac3-4995-97c6-947086fcaa3d">


\
Rechazamos $H_0$, no se presenta raíz unitaria



### 2° Test de Dickey-Fuller ampliado

Con este comando vemos la cantidad de rezagos óptimos en nuestro modelo, optamos por un rezago por criterio Bayesiano

```
1) busco cantidad de lags */
varsoc dlogIPC if Fecha<=tm(2023m9),maxlag (36)
```


\
Nuestra salida en STATA es la siguiente:



<img width="626" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/affca1b4-ccdd-4820-9fdd-78f61aa9967c">



\
Luego testeo a traves de estos rezagos en el test:


```
/* 2) Testeo */

dfuller dlogIPC if Fecha<=tm(2023m9),lag(1)

```

\
Nuestra salida en STATA es la siguiente:



<img width="640" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/50dfe3c9-3745-4c74-beee-ffb915e5fc97">



\
También a través del Test puedo observar la significancia de la tendencia  




```
*Si quiero identificar si la serie tiene una tendencia deterministica (si por el mero paso del tiempo se modifica la Y_t)

dfuller dlogIPC if Fecha<=tm(2023m9),lag(1) trend reg
```


Nuestra salida en STATA:

<img width="649" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/ee13924f-d4c8-4aa0-a105-7ad19752765b">




\
En este caso no es significativo


# Correlograma de la serie

---------
**arma**

La serie de tiempo depende de una **constante + algunos rezagos donde aparece el rezago de la variable independiente + otros rezagos a traves de los shocks**


Para los MA -> función de autocorrelación. 

comando

```
ac variable

```
Calcula los coeficientes que se corresponden con la autocorrelación, los que estan por fuera del intervalo de confianza son estadisticamente significativos

**OBSERVACIÓN: LA AUTOCORRELACION SIRVE COMO FORMA DE ELEGIR LA CANTIDAD DE REZAGOS DE LOS PROMEDIOS MOVILES**
```
corrgram variable
```
calcula todas las autocorrelaciones y las expone en una tabla

**FUNCION DE AUTOCORRELACION**: nos dice las correlaciones que hay entre cada periodo y sus rezagos, entonces nos va a servir para ver si hay estacionalidad

La funcion de autocorrelacion no nos sirve para elegir la cantidad de rezagos de la parte AR, en general, lo que se ve es que la autocorrelación va cayendo lentamente. 

**FUNCION DE AUTOCORRELACION PARCIAL** -> orden AR.
Para estimar el valor de la autocorrelacion parcial de orden 2 estoy controlando de forma parcial por el efecto de lo que paso en $t-1$.

comando

```
pac variable
```

--------
Una aclaración importante es que debemos trabajar sobre nuestra serie estacionaria (Sin tendencia ni estacionalidad)

Realizando un **analisis subjetivo** en base a los **gráficos** de la **función de autorcorrelación (AC)** y la **funcion de autocorrelación parcial (PAC)**, nos acercamos a la cantidad de rezagos de la parte MA y la parte AR del modelo.


Una primera aproximación es utilizar el siguiente comando:


```
corrgram CLEAN_INF2

```

La salida en STATA es la siguiente:

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/374e339d-c66a-4e64-81ed-8070c21bb19f)


\
Podemos inferir de forma mas precisa graficando la **función de autorcorrelación (AC)** y la **funcion de autocorrelación parcial (PAC)** con los siguientes comandos:

```
ac CLEAN_INF2, name(auto)
pac CLEAN_INF2, name(partauto)
graph combine auto partauto 
```


Los gráficos son los siguientes:


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/087507ce-6be6-4218-96d2-e06cd767d28b)


¿Que pueden inferir graficamente?


Para realizar una análisis mas robusto utilizamos **los critorios de información**, probamos distintos modelos ARMA. A la hora de elegir nos quedamos con el modelo que reporte el **minimo** valor

### **ARIMA (1,0,0)**

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/a853b0b1-1f02-404f-a806-edc2336e3f35)



### **ARIMA (0,0,1)**

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/e58e787e-7b59-4e1c-8530-736d5459b9ed)


### **ARIMA (1,0,1)**


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/a8855220-ca56-4bfe-ae05-811e4c0b0164)


### **ARIMA (2,0,0)**


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/5f3fde9b-41fd-4104-a902-862dc575791b)


### **ARIMA (2,0,1)**


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/be4f3057-e876-4e19-ac11-d1bb7dfba6b3)



### **ARIMA (3,0,0)**


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/0ff1d045-81af-4240-96c9-2f260ab1cbc8)


### MA(1)
```
-----------------------------------------------------------------------------
       Model |        Obs  ll(null)  ll(model)      df         AIC        BIC
-------------+---------------------------------------------------------------
           . |        237         .   1060.197       3   -2114.394   -2103.99
-----------------------------------------------------------------------------
```

### ARMA(1,1)

```
-----------------------------------------------------------------------------
       Model |        Obs  ll(null)  ll(model)      df         AIC        BIC
-------------+---------------------------------------------------------------
           . |        237         .   1061.282       4   -2114.564  -2100.692
-----------------------------------------------------------------------------

```
### AR(2)

```
-----------------------------------------------------------------------------
       Model |        Obs  ll(null)  ll(model)      df         AIC        BIC
-------------+---------------------------------------------------------------
           . |        237         .   1061.153       4   -2114.306  -2100.433
-----------------------------------------------------------------------------
```

En base a los criterios de información el ARMA(1,1) y MA(1) son los mejores modelos a utilizar. A fines prácticos trabajaremos con los 3 modelos seleccionados para ver diferencias y similitudes.


Como siguiente paso verificamos si existe correlación con los ruidos blancos de nuestros modelos. Corremos en STATA el correlograma de los errores de predicción del modelo AR (2) para los datos de nuestra base de inflación para tener más información sobre la elección del modelo además de los criterios de información


```
arima CLEAN_INF2, arima (2,0,0)
predict er, resid
corrgram er
drop er

```

La salida de STATA es la siguiente:

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/f93bd0d4-4f7e-4568-ba98-903d9858ae59)


\
Corremos en STATA el correlograma de los errores de predicción del modelo ARMA (1,1) para los datos de nuestra base de inflación para tener más información sobre la elección del modelo además de los criterios de información

```
arima CLEAN_INF2, arima (1,0,1)
predict er2, resid
corrgram er2
drop er2
```

La salida de STATA es la siguiente:

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/e3ad44ae-968f-4231-8889-21420c9c133d)


Corremos en STATA el correlograma de los errores de predicción del modelo MA (1) para los datos de nuestra base de inflación para tener más información sobre la elección del modelo además de los criterios de información.


```
arima CLEAN_INF2, arima (0,0,1)
predict er3, resid
corrgram er3
drop er3
```

La salida de STATA es la siguiente:


![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/181bceec-f701-428b-912f-c0b2fb1ccd3b)


# Predicción

Presentando los datos de la inflación de 2023 hasta el mes de septiembre, evaluaremos qué tan bien predice nuestro modelo constrastando la inflación realizada. Existen 3 instancias para entrenar nuestro modelo:


1) In sample forecast o parte training: entrenamos al modelo dentro de la muestra con valores conocidos (estimación de regresión arima)

2) Ex post out of sample forecast o parte testing: pronóstico mas allá de la muestra de la regresión testeando contra valores conocidos (ya realizados ex-post)

3) Ex ante out of sample forecast: pronóstico mas allá de la muestra de la regresión y de los valores conocidos (estimo el futuro)


### Modelo ARMA(1,1)

1) In sample forecast o parte training

```
arima CLEAN_INF2 if Fecha < tm(2023m1), arima (1,0,1)
```

2) Ex post out of sample forecast o parte testing:


```
predict inf_pred1, dynamic (tm(2023m1)) 
gen inhat20=inf_pred1+trend + estacio


```

**Debemos sumar la tendencia y la estacionalidad**

Graficamos

```
tsline inhat20 dlogIPC if Fecha>tm(2022m12) & Fecha<=tm(2023m9)

tsline inhat20 dlogIPC  if Fecha>tm(2020m12) & Fecha<=tm(2023m9)
```

El primer gráfico muestra nuestra predicción a lo largo del año 2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/d1c909b9-4c5a-42ce-8fbf-5e62bfd035c0)

El segundo gráfico muestra nuestro modelo junto con la predicción del año 2023 para el período 2021-2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/c065c50e-f6ae-4eb4-b4f0-a0333bd88e69)



### Modelo AR (2)

Siguiendo los mismos pasos que utilizamos en el modelo anterior obtenemos los siguientes resultados:


Graficamos

```
tsline inhat20_2 dlogIPC if Fecha>tm(2022m12) & Fecha<=tm(2023m9)

tsline inhat20_2 dlogIPC  if Fecha>tm(2020m12) & Fecha<=tm(2023m9)
```

El primer gráfico muestra nuestra predicción a lo largo del año 2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/2d69141c-99ba-4043-bdba-cfc3ec66b437)


El segundo gráfico muestra nuestro modelo junto con la predicción del año 2023 para el periodo 2021-2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/45779042-54e0-41f5-96cc-5d64627e0332)


### Modelo MA(1)

Siguiendo los mismos pasos que utilizamos en el modelo anterior obtenemos los siguientes resultados:


Graficamos

```
tsline inhat20_2 dlogIPC if Fecha>tm(2022m12) & Fecha<=tm(2023m9)

tsline inhat20_2 dlogIPC  if Fecha>tm(2020m12) & Fecha<=tm(2023m9)
```

El primer gráfico muestra nuestra predicción a lo largo del año 2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/bd997b5b-0e13-4cb4-bf3d-c06f15d2786b)


El segundo gráfico muestra nuestro modelo junto con la predicción del año 2023 para el periodo 2021-2023

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/3d00234f-ef12-4036-8ea1-1e2814593a1e)



### Comparación de pronósticos

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/a024a784-6414-46ad-9763-b438140a44ed)


Una aplicación util es comparar los errores de nuestro modelo:

```
gen error_pron= dlogIPC - inhat20 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
gen error_pron_cuad=error_pron^2 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
rename error_pron ehat_arma11


gen error_pron2= dlogIPC - inhat20_2 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
gen error_pron_cuad2=error_pron2^2 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
rename error_pron2 ehat_ar2


gen error_pron3= dlogIPC - inhat20_3 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
gen error_pron_cuad3=error_pron3^2 if Fecha>tm(2022m12) & Fecha<=tm(2023m9)
rename error_pron3 ehat_ma1

sum ehat_ar1 ehat_arma11

```

![image](https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/03a37b75-bca9-4d5e-9c3f-006b547f6030)



### Pronóstico Ex ante out of sample forecast

### ARMA (1,1)

```
arima CLEAN_INF2 if Fecha < tm(2023m9), arima (1,0,1)
predict inf_pred2_expost, dynamic(tm(2023m10))
gen inhat21= inf_pred2_expost+trend + estacio
label variable inhat21 "Pronóstico ARMA (1,1)"

tsline inhat21 dlogIPC if Fecha>(tm(2022m12)) &  Fecha <= (tm(2023m12)) 

```

<img width="1357" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/ba429bb6-d3c2-4b41-9ff4-2b8d6855436e">


### MA (1)

```
arima CLEAN_INF2 if Fecha < tm(2023m9), arima (0,0,1)
predict inf_pred2_expost2, dynamic(tm(2023m10))
gen inhat21_2= inf_pred2_expost2 +trend + estacio
label variable inhat20_2 "Pronóstico MA (1)"


tsline inhat21_2 dlogIPC if Fecha>(tm(2023m1)) &   Fecha <= (tm(2023m12)) 

```

<img width="1294" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/f15269cc-bab1-4ec3-88de-f6d2d0984aeb">



### Comparación: ARMA (1,1) vs MA (1)

<img width="1255" alt="image" src="https://github.com/lucassebaord29/PracticaEconMontesRojas/assets/67765423/adee24d7-7ac6-4e3c-ab79-a2826a04a794">


