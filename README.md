# SRA_Stochastic_Resonance_Algorithm

## Introducción: El Problema del Viajante (TSP)

El **Travelling Salesman Problem (TSP)** es uno de los problemas más estudiados en la teoría de optimización combinatoria. En su forma más conocida, se trata de encontrar la ruta más corta que permita a un vendedor visitar un conjunto de ciudades una sola vez y regresar al punto de partida.

A pesar de su simplicidad aparente, el TSP es un problema **NP-hard**, lo que significa que no se conoce un algoritmo que lo resuelva de forma exacta en tiempo polinomial para cualquier número de nodos. Esto ha motivado el desarrollo de diversas aproximaciones, entre ellas:

- Algoritmos exactos (como Concorde)
- Métodos heurísticos clásicos (como Vecino Más Cercano o Inserción)
- Metaheurísticas avanzadas (Simulated Annealing, Algoritmos Genéticos, Ant Colony Optimization, entre otros)

El TSP tiene aplicaciones reales en planificación de rutas, logística, fabricación de circuitos integrados, secuenciación de ADN, mapeo de drones, exploración planetaria, organización de inventarios, y más. Su popularidad lo convierte en un campo ideal para explorar nuevas ideas de optimización inspiradas en la física, biología o sistemas dinámicos.

---

## Motivación: ¿Por qué un nuevo algoritmo?

En un intento de explorar nuevos enfoques conceptuales, se propuso modelar el espacio de soluciones del TSP como un **paisaje energético** no lineal, donde cada ruta candidata representa un estado del sistema. Esta perspectiva permite adaptar principios de la física no lineal, y en particular, el fenómeno de **resonancia estocástica**, que describe cómo un sistema puede amplificar señales débiles mediante la presencia de ruido.

Inspirado en este concepto, se desarrolló el algoritmo **SRA (Stochastic Resonance Algorithm)**, que introduce un ruido cuidadosamente controlado en el proceso de búsqueda para ayudar a escapar de mínimos locales y mejorar la calidad global de las soluciones. Este enfoque intenta equilibrar la exploración y la explotación de una manera dinámica y adaptativa.

---

## Fundamento físico y modelo matemático

La resonancia estocástica se ha estudiado en sistemas biológicos, electrónicos y neuronales. En nuestro contexto, la idea es simple: añadir ruido no de forma aleatoria, sino **resonante**, es decir, calibrado en función de una frecuencia característica del sistema.

La versión clásica de la fórmula para la amplitud del ruido es:

```
eta(t) = eta0 * exp( - ((|f(t) - f_r(t)| / sigma)^2) )
```

Donde:
- `eta(t)` es la amplitud del ruido en el tiempo `t`
- `eta0` es la amplitud máxima del ruido
- `f(t)` es la frecuencia actual del sistema (estimada por cambios en la energía)
- `f_r(t)` es la frecuencia de resonancia estimada (varía suavemente con el tiempo)
- `sigma` controla el ancho de banda del "filtro de resonancia"

Esta fórmula corresponde a una distribución gaussiana centrada en `f_r`, lo que hace que el ruido disminuya rápidamente a medida que `f(t)` se aleja de la frecuencia resonante.

---

## La extensión con parámetro de forma alpha

En el desarrollo del algoritmo se detectó una limitación importante: la gaussiana clásica es poco flexible. En fases tempranas de la búsqueda, cuando se necesita mayor exploración, su rápida caída impide que el ruido actúe. Por el contrario, en fases finales se necesita mayor precisión.

Para solventar esto, se introdujo una generalización del exponente cuadrático, usando un **parámetro de forma `alpha`** que permite controlar la pendiente de la curva. La fórmula extendida es:

```
eta(t) = eta0 * exp( - ((|f(t) - f_r(t)| / sigma)^alpha) )
```

- Si `alpha = 2`, se mantiene la versión gaussiana clásica
- Si `alpha < 2`, la función es más plana, lo que favorece la exploración
- Si `alpha > 2`, la función es más aguda, lo que favorece la explotación

Esto convierte a `alpha` en un **control continuo** entre exploración y explotación dentro del algoritmo. Su introducción transforma una formulación rígida en un modelo más versátil capaz de adaptarse a distintos tipos de problemas y configuraciones de entrada.

---

## Interpretación física e inspiración natural

El concepto de modificar la forma de una curva de resonancia no es nuevo en física ni biología. Se han documentado situaciones análogas en distintos sistemas naturales, tales como:

- En **neurociencia**, ciertos tipos de neuronas ajustan su sensibilidad al ruido adaptando el perfil de resonancia, lo que permite detectar señales débiles en entornos caóticos o cambiantes.
- En **óptica**, los filtros Gauss-Lorentz permiten cambiar la pendiente de una función de transmisión sin mover el centro, un principio similar al uso de `alpha`.
- En **procesamiento sensorial**, algunos insectos y mamíferos ajustan la pendiente de respuesta de sus sensores auditivos o visuales dependiendo del entorno, buscando un compromiso entre sensibilidad y estabilidad.

---

## Implementación práctica en código

La fórmula `eta(t)` se implementa en una sola línea dentro del código del algoritmo:

```python
exponent = - ((abs(current_freq - resonance_freq) / self.sigma) ** self.shape_param)
amplitude = self.eta0 * np.exp(exponent)
```

Este valor `amplitude` se utiliza para determinar cuántos cambios aleatorios deben introducirse en una solución candidata durante cada iteración del algoritmo.

La inclusión de `shape_param` en la clase `SRA_TSP` permite que cualquier investigador o ingeniero pueda ajustar el comportamiento de la resonancia para experimentar con distintos niveles de agresividad en la búsqueda.

---

## Resultados y parámetros derivados automáticamente

Para ajustar el rendimiento del algoritmo a diferentes tamaños de problema, se desarrolló una función `autoconfig(num_cities)` que calcula valores empíricamente efectivos para:

- Número de partículas (`num_particles`)
- Iteraciones máximas (`max_iterations`)
- Valor base del ruido (`eta0`)
- Ancho de banda (`sigma`)
- Factor de acoplamiento (`coupling_factor`)
- Parámetro de forma `alpha`

Estos valores fueron obtenidos tras ejecutar un barrido aleatorio de combinaciones, evaluando la mejora respecto al algoritmo de vecino más cercano. Se identificaron patrones consistentes que permitieron ajustar funciones de escala:

```python
num_particles = max(10, min(300, int(0.5*np.sqrt(num_cities))))
max_iterations = max(30, int(8 * (num_cities ** 0.35)))
eta0 = 0.045 * (num_cities / 10000) ** -0.25
sigma = 0.35 * (num_cities / 10000) ** -0.15
coupling_factor = min(0.7, 0.25 + 0.15*np.log10(num_cities))
shape_param = 1.8 + 0.4 * (num_cities / 10000)
```

---

## Gráficos de convergencia y ejemplo visual

Se observa en los experimentos una curva plana de energía (distancia) óptima desde las primeras iteraciones, mientras que la frecuencia del sistema y la amplitud del ruido fluyen de forma dinámica. Esto indica que el sistema no genera mejoras adicionales una vez ha alcanzado un mínimo profundo, y que el mecanismo de resonancia mantiene su funcionamiento adaptativo.

![Curva de convergencia](c05c5250-e9e0-4054-a1a6-5f4f47a40f4d.png)

Además, la forma espacial de la ruta obtenida muestra que la optimización logra evitar rutas erráticas o zigzags excesivos, manteniendo una suavidad general notable:

![Visualización del tour](8a20819f-9a88-4472-8413-3e5f9ff490f6.png)

Esta imagen representa una solución generada con 10 000 ciudades distribuidas aleatoriamente. La distribución del camino sigue un patrón suave y compacto, resultado de la combinación del gradiente local, acoplamiento y resonancia estocástica.

---

## Comparación con benchmarks estándar (TSPLIB)

Para evaluar objetivamente el rendimiento del algoritmo, se realizaron pruebas sobre el conjunto de datos estándar **TSPLIB**, que incluye instancias desde 52 hasta 3 038 nodos. Se organizaron por nivel de dificultad:

- Nivel básico: `berlin52`, `eil76`, `eil101`
- Nivel intermedio: `pcb442`, `lin318`, `pr439`
- Nivel avanzado: `rat783`, `pr1002`
- Nivel extremo: `rl1323`, `d1655`, `d2103`, `pcb3038`

Se ejecutaron **50 corridas por instancia**, midiendo distancia final, tiempo y mejora relativa respecto al óptimo conocido. Aunque el algoritmo no supera a LKH, muestra consistencia en todos los tamaños y revela su potencial como herramienta de exploración y ajuste dinámico de ruido.

Los resultados detallados por instancia incluyen estadísticas agregadas (media, desviación, mínimo, máximo) y confirman que el modelo responde a escalas grandes sin necesidad de rediseño manual.

---

## Comentario final

El valor de esta modificación no está en superar algoritmos exactos como Concorde o LKH, que están altamente optimizados, sino en proponer un **modelo sencillo, replicable y controlable** que permita explorar nuevas dinámicas dentro del TSP y potencialmente en otros problemas combinatorios. La posibilidad de ajustar el comportamiento del ruido a través del parámetro `alpha` agrega una capa de flexibilidad que puede ser aprovechada en futuros desarrollos o integraciones híbridas con otras heurísticas.

Además, la simplicidad de su implementación, la visualización clara de su dinámica y su capacidad de adaptación lo convierten en un excelente punto de partida para quienes buscan experimentar con **modelos inspirados en fenómenos físicos** dentro del campo de la optimización heurística.



