
# Horas y fechas

## Introducción

Este capítulo te mostrará cómo trabajar con fechas y horas en R. A primera vista, esto parece sencillo. Las usas en todo momento en tu vida regular, y no parecen causar demasiada confusión. Sin embargo, cuanto más aprendes de horas y fechas, más complicadas se vuelven. Para prepararnos, intenta estas preguntas sencillas:

* ¿Todos los años tienen 365 días?
* ¿Todos los días tienen 24 horas?
* ¿Cada minuto tiene 60 segundos?

Estoy seguro que sabes que no todos los años tienen 365 días ¿pero acaso conoces la regla entera para determinar si un año es bisiesto? (Tiene tres partes, de hecho). Puedes recordar que muchas partes del mundo usan horarios de verano (las siglas en inglés son DST, por _daylight savings time_), así que algunos días tienen 23 horas, y otros tienen 25. Puede ser que no supieras que algunos minutos tienen 61 segundos, porque de vez en cuando se agregan segundos adicionales ya que la rotación de la tierra se hace cada vez más lenta.

Las fechas y las horas son complicadas porque tienen que reconciliar dos fenómenos físicos (la rotación de la Tierra, y su órbita alrededor del sol), con todo un conjunto de fenómenos geopolíticos que incluyen a los meses, los husos horarios y los horarios de verano (DST). Este capítulo no te enseñará cada detalle sobre las horas y fechas, pero te dará un sólido fundamento de habilidades prácticas que te ayudarán con los desafíos más comunes de análisis de datos.

### Requisitos previos

Este capítulo se centra en el paquete __lubridate__, que simplifica el trabajo con fechas y tiempo en R. __lubridate__ no es parte de los paquetes centrales de __tidyverse__ porque sólo se necesita al trabajar con fechas/horas. A su vez, necesitaremos `vuelos` del conjunto de datos traducidos que acompañan a este libro.


```r
library(tidyverse)

library(lubridate)
library(datos)
```

## Creando horas/fechas

Hay tres tipos de datos de horas/fechas que se refieren a un instante en el tiempo:

* Un `date` o fecha (del inglés, _fecha_). Un _tibble_ lo imprime como `<date>`.

* Un `time` u hora (del ingles, _hora_) dentro de un día. Los _tibbles_ lo imprimen como `<time>`.

* Un  `date-time` o fecha-hora (del inglés, _fecha-hora_) es una fecha con una hora adjunta:  identifica de forma única como un instante en el tiempo (típicamente al segundo más cercano). Los _tibbles_ imprimen esto como `<dttm>`. En otras partes de R, éstos se llaman POSIXct, pero yo no creo que sea un nombre muy útil.

En este capítulo sólo nos concentraremos en fechas (`date`) y fechas con horas (`date-time`) ya que R no tiene una clase nativa para almacenar horas. Si necesitas una, puedes usar el paquete __hms__.
  
Siempre deberías usar el tipo de datos más sencillo que se ajusta a tus necesidades. Esto significa que si puedes usar un `date` en lugar de un `date-time`, deberías hacerlo. Las fechas-horas son sustancialmente más complicadas porque necesitas gestionar los husos horarios, a los cuales estudiaremos al final del capítulo.

Para obtener la fecha (`date`) o fecha-hora (`date-time`) actual, puedes usar `today()` (del inglés, _hoy_) o `now()` (del inglés, _ahora_):


```r
today()
#> [1] "2018-09-12"
now()
#> [1] "2018-09-12 20:32:02 UTC"
```

De otra forma, hay tres modos en los que puedes crear una fecha/hora:

* Desde una cadena de caracteres (o _string_, en inglés).
* Desde componentes `date-time` individuales. 
* Desde un objeto fecha-hora existente.

Éstos funcionan de la siguiente manera.


### Desde cadenas de caracteres

Los datos de fecha/hora a menudo vienen como cadenas de caracteres. Ya has visto una forma de análisis gramatical de _strings_ hacia `date-times` en [date-times](#readr-datetimes). Otra forma es usar los ayudantes provistos por __lubridate__. Ellos automáticamente trabajan el formato una vez que especificas el orden de los componentes. Para usarlos, identifica el orden en el cual el año, mes y día aparecen en tus fechas, y luego agrega "y" (del inglés _year_), "m" (mes) y "d" (día) en el mismo orden. Esto te da el nombre de la función __lubridate__ que traducirá tu fecha. Por ejemplo:


```r
ymd("2017-01-31")
#> [1] "2017-01-31"
mdy("Enero 31, 2017")
#> Warning: All formats failed to parse. No formats found.
#> [1] NA
dmy("31-Ene-2017")
#> Warning: All formats failed to parse. No formats found.
#> [1] NA
```

Estas funciones también reciben números sin comillas. Esta es la forma más concisa de crear un sólo objeto fecha/hora, ya que puedes necesitarlo cuando filtres datos temporales. `ymd()` (del inglés _año-mes-día_)  es corto y no ambigüo:


```r
ymd(20170131)
#> [1] "2017-01-31"
```

`ymd()` y funciones amigas crean fechas (`date`). Para generar un `date-time`, agrega un guión bajo y uno o más de "h", "m" y "s" al nombre de la función de análisis:


```r
ymd_hms("2017-01-31 20:11:59")
#> [1] "2017-01-31 20:11:59 UTC"
mdy_hm("01/31/2017 08:01")
#> [1] "2017-01-31 08:01:00 UTC"
```

También puedes forzar la creación de un `date-time` desde una fecha, al proveer un huso horario:


```r
ymd(20170131, tz = "UTC")
#> [1] "2017-01-31 UTC"
```

### Desde componentes individuales

En lugar de una cadena de caracteres simple, a veces tienes los componentes individuales de una fecha/hora esparcidos en múltiples columnas. Esto es lo que tenemos en los datos de vuelos:


```r
vuelos %>% 
  select(anio, mes, dia, hora, minuto)
#> # A tibble: 336,776 x 5
#>    anio   mes   dia  hora minuto
#>   <int> <int> <int> <dbl>  <dbl>
#> 1  2013     1     1     5     15
#> 2  2013     1     1     5     29
#> 3  2013     1     1     5     40
#> 4  2013     1     1     5     45
#> 5  2013     1     1     6      0
#> 6  2013     1     1     5     58
#> # ... with 3.368e+05 more rows
```

Para crear una fecha y hora desde este tipo de entrada, usa `make_date()` (del inglés, _crear fechas_) para las fechas, o `make_datetime()` (del inglés _crear fecha-hora_) para las fechas-horas:


```r
vuelos %>% 
  select(anio, mes, dia, hora, minuto) %>% 
  mutate(salida = make_datetime(anio, mes, dia, hora, minuto))
#> # A tibble: 336,776 x 6
#>    anio   mes   dia  hora minuto salida             
#>   <int> <int> <int> <dbl>  <dbl> <dttm>             
#> 1  2013     1     1     5     15 2013-01-01 05:15:00
#> 2  2013     1     1     5     29 2013-01-01 05:29:00
#> 3  2013     1     1     5     40 2013-01-01 05:40:00
#> 4  2013     1     1     5     45 2013-01-01 05:45:00
#> 5  2013     1     1     6      0 2013-01-01 06:00:00
#> 6  2013     1     1     5     58 2013-01-01 05:58:00
#> # ... with 3.368e+05 more rows
```

Hagamos esto mismo para cada una de las cuatro columnas de tiempo en `vuelos`. Las horas están representadas en un formato ligeramente más extraño, así que usamos el módulo aritmético para extraer los componentes de horas y minutos. Una vez que hayamos creado las variables `date-time`, nos centraremos en las variables que usaremos por el resto del capítulo^[N. del T.: El ultimo `select` de esta parte difiere del original o de lo contrario obtenemos un dataset con menos columnas que en el original haciendo que fallen los ejemplos posteriores. Esto se hizo ya que el castellano algunas palabras se invierten de orden.].


```r
hacer_fechahora_100 <- function(anio, mes, dia, tiempo) {
  make_datetime(anio, mes, dia, tiempo %/% 100, tiempo %% 100)
}

vuelos_dt <- vuelos %>% 
  filter(!is.na(horario_salida), !is.na(horario_llegada)) %>% 
  mutate(
    horario_salida = hacer_fechahora_100(anio, mes, dia, horario_salida),
    horario_llegada = hacer_fechahora_100(anio, mes, dia, horario_llegada),
    salida_programada = hacer_fechahora_100(anio, mes, dia, salida_programada),
    llegada_programada = hacer_fechahora_100(anio, mes, dia, llegada_programada)
  ) %>% 
  select(origen, destino, starts_with("atraso"), starts_with("horario"), ends_with("programada"), tiempo_vuelo)

vuelos_dt
#> # A tibble: 328,063 x 9
#>   origen destino atraso_salida atraso_llegada horario_salida     
#>   <chr>  <chr>           <dbl>          <dbl> <dttm>             
#> 1 EWR    IAH                 2             11 2013-01-01 05:17:00
#> 2 LGA    IAH                 4             20 2013-01-01 05:33:00
#> 3 JFK    MIA                 2             33 2013-01-01 05:42:00
#> 4 JFK    BQN                -1            -18 2013-01-01 05:44:00
#> 5 LGA    ATL                -6            -25 2013-01-01 05:54:00
#> 6 EWR    ORD                -4             12 2013-01-01 05:54:00
#> # ... with 3.281e+05 more rows, and 4 more variables:
#> #   horario_llegada <dttm>, salida_programada <dttm>,
#> #   llegada_programada <dttm>, tiempo_vuelo <dbl>
```

Con estos datos, puedo visualizar la distribución de las horas de salida a lo largo del año:


```r
vuelos_dt %>% 
  ggplot(aes(horario_salida)) + 
  geom_freqpoly(binwidth = 86400) # 86400 segundos = 1 día
```

<img src="datetimes_files/figure-html/unnamed-chunk-10-1.png" width="70%" style="display: block; margin: auto;" />

O para un solo día:


```r
vuelos_dt %>% 
  filter(horario_salida < ymd(20130102)) %>% 
  ggplot(aes(horario_salida)) + 
  geom_freqpoly(binwidth = 600) # 600 segundos = 10 minutos
```

<img src="datetimes_files/figure-html/unnamed-chunk-11-1.png" width="70%" style="display: block; margin: auto;" />

Mira con detenimiento, ya que cuando usas `date-times` en un contexto numérico (como en un histograma), 1 significa un segundo, entonces un `binwidth` (del inglés, _ancho del cajón_) de 86400 significa un día. Para las fechas, 1 significa un día.

### Desde otros tipos

Puedes querer cambiar entre una fecha-hora y una fecha. Ese es el trabajo de  `as_datetime()` (del inglés, _como fecha-hora_) y `as_date()` (del inglés, _como fecha_):


```r
as_datetime(today())
#> [1] "2018-09-12 UTC"
as_date(now())
#> [1] "2018-09-12"
```

A veces, tendrás fechas/horas como desfasajes numéricos de la "Época Unix" similares a 1970-01-01. Si el desfasaje es un segundos, usa `as_datetime()`; si es en días, usa `as_date()`.


```r
as_datetime(60 * 60 * 10)
#> [1] "1970-01-01 10:00:00 UTC"
as_date(365 * 10 + 2)
#> [1] "1980-01-01"
```

### Ejercicios

1. ¿Qué sucede si analizas una cadena de caracteres que contiene fechas inválidas?

    
    ```r
    ymd(c("2010-10-10", "bananas"))
    ```

1. ¿Qué hace el argumento `tzone` (del inglés, _huso horario_ abreviado) para `today()`? ¿Por qué es importante?

1. Usa la función de __lubridate__ apropiada para analizar las siguientes fechas:

    
    ```r
    d1 <- "Enero 1, 2010"
    d2 <- "2015-Mar-07"
    d3 <- "06-Jun-2017"
    d4 <- c("Agosto 19 (2015)", "Julio 1 (2015)")
    d5 <- "12/30/14" # Diciembre 30, 2014
    ```

## Componentes fecha-hora

Ahora que ya conoces cómo tener datos de fechas y horas en las estructuras de datos de R, vamos a explorar qué puedes hacer con ellos. Esta sección se concentrará en las funciones de acceso que te permiten obtener y configurar componentes individuales. La siguiente sección mirará el trabajo aritmético con fechas-horas.

### Obteniendo los componentes

Puedes obtener las partes individuales de una fecha con las funciones de acceso `year()` (del inglés, _año_), `month()` (del inglés, _mes_), `mday()` (del inglés, _día del mes_), `yday()` (del inglés, _día del año_), `wday()` (del inglés, _día de la semana_), `hour()` (del inglés, _hora_), `minute()` (del inglés, _minuto_), y `second()` (del inglés, _segundo_). 


```r
fechahora <- ymd_hms("2016-07-08 12:34:56")

year(fechahora)
#> [1] 2016
month(fechahora)
#> [1] 7
mday(fechahora)
#> [1] 8

yday(fechahora)
#> [1] 190
wday(fechahora)
#> [1] 6
```

Para `month()` y `wday()` puedes configurar `label = TRUE` para retornar el nombre abreviado del mes o del día de la semana. Usa `abbr = FALSE` para retornar el nombre completo.


```r
month(fechahora, label = TRUE)
#> [1] Jul
#> 12 Levels: Jan < Feb < Mar < Apr < May < Jun < Jul < Aug < Sep < ... < Dec
wday(fechahora, label = TRUE, abbr = FALSE)
#> [1] Friday
#> 7 Levels: Sunday < Monday < Tuesday < Wednesday < Thursday < ... < Saturday
```

Podemos usar `wday()` para ver que más vuelos salen durante la semana que durante el fin de semana:


```r
vuelos_dt %>% 
  mutate(dia_semana = wday(horario_salida, label = TRUE)) %>% 
  ggplot(aes(x = dia_semana)) +
    geom_bar()
```

<img src="datetimes_files/figure-html/unnamed-chunk-18-1.png" width="70%" style="display: block; margin: auto;" />

Hay un patrón interesante si miramos a la demora promedio por minuto dentro de la hora. ¡Parece que los vuelos que salen en los minutos 20-30 y 50-30 tienen mucho más demora que en el resto de las horas!


```r
vuelos_dt %>% 
  mutate(minuto = minute(horario_salida)) %>% 
  group_by(minuto) %>% 
  summarise(
    atraso_promedio = mean(atraso_llegada, na.rm = TRUE),
    n = n()) %>% 
  ggplot(aes(minuto, atraso_promedio)) +
    geom_line()
```

<img src="datetimes_files/figure-html/unnamed-chunk-19-1.png" width="70%" style="display: block; margin: auto;" />

De forma interesante, si miramos al horario _programado_ de salida, no vemos un patrón tan prominente:


```r
  salida_programada <- vuelos_dt %>% 
    mutate(minuto = minute(salida_programada)) %>% 
    group_by(minuto) %>% 
    summarise(
      atraso_promedio = mean(atraso_llegada, na.rm = TRUE),
      n = n())
  
  ggplot(salida_programada, aes(minuto, atraso_promedio)) +
    geom_line()
```

<img src="datetimes_files/figure-html/unnamed-chunk-20-1.png" width="70%" style="display: block; margin: auto;" />

Entonces ¿por qué vemos ese patrón con los horarios reales de salida? Bueno, como muchos datos recolectados por los humanos, hay un sesgo importante hacia los vuelos que salen en horas "lindas". ¡Mantente siempre alerta respecto a este tipo de patrón, cuando sea que trabajes con datos que involucran al juicio humano!



```r
ggplot(salida_programada, aes(minuto, n)) +
  geom_line()
```

<img src="datetimes_files/figure-html/unnamed-chunk-21-1.png" width="70%" style="display: block; margin: auto;" />

### Redondeo

Un método alternativo para graficar los componentes individuales es redondear la fecha a una unidad de tiempo cercana, con `floor_date()` (del inglés, _redondear fecha hacia abajo_), `round_date()` (del inglés, _redondear fecha_), y `ceiling_date()` (del inglés, _redondear fecha hacia arriba_). Cada función toma un vector de fechas para ajustarlas, y luego el nombre de la unidad redondeada hacia abajo (con _floor_), hacia arriba (con _ceiling_), o al mas cercano. Esto, por ejemplo, nos permite graficiar el número de vuelos por semana:


```r
vuelos_dt %>% 
  count(semana = floor_date(horario_salida, "week")) %>% 
  ggplot(aes(semana, n)) +
    geom_line()
```

<img src="datetimes_files/figure-html/unnamed-chunk-22-1.png" width="70%" style="display: block; margin: auto;" />

Calcular la diferencia entre una fecha redondeada y una sin redondear puede ser particularmente útil.

### Configurando componentes

También puedes usar las funciones de acceso para darle un valor a los componentes de las fechas/horas:


```r
(fechahora <- ymd_hms("2016-07-08 12:34:56"))
#> [1] "2016-07-08 12:34:56 UTC"

year(fechahora) <- 2020
fechahora
#> [1] "2020-07-08 12:34:56 UTC"
month(fechahora) <- 01
fechahora
#> [1] "2020-01-08 12:34:56 UTC"
hour(fechahora) <- hour(fechahora) + 1
fechahora
#> [1] "2020-01-08 13:34:56 UTC"
```

Alternativamente, en lugar de modificar en un sólo lugar, puedes crear una nueva fecha-hora con `update()` (del inglés, _actualizar_). Esto también te permite configurar múltiples valores al mismo tiempo.


```r
update(fechahora, year = 2020, month = 2, mday = 2, hour = 2)
#> [1] "2020-02-02 02:34:56 UTC"
```

Si los valores son demasiado grandes, darán la vuelta:


```r
ymd("2015-02-01") %>% update(mday = 30)
#> [1] "2015-03-02"
ymd("2015-02-01") %>% update(hour = 400)
#> [1] "2015-02-17 16:00:00 UTC"
```

Puedes usar `update()` para mostrar la distribución de los vuelos a lo largo del día para cada día del año: 


```r
vuelos_dt %>% 
  mutate(horario_salida = update(horario_salida, yday = 1)) %>% 
  ggplot(aes(horario_salida)) +
    geom_freqpoly(binwidth = 300)
```

<img src="datetimes_files/figure-html/unnamed-chunk-26-1.png" width="70%" style="display: block; margin: auto;" />

Fijar los componentes más grandes de una fecha con una constante es una técnica que te permite explorar patrones en los componentes más pequeños.


### Ejercicios

1. ¿Cómo cambia la distribución de las horas de los vuelos dentro de un día, a lo largo del año?
    
1. Compara `horario_salida`, `salida_programada` and `atraso_salida`. ¿Son consistentes? Explica tus hallazgos.

1. Compara `tiempo_vuelo` con la duración entre la salida y la llegada. Explica tus hallazgos. (Pista: considera la ubicación del aeropuerto).

1. ¿Cómo cambia la demora promedio durante el curso de un día? ¿Deberías usar `horario_salida` o `salida_programada`? ¿Por qué?

1. ¿En qué día de la semana deberías salir, si quieres minimizar las posibilidades de una demora?

1. ¿Qué hace que la distribución de `diamantes$carat` y `vuelos$salida_programada` sean similares? 

1. Confirma mi hipótesis de que salidas programas en los minutos 20-30 y 50-60 están casuadas por los vuelos programados que salen más temprano. Pista: crea una variable binaria que te diga si un vuelo tuvo o no demora.


## Plazos de tiempo

Ahora, aprenderás cómo trabaja la aritmética con fechas, incluyendo la sustracción, adición y división. En el camino, aprenderás sobre tres importantes clases que representan períodos de tiempo:

* __durations__ (del inglés, _duraciones_), que representa un número exacto de segundos.
* __periods__ (del inglés, _períodos_), que representan unidades humanas como semanas o meses.
* __intervals__ (del inglés, _intervalos_), que representan un punto de inicio y uno de finalización.

### Duraciones

En R, cuando restas dos fechas, obtienes un objeto de diferencia temporal (en inglés, _difftimes_):


```r
# ¿Cuán viejo es Hadley?
edad_h <- today() - ymd(19791014)
edad_h
#> Time difference of 14213 days
```

Un objeto de clase _difftime_ registra un período de tiempo se segundos, minutos, horas, días o semanas. Esta ambiguedad hacia que los _difftimes_ sean un poco complicados de trabjar, por lo que __lubridate__ provee una alternativa que siempre usa segundos: la __duration__.


```r
as.duration(edad_h)
#> [1] "1228003200s (~38.91 years)"
```

Las duraciones traen un conveniente grupo de constructores:


```r
dseconds(15)
#> [1] "15s"
dminutes(10)
#> [1] "600s (~10 minutes)"
dhours(c(12, 24))
#> [1] "43200s (~12 hours)" "86400s (~1 days)"
ddays(0:5)
#> [1] "0s"                "86400s (~1 days)"  "172800s (~2 days)"
#> [4] "259200s (~3 days)" "345600s (~4 days)" "432000s (~5 days)"
dweeks(3)
#> [1] "1814400s (~3 weeks)"
dyears(1)
#> [1] "31536000s (~52.14 weeks)"
```

Las duraciones siempre registran el período de tiempo en segundos. Las unidades más grandes se crean al convertir minutos, horas, días, semanas y años a segundos, mediante una conversión estándar (60 segundos en un minuto, 60 minutos en una hora, 24 horas en un día, 7 días en una semana, 365 días en un año).

Puedes agregar y multiplicar duraciones:


```r
2 * dyears(1)
#> [1] "63072000s (~2 years)"
dyears(1) + dweeks(12) + dhours(15)
#> [1] "38847600s (~1.23 years)"
```

Puedes sumar y restar duraciones a días:


```r
ayer <- today() + ddays(1)
anio_pasado <- today() - dyears(1)
```

Sin embargo, como las duraciones representan un número exacto de segundos, a veces puedes obtener un resultado inesperado:


```r
una_pm <- ymd_hms("2016-03-12 13:00:00", tz = "America/New_York")

una_pm
#> [1] "2016-03-12 13:00:00 EST"
una_pm + ddays(1)
#> [1] "2016-03-13 14:00:00 EDT"
```

¿¡Por qué un día antes de la 1PM del 12 de Marzo, resulta ser las 2PM del 13 de Marzo!? Si miras con cuidado a la fecha, te darás cuenta que los husos horarios han cambiado. Debido al horario de verano (DST, o EDT para el horario de verano de la costa Este), el 12 de Marzo sólo tiene 23 horas, por lo que si agregamos un día entero de segundos, terminamos con una hora diferente.


### Períodos

Para resolver este problema, __lubridate__ provee a __periods__ (del inglés, _períodos_). Estos son plazos de tiempo que no tienen un largo fijo en segundos, sino que funcionan con tiempos "humanos", como días o meses. Esto les permite trabajar en una forma más intuitiva:


```r
una_pm
#> [1] "2016-03-12 13:00:00 EST"
una_pm + days(1)
#> [1] "2016-03-13 13:00:00 EDT"
```

Como las duraciones, los períodos pueden ser creados mediante un número de funciones constructoras amigables.


```r
seconds(15)
#> [1] "15S"
minutes(10)
#> [1] "10M 0S"
hours(c(12, 24))
#> [1] "12H 0M 0S" "24H 0M 0S"
days(7)
#> [1] "7d 0H 0M 0S"
months(1:6)
#> [1] "1m 0d 0H 0M 0S" "2m 0d 0H 0M 0S" "3m 0d 0H 0M 0S" "4m 0d 0H 0M 0S"
#> [5] "5m 0d 0H 0M 0S" "6m 0d 0H 0M 0S"
weeks(3)
#> [1] "21d 0H 0M 0S"
years(1)
#> [1] "1y 0m 0d 0H 0M 0S"
```

Puedes sumar y multiplicar períodos: 


```r
10 * (months(6) + days(1))
#> [1] "60m 10d 0H 0M 0S"
days(50) + hours(25) + minutes(2)
#> [1] "50d 25H 2M 0S"
```

Y, por supuesto, puedes sumarlos a las fechas. Comparados a las duraciones, los períodos son más propensos a hacer lo que esperas que hagan:



```r
# Un año bisiesto
ymd("2016-01-01") + dyears(1)
#> [1] "2016-12-31"
ymd("2016-01-01") + years(1)
#> [1] "2017-01-01"

# Horarios de verano
una_pm + ddays(1)
#> [1] "2016-03-13 14:00:00 EDT"
una_pm + days(1)
#> [1] "2016-03-13 13:00:00 EDT"
```

Usemos los períodos para arreglar una rareza relacionada a nuestras fechas de vuelos. Algunos aviones parecen arrivar a su destino _antes_ de salir de la ciudad de Nueva York.


```r
vuelos_dt %>% 
  filter(horario_llegada < horario_salida) 
#> # A tibble: 10,633 x 9
#>   origen destino atraso_salida atraso_llegada horario_salida     
#>   <chr>  <chr>           <dbl>          <dbl> <dttm>             
#> 1 EWR    BQN                 9             -4 2013-01-01 19:29:00
#> 2 JFK    DFW                59             NA 2013-01-01 19:39:00
#> 3 EWR    TPA                -2              9 2013-01-01 20:58:00
#> 4 EWR    SJU                -6            -12 2013-01-01 21:02:00
#> 5 EWR    SFO                11            -14 2013-01-01 21:08:00
#> 6 LGA    FLL               -10             -2 2013-01-01 21:20:00
#> # ... with 1.063e+04 more rows, and 4 more variables:
#> #   horario_llegada <dttm>, salida_programada <dttm>,
#> #   llegada_programada <dttm>, tiempo_vuelo <dbl>
```

Estos son vuelos nocturnos. Usamos la misma información de fecha para los horarios de salida y llegada, pero estos vuelos llegaron en el día siguiente. Podemos arreglarlo al sumar `days(1)` a la fecha de llegada de cada vuelo nocturno.


```r
vuelos_dt <- vuelos_dt %>% 
  mutate(
    nocturno = horario_llegada < horario_salida,
    horario_llegada = horario_llegada + days(nocturno * 1),
    llegada_programada = llegada_programada + days(nocturno * 1)
  )
```

Ahora todos los vuelos obedecen a las leyes de la física.


```r
vuelos_dt %>% 
  filter(nocturno, horario_llegada < horario_salida) 
#> # A tibble: 0 x 10
#> # ... with 10 variables: origen <chr>, destino <chr>, atraso_salida <dbl>,
#> #   atraso_llegada <dbl>, horario_salida <dttm>, horario_llegada <dttm>,
#> #   salida_programada <dttm>, llegada_programada <dttm>,
#> #   tiempo_vuelo <dbl>, nocturno <lgl>
```

### Intervalos

Resulta obvio lo que `dyears(1) / ddays(365)` debería retornar: uno, porque las duraciones siempre se representan por un número de segundos, y la duración de un año se define como 365 días convertidos a segundos.

¿Qué debería devolver `years(1) / days(1)`? Bueno, si el año fuera 2015 debería retornar 365, ¡pero si fuera 2016 debería retornar 366! No hay suficiente información para que __lubridate__ nos de una sóla respuesta sencilla. Entonces, lo que hace es darnos una estimación, con una advertencia:


```r
years(1) / days(1)
#> estimate only: convert to intervals for accuracy
#> [1] 365
```

Si quieres una medida más precisa, tendrás que usar un __interval__ (del inglés, _intervalo_). Un intervalo es una duración con un punto de partida: eso lo hace preciso, porque puedes determinar exactamente cuán largo es:


```r
siguiente_anio <- today() + years(1)
(today() %--% siguiente_anio) / ddays(1)
#> [1] 365
```

Para encontrar cuántos períodos caen dentro de un intervalo, tienes que usar la división entera:


```r
(today() %--% siguiente_anio) %/% days(1)
#> Note: method with signature 'Timespan#Timespan' chosen for function '%/%',
#>  target signature 'Interval#Period'.
#>  "Interval#ANY", "ANY#Period" would also be valid
#> [1] 365
```

### Resumen

¿Cómo eliges entre duraciones, períodos e intervalos? Como siempre, selecciona la estructura de datos más sencilla que resuelva tu problema. Si sólo necesitas tiempo físico, usa una duración; si necesitas agregar tiempos humanos, usa un período; si tienes que deducir cuán largo es un plazo en unidades humanas, usa un intervalo.

La figura \@ref(fig:dt-algebra) resume las operaciones artiméticas permitidas entre los tipos de datos.

<div class="figure" style="text-align: center">
<img src="diagrams/datetimes-arithmetic.png" alt="Las operaciones artiméticas permitidas entre pares de clases fecha/hora." width="70%" />
<p class="caption">(\#fig:dt-algebra)Las operaciones artiméticas permitidas entre pares de clases fecha/hora.</p>
</div>

### Ejercicios

1. ¿Por qué hay `months()` pero no `dmonths()` (del inglés, _días del mes_)?

1. Explica `days(nocturno * 1)` a alguien que apenas comienza a aprender R. ¿Cómo funciona?

1. Crea un vector de fechas dando el primer día de cada mes en 2015. Crea un vector de fechas dando el primer día de cada mes del año _actual_.

1. Crea una función en la cual, dado tu cumpleaños (como una fecha), retorne cuán viejo eres en años.

1. ¿Por qué no puedes hacer funcionar a `(today() %--% (today() + years(1)) / months(1)` ?

## Husos horarios

Los husos horarios son un tema enormemente complicado, debido a su interacción con entidades geopolíticas. Afortunadamente, no necesitamos excarbar en todos los detalles, ya que no son tan necesarios para el análisis de datos, pero hay algunos desafíos sobre los que tenemos que trabajar.

El primer desafío es que los nombres comunes de los husos horarios tienden a ser ambiguos. Por ejemplo, si eres Americano probablemente estés familiarizado con EST, (siglas en inglés de _Tiempo Este Estándar_). Sin embargo, ¡Canadá y Australia también tienen EST! Para evitar la confusión, R usa el estándar internacional IANA para husos horarios. Estos tienen un esquema de nombres que sigue el formato "<área>/<ubicación>", típicamente escrito como "\<continente\>/\<ciudad\>", en idioma inglés (hay algunas pocas excepciones porque no todos los países están ubicados en un continente). Ejemplos incluyen: "America/New_York", "Europe/Paris", and "Pacific/Auckland".

Puedes preguntarte por qué un huso horario usa una ciudad, cuando típicamente piensas en ellos como asociados a un país, o a una región dentro de un país. Esto es porque la base de datos de IANA tiene que registrar décadas de reglamentos sobre husos horarios. En el curso de las décadas, los países cambian nombres (o desaparecen) de forma bastante frecuente, pero los nombres de las ciudades tienden a mantenerse igual. Otro problema es que los nombres tienen que reflejar no sólo el comportamiento actual, sino también la historia completa. Por ejemplo, hay husos horarios tanto para "America/New_York" como para "America/Detroit". Actualmente, ambas ciudades usan el EST, pero entre 1969 y 1972, Michigan (el estado en el cual Detroit está ubicado), no empleaba el horario de verano, así que necesita un nombre diferente. ¡Vale mucho leer la base de datos cruda (disponible en <http://www.iana.org/time-zones>) sólo para enterarse de algunas de estas historias!

Puedes encontrar cuál es tu huso horario actual para R, usando `Sys.timezone()` (del inglés, _sistema, huso horario_):


```r
Sys.timezone()
#> [1] "UTC"
```

(Si R no lo sabe, obtendrás un `NA`.)

Y puedes ver la lista completa de todos los husos horarios con `OlsonNames()`:


```r
length(OlsonNames())
#> [1] 606
head(OlsonNames())
#> [1] "Africa/Abidjan"     "Africa/Accra"       "Africa/Addis_Ababa"
#> [4] "Africa/Algiers"     "Africa/Asmara"      "Africa/Asmera"
```

En R, el huso horario es un atributo de la fecha-hora (`date-time`) que sólo controla la impresión. Por ejemplo, estos tres objetos representan el mismo instante en el tiempo:


```r
(x1 <- ymd_hms("2015-06-01 12:00:00", tz = "America/New_York"))
#> [1] "2015-06-01 12:00:00 EDT"
(x2 <- ymd_hms("2015-06-01 18:00:00", tz = "Europe/Copenhagen"))
#> [1] "2015-06-01 18:00:00 CEST"
(x3 <- ymd_hms("2015-06-02 04:00:00", tz = "Pacific/Auckland"))
#> [1] "2015-06-02 04:00:00 NZST"
```

Puedes verificar que son lo mismo al usar una resta:


```r
x1 - x2
#> Time difference of 0 secs
x1 - x3
#> Time difference of 0 secs
```

Excepto que sea especificado claramente, __lubridate__ siempre usa UTC. UTC significa _Tiempo Universal Coordinado_ (por las siglas en inglés), y es el huso horario estándar empleado por la comunidad científica, y es aproximadamente equivalente a su predecesor GMT (_Tiempo del Meridiano de Greenwich_, por las siglas en inglés). No tiene horario de verano, por lo que resulta una representación conveniente para la computación. Las operaciones que combinan fechas y horas, como `c()`, a menudo descartan el huso horario. En ese caso, las fechas y horas se muestran en tu huso local:


```r
x4 <- c(x1, x2, x3)
x4
#> [1] "2015-06-01 12:00:00 EDT" "2015-06-01 12:00:00 EDT"
#> [3] "2015-06-01 12:00:00 EDT"
```

Puedes cambiar el huso horario de dos formas:

* Mantener el instante en el tiempo igual, y cambiar sólo cómo se representa. Usa esto cuando el instante es correcto, pero quieres una visualización más natural.
  
    
    ```r
    x4a <- with_tz(x4, tzone = "Australia/Lord_Howe")
    x4a
    #> [1] "2015-06-02 02:30:00 +1030" "2015-06-02 02:30:00 +1030"
    #> [3] "2015-06-02 02:30:00 +1030"
    x4a - x4
    #> Time differences in secs
    #> [1] 0 0 0
    ```
    
    (Esto también ilustra otro desafío de los husos horarios: ¡no todos los desfasajes son horas como números enteros!)

* Cambia el instante en el tiempo. Usa esto cuando tienes un instante que ha sido etiquetado con un huso horario incorrecto, y necesitas arreglarlo.

    
    ```r
    x4b <- force_tz(x4, tzone = "Australia/Lord_Howe")
    x4b
    #> [1] "2015-06-01 12:00:00 +1030" "2015-06-01 12:00:00 +1030"
    #> [3] "2015-06-01 12:00:00 +1030"
    x4b - x4
    #> Time differences in hours
    #> [1] -14.5 -14.5 -14.5
    ```
