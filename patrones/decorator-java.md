## Decorator en Java

A continuación se muestra un ejemplo práctico y realista del patrón Decorator en Java, aplicado a la decoración de un servicio de acceso a datos. En entornos empresariales es común necesitar agregar funcionalidades adicionales (como registro de actividad, caching, seguridad, etc.) a un servicio sin modificar su implementación base. Con el Decorator, podemos “envolver” la funcionalidad básica y añadirle comportamientos extra de forma transparente y flexible.

---

### Definición de la Interfaz del Servicio

```java
public interface DataService {
    String getData(String parameter);
}
```

### Implementación Base del Servicio

Esta clase simula la operación costosa de obtener datos (por ejemplo, de una base de datos o servicio remoto).

```java
public class BasicDataService implements DataService {
    @Override
    public String getData(String parameter) {
        // Simula una operación costosa.
        return "Datos para " + parameter;
    }
}
```

### Decorador para Logging

Este decorador añade registro (logging) antes y después de la llamada al método del servicio.

```java
public class LoggingDataServiceDecorator implements DataService {
    private final DataService wrapped;

    public LoggingDataServiceDecorator(DataService wrapped) {
        this.wrapped = wrapped;
    }

    @Override
    public String getData(String parameter) {
        System.out.println("LOG: Iniciando getData con parámetro: " + parameter);
        String result = wrapped.getData(parameter);
        System.out.println("LOG: Finalizando getData con resultado: " + result);
        return result;
    }
}
```

### Decorador para Caching

Este decorador almacena en caché los resultados para evitar repetir operaciones costosas.

```java
import java.util.HashMap;
import java.util.Map;

public class CachingDataServiceDecorator implements DataService {
    private final DataService wrapped;
    private final Map<String, String> cache = new HashMap<>();

    public CachingDataServiceDecorator(DataService wrapped) {
        this.wrapped = wrapped;
    }

    @Override
    public String getData(String parameter) {
        if (cache.containsKey(parameter)) {
            System.out.println("CACHE: Valor obtenido de la caché para: " + parameter);
            return cache.get(parameter);
        }
        System.out.println("CACHE: Valor no encontrado en caché para: " + parameter);
        String result = wrapped.getData(parameter);
        cache.put(parameter, result);
        return result;
    }
}
```

### Decorador para Seguridad

Este decorador verifica que el usuario tenga los permisos necesarios antes de permitir el acceso al servicio.

```java
public class SecurityDataServiceDecorator implements DataService {
    private final DataService wrapped;
    private final String userRole;

    public SecurityDataServiceDecorator(DataService wrapped, String userRole) {
        this.wrapped = wrapped;
        this.userRole = userRole;
    }

    @Override
    public String getData(String parameter) {
        if (!"ADMIN".equalsIgnoreCase(userRole)) {
            throw new SecurityException("Acceso denegado para el rol: " + userRole);
        }
        return wrapped.getData(parameter);
    }
}
```

### Ejemplo de Uso Combinado

En el método `main` se combinan los decoradores para crear un servicio con logging, caching y seguridad. Así, el cliente utiliza el servicio sin preocuparse de los detalles adicionales.

```java
public class DecoratorDemo {
    public static void main(String[] args) {
        // Servicio base
        DataService basicService = new BasicDataService();

        // Agregamos funcionalidad de logging
        DataService loggingService = new LoggingDataServiceDecorator(basicService);

        // Agregamos caching sobre el servicio con logging
        DataService cachingService = new CachingDataServiceDecorator(loggingService);

        // Finalmente, se añade la verificación de seguridad (solo usuarios ADMIN pueden acceder)
        DataService securedService = new SecurityDataServiceDecorator(cachingService, "ADMIN");

        // Uso del servicio decorado
        System.out.println(securedService.getData("param1"));
        // Segunda llamada con el mismo parámetro debe obtener el valor de la caché
        System.out.println(securedService.getData("param1"));

        // Si se cambia el rol a uno sin permisos, se lanzará una excepción:
        // DataService securedServiceUser = new SecurityDataServiceDecorator(cachingService, "USER");
        // securedServiceUser.getData("param2"); // Lanza SecurityException
    }
}
```

---

### Explicación del Ejemplo

- **Composición Dinámica:**  
  Cada decorador recibe como parámetro una implementación de `DataService` y añade su funcionalidad extra sin alterar la lógica base. Esto permite componer comportamientos de forma flexible.

- **Desacoplamiento:**  
  El cliente solo interactúa con la interfaz `DataService`. Los detalles de logging, caching y seguridad se aplican de manera transparente.

- **Aplicaciones Reales:**  
  Este patrón es común en aplicaciones empresariales para interceptar y extender la funcionalidad de servicios (por ejemplo, en frameworks de inyección de dependencias, API gateways, middleware, etc.), permitiendo agregar características transversales sin modificar el servicio principal.

Este ejemplo demuestra un caso de uso profesional y práctico del patrón Decorator en Java, permitiendo mejorar la mantenibilidad y extensibilidad de un sistema real.
