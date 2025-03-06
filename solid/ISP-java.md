## Principio de Segregación de Interfaces (Interface Segregation Principle, ISP)

El ISP establece que una clase no debe verse forzada a implementar interfaces que no utiliza. En otras palabras, es preferible tener varias interfaces específicas en lugar de una única interfaz general, permitiendo que las clases dependan solo de los métodos que realmente necesitan.

A continuación, se muestra un ejemplo en Java que ilustra cómo refactorizar para cumplir con el ISP:

### Ejemplo Sin Aplicar el ISP

En este ejemplo, se define una interfaz general que obliga a implementar métodos que algunas clases no requieren:

```java
// Interfaz general que agrupa varias responsabilidades
public interface AllInOnePrinter {
    void print(String document);
    void scan(String document);
    void fax(String document);
}

public class BasicPrinter implements AllInOnePrinter {
    @Override
    public void print(String document) {
        System.out.println("Imprimiendo: " + document);
    }

    @Override
    public void scan(String document) {
        // BasicPrinter no soporta escanear, pero se ve obligado a implementarlo.
        throw new UnsupportedOperationException("Scan not supported");
    }

    @Override
    public void fax(String document) {
        // BasicPrinter no soporta fax, pero se ve obligado a implementarlo.
        throw new UnsupportedOperationException("Fax not supported");
    }
}
```

### Ejemplo Refactorizado Aplicando el ISP

Se separa la interfaz en varias interfaces específicas para que cada clase solo implemente lo que necesita:

```java
// Interfaz específica para la impresión
public interface Printer {
    void print(String document);
}

// Interfaz específica para el escaneo
public interface Scanner {
    void scan(String document);
}

// Interfaz específica para el fax
public interface Fax {
    void fax(String document);
}

// Clase que solo implementa la funcionalidad de impresión
public class BasicPrinter implements Printer {
    @Override
    public void print(String document) {
        System.out.println("Imprimiendo: " + document);
    }
}

// Una clase multifuncional que puede implementar varias interfaces según sea necesario
public class MultiFunctionPrinter implements Printer, Scanner, Fax {
    @Override
    public void print(String document) {
        System.out.println("Imprimiendo: " + document);
    }

    @Override
    public void scan(String document) {
        System.out.println("Escaneando: " + document);
    }

    @Override
    public void fax(String document) {
        System.out.println("Enviando fax: " + document);
    }
}
```

### Conclusión

Aplicar el Principio de Segregación de Interfaces mejora la cohesión del código, ya que las clases solo dependen de las interfaces que realmente utilizan, evitando implementar métodos innecesarios y reduciendo el riesgo de errores en tiempo de ejecución. Este es un caso clásico de refactorización en muchas aplicaciones para mantener el código limpio y fácilmente extensible.
