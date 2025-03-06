## Iterator en C#

A continuación se muestra un ejemplo práctico y profesional del patrón Iterator en C#, aplicado a un sistema de gestión de pedidos. En este escenario, se tiene una colección de pedidos y se requiere iterar sobre ellos de manera personalizada, por ejemplo, para procesar únicamente los pedidos "Pendientes". Este ejemplo ilustra cómo implementar un iterador personalizado (sin recurrir únicamente a la sintaxis “yield return”) que te permite filtrar elementos durante la iteración, algo muy útil en aplicaciones reales donde se requiere procesar subconjuntos de datos de una colección.

---

### Código de Ejemplo

#### 1. Clase Order

Define el objeto Pedido con propiedades básicas.

```csharp
public class Order
{
    public int OrderId { get; }
    public string Status { get; }

    public Order(int orderId, string status)
    {
        OrderId = orderId;
        Status = status;
    }

    public override string ToString() => $"OrderId: {OrderId}, Status: {Status}";
}
```

#### 2. Colección de Pedidos (Aggregate)

La clase `OrderCollection` mantiene una lista de pedidos e implementa `IEnumerable<Order>` para poder iterar sobre todos ellos. Además, proporciona un método para obtener un iterador filtrado (solo pedidos pendientes).

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class OrderCollection : IEnumerable<Order>
{
    private readonly List<Order> _orders = new List<Order>();

    public void AddOrder(Order order) => _orders.Add(order);

    // Iterador general: permite recorrer todos los pedidos.
    public IEnumerator<Order> GetEnumerator() => _orders.GetEnumerator();

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();

    // Método que devuelve un iterador personalizado para pedidos "Pendientes".
    public IEnumerator<Order> GetPendingOrdersIterator() => new PendingOrderIterator(_orders);
}
```

#### 3. Iterador Personalizado (PendingOrderIterator)

Implementa `IEnumerator<Order>` para recorrer la lista de pedidos, pero solo retorna aquellos cuyo estado es "Pending". Este patrón te enseña cómo encapsular la lógica de filtrado en el iterador, separándola de la lógica de la colección.

```csharp
using System;
using System.Collections;
using System.Collections.Generic;

public class PendingOrderIterator : IEnumerator<Order>
{
    private readonly List<Order> _orders;
    private int _position = -1;

    public PendingOrderIterator(List<Order> orders)
    {
        _orders = orders;
    }

    public Order Current
    {
        get
        {
            if (_position >= 0 && _position < _orders.Count)
                return _orders[_position];
            throw new InvalidOperationException();
        }
    }

    object IEnumerator.Current => Current;

    // MoveNext avanza el puntero hasta encontrar un pedido con Status "Pending".
    public bool MoveNext()
    {
        while (++_position < _orders.Count)
        {
            if (_orders[_position].Status.Equals("Pending", StringComparison.OrdinalIgnoreCase))
                return true;
        }
        return false;
    }

    public void Reset() => _position = -1;

    public void Dispose() { /* No se necesitan liberar recursos en este ejemplo */ }
}
```

#### 4. Ejemplo de Uso

Se crea una colección de pedidos con distintos estados y se demuestra el uso del iterador general y el iterador filtrado para procesar solo los pedidos pendientes.

```csharp
using System;
using System.Collections.Generic;

public class Program
{
    public static void Main(string[] args)
    {
        OrderCollection orderCollection = new OrderCollection();
        orderCollection.AddOrder(new Order(1, "Pending"));
        orderCollection.AddOrder(new Order(2, "Shipped"));
        orderCollection.AddOrder(new Order(3, "Pending"));
        orderCollection.AddOrder(new Order(4, "Delivered"));
        orderCollection.AddOrder(new Order(5, "Pending"));

        Console.WriteLine("Todos los pedidos:");
        foreach (Order order in orderCollection)
        {
            Console.WriteLine(order);
        }

        Console.WriteLine("\nPedidos Pendientes:");
        IEnumerator<Order> pendingIterator = orderCollection.GetPendingOrdersIterator();
        while (pendingIterator.MoveNext())
        {
            Console.WriteLine(pendingIterator.Current);
        }
    }
}
```

---

### Explicación

- **Encapsulación de la Lógica de Iteración:**  
  Se separa la lógica para recorrer la colección (la clase `OrderCollection`) y la lógica para filtrar solo los pedidos pendientes (la clase `PendingOrderIterator`). Esto permite reutilizar y modificar el comportamiento de iteración sin afectar la estructura de la colección.

- **Uso Profesional y Situaciones Comunes:**  
  En sistemas empresariales es habitual tener colecciones de objetos (por ejemplo, pedidos, clientes, registros de log) y requerir iteradores personalizados que permitan filtrar o transformar datos durante la iteración. Este patrón es fundamental para diseñar aplicaciones escalables y mantenibles.

- **Flexibilidad:**  
  El patrón Iterator te permite recorrer cualquier tipo de colección sin exponer su estructura interna, facilitando la integración con otras partes del sistema y el manejo de colecciones complejas.

Este ejemplo demuestra cómo aplicar el patrón Iterator en C# para manejar situaciones comunes del día a día de un programador, como la necesidad de filtrar y procesar datos de colecciones en aplicaciones profesionales.
