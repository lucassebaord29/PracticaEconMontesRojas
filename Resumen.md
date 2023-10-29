# Tp2-series-time
***
****Series de tiempo******
***
El objetivo es tratar de desmenuzar la serie temporal en cosas inexplicables, y estas se 
llaman ruidos blancos.
La serie de tiempo lineales son promedios ponderados de cosas que fueron ocurriendo
en el pasado. 

***
**Procesos autorregresivos AR(1)**
***

Lo que observamos hoy es en parte explicado en forma deterministica por los parametros y por
el pasado en un rezago + una parte aleatoria que una innovacion o perturbación $e_t$


**Funcion de autocovarianzas**: nos interesan las relaciones entre el pasado y el presente.
La covarianza entre lo que pasa en cualquier periodo t y otro periodo con un rezago k

la $cov(y_{t-1},e_t)=0$ porque el shock no ha ocurrido aún. 

**correlacion**:  $corr(x,y)= cov (x,y)$ /la raiz cuadrada de la var(x)var(y)*/

**Ejemplo con IPC con AR(1)**

```
tsset date  // indexo por la variable tiempo
reg inf L.inf // la inflacion si es un autorregresivo, de la inflacion del periodo anterior se traslada el x% (que es el dato de L1) al futuro
```
**Promedios Móviles**: hacen un promedio que se va moviendo a lo largo del tiempo.
Las series de tiempo van avanzando a partir de un shock de ese periodo y los shocks del periodo pasado.
El parametro a estimar es un parametro autorregresivo que tiene que ver con la estructura de los errores.
Los errores son ruidos blancos, se permita que haya hasta q rezagos hacia atras donde los shocks
forman parte de la serie de tiempo.
Los shocks tienen distintas ponderaciones y estas ponderaciones son los que vamos a estimar. 


En los **promedios autorregresivos** las cov y las correlaciones dependian de la cantidad de rezagos, pero
si nos ibamos cada vez más al pasado los efectos seguian perdurando. 
Ahora las cov se calculan viendo cuando coinciden el periodo de los shocks. 

$cov(y_t;y_{t-1})$ escribo lo que es cada uno y ver adonde coinciden.
Pero ojo cuando me voy a un valor más alto de autocovarianza todos estos tiende cero. 

**Calcular la $cov(y_t;y_{t-1})$ para un promedio movil de orden 1.**

Los autorregresivos tienen una persistencia del pasado más patente, el pasado aparece en forma explicita con todos sus componente como un factor explicativo
de lo que pasa en $y_t$. Sin embargo, en los promedios moviles el pasado aparece solamente a traves de los shocks y la persistencia se cae abruptamente una vez que cruzamos el umbral que corresponde al orden de los promedios moviles

**arma**

La serie de tiempo depende de una constante + algunos rezagos donde aparece el rezago de la variable independiente + otros rezagos a traves de los shocks


Para los MA -> función de autocorrelación. 

comando

```
ac variable

```
me calcula los coeficiente que se corresponden con la autocorrelación, los que estan por fuera del intervalo de confianza son estadisticamente significativos

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
***
**Criterios de informacion**
***

AKAIKE -> la varianza de los errores, mas variables hace que el termino sea más chico porque se aplica el log. 

```
estat ic 
```
para reportar el valor del arkaike y el aic

***
**ESTACIONARIEDAD**
***

**Estricta** -> si la distribucion conjunta de cualquier coleccion de valores en distintos periodos del tiempo, la distribucion es la misma si se traslada hacia adelante o hacia atras para cualquier cantidad de periodos. 
La relacion estocastica entre las variables se sigue manteniendo a lo largo del tiempo y lo unico que nos importa son los rezagos o lags. 

**Estacionariedad debil** -> la esperanza es siempre la misma sin importar adonde estemos parando, la estructura de cov no depende de t, solo depende de la estructuras de rezagos. 

**Procesos debilmente dependientes** -> Como afecta el pasado lejano a la realizacion de la variable aleatoria. 
La cov entre $y_t$ y $y_{t+h} (h regazos hacia adelante o hacia atras) se hace cero cuando h va a infinitco ( es decir, cuando vamos a un pasado muy lejano)


*Observación*
Necesitamos que se mantenga la estacionariedad para poder utilizar el pasado para pronosticar hacia el futuro.

*Importante*: si la serie no es estacionaria no podemos elegir un modelo porque la serie va cambiando a lo largo del tiempo

***
**PREDICIÓN**
***

Conjunto de información: tiene lo que vimos (los datos del pasado) 

Para la prediccion vamos estimar la esperanza de $y_{t+1}$ condicional a la toda la información que tenemos hasta T. 

*Observacion*: se pueden armar los pronosticos de manera iterativa todos los periodos hacia adelante que queramos

*Observacion*: **la prediccion hacia un futuro sufientemente lejano se vuelve la esperanza no condicional**, es decir, el hecho de que nosotros condicionemos con el pasado hace que ese condicionamiento no juegue ningun rol cuando nos movemos hacia un pasado muy lejano (para todos los modelos), el pronostico se vuelve la esperanza no condicional

*******************************
SERIES DE TIEMPO II (SERIES NO ESTACIONARIAS)
*********************************

Random walk -> paseo aleatorio. 
No podemos aplicar la metodologia de Box

Si calculamos las experantas condicionales nos va a dar $y_t$, es decir, lo mejor que podemos predecir acerca de lo que va a pasar en el futuro tiene que ver con la infomacion que tenemos en el presente.

Se puede hacer el reemplazo iterativo hacia atras y asi vamos a llegar que cada valor de la serie es la suma de todos los shocks que ocurriendo en el pasado, donde cada shock tiene la misma ponderación.
La experanza no condicional de la serie va a depende del lugar de adonde partimos

No es una serie estacionaria porque no cumple con la condicion de estacionalidad debil, de varianzas constantes. 

No se puede predecir hacia adonde va a ir, porque la variabilidad se hace cada vez más grandes. 
El paseo aleatorio tiene demasiada información acumulada del pasado, entonces no vamos a poder discernir lo que es un shock de lo que es la información del pasado.

Como las relaciones entre los periodos rezagados va cambiando a lo largo del tiempo no podemos usar la informacion del pasado para explicar el futuro. 

*Observación*:Cada variantes de los paseos aleatorios da lugar a procesos diferentes, con experanzas, medias, var que pueden o no ser diferentes. 

Calcular: la experanza no condicional de $y_t$, la experanza condicionales,estructura de cov

***
ORDEN DE INTEGRACION
***

 Series debilmente dependiente: la influencia del pasado lejano se hace cero a medida que nos vamos hacia ese pasado (integrada de orden cero)
 A estas series se le puede aplicar una reg y hacer inferencia ( de orden cero).

 Cuando la series es integrada de orden 1, trabajamos con la diferencia, hacemos inferencia y pronosticamos con la diferencia y con ese pronostico despues podemos armar un pronostico sobre la serie en niveles. 
 
 **Contraste para raices unitarias**
 $$H0: p=1$$
 
Podria correr una reg de $y_t$ en $y_{t-1}$ que estimaria el parametro $\rho$, sobre este podriamos hacer inferencia siempre y cuando la serie no sea raiz unitaria. 

Importante: para hacer inferencia en una reg imponemos que la var es estable y con los paseos aleatorios no sucede. Cuando $|\rho|<1$ el estimador es asintoticamente normal.
Pero si $\rho=1$ la teoria asintotica no funciona.

***
*CONTRASTE DE DICKEY-FULLER*
***

1. Agarramos el modelo original y le resto a ambos lados de la igualdad $y_{t-1}$ 
2. Evaluar que $\rho=1$ es equivalente a evaluar que $\theta=0$ 
3. H0: $\theta=0$ 

El contraste corre una reg que tiene a la diferencia como variable independiente y al rezago como variable dependiente.

***
*CONTRASTE DE DICKEY-FULLER AUMENTADO*
***

**TENDENCIA** -> El hecho de que una serie tiene tendencia rompe con el supuesto de estacionariedad de que la media es constante.

Otro problema es que el hecho de que existe tendencia en la serie hace que nosotros, muchas veces, asociemos relaciones causales de vs
donde no lo hay, correlaciones espurias. 

Tampoco podemos aplicar la teoria asintotica estandar. 

La variable explicativa puede estar potencialmente no acotada y esto puede dar lugar a ciertos problemas en las relaciones asintoticas (OTRO PROBLEMA)

Cuando tenemos una serie con tendencia podemos descomponerla en dos partes, una se modela en forma deterministica como una tendencia $\delta *t$ y despues trabajamos con el resto con la teoria de Box y Jeckings.

PASOS:
1. Agarra la serie original 
2. reg y t (regresion de la serie original con respecto al tiempo)
```
reg y t
```
3. predict yh, resid (generamos una nueva variable que no tenga que ver con la tendencia)

```
predict y_r, residuales
```

Recordar que los residuos es todo aquello que no tenga que ver con las varibles explicativas que se uso en la reg original. 
Y con estos residuos es que voy a trabajar en termino de los componentes ARMA. 

**OJO ESTO ES ASI SIEMPRE Y CUANDO UNO ASUMA QUE TIENE UNA TENDENCIA LINEAL**

Pero no hay problema de todas formas, porque uno puede usar cualquier modelo en t para ajustar a la tendencia. 

***
*MODELO DE FILTROS*
***

En vez de asumir que la tendencia es deterministica, se asume que la tendencia no es deterministica sino que se va ajustando a las caracteristica de como va evolucionando. 

La tendencia es adaptativa. 

Se penaliza por grandes fluctuaciones. 

*USAR UNA TENDENCIA QUE SEA ESTOCASTICA Y NO DETERMINISTICA*

Usamos el ciclo de H-P.

```

tsfilter hp infciclo=inf, trend(inftrend) 
```

(infcliclo: nombre de la variable) permite que la tendencia vaya ajustando suavemnete a las fluctuaciones de la variable

El modelo de filtro, filtra una serie en distintos componentes -> componente de tendencia, de estacionalidad, componente irregular.

Observación: Como las cosas van cambiando por estacion deberiamos controlar por ese efecto. 


SOLUCIÓN:
Si uno sospecha que hay estacionalidad hay que controlar por esa estacionaridad y esto se hace con varibless dummies. + LO QUE CORRESPONDA A LA TENDENCIA. 
Una vez que tenemos esto sacamos los residuos y con estos residuos trabajamos con la metodologia de BOX y Jeckings. 

corro un modelo en la cual en la parte deterministica de la serie tengo estacionariedad y tengo tendencia, a su vez, si me fijo en la significatividad de las vs puedo testear por la tendencia y la estacionalidad*/
```
reg inf i.mes t t2 
```

Si me da que tiene estacinalidad y tendencia

```
predict u, resid. 

line u t 
```
CONCLUSION
Siempre voy a tener dos partes por separado, una modelizacion de las series de tiempo a partid de los residuos a lo cual tengo que eventualmente agregarle la modelizacion deterministica que viene por estacionalidad y tendencia.

Para armar un pronostico -> separo en dos partes, trabajo con los residuos con la metodologia de Box y Jeckings, y despues juntarlas de acuerdo a lo que necesite. 
