---
title: Ejercicios de Limites
date: 2024-05-06 21:04:10 +/-TTTT
categories: [Wolfram 1]
tags: [wolfram, matemáticas, cálculo, facíl]     # TAG names should always be lowercase
math: true

image:
  path: /assets/img/Posts/conociendo_wolfram/wolfram1_1.jpg
  alt: Ejercicios de Limites
---

El dia de hoy toca aprender limites e intentare aplicar el metodo de aprender haciendo, la idea es que tengas algun conocimiento previo de los limites y conocimientos basicos de wolfram mathematica. Para resolver los ejercicios intenta hacer lo que se te ocurra y si no se te ocurre nada significa que te falta estudiar lo previo asi que vuelve a estudiar de nuevo hasta que se te ocurra como hacerlo. Una vez que se te ocurrio como hacerlo trata de hacerlo por tu cuenta y si te sale a la primera, fantastico, pero si te quedas trabado puedes revisar la solucion al ejercicio que te presento en este post. Sin mas que comentar partamos.

## Ejercicios:

1) En cuales de los siguientes casos existe el límite en x=0:

a.) f(x)=1/x

b.) f(x)=Sin[x]/x

c.) f(x)=Sin[1/x]

Solucion: A priori ninguna funcion parece cumplir con la condicion ya que todas las funciones estan restringidas por el denominador, pero veamos la grafica de cada una de ellas.

##### Grafica de a): 

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/grafica_a.JPG" alt="" >
</div>

Como se aprecia no existe un limite ya que al aproximarnos a 0 por ambos laterales llegamos a valores de f(x) muy diferentes, por arriba llegamos a mas infinito y abajo a menos infinito. Esto lo podemos comprobar con el comando limit:

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/comprobaciona.JPG" alt="" >
</div>

##### Grafica de b): 

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/grafica_b.JPG" alt="" >
</div>

En este caso podemos ver claramente que existe un limite ya que que por ambos lados obtenemos 1 al acercarnos al 0. Esto lo podemos corroborar con el comando limit:

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/comprobacionb.JPG" alt="" >
</div>


##### Grafica de c): 

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/grafica_c.JPG" alt="" >
</div>

Por simple inspeccion a esta distancia no se alcanza a distinguir si existe limite en el 0 para la funcion C, asi que haremos un zoom en la vecindad de 0.

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/grafica_c2.JPG" alt="" >
</div>

De esta imagen tampoco podemos concluir nada ya que el procesador del computador tiene problemas al hacer calculos con numeros decimales tan pequenos por lo que tira resultados con muchos errores y al graficarlos solo se ve ruido.

Por lo tanto de las graficas solo podemos concluir de que el limite en 0 solo existe en la grafica b. Dado que el comando limit hace uso del mismo procesador tampoco obtendremos resultados concluyentes como se observa a continuacion:

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/comprobacionc.JPG" alt="" >
</div>

Entonces diremos que en c el limite tampoco existe porque a medida que nos acercamos al 0 el valor del seno se hace cada vez mas grande de manera no lineal por lo tanto no predecible ya que tiende a infinito, lo que conduce a que no exista limite en 0.

2) Sea  f(x) = Sqrt[x^3-2] y g(x) =x^2+2 x+1,  calcule lím x->3 f(x) menos lím x->3 g(x) y luego el lím x->3 f(x) - g(x):

Este es un ejercicio bastante simple que lo que busca es que veamos como funciona al algebra de limites. Primero calculamos el primer resultado que nos dan:

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/calculo1.JPG" alt="" >
</div>

Y ahora calculamos el segundo enunciado:

<div style="text-align:center">
    <img src="/assets/img/Posts/Ejercicios de Limites/calculo2.JPG" alt="" >
</div>

Como vemos ambos resultados dan lo mismo por lo tanto el algebra de limites tambien es valido en wolfram mathematica.

Esos han sido los resultados por ahora, espero que te hayan servido.





