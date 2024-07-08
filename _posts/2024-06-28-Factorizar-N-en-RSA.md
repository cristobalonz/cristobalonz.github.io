---
title: Factorizar N en RSA con N, e, d
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
desde la publicacion de RSA. Pero ademas agregare una optimizacion que se le puede hacer al algoritmo.

## Trasfondo matematico:

El trasfondo matematico de este problema es bastante complejo y requiere un alto nivel de matematicas, hace uso del test de primalidad de miller-rabin y la funcion lambda 

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
    $\Large \varphi (N) \cdot a = d \cdot e \ - 1 , a \in \mathbb{Z}$ <br>
</div> 
<br>
Ademas el totiente de N $\varphi (N)$ siempre es par, por lo que $\varphi (N) \cdot a$ tambien siempre sera par. Para simplificar las cosas definamos $k = \varphi (N) \cdot a$. Ahora notemos que tenemos el siguiente teorema:
<br>

> **Teorema 1:** Todo numero par $k$ se puede escribir como: 
<div style="text-align: center;">
$\Large k = 2^{t} \cdot r \ , \ t \in \mathbb{Z}, t \geq 1 \ , \ r \ impar$
</div>
{: .prompt }
<br>
Ademas tenemos el teorema de euler que relaciona el resultado anterior con una nueva identidad.
<br>

> **Teorema 2 (euler):** Sea N un numero natural se tiene la siguiente relacion: <br> 
<div style="text-align: center;">
$\Large \ g^{\varphi (N) \cdot a } = 1 \ mod \ N$
</div>
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
El algoritmo original plantea de que se escoja un $g$ aleatorio del conjunto $\set{2, .., N-1}$ y le saquemos la raiz cuadrada, es decir que lo elevemos a la mitad y alguna de esas raices sera la raiz cuadrada de 1 que nos revelara la factorizacion de N con una probabilidad de 0.5 para cada valor escogido. Deberemos cambiar de g si ya no es posible seguir sacando raices cuadradas es decir que el exponente no es divisible por 2.
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
Y podemos obtener el menor de p y q mediante $\Large y = gcd(g^{\frac{d \cdot e - 1}{2^{i}}} \ - \ 1, N)$ y si $\Large \ y >  1 \ x >  1$ significa que hemos encontrado nuestros factores. Y podemos calcularlos como:
<div style="text-align:center">
$\Large p = y \ , \ q = N \div p$
</div> 

Antes de hacer la implementacion en python veamos el siguiente pseudocodigo:
<div style="text-align:center">
    <img src="/assets/img/Posts/Factorizar N en RSA/pseudocodigo_original.JPG" alt="" >
</div>
El algoritmo tiene tiempo de ejecucion $\mathcal{O}(n^3)$ con $n=\log_{2} N$ .
## Implementacion:

A continuacion se muestra la siguiente implementacion en python de la funcion descrita en el pseudocodigo

```python
import random, math

def Factorizador_de_N(N, d, e):
    k = d * e - 1

    #Primero verificamos que se 
    #pueda calcular
    if k % 2 == 1:
        print("Error: k debe ser par.")
        return

    while True:
        g = random.randint(2, N - 1)
        t = k
        while t % 2 == 0:
            t = t // 2
            x = pow(g, t, N)
            if x > 1 and math.gcd(x - 1, N) > 1:
                p = math.gcd(x - 1, N)
                q = N // p
                print(f"p: {p}, q: {q}")
                return
```

Verificando el codigo con numeros obtenidos de un cifrado RSA-2048 bit se tiene la siguiente salida:

```python
N=21711308225346315542706844618441565741046498277716979943478360598053144971379956916575370343448988601905854572029635846626259487297950305231661109855854947494209135205589258643517961521594924368498672064293208230802441077390193682958095111922082677813175804775628884377724377647428385841831277059274172982280545237765559969228707506857561215268491024097063920337721783673060530181637161577401589126558556182546896783307370517275046522704047385786111489447064794210010802761708615907245523492585896286374996088089317826162798278528296206977900274431829829206103227171839270887476436899494428371323874689055690729986771
e=0x10001
d=2734411677251148030723138005716109733838866545375527602018255159319631026653190783670493107936401603981429171880504360560494771017246468702902647370954220312452541342858747590576273775107870450853533717116684326976263006435733382045807971890762018747729574021057430331778033982359184838159747331236538501849965329264774927607570410347019418407451937875684373454982306923178403161216817237890962651214718831954215200637651103907209347900857824722653217179548148145687181377220544864521808230122730967452981435355334932104265488075777638608041325256776275200067541533022527964743478554948792578057708522350812154888097
Factorizador_de_N(N, d, e)
```

**Output**:
```bash
$python pq_reconstruct.py 
p: 161469718942256895682124261315253003309512855995894840701317251772156087404025170146631429756064534716206164807382734456438092732743677793224010769460318383691408352089793973150914149255603969984103815563896440419666191368964699279209687091969164697704779792586727943470780308857107052647197945528236341228473, q: 134460556242811604004061671529264401215233974442536870999694816691450423689575549530215841622090861571494882591368883283016107051686642467260643894947947473532769025695530343815260424314855023688439603651834585971233941772580950216838838690315383700689885536546289584980534945897919914730948196240662991266027
```

El tiempo que le tomo al programa fue menos de un segundo y este es el problema que se resuelve en librerias como pycrypto que entregan esto mismo a traves de funciones como: ```Crypto.PublicKey.RSA.construct(n, e, d)```




