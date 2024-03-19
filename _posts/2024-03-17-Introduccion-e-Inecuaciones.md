---
title: Conociendo Wolfram e Inecuaciones
date: 2024-03-17 00:04:05 +/-TTTT
categories: [Wolfram 1]
tags: [wolfram, matemáticas, cálculo, facíl]     # TAG names should always be lowercase
math: true

image:
  path: ./assets/img/Posts/conociendo_wolfram/wolfram1_1.JPG
  alt: Elasticidad economica
---

En este Post que es el segundo de la serie de Wolfram 1, veremos una introduccion a los comandos de wolfram mathematica para hacer cosas como graficar, resolver inecuaciones y resolver ecuaciones.

# Principales comandos

* N[fracción, cantidad de digitos]: este comando entrega el resultado numérico de una fracción. Donde la cantidad de dígitos se calcula usando la siguiente regla: 1 (1 dígito), 12 (2 dígitos), 1.2 (2 dígitos), 1.234 (4 dígitos).

* Plot[{funcion_1, funcion_2, .., funcion_n}, {x,xmin,xmax}, Atributo_1, Atributo_2, .., Atributo_n]: este comando nos permite graficar funciones de una variable y decorarlas de tantas maneras como Wolfram Mathematica lo permita. A continuación se grafican las funciones $$j=sin⁡(3x)+cos⁡(2x)$$ y $$j=0.5$$ pintado el área bajo la curva de ambas funciones.

<div style="text-align:center">
    <img src="./assets/img/Posts/conociendo_wolfram/imagen1.jpg" alt="Grafica del par de funciones" >
</div>
<center>Imagen 1</center>

* RegionPlot[{funcion_1, funcion_2, .., funcion_n}, {x,xmin,xmax},{y,ymin,ymax}]: sirve para graficar regiones completas donde se cumple una desigualdad para una función definida de forma implícita, a continuación se grafican las regiones para las cuales las siguientes inecuaciones son mayores a 0.
<div style="text-align:center">
    <img src="./assets/img/Posts/conociendo_wolfram/imagen2.jpg" alt="Grafica del par de regiones" >
</div>
<center>Imagen 2</center>

* Solve[ecuacion, x, Reals]: resuelve una ecuación en los números reales.

* Solve[ecuacion, x]: resuelve una ecuación en los complejos.

> Puedes cambiar Solve Por NSolve para recibir el valor numérico, pero funcionan de manera equivalente.
{: .block-tip }

* FindRoot[ecuacion,{x, numero de referencia}]: Encuentra la solución de la ecuación mas cercana al valor de referencia que le entreguemos.

<div style="text-align:center">
    <img src="./assets/img/Posts/conociendo_wolfram/imagen3.jpg" alt="Uso del comando FindRoot" >
</div>
<center>Imagen 3</center>

* Reduce[inecuacion, x]: resuelve una inecuación para la variable x.

* Simplify[Expresion, Element[x, Reals]]: Simplifica una expresión algebraica en funcion de x suponiendo que x es real. 

* RGBColor[rojo, verde, azul]: Define un color en el formato red, green and blue, es útil para combinarlo con Plot de la siguienete manera:  

```python
Plot[{Abs[x-2],x-2},{x,-3,3},PlotStyle->{RGBColor[1.000,0.000,0.000],RGBColor[0.000,1.000,0.000]}]
```
<center>Código 1</center>

* ConditionalExpression[expresión, condición]: grafica una función para condiciones especificas como que la función sea mayor a 0.