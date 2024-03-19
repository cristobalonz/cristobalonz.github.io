---
title: Elasticidad Económica en Wolfram Mathematica
date: 2024-03-05 23:01:05 +/-TTTT
categories: [Wolfram 1]
tags: [wolfram, matemáticas, cálculo, facíl]     # TAG names should always be lowercase
math: true

image:
  path: ./assets/img/Posts/elasticidad.jpg
  alt: Elasticidad economica
---

Este es el primer ejemplo que mostraré en la serie de Wolfram Mathematica y pretende servir de motivación para que tú, estudiante de este software, te animes a aventurarte por este bello mundo de los software computacionales. En el presente texto me enfocaré en definir lo que es la elasticidad económica y aplicarla en Wolfram Mathematica en un ejemplo donde se analiza un estudio de mercado:

La elasticidad económica es un concepto que viene del estudio de la física de los elásticos, que predice cuánta fuerza acumula un resorte (modelo simplificado de un elástico) al estirarlo o comprimirlo. En economía se tomaron estas ideas para predecir cuánta demanda tendrá un producto, bien o servicio si experimenta una variación en su precio, competencia, durabilidad, gustos del consumidor, etc. Para ello se hizo todo un desarrollo matemático que no entraré en detalle porque es lo que se aprende en un curso de cálculo 1 y no todos los lectores podrán tener esos conocimientos, pero sí diré que para calcular la elasticidad de demanda necesitamos sí o sí una función o un conjunto de puntos que relacionen el precio, competencia, durabilidad o gustos del consumidor con la demanda de un producto. Esta relación se puede modelar matemáticamente mediante:
<div style="text-align:center">
$$Demanda = D(Precio)$$
</div>

Donde $$D()$$ es la función que modela la demanda y recibe como entrada el $$Precio$$, entonces la elasticidad de demanda esta dada por la fórmula: 

<div style="text-align:center">
$$
\text{Elasticidad de demanda} = \frac{\text{Cambio porcentual en la cantidad demandada}}{\text{Cambio porcentual en el precio}} 
$$
</div>

> Si estás un poco confundido, no te preocupes. A continuación, vamos a aplicar todo a un ejemplo donde aprenderás cómo usar esos conceptos.
{: .prompt-warning }


Imagina que una empresa nos pidió hacer un estudio de mercado donde analizáramos la elasticidad de la demanda para 13 valores cualquiera de precio en el rango de 70USD a 100USD, con el fin de determinar qué precio debe tener el producto para aumentar al máximo las ganancias de la empresa. Después de un mes variando precios, obtuvimos el siguiente conjunto de datos tabulados:


| Precio USD | Unidades Vendidas |
|------------|-------------------|
| 70  | 166 |
| 72  | 138 |
| 75  | 121 |
| 77  | 110 |
| 80  | 101 |
| 82  | 115 |
| 85  | 118 |
| 87  | 100 |
| 90  | 90  |
| 92  | 85  |
| 95  | 78  |
| 97  | 63  |
| 100 | 52  |

A continuación se muestra el código Wolfram Script donde en las líneas 1 a 3 declaramos una tabla donde sus entradas están en el formato ```{Precio USD,Unidades Vendidas}```. Luego, en las líneas 5 en adelante, graficamos dicha tabla. La gráfica resultante se muestra en la Imagen 1.

```python
RelacionPrecioVentas = { {70, 166}, {72, 138}, {75, 121}, {77, 
    110}, {80, 101}, {82, 115}, {85, 118}, {87, 100}, {90, 90}, {92, 
    85}, {95, 78}, {97, 63}, {100, 52} };

ListPlot[RelacionPrecioVentas, 
 AxesLabel -> {Precio USD, Unidades Vendidas}, 
 LabelStyle -> Directive[Blue, Bold]]
```
<center>Código 1</center>

<div style="text-align:center">
    <img src="./assets/img/Posts/grafica1.JPG" alt="Grafica del estudio de mercado" >
</div>
<center>Imagen 1</center>

A modo de prueba de concepto, calcularé la elasticidad de la demanda entre los puntos 80USD y 82USD. Para ello, utilizaremos la fórmula:

<div style="text-align:center">
$$
\text{Cambio porcentual en la cantidad demandada} = \frac{115 -101}{101} = 0.139 = 13.9\%
$$
$$
\text{Cambio porcentual en el precio} = \frac{82 - 80}{80} = 0.025 = 2.5\%
$$
$$
\text{Elasticidad de demanda} = \frac{13.9\%}{2.5\%} = 5.56
$$
</div>

Por lo tanto, se puede concluir que la elasticidad de la demanda es mayor a 1, lo que quiere decir que la demanda es elástica y la cantidad de demanda cambia más rápido que el precio. Por lo tanto, si el precio se va a fijar entre 80USD y 82USD, debería quedarse en 80USD para aumentar los ingresos totales.

Pero como dije al principio, esto es solo una prueba de concepto y para determinar el precio final que debe tener el producto, debemos hacer un análisis elástico para todos los intervalos de la gráfica. Para ello, podemos usar el siguiente código en Wolfram:

```python
elasticidades = {};
For[i = 1, i <= Length[RelacionPrecioVentas] - 1, i++,
  PrecioAntes =  RelacionPrecioVentas[[i, 1]];
  PrecioDespues =  RelacionPrecioVentas[[i + 1, 1]];
  unidadesVendidasAntes =  RelacionPrecioVentas[[i, 2]];
  unidadesVendidasDespues =  RelacionPrecioVentas[[i + 1, 2]];
  cambioPrecio = (PrecioDespues - PrecioAntes) /PrecioAntes;
  cambioDemanda = (unidadesVendidasDespues - unidadesVendidasAntes )/
    unidadesVendidasAntes ;
  elasticidad = cambioDemanda /cambioPrecio ;
  intervalo = 
   StringTemplate["$`a`USD - $`b`USD"][<|"a" -> PrecioAntes, 
     "b" -> PrecioDespues|>];
  AppendTo[elasticidades, {N[elasticidad, 3], intervalo}];
  ];

elasticidades
```
<center>Código 2</center>
El resultado que arroja el código es la siguiente tabla:

```
{ {-5.90, "$70USD - $72USD"}, {-2.96, "$72USD - $75USD"}, {-3.41, 
  "$75USD - $77USD"}, {-2.10, "$77USD - $80USD"}, {5.54, 
  "$80USD - $82USD"}, {0.713, "$82USD - $85USD"}, {-6.48, 
  "$85USD - $87USD"}, {-2.90, "$87USD - $90USD"}, {-2.50, 
  "$90USD - $92USD"}, {-2.53, "$92USD - $95USD"}, {-9.13, 
  "$95USD - $97USD"}, {-5.65, "$97USD - $100USD"} }
```
<center>Código 3</center>

La cual tabulada se ve así:

|   elasticidad    |        Intervalo        |
|------------|-------------------------|
|   -5.90    |    70USD - 72USD       |
|   -2.96    |    72USD - 75USD       |
|   -3.41    |    75USD - 77USD       |
|   -2.10    |    77USD - 80USD       |
|    5.54    |    80USD - 82USD       |
|   0.713    |    82USD - 85USD       |
|   -6.48    |    85USD - 87USD       |
|   -2.90    |    87USD - 90USD       |
|   -2.50    |    90USD - 92USD       |
|   -2.53    |    92USD - 95USD       |
|   -9.13    |    95USD - 97USD       |
|   -5.65    |   97USD - 100USD       |

Por lo que le reportamos a la empresa que la demanda elástica ocurre solo en el intervalo: ```80USD - 82USD``` ya que solo en ese intervalo la elasticidad es mayor a 1, en consecuencia, el precio debe fijarse en 80USD para maximizar las ganancias.