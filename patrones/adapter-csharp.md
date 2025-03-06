A continuación se muestra un ejemplo práctico y profesional del patrón Adapter en C#. En entornos reales es común encontrarse con sistemas legados o librerías de terceros que implementan interfaces o métodos diferentes a los que utiliza nuestra aplicación. En este ejemplo, integramos un sistema de procesamiento de pagos legado en una aplicación moderna. Nuestra aplicación define una interfaz estándar para procesar pagos, mientras que el sistema legado expone un método distinto. Usando el Adapter, logramos que el sistema legado se adapte a nuestra interfaz sin modificar su código.

---

### Definición de la Interfaz Objetivo

Esta es la interfaz que nuestra aplicación utiliza para procesar pagos.

```csharp
// Interfaz que define el contrato que espera nuestra aplicación.
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
}
```

---

### Sistema Legado (Proveedor de Pago)

El sistema legado expone un método con una firma distinta, por ejemplo, recibe el monto como tipo double y requiere especificar la moneda.

```csharp
// Sistema de pago legado (por ejemplo, una librería de terceros).
public class LegacyPaymentGateway
{
    public void MakePayment(double amount, string currency)
    {
        // Lógica de procesamiento en el sistema legado.
        Console.WriteLine($"Procesando pago legado de {amount} {currency}");
    }
}
```

---

### Adaptador del Sistema Legado

El adaptador implementa la interfaz estándar y encapsula la instancia del sistema legado, traduciendo las llamadas al formato que éste espera.

```csharp
// Adaptador que permite usar el LegacyPaymentGateway a través de la interfaz IPaymentProcessor.
public class LegacyPaymentGatewayAdapter : IPaymentProcessor
{
    private readonly LegacyPaymentGateway _legacyGateway;
    private readonly string _currency;
    
    public LegacyPaymentGatewayAdapter(LegacyPaymentGateway legacyGateway, string currency)
    {
        _legacyGateway = legacyGateway;
        _currency = currency;
    }
    
    public void ProcessPayment(decimal amount)
    {
        // Convertimos el monto de decimal a double (teniendo en cuenta las conversiones necesarias).
        double legacyAmount = (double) amount;
        // Delegamos la llamada al método del sistema legado.
        _legacyGateway.MakePayment(legacyAmount, _currency);
    }
}
```

---

### Uso en la Aplicación

La aplicación utiliza la interfaz IPaymentProcessor sin preocuparse de la implementación concreta. Gracias al Adapter, podemos integrar el sistema legado sin modificar su código ni la lógica de negocio de la aplicación.

```csharp
using System;

public class Program
{
    public static void Main(string[] args)
    {
        // Instanciamos el sistema de pago legado.
        LegacyPaymentGateway legacyGateway = new LegacyPaymentGateway();
        
        // Creamos el adaptador, especificando la moneda (por ejemplo, "USD").
        IPaymentProcessor paymentProcessor = new LegacyPaymentGatewayAdapter(legacyGateway, "USD");
        
        // Procesamos un pago utilizando la interfaz común.
        paymentProcessor.ProcessPayment(150.00m);
        
        // La salida será:
        // "Procesando pago legado de 150 USD"
    }
}
```

---

### Explicación

- **Desacoplamiento y Flexibilidad:**  
  La aplicación depende únicamente de la interfaz `IPaymentProcessor`. El adaptador se encarga de traducir la llamada a la implementación del sistema legado, lo que permite cambiar o actualizar el componente de pago sin afectar el resto del código.

- **Integración de Sistemas Legados:**  
  Este patrón es especialmente útil cuando se integra software heredado o librerías de terceros que no se pueden modificar. El Adapter actúa como puente entre dos interfaces incompatibles.

- **Manejo de Situaciones Comunes:**  
  En el día a día, es frecuente que los requisitos del negocio obliguen a integrar sistemas con APIs dispares. Usar el patrón Adapter permite encapsular las diferencias y ofrecer una interfaz unificada al resto de la aplicación.

Este ejemplo ilustra un caso de uso clásico y práctico del patrón Adapter en C#, mostrando cómo manejar situaciones comunes en aplicaciones profesionales al integrar sistemas con interfaces diferentes.
