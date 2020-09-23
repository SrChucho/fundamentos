# Tareas {-}

* Las tareas se envían por correo a <teresa.ortiz.mancera@gmail.com> con título: 
fundamentos-tareaXX (donde XX corresponde al número de tarea, 01..). 

* Las tareas deben incluir código y resultados (si conocen [Rmarkdown](https://rmarkdown.rstudio.com) es muy conveniente para este propósito).

## 1. Anáslisis Exploratorio {-}

Realicen los ejercicios del script 01_exploratorio.R que vimos la clase pasada (RStudio.cloud proyecto 01-exploratorio). Escriban las respuestas en un reporte 
que incluya código y resultados (puede ser word, html, pdf,...).


## 2. Loess {-}

La tarea 2 es el proyecto de RStudio.Cloud con este nombre, el ejercicio está
en un archvio de R Markdown, para los que son nuevos con R Markdown deben 
escribir su código en los bloques de código (secciones grises entre 
\```{r}  codigo \```) una vez que escriban sus respuestas pueden generar el reporte tejiendo el documento, para esto deben presionar el botón *Knit*. Envíen el 
reporte por correo electrónico (con título fundamentos-tarea02).

### Solución: Series de tiempo {-}

Podemos usar el suavizamiento loess para entender y describir el comportamiento
de series de tiempo, en las cuales intentamos entender la dependencia de una
serie de mediciones indexadas por el tiempo. Típicamente es necesario utilizar 
distintas *componentes* para describir exitosamente una serie de tiempo, y para
esto usamos distintos tipos de suavizamientos. Veremos que distintas
*componentes* varían en distintas escalas de tiempo (unas muy lentas, cono la
tendencia, otras más rapidamente, como variación quincenal, etc.).

En el siguiente ejemplo consideramos la ventas semanales de un producto a lo 
largo de 5 años. Veamos que existe una tendencia a largo plazo (crecimientos
anuales) y también que existen patrones de variación estacionales.


```r
library(tidyverse)
ventas <- read.csv("./data/ventas_semanal.csv")
head(ventas)
```

```
##   period sales.kg
## 1      1  685.537
## 2      2  768.234
## 3      3  894.643
## 4      4  773.501
## 5      5  954.600
## 6      6  852.853
```

```r
ggplot(ventas, aes(x = period, y = sales.kg)) + geom_line(size = 0.3)
```

<img src="98-tareas_files/figure-html/unnamed-chunk-1-1.png" width="528" />

Intentaremos usar suavizamiento para capturar los distintos tipos de variación
que observamos en la serie. En primer lugar, si suavizamos poco (por ejemplo
$\aplha = 0.1$), vemos que capturamos en parte la tendencia y en parte la 
variación estacional.


```r
ggplot(ventas, aes(x = period, y = log(sales.kg))) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.1, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-2-1.png" width="528" />

Es mejor comenzar capturando la tendencia, y poco de la componente estacional:


```r
ggplot(ventas, aes(x = period, y = log(sales.kg))) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.3, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-3-1.png" width="528" />

```r
ajuste.trend.1 <- loess(log(sales.kg) ~ period, ventas, span = 0.3, degree = 1)
ventas$trend.1 <- ajuste.trend.1$fitted
ventas$res.trend.1 <- ajuste.trend.1$residuals
```

Ahora calculamos los residuales de este ajuste e intentamos describirlos 
mediante un suavizamiento más fino. Verificamos que hemos estimado la mayor
parte de la tendencia, e intentamos capturar la variación estacional de los 
residuales.


```r
ggplot(ventas, aes(x = period, y = res.trend.1)) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.15, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-4-1.png" width="528" />

```r
ajuste.est1.1 <- loess(res.trend.1 ~ period, ventas, span = 0.15, degree = 1)
ventas$est1.1 <- ajuste.est1.1$fitted
ventas$res.est1.1 <- ajuste.est1.1$residuals
```

Y graficamos los residuales obtenidos después de ajustar el componente 
estacional para estudiar la componente de mayor frecuencia.


```r
ggplot(ventas, aes(x = period, y = res.est1.1)) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.06, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-5-1.png" width="528" />

```r
ajuste.est2.1 <- loess(res.est1.1 ~ period, ventas, span = 0.06, degree = 1)
ventas$est2.1 <- ajuste.est2.1$fitted
ventas$res.est2.1 <- ajuste.est2.1$residuals
```

Ahora que tenemos nuestra primera estimación de cada una de las componentes, 
podemos regresar a hacer una mejor estimación de la tendencia. La ventaja de 
volver es que ahora podemos suavizar más sin que en nuestra muestra compita
tanto la variación estacional. Por tanto podemos suavizar menos:


```r
ventas$sales.sin.est.1 <- log(ventas$sales.kg) - ajuste.est1.1$fitted - 
  ajuste.est2.1$fitted

ggplot(ventas, aes(x = period, y = sales.sin.est.1)) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.09, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-6-1.png" width="528" />

```r
ajuste.trend.2 <- loess(sales.sin.est.1 ~ period, ventas, span = 0.08, degree = 1)
ventas$trend.2 <- ajuste.trend.2$fitted
ventas$res.trend.2 <- log(ventas$sales.kg) - ventas$trend.2
```

Y ahora nos concentramos en la componente anual.


```r
ventas$sales.sin.est.2 <- log(ventas$sales.kg) - ajuste.trend.2$fitted -
  ajuste.est2.1$fitted
ggplot(ventas, aes(x = period, y = sales.sin.est.2)) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.15, method.args = list(degree = 2), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-7-1.png" width="528" />

```r
ajuste.est1.2 <- loess(sales.sin.est.2 ~ period, ventas, span = 0.15, degree = 1)
ventas$est1.2 <- ajuste.est1.2$fitted
ventas$res.est1.2 <- ajuste.est1.2$residuals
```

Finalmente volvemos a ajustar la componente de frecuencia más alta:


```r
ventas$sales.sin.est.3 <- log(ventas$sales.kg) - ajuste.trend.2$fitted -
  ajuste.est1.2$fitted

ggplot(ventas, aes(x = period, y = sales.sin.est.3)) +
  geom_line(size = 0.3) +
  geom_smooth(method = "loess", span = 0.06, method.args = list(degree = 1), 
              se = FALSE, size = 1, 
    color = "red")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-8-1.png" width="528" />

```r
ajuste.est2.2 <- loess(sales.sin.est.3 ~ period, ventas, span = 0.06, degree = 1)
ventas$est2.2 <- ajuste.est2.2$fitted
ventas$res.est2.2 <- ajuste.est2.2$residuals
```

Verificamos nuestra descomposición y visualizamos el ajuste:


```r
ventas$log.sales <- log(ventas$sales.kg)
ventas.2 <- dplyr::select(ventas, period, trend.2, est1.2, est2.2, res.est2.2, 
  log.sales)
max(abs(apply(ventas.2[, 2:4], 1, sum) - ventas.2$log.sales))
```

```
## [1] 0.240316
```

```r
ventas.2.m <- gather(ventas.2, componente, valor, -period)

ventas.2.m.c <- ventas.2.m %>%
  group_by(componente) %>%
  mutate(
    valor.c = valor - mean(valor), 
    componente = factor(componente, c("trend.2", "est1.2", "est2.2", 
                                           "res.est2.2", "log.sales"))
  )

ggplot(ventas.2.m.c, aes(x = period, y = valor.c)) +
  geom_vline(xintercept = c(0, 52 - 1, 52 * 2 - 1, 52 * 3 - 1, 52 * 4 - 1), color = "gray") +
  geom_line(size = 0.3) +
  facet_wrap(~ componente, ncol = 1)
```

<img src="98-tareas_files/figure-html/unnamed-chunk-9-1.png" width="528" />

Y vemos que es razonable describir los residuales con una distribución normal 
(con desviación estándar alrededor de 8% sobre el valor ajustado):


```r
sd(ventas$res.est2.2)
```

```
## [1] 0.08093687
```

```r
ventas.ord <- arrange(ventas, res.est2.2)
ventas.ord$q.normal <- qnorm((1:nrow(ventas) - 0.5) / nrow(ventas))
ggplot(ventas.ord, aes(x = q.normal, y = res.est2.2)) +
  geom_point(size = 1.2) +
  geom_smooth(method = "lm")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

<img src="98-tareas_files/figure-html/unnamed-chunk-10-1.png" width="345.6" />

Aún no estamos convencidos de que podemos hacer *pooling* de los residuales. 
Para checar esto, vemos la gráfica de dependencia de residuales.


```r
ggplot(ventas, aes(x = period, y = res.est2.2)) + geom_line(size = 0.3) + 
  geom_point(size = 1.2)
```

<img src="98-tareas_files/figure-html/unnamed-chunk-11-1.png" width="528" />

¿Puedes notar alguna dependencia en los residuales?


```r
library(nullabor)
ventas.res <- dplyr::select(ventas, period, res.est2.2)
ventas.null <- lineup(null_dist(var = 'res.est2.2', dist = 'normal', 
  params = list(mean = 0, sd = 0.08)), n = 10, ventas.res)
```

```
## decrypt("bhMq KJPJ 62 sSQ6P6S2 ua")
```

```r
ggplot(ventas.null, aes(x = period, y = res.est2.2)) +
  facet_wrap(~ .sample, ncol = 2) + 
  geom_line(size = 0.3) +
  geom_vline(xintercept = c(0, 52 - 1, 52 * 2 - 1, 52 * 3 - 1, 52 * 4 - 1), color = "gray") + 
  geom_point(size = 1.2) 
```

<img src="98-tareas_files/figure-html/unnamed-chunk-12-1.png" width="960" />

Hay dos cosas que nos falta explicar, en primer lugar, las caídas alrededor de
principios/finales de cada año (que son de hasta -0.2), y segundo que esta 
gráfica parece oscilar demasiado. La estructura que aún no hemos explicado se
debe a que las semanas que caen en quincena tienden a tener compras más 
grandes que las que están justo antes de quincena o fin de mes.

Por el momento detendremos el análisis aquí y explicamos un proceso iterativo
para proceder en nuestro análisis exploratorio:

<div class="caja">
**Iterando ajuste de loess.** Cuando queremos ajustar con tres componentes: 
  tendencia, estacionalidad y residuales, podemos seguir el siguiente proceso,

1. Ajustar la primera componente a los datos (tendencia).

2. Ajustar la segunda componente a los residuales del paso anterior 
(estacionalidad).

3. Restar de los datos originales la segunda componente ajustada 
(estacionalidad).

4. Ajustar a los residuales del paso anterior una nueva componente (tendencia).

5. Restar a los datos originales la componente ajustada en el paso anterior.

6. Ajustar a los residuales del paso anterior una nueva componente 
(estacionalidad).

7. Checar ajuste y si es necesario iterar de 3 a 6 con las nuevas componentes.
</div>

La idea es que cada componente compite para explicar los datos (cada una gana
más al bajar el parámetro $\alpha$). El conflicto es que si suavizamos mucho
cada componente (por ejemplo la tendencia), entonces parte de la variación 
que debería ir en ella queda en los residuales, y se intenta ajustar 
posteriormente por una componente distinta (estacionalidad). Sin embargo, si 
suavizamos poco, entonces parte de la variación de la segunda componente es
explicada por el ajuste de la primera. Entonces, la solución es ir poco a poco
adjudicando variación a cada componente. En nuestro ejemplo de arriba, podemos
comenzar suavizando de menos el primer ajsute de la tendencia, luego ajustar
estacionalidad, restar a los datos originales esta estacionalidad, y ajustar a
estos datos una componente más suave de tandencia. Es posible suavizar más la
tendencia justamente porque ya hemos eliminado una buena parte de la
estacionalidad.


## 3. Tipos de estudio y PGD {-}

Para cada uno de los siguientes estudios, ubícalos en el recuadro y contesta lo 
que se pide. Envíen las respuestas por correo electrónico (con título 
fundamentos-tarea03).

![Inferencia estadística de acuerdo al tipo del diseño [@ramsey].](images/03_inferencia-muestra.png)

1. En 1930 se realizó un experimento en 20,000 niños de edad escolar de Inglaterra.
Los maestros fueron los responsables de asignar a los niños de manera aleatoria al
grupo de tratamiento -que consistía en recibir 350 ml de leche diaria - o al 
grupo de control, que no recibía suplementos alimenticios. Se registraron peso y 
talla antes y después del experimento. El estudio descubrió que los niños que
recibieron la leche ganaron más en peso en el lapso del estudio. Una 
investigación posterior descubrió que los niños del grupo control eran de mayor 
peso y talla que los del grupo de intervención, antes de iniciar el tratamiento. 
¿Qué pudo haber ocurrido? ¿Podemos 
utilizar los resultados del estudio para inferir causalidad?

1. Supongamos que de los registros de un conjunto de doctores se slecciona una 
muestra aleatoria de individuos americanos caucásicos y de americanos de 
ascendencia china, con el objetivo de comparar la presión arterial de las dos
poblaciones. Supongamos que a los seleccionados se les pregunta si quieren
participar y algunos rechazan. Se compara la distribución de presión arterial
entre los que accedieron a participar. ¿En que cuadro cae este estudio? ¿Qué 
supuesto es necesario para permitir inferencias a las poblaciones muestreadas?

1. Un grupo de investigadores reportó que el consumo moderado de alcohol estaba
asociado con un menor riesgo de demencia (Mukamal et al. (2003)). Su muestra 
consistía en 373 personas con demencia y 373 sin demencia. A los participantes 
se les pregintó cuánta cerveza, vino, o licor consumían. Se observó que aquellos
que consumían de 1-6 bebidas por semana tenían una incidencia menor de demencia
comparado a aquellos que se abstenían del alcohol. ¿se puede inferir causalidad?

2. Un estudio descubrió que los niños que ven más de dos horas diarias de 
televisión tienden a tener mayores niveles de colesterol que los que ven menos 
de dos horas diarias. ¿Cómo se pueden utilizar estos resultados?

1. Más gente se enferma de gripa en temporada de invierno, ¿esto prueba que las
temperaturas bajas ocasionan las gripas? ¿Qué otras variables podrían estar 
involucradas?

2. ¿Cuál es la diferencia entre un experimento aleatorizado y una muestra 
aleatoria?


## 4. Pruebas de hipótesis visuales y permutación {-}

La tarea 4 es el proyecto de RStudio.Cloud con este nombre, el ejercicio está
en un archvio de R Markdown. Envíen el reporte por correo electrónico (con 
título fundamentos-tarea04).

### Solución: Pruebas pareadas {-}

En este ejemplo buscamos comparar la diferencia entre dos medicinas 
para dormir. 
  - ID es el identificador de paciente, y medicina_1 y medicina_2 son las
  horas extras de sueño vs. no usar medicina.  
  - Examina los datos.  



```r
library(tidyverse)

dormir <- sleep %>% 
  pivot_wider(names_from = group, 
              names_prefix = "medicina_",
              values_from = extra)

dormir
```

```
## # A tibble: 10 x 3
##    ID    medicina_1 medicina_2
##    <fct>      <dbl>      <dbl>
##  1 1            0.7        1.9
##  2 2           -1.6        0.8
##  3 3           -0.2        1.1
##  4 4           -1.2        0.1
##  5 5           -0.1       -0.1
##  6 6            3.4        4.4
##  7 7            3.7        5.5
##  8 8            0.8        1.6
##  9 9            0          4.6
## 10 10           2          3.4
```

La pregunta de interés es si una medicina es mejor que otra para prolongar el 
sueño. Nótese que en este caso, no tenemos grupos, sino mediciones repetidas.

- Escribe la hipótesis nula.

**No hay diferencia entre las medicinas.**

- Nuestra estadśtica de interés es media de las diferencias entre las medicinas. Calcula la diferencia observada.


```r
dif_obs <- dormir %>% 
  mutate(dif = medicina_1 - medicina_2) %>% 
  summarise(dif_obs = mean(dif)) %>% 
  pull(dif_obs)
dif_obs
```

```
## [1] -1.58
```

- Hay variación entre los pacientes. ¿Tenemos evidencia para rechazar que son 
iguales? ¿Cómo hacemos nuestra distribución de referencia?

**Bajo la hipótesis nula las medicinas son intercambiables, debemos
la columna de medicina _por individuo_ pues cada uno debe
tener un valor de medicina 1 y uno de medicina 2**.


```r
dormir_larga <- dormir %>% 
  pivot_longer(cols = contains("medicina"))
# Hacemos una permutación 
dormir_perm <- dormir_larga %>% 
  group_by(ID) %>% 
  mutate(name = sample(c("medicina_1", "medicina_2"))) %>% 
  ungroup()
# Calculamos la estadística en esta permutación
dormir_perm %>%
  pivot_wider(names_from = "name", values_from = "value") %>% 
  mutate(dif = medicina_1 - medicina_2) %>% 
  summarise(media_dif = mean(dif)) %>% 
  pull(media_dif)
```

```
## [1] -0.28
```

```r
# Lo hacemos una función 
calcula_est <- function(){
  dormir_perm <- dormir_larga %>% 
    group_by(ID) %>% 
    mutate(name = sample(c("medicina_1", "medicina_2"))) %>% 
    ungroup()
  dormir_perm %>%
    pivot_wider(names_from = "name", values_from = "value") %>% 
    mutate(dif = medicina_1 - medicina_2) %>% 
    summarise(media_dif = mean(dif)) %>% 
    pull(media_dif)
}
# La llamamos 1000 veces
calcula_est()
```

```
## [1] -0.46
```

```r
difs <- rerun(1000, calcula_est()) %>% flatten_dbl()
```


- Haz una gráfica de la distribución de referencia y grafica encima el valor 
observado en los datos originales.


```r
perms <- tibble(sims = 1:1000, difs = difs)
ggplot(perms, aes(x = difs)) +
  geom_histogram(binwidth = 0.20) +
  geom_vline(xintercept = dif_obs, color = "red")
```

<img src="98-tareas_files/figure-html/unnamed-chunk-16-1.png" width="336" />


## 5. Distribución muestral y remuestreo {-}

La tarea 5 es el proyecto de RStudio.Cloud con este nombre, el ejercicio está
en un archvio de R Markdown. Envíen el reporte por correo electrónico (con 
título fundamentos-tarea05).

## 6. TCL e introducción a bootstrap {-}

La tarea 6 es el proyecto de RStudio.Cloud con este nombre, los ejercicios
están descritos en un archivo de R. Envíen un reporte por correo electrónico con 
las respuestas (con título fundamentos-tarea06).

### Solución y discusión de media {-}

---
title: "Bootstrap - Media"
output: html_document
---




#### Ejemplo: el error estándar de una media {-}

Supongamos que $x$ es una variable aleatoria que toma valores en los reales con 
distribución de probabilidad $P$. Denotamos por $\mu_F$ y $\sigma_F^2$ la 
media y varianza de $P$,

$$\mu_F = E_F(x),$$ 
$$\sigma_F^2=var_F(x)=E_F[(x-\mu)^2]$$

en la notación enfatizamos la dependencia de la media y varianza en la 
distribución $P$. 

Ahora, sea $(x_1,...,x_n)$ una muestra aleatoria de $P$, de tamaño $n$, 
la media de la muestra $\bar{x}=\sum_{i=1}^nx_i/n$ tiene:

* esperanza $\mu_F$,

* varianza $\sigma_F^2/n$.

En palabras: la esperanza de $\bar{x}$ es la misma que la esperanza de $x$, pero
la varianza de $\bar{x}$ es $1/n$ veces la varianza de $x$, así que entre
mayor es la $n$ tenemos una mejor estimación de $\mu_P$.

En el caso de la media $\bar{x}$, el error estándar, que denotamos 
$se_P(\bar{x})$, es la raíz de la varianza de $\bar{x}$,

$$se_F(\bar{x}) = [var_F(\bar{x})]^{1/2}= \sigma_F/ \sqrt{n}.$$

En este punto podemos usar el principio del _plug-in_, simplemente sustituimos
$P_n$ por $P$ y obtenemos, primero, una estimación de $\sigma_P$:
$$\hat{\sigma}=\hat{\sigma}_{\hat{F}} = \bigg\{\frac{1}{n}\sum_{i=1}^n(x_i-\bar{x})^2\bigg\}^{1/2}$$

de donde se sigue la estimación del error estándar:

$$\hat{ee}(\bar{x})=\hat{\sigma}_{\hat{F}}/\sqrt{n}=\bigg\{\frac{1}{n^2}\sum_{i=1}^n(x_i-\bar{x})^2\bigg\}^{1/2}$$

Notemos que usamos el principio del _plug-in_ en dos ocasiones, primero para 
estimar la esperanza $\mu_P$ mediante $\mu_{\hat{F}}$ y luego para estimar el 
error estándar $ee_F(\bar{x})$. 


Consideramos los datos de ENLACE edo. de México 
(`enlace`), y la columna de calificaciones de español 3^o^ de primaria (`esp_3`). 


```r
enlace <- read_csv("data/enlace_15.csv")
```

```
## Parsed with column specification:
## cols(
##   id = col_double(),
##   cve_ent = col_double(),
##   turno = col_character(),
##   tipo = col_character(),
##   esp_3 = col_double(),
##   esp_6 = col_double(),
##   n_eval_3 = col_double(),
##   n_eval_6 = col_double()
## )
```
Suponemos que me interesa hacer inferencia del promedio de las 
calificaciones de los estudiantes de tercero de primaria en el Estado de México.

En este ejercicio planteamos $3$ escenarios (que simulamos): 1) que tengo una 
muestra de tamaño $10$, 2) que tengo una muestra de tamaño $100$, y 3) que tengo una 
muestra de tamaño $1000$. 

- Selección de muestras:


```r
set.seed(373783326)
muestras <- tibble(tamanos = c(10, 100, 1000)) %>% 
    mutate(muestras = map(tamanos, ~sample(enlace$esp_3, size = .)))
```

Ahora procedemos de manera *usual* en estadística (usando fórmulas y no 
simulación), estimo la media de la muestra con el estimador *plug-in* 

$$\bar{x}={1/n\sum x_i}$$ 

y el error estándar de $\bar{x}$ con el estimador *plug-in*, en el caso de la 
media hay una fórmula, pero cómo calculamos el error estándar de una mediana?, 
o de una correlación?

$$\hat{ee}(\bar{x}) =\bigg\{\frac{1}{n^2}\sum_{i=1}^n(x_i-\bar{x})^2\bigg\}^{1/2}$$

- Estimadores *plug-in*:


```r
se_plug_in <- function(x){
    x_bar <- mean(x)
    n_x <- length(x)
    var_x <- 1 / n_x ^ 2 * sum((x - x_bar) ^ 2)
    sqrt(var_x)
}
muestras_est <- muestras %>% 
    mutate(
        medias = map_dbl(muestras, mean), 
        e_estandar_plug_in = map_dbl(muestras, se_plug_in)
    )
muestras_est
```

```
## # A tibble: 3 x 4
##   tamanos muestras      medias e_estandar_plug_in
##     <dbl> <list>         <dbl>              <dbl>
## 1      10 <dbl [10]>      602.              19.3 
## 2     100 <dbl [100]>     553.               6.54
## 3    1000 <dbl [1,000]>   552.               1.90
```

Ahora, recordemos que la distribución muestral es la distribución de una
estadística, considerada como una variable aleatoria. Usando esta definción 
podemos aproximarla, para cada tamaño de muestra, simulando:  

1) simulamos muestras de tamaño $n$ de la población,   
2) calculamos la estadística de interés (en este caso $\bar{x}$),  
3) vemos la distribución de la estadística a lo largo de simulaciones.

- Histogramas de distribución muestral y aproximación de errores estándar con 
simulación 


```r
muestras_sims <- muestras_est %>%
    mutate(
        sims_muestras = map(tamanos, ~rerun(10000, sample(enlace$esp_3, 
            size = ., replace = TRUE))), 
        sims_medias = map(sims_muestras, ~map_dbl(., mean)), 
        e_estandar_aprox = map_dbl(sims_medias, sd)
        )
sims_medias <- muestras_sims %>% 
    select(tamanos, sims_medias) %>% 
    unnest(sims_medias) 

ggplot(sims_medias, aes(x = sims_medias)) +
    geom_histogram(binwidth = 2) +
    facet_wrap(~tamanos, nrow = 1) +
    theme_minimal()
```

<img src="98-tareas_files/figure-html/unnamed-chunk-21-1.png" width="672" height="350px" />

Notamos que la variación en la distribución muestral decrece conforme aumenta
el tamaño de muestra, esto es esperado pues el error estándar de una media 
es $\sigma_P / \sqrt{n}$, y dado que en este ejemplo estamos calculando la media 
para la misma población el valor poblacional $\sigma_P$ es constante y solo 
cambia el denominador.

Nuestros valores de error estándar con simulación están en la columna 
`e_estandar_aprox`:


```r
muestras_sims %>% 
    select(tamanos, medias, e_estandar_plug_in, e_estandar_aprox)
```

```
## # A tibble: 3 x 4
##   tamanos medias e_estandar_plug_in e_estandar_aprox
##     <dbl>  <dbl>              <dbl>            <dbl>
## 1      10   602.              19.3             18.9 
## 2     100   553.               6.54             5.92
## 3    1000   552.               1.90             1.87
```

En este ejercicio estamos simulando para examinar las distribuciones muestrales
y para ver que podemos aproximar el error estándar de la media usando 
simulación; sin embargo, dado que en este caso hipotético conocemos la varianza 
poblacional y la fórmula del error estándar de una media, por lo que podemos 
calcular el verdadero error estándar para una muestra de cada tamaño.

- Calcula el error estándar de la media para cada tamaño de muestra usando la
información poblacional:


```r
muestras_sims_est <- muestras_sims %>% 
    mutate(e_estandar_pob = sd(enlace$esp_3) / sqrt(tamanos))
muestras_sims_est %>% 
    select(tamanos, e_estandar_plug_in, e_estandar_aprox, e_estandar_pob)
```

```
## # A tibble: 3 x 4
##   tamanos e_estandar_plug_in e_estandar_aprox e_estandar_pob
##     <dbl>              <dbl>            <dbl>          <dbl>
## 1      10              19.3             18.9           18.7 
## 2     100               6.54             5.92           5.93
## 3    1000               1.90             1.87           1.87
```

En la tabla de arriba podemos comparar los $3$ errores estándar que calculamos, 
recordemos que de estos $3$ el *plug-in* es el único que podríamos obtener en 
un escenario real pues los otros dos los calculamos usando la población. 

Una alternativa al estimador *plug-in* del error estándar es usar *bootstrap* 
(en muchos casos no podemos calcular el error estándar *plug-in* por falta de 
fórmulas) pero podemos usar *bootstrap*: utilizamos una 
estimación de la distribución poblacional y calculamos el error estándar 
bootstrap usando simulación. Hacemos el mismo procedimiento que usamos para 
calcular *e_estandar_apox* pero sustituimos la distribución poblacional por la 
distriución empírica. Hagámoslo usando las muestras que sacamos en el primer 
paso:


```r
muestras_sims_est_boot <- muestras_sims_est %>% 
    mutate(
        sims_muestras_boot = map2(muestras, tamanos,
            ~rerun(10000, sample(.x, size = .y, replace = TRUE))), 
        sims_medias_boot = map(sims_muestras_boot, ~map_dbl(., mean)), 
        e_estandar_boot = map_dbl(sims_medias_boot, sd)
        )
muestras_sims_est_boot
```

```
## # A tibble: 3 x 11
##   tamanos muestras medias e_estandar_plug… sims_muestras sims_medias
##     <dbl> <list>    <dbl>            <dbl> <list>        <list>     
## 1      10 <dbl [1…   602.            19.3  <list [10,00… <dbl [10,0…
## 2     100 <dbl [1…   553.             6.54 <list [10,00… <dbl [10,0…
## 3    1000 <dbl [1…   552.             1.90 <list [10,00… <dbl [10,0…
## # … with 5 more variables: e_estandar_aprox <dbl>, e_estandar_pob <dbl>,
## #   sims_muestras_boot <list>, sims_medias_boot <list>, e_estandar_boot <dbl>
```

Graficamos los histogramas de la distribución bootstrap para cada muestra.


```r
sims_medias_boot <- muestras_sims_est_boot %>% 
    select(tamanos, sims_medias_boot) %>% 
    unnest(sims_medias_boot) 

ggplot(sims_medias_boot, aes(x = sims_medias_boot)) +
    geom_histogram(binwidth = 4) +
    facet_wrap(~tamanos, nrow = 1) +
    theme_minimal()
```

<img src="98-tareas_files/figure-html/unnamed-chunk-25-1.png" width="672" height="350px" />

Y la tabla con todos los errores estándar quedaría:


```r
muestras_sims_est_boot %>% 
    select(tamanos, e_estandar_boot, e_estandar_plug_in, e_estandar_aprox, 
        e_estandar_pob)
```

```
## # A tibble: 3 x 5
##   tamanos e_estandar_boot e_estandar_plug_in e_estandar_aprox e_estandar_pob
##     <dbl>           <dbl>              <dbl>            <dbl>          <dbl>
## 1      10           19.3               19.3             18.9           18.7 
## 2     100            6.53               6.54             5.92           5.93
## 3    1000            1.89               1.90             1.87           1.87
```

Observamos que el estimador bootstrap del error estándar es muy similar al 
estimador plug-in del error estándar, esto es esperado pues se calcularon con la 
misma muestra y el error estándar *bootstrap* converge al *plug-in* conforme 
incrementamos el número de muestras *bootstrap*.


#### ¿Por qué bootstrap? {-}

* En el caso de la media $\hat{\theta}=\bar{x}$ la aplicación del principio del 
_plug-in_ para el cálculo de errores estándar es inmediata; sin embargo, hay 
estadísticas para las cuáles no es fácil aplicar este método.

* El método de aproximarlo con simulación, como lo hicimos en el ejercicio de 
arriba no es factible pues en la práctica no podemos seleccionar un número 
arbitrario de muestras de la población, sino que tenemos únicamente una muestra. 

* La idea del *bootstrap* es replicar el método de simulación para aproximar
el error estándar, esto es seleccionar muchas muestras y calcular la estadística 
de interés en cada una, con la diferencia que las muestras se seleccionan de la
distribución empírica a falta de la distribución poblacional.

