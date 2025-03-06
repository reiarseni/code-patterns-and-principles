## Observer en C#

A continuación se muestra un ejemplo práctico en C# del patrón Observer aplicado a una estación meteorológica, un caso clásico y real usado en muchas aplicaciones para actualizar múltiples componentes cuando cambian los datos.

---

### Definición de Interfaces

Se definen dos interfaces:  
- **ISubject:** para gestionar los observadores.  
- **IObserver:** para que los observadores reciban actualizaciones.

```csharp
public interface IObserver
{
    void Update(float temperature, float humidity, float pressure);
}

public interface ISubject
{
    void RegisterObserver(IObserver observer);
    void RemoveObserver(IObserver observer);
    void NotifyObservers();
}
```

---

### Implementación del Sujeto (WeatherStation)

La clase `WeatherStation` mantiene una lista de observadores y notifica los cambios en las medidas meteorológicas.

```csharp
using System;
using System.Collections.Generic;

public class WeatherStation : ISubject
{
    private List<IObserver> observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherStation()
    {
        observers = new List<IObserver>();
    }

    public void RegisterObserver(IObserver observer)
    {
        observers.Add(observer);
    }

    public void RemoveObserver(IObserver observer)
    {
        observers.Remove(observer);
    }

    public void NotifyObservers()
    {
        foreach (var observer in observers)
        {
            observer.Update(temperature, humidity, pressure);
        }
    }

    // Método para actualizar las mediciones y notificar a los observadores
    public void SetMeasurements(float temperature, float humidity, float pressure)
    {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        NotifyObservers();
    }
}
```

---

### Implementación del Observador (CurrentConditionsDisplay)

El observador `CurrentConditionsDisplay` muestra las condiciones actuales cada vez que se recibe una actualización.

```csharp
using System;

public class CurrentConditionsDisplay : IObserver
{
    private float temperature;
    private float humidity;

    public void Update(float temperature, float humidity, float pressure)
    {
        this.temperature = temperature;
        this.humidity = humidity;
        Display();
    }

    public void Display()
    {
        Console.WriteLine($"Condiciones actuales: {temperature}°F y {humidity}% de humedad.");
    }
}
```

---

### Ejecución del Ejemplo

En el método `Main`, se crea una instancia de `WeatherStation`, se registra el observador y se simulan cambios en las medidas para demostrar cómo se notifica automáticamente al observador.

```csharp
using System;

class Program
{
    static void Main(string[] args)
    {
        WeatherStation weatherStation = new WeatherStation();
        CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay();
        
        // Registrar el observador
        weatherStation.RegisterObserver(currentDisplay);
        
        // Simular cambios en las condiciones meteorológicas
        weatherStation.SetMeasurements(80, 65, 30.4f);
        weatherStation.SetMeasurements(82, 70, 29.2f);
        weatherStation.SetMeasurements(78, 90, 29.2f);
        
        Console.ReadLine();
    }
}
```

---

### Conclusión

Este ejemplo demuestra el uso del patrón Observer en un escenario real:  
- **Desacoplamiento:** La `WeatherStation` no necesita conocer los detalles de cómo se muestran o procesan los datos, solo notifica a los observadores.  
- **Flexibilidad:** Se pueden añadir nuevos observadores (por ejemplo, gráficos, alertas, etc.) sin modificar el código del sujeto.  
- **Escalabilidad:** Es un patrón ampliamente utilizado en sistemas de eventos, interfaces gráficas y notificaciones en tiempo real.
