---
title: Factorizar N en RSA 
date: 2024-06-28 22:36:10 +/-TTTT
categories: [Criptografia]
tags: [Criptografia, matemáticas, rsa, medio]     # TAG names should always be lowercase
math: true

header-includes:
  - \usepackage[ruled,vlined,linesnumbered]{algorithm2e}

image:
  path: /assets/img/Posts/Factorizar N en RSA/RSA.png
  alt: RSA
---

El algortimo de cifrado RSA es ampliamente utilizado en aplicaciones de software de encriptado como lo son protocolos del stack TCP/IP,
garantizando la confidencialidad entre 2 partes, ademas tiene un modo de uso como firma digital para garantizar la autenticidad
de la informacion. En este articulo me centrare en explicar como factorizar el numero N conocidos e y d, este es un ataque conocido
desde la publicacion de RSA. Pero ademas agregare una optimizacion que descubri en el proceso que estudiaba el ataque.

## Trasfondo matematico:

La formula para calcular $d$ (clave privada) a partir de $e$ (clave publica):
<div style="text-align:center">
    $\Large d = e^{-1} \ mod \  \varphi (N)$ <br>
</div>
multiplicando por $e^{-1}$ a ambos lados: <br>
<div style="text-align:center">
    $\Large d \cdot e = 1 \ mod \  \varphi (N)$ <br>
</div>
Por la definicion de modulo se tiene: <br>
<div style="text-align:center">
    $\Large \varphi (N) \cdot a + 1 = d \cdot e \ , a \in \mathbb{Z}$ <br>
</div> 
<br>
Ademas el totiente de N $\varphi (N)$ siempre es par, por lo que $\varphi (N) \cdot a$ tambien siempre sera par. Para simplificar las cosas definamos $k = \varphi (N) \cdot a$. Ahora notemos que tenemos el siguiente teorema:
<br>

> **Teorema 1:** Todo numero par $k$ se puede escribir como: $\Large k = 2^{t} \cdot r \ , \ t \in \mathbb{Z}, t \geq 1 \ , \ r \ impar$
{: .prompt }
<br>
Ademas tenemos el teorema de euler que relaciona el resultado anterior con una nueva identidad.
<br>

> **Teorema 2 (euler):** $\Large \ g^{\varphi (N) \cdot a } = 1 \ mod \ N$
{: .prompt }
<br>

Entonces por el teorema 1 y 2 podemos plantear la siguiente identidad:
<div style="text-align:center">
    $\Large g^{2^{t} \cdot \ r} = 1 \ mod \ N$
    <br>
</div> 


Y nuestro objetivo sera aplicar raices cuadradas a ese resultado de la siguiente manera:
<br>
<div style="text-align:center;font-size: 20px;">
    $\Large x = g^{\frac{d \cdot e - 1}{2^{i}}} = 1 \ mod \ N \equiv$
    $\Large g^{\frac{k}{2^{i}}} = 1 \ mod \ N \equiv$
    $\Large g^{\frac{\varphi (N) \cdot a}{2^{i}}} = 1 \ mod \ N    \equiv$
    $\Large g^{\frac{2^{t} \cdot \ r}{2^{i}}} = 1 \ mod \ N$
</div> 


<br>
El algoritmo original plantea de que se escoja un $g$ aleatorio del conjunto $\set{2, .., N-1}$ y le saquemos la raiz cuadrada, es decir que lo elevemos a la mitad y alguna de esas raices sera la raiz cuadrada de 1 que nos revelara la factorizacion de N. Deberemos cambiar de g si ya no es posible seguir sacando raices cuadradas es decir que el exponente no es divisible por 2.
<div style="text-align:center">
    $\Large g^{\frac{d \cdot e - 1}{2^{i}}} = 1 \ mod \ N$<br>
</div> 
Cuando lleguemos a una nueva raiz podemos usar el teorema chino del resto que nos dice que podemos armar el siguiente sistema de ecuaciones:
<br>
<div style="text-align:center">
$$
\Large
\begin{cases}
    g^{\frac{d \cdot e - 1}{2^{i}}} = 1 \ mod \ p
    \\
    g^{\frac{d \cdot e - 1}{2^{i}}} = 1 \ mod \ q
\end{cases}
$$
</div> 

<br>
Y podemos obtener el menor de p y q mediante $\Large y = gcd(g^{\frac{d \cdot e - 1}{2^{i}}} \ - \ 1, N)$ y si $\Large \ y \geq  1$ significa que hemos encontrado nuestros factores. Y podemos calcularlos como:
<div style="text-align:center">
$\Large p = y \ , \ q = N \div p$
</div> 

Antes de hacer la implementacion en python veamos el siguiente pseudocodigo:
<div style="text-align:center">
    <img src="/assets/img/Posts/Factorizar N en RSA/pseudocodigo_original.JPG" alt="" >
</div>
El algoritmo tiene tiempo de ejecucion $\mathcal{O}(n^3)$ con $n=\log_{2} N$ .
## Implementacion:




