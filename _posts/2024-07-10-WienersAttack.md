---
title: Ataque de Wieners 
date: 2024-07-10 9:41:10 +/-TTTT
categories: [Criptografia]
tags: [Criptografia, matemáticas, rsa, medio]     # TAG names should always be lowercase
math: true

header-includes:
  - \usepackage[ruled,vlined,linesnumbered]{algorithm2e}

image:
  path: /assets/img/Posts/Ataque_de_Wieners/portadaWieners.JPG
  alt: RSA
---

El ataque de Wieners nos permite obtener la clave privada d cuando $$\Large d < \frac{1}{3} \cdot \sqrt[4]{N}$$, esto en la practica sucede cuando la clave publica e es muy grande. La logica es la siguiente:
<br>
<br>

1) De la ecuacion que relaciona la clave publica con la clave privada se tiene la siguiente relacion:
<div style="text-align: center;">
 $\Large e \cdot d = k \cdot \phi (n) + 1$
</div>
<br>
2)Ademas:
<div style="text-align: center;">
 $\Large \phi (n) = (p-1)(q-1)$
 <br>
 $\Large \phi (n) = p \cdot q - (p+q) + 1$
</div>
 <br>
En este caso se tiene que $$p \cdot q = n$$ es el doble de largo en bits que $$(p+q)+1$$ por lo que podemos despreciar este ultimo termino quedandonos la siguiente igualdad:
<div style="text-align: center;">
 $\Large \phi (n) \approx n $
</div>
<br>
3)Tambien de la congruencia anterior que relacionaba las claves podemos observar lo siguiente:
<div style="text-align: center;">
 $\Large e \cdot d = k \cdot \phi (n) + 1$
<br>
 $\Large \frac{e}{\phi(n)} - \frac{k}{d} = \frac{1}{d \cdot \phi(n)}$
</div>
Como se puede ver el termino $$\Large \frac{1}{d \cdot \phi(n)}$$ sera muy cercano a 0 lo que indica que $$\Large \frac{e}{\phi(n)} - \frac{k}{d}$$ seran valores muy parecidos.

4)Usando fracciones continuas y las aproximaciones anteriores podemos darnos cuenta de que podemos aproximar $$\Large \frac{k}{d}$$ usando los valores conocidos de $$\Large \frac{e}{N}$$ Esta busqueda la podemos hacer como lo indica el siguiente algoritmo:

### Algoritmo para ataque de Wieners:
1) Generamos una lista de todos los cocientes k que se forman de dividir $$\frac{e}{N}$$ y el resto de la division lo usamos como el $$n$$ de la division siguiente y el valor que en la division actual era $$n$$ en la division siguiente sera $$e$$.

2) Vamos por cada cociente $$a_{i} \ para \ 0 \geq i \geq k$$ encontrado armando la siguiente secuencia de fracciones continuas:

<div style="text-align: center;">
 1) $\Large \frac{a_0}{1}$
 <br>
 2) $\Large a_0 + \frac{1}{a_1}$
 <br>
 3) $\Large a_0 + \frac{1}{a_1 + \frac{1}{a_2}}$
 <br>
 4) $\Large a_0 + \frac{1}{a_1 + \frac{1}{a_2 + \frac{1}{a_3}}}$
</div>
Se sigue asi hasta llegar al termino a_k.
Por cada paso se debe verificar que d sea impar y que k sea entero es decir $\Large e \cdot d \equiv 1 \ mod \ k$.

Si se cumplen ambas condiciones se deben encontrar las raices del polinomio $$x^2 - (N - \phi (N) + 1)x + N = 0$$ Si son enteras hemos encontrado la solucion de lo contrario debemos seguir buscando. Una herramienta interesante para resolver Wiener es la siguiente:

```python
import math
import random

############################
## Wiener's Attack module ##
############################

# Calculates bitlength


def bitlength(x):
    assert x >= 0
    n = 0
    while x > 0:
        n = n + 1
        x = x >> 1
    return n

# Squareroots an integer


def isqrt(n):
    if n < 0:
        raise ValueError('square root not defined for negative numbers')
    if n == 0:
        return 0
    a, b = divmod(bitlength(n), 2)
    x = 2**(a + b)
    while True:
        y = (x + n // x) // 2
        if y >= x:
            return x
        x = y

# Checks if an integer has a perfect square


def is_perfect_square(n):
    h = n & 0xF  # last hexadecimal "digit"
    if h > 9:
        return -1  # return immediately in 6 cases out of 16.
    # Take advantage of Boolean short-circuit evaluation
    if (h != 2 and h != 3 and h != 5 and h != 6 and h != 7 and h != 8):
        # take square root if you must
        t = isqrt(n)
        if t * t == n:
            return t
        else:
            return -1
    return -1

# Calculate a sequence of continued fractions


def partial_quotiens(x, y):
    partials = []
    while x != 1:
        partials.append(x // y)
        a = y
        b = x % y
        x = a
        y = b
    # print partials
    return partials

# Helper function for convergents


def indexed_convergent(sequence):
    i = len(sequence) - 1
    num = sequence[i]
    denom = 1
    while i > 0:
        i -= 1
        a = (sequence[i] * num) + denom
        b = num
        num = a
        denom = b
    #print (num, denom)
    return (num, denom)

# Calculate convergents of a  sequence of continued fractions


def convergents(sequence):
    c = []
    for i in range(1, len(sequence)):
        c.append(indexed_convergent(sequence[0:i]))
    # print c
    return c

# Calculate `phi(N)` from `e`, `d` and `k`


def phiN(e, d, k):
    return ((e * d) - 1) / k

# Wiener's attack, see http://en.wikipedia.org/wiki/Wiener%27s_attack for
# more information


def wiener_attack(N, e):
    (p, q, d) = (0, 0, 0)
    conv = convergents(partial_quotiens(e, N))
    for frac in conv:
        (k, d) = frac
        if k == 0:
            continue
        y = -(N - phiN(e, d, k) + 1)
        discr = y * y - 4 * N
        if(discr >= 0):
            # since we need an integer for our roots we need a perfect squared
            # discriminant
            sqr_discr = is_perfect_square(discr)
            # test if discr is positive and the roots are integers
            if sqr_discr != -1 and (-y + sqr_discr) % 2 == 0:
                p = ((-y + sqr_discr) / 2)
                q = ((-y - sqr_discr) / 2)
                return p, q, d
    return p, q, d
#Solo numeros de ejemplo
e = 0x0285F8D4FE29CE11605EDF221889ED2
n = 0x02AEB637F6152AFD4FB3A2DD165AEC9
p, q, d = wiener_attack(n, e)
print p
print q
print d
################################
## End Wiener's Attack module ##
################################
```