## Algoritmos Recursivos vs. Iterativos en Python

Este tutorial muestra un ejemplo clásico del cálculo del factorial de un número utilizando dos enfoques: recursivo e iterativo.

## Enfoque Recursivo

La recursión implica que una función se llame a sí misma hasta alcanzar un caso base que detenga la recursión. El ejemplo clásico es el cálculo del factorial:

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)

print(factorial(5))  # Salida: 120
```

### Explicación:
- **Caso base:** Cuando `n == 0`, se retorna 1 para detener la recursión.
- **Caso recursivo:** La función se llama a sí misma con `n - 1`, multiplicando el resultado por `n`.

## Enfoque Iterativo

La iteración utiliza bucles para repetir operaciones, evitando llamadas recursivas. Aquí se muestra cómo calcular el factorial de forma iterativa:

```python
def factorial_iterativo(n):
    result = 1
    for i in range(1, n + 1):
        result *= i
    return result

print(factorial_iterativo(5))  # Salida: 120
```

### Explicación:
- Se inicia `result` en 1.
- Se utiliza un bucle `for` que itera desde 1 hasta `n`, multiplicando `result` por cada valor de `i`.
- Finalmente, se retorna el valor de `result`.

## Comparación entre Recursividad e Iteración

| Característica       | Recursivo                                    | Iterativo                               |
|----------------------|----------------------------------------------|-----------------------------------------|
| **Simplicidad**      | Código conciso y elegante                    | Más explícito y directo                 |
| **Legibilidad**      | Fácil de entender en problemas naturalmente recursivos | Puede ser más intuitivo para operaciones repetitivas simples |
| **Uso de memoria**   | Mayor consumo de memoria por las llamadas en pila | Menor uso de memoria, al evitar la pila de llamadas |
| **Riesgo de errores**| Posible desbordamiento de pila en casos muy profundos | Menor riesgo de desbordamiento          |
| **Rendimiento**      | Puede ser más lento por la sobrecarga de llamadas | Generalmente más rápido y eficiente     |

## Conclusión

Ambos enfoques son válidos para calcular el factorial. La elección entre recursividad e iteración dependerá del problema específico, consideraciones de rendimiento y preferencia en legibilidad del código. Para problemas que requieran una solución más natural y elegante, la recursión puede ser adecuada, mientras que la iteración es recomendable para optimizar el uso de memoria y evitar desbordamientos en casos de gran profundidad.
