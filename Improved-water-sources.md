**Población que utiliza fuentes mejoradas de abastecimiento de agua
potable**

Utilizaremos los paquetes survey para el analisis de encuestas y el
conjunto de paquetes Tidyverse para el manejo de los datos.

``` r
library(survey)
```

    ## Warning: package 'survey' was built under R version 4.0.3

``` r
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.0.3

Descargamos la base de datos “Hogares” de la ENHOGAR, en el siguiente
enlace:
<a href="https://www.one.gob.do/recursos-automatizados/bases-de-datos" class="uri">https://www.one.gob.do/recursos-automatizados/bases-de-datos</a>

``` r
enhogar18 <- read.csv("Hogares_ENH18.csv") # Cargamos la base de datos
```

Acontinuacion seleccinamos y renombramos las variables a utilizar.

``` r
BD <- enhogar18 %>%
  select(Region, HPROVI, HZONA, H109, Factor_expansion) %>% 
    rename(provincia   = HPROVI,
           zona        = HZONA,
           fuente_agua = H109)
```

Omitimos los valores perdidos.

``` r
BD <- na.omit(BD)
```

Recodificamos los valores de acuerdo con el diccionario de la encuesta.
ver:<a href="https://www.one.gob.do/catalogo-datos/datasets/ENHOGAR/ENHOGAR-2018-Base-SPSS-PUB/Hogares_ENH2018.htm" class="uri">https://www.one.gob.do/catalogo-datos/datasets/ENHOGAR/ENHOGAR-2018-Base-SPSS-PUB/Hogares_ENH2018.htm</a>

``` r
# Recodificacion de Regiones

BD$Region <- recode(BD$Region,             
                    `1` = "Cibao Norte",
                    `2` = "Cibao Sur",
                    `3` = "Cibao Nordeste",
                    `4` = "Cibao Noroeste",
                    `5` = "Valdesia",
                    `6` = "Enriquillo",
                    `7` = "El Valle",
                    `8` = "Del Yuma",
                    `9` = "Higuamo",
                    `10` = "Metropolitana")



# Recodificacion de Provincias

BD$provincia <- recode(BD$provincia,       
                       `1` = "Distrito Nacional",
                    `2` = "Azua",
                    `3` = "Bahoruco",
                    `4` = "Barahona",
                    `5` = "Dajabón",
                    `6` = "Duarte",
                    `7` = "Elías Piña",
                    `8` = "El Seibo",
                    `9` = "Espaillat",
                    `10` = "Independencia",
                     `11` = "La Altagracia",
                    `12` = "La Romana",
                    `13` = "La Vega",
                    `14` = "María Trinidad Sánchez",
                    `15` = "Monte Cristi",
                    `16` = "Pedernales",
                    `17` = "Peravia",
                    `18` = "Puerto Plata",
                    `19` = "Hermanas Mirabal",
                    `20` = "Samaná",
                     `21` = "San Cristóbal",
                    `22` = "San Juan",
                    `23` = "San Pedro de Macorís",
                    `24` = "Sánchez Ramírez",
                    `25` = "Santiago",
                    `26` = "Santiago Rodríguez",
                    `27` = "Valverde",
                    `28` = "Monseñor Nouel",
                    `29` = "Monte Plata",
                    `30` = "Hato Mayor",
                    `31` = "San José de Ocoa",
                    `32` = "Santo Domingo")


# Recodificacion los valores de Zona de residencia

BD$zona <- recode(BD$zona,               
                  `1` = "Urbano", 
                  `2`= "Rural") 


# Recodificacion los valores de las fuentes de agua

BD$fuente_agua <- recode(BD$fuente_agua,
                          `1` = "Del acueducto dentro de la vivienda",
                    `2` = "Del acueducto en el patio de la vivienda",
                    `3` = "De una llave de otra vivienda",
                    `4` = "Del acueducto pero de una llave pública",
                    `5` = "Manantial, rio, arroyo, canal",
                    `6` = "Pozo",
                    `7` = "Lluvia",
                    `8` = "De camión tanque",
                    `96` = "Otro",
                    `99` = "Sin información",)
```

Segmentamos entre las fuentes mejoradas, las no mejoradas y otras
fuentes. Metodologia
CEPALSTATS:<a href="https://cepalstat-prod.cepal.org/cepalstat/tabulador/SisGen_MuestraFicha_puntual.asp?id_aplicacion=1&amp;id_estudio=1&amp;indicador=97&amp;idioma=e" class="uri">https://cepalstat-prod.cepal.org/cepalstat/tabulador/SisGen_MuestraFicha_puntual.asp?id_aplicacion=1&amp;id_estudio=1&amp;indicador=97&amp;idioma=e</a>

``` r
BD$estado_fuente <- recode(BD$fuente_agua,
                           `Del acueducto dentro de la vivienda` = "Mejorada",
                           `Del acueducto en el patio de la vivienda` = "Mejorada",
                           `De una llave de otra vivienda` = "Mejorada",
                           `Del acueducto pero de una llave pública` = "Mejorada",
                           `Manantial, rio, arroyo, canal` = "No Mejorada",
                           `Pozo` = "Mejorada",
                           `Lluvia` = "Mejorada",
                           `De camión tanque` = "No Mejorada")
```

A continuacion calculamos la poblacion con acceso a fuentes mejoradas

``` r
# Especificamos el diseño de la muestra con la funcion svydesign del paquete survey 

ds <- svydesign(data = BD, ids = ~1, weights = BD$Factor_expansion)

# Extraemos los resultados

options(scipen = 999, digits = 2)

# Extraemos una tabla con las cantidades por fuente y sus porcentajes.

  knitr::kable( svytable(~fuente_agua, design = ds)%>% 
  as.data.frame() %>%
  mutate(Proporción = Freq/sum(Freq)*100),
  caption = "Proporción de la población segun fuente de agua utilizada")
```

| fuente\_agua                             |     Freq|  Proporción|
|:-----------------------------------------|--------:|-----------:|
| De camión tanque                         |   163446|        4.92|
| De una llave de otra vivienda            |   171354|        5.16|
| Del acueducto dentro de la vivienda      |  1793669|       53.97|
| Del acueducto en el patio de la vivienda |   647171|       19.47|
| Del acueducto pero de una llave pública  |    29877|        0.90|
| Lluvia                                   |    19218|        0.58|
| Manantial, rio, arroyo, canal            |    86411|        2.60|
| Otro                                     |    15871|        0.48|
| Pozo                                     |   396559|       11.93|
| Sin información                          |       59|        0.00|

``` r
# Extraemos una tabla con la condición de la fuente de agua 

knitr::kable(svytable(~estado_fuente , design = ds)%>% 
  as.data.frame() %>%
  mutate(proporción = Freq/sum(Freq)*100), 
  caption = "Proporción de la población según condición de la fuente de agua utilizada")
```

| estado\_fuente  |     Freq|  proporción|
|:----------------|--------:|-----------:|
| Mejorada        |  3057848|       92.00|
| No Mejorada     |   249857|        7.52|
| Otro            |    15871|        0.48|
| Sin información |       59|        0.00|

``` r
db<- svytable(~zona+estado_fuente, design = ds)%>% 
  as.data.frame() 
```

``` r
options(scipen = 999, digits = 2)

db %>%
  group_by(zona) %>%
  mutate(p = 100*Freq/sum(Freq))%>%
  filter( estado_fuente %in% c("Mejorada","No Mejorada"))%>%
  ggplot(aes(x= zona, y= p, fill = estado_fuente))+
  geom_bar(stat = "identity", position = position_dodge()) +
  scale_fill_manual(values=c('#1E90FF','#DEB887'))+
  labs(title = "Poblacion con acceso a fuentes de agua, por tipo de fuente y zona",
       y= "Porcentaje (%)", x = "Zona de Residencia",
       fill = "Estado fuente",caption = "Fuente: ENHOGAR 2018 & JMP")+
  geom_text( aes(label= as.integer(p),vjust = -0.4), 
             position = position_dodge(width = 1))+
  theme_minimal()+
  theme(plot.caption = element_text(size=8),
        plot.title = element_text(face = "bold"))
```

<img src="Improved-water-sources_files/figure-markdown_github/unnamed-chunk-8-1.png" style="display: block; margin: auto;" />

``` r
# Estimamos la proporción de la población con acceso fuentes de agua mejoradas

 svyciprop(~estado_fuente=="Mejorada", ds, method="li")
```

    ##                                    2.5% 97.5%
    ## estado_fuente == "Mejorada" 0.920 0.917  0.92

``` r
# Estadistica de acceso a fuentes de agua mejorada por zona de residencia

 knitr::kable(svytable(~zona+estado_fuente, ds))
```

|        |  Mejorada|  No Mejorada|  Otro|  Sin información|
|--------|---------:|------------:|-----:|----------------:|
| Rural  |    506283|       110890|  7014|               59|
| Urbano |   2551564|       138967|  8857|                0|

``` r
# Estimamos la proporción de la población con acceso fuentes de agua mejoradas por zona

knitr::kable(svyby(~estado_fuente=="Mejorada",~zona,design=ds, svyciprop,method="li"))
```

|        | zona   |  estado\_fuente == “Mejorada”|  se.as.numeric(estado\_fuente == “Mejorada”)|
|--------|:-------|-----------------------------:|--------------------------------------------:|
| Rural  | Rural  |                          0.81|                                            0|
| Urbano | Urbano |                          0.95|                                            0|
