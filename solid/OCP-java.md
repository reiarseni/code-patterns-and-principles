## Principio de Abierto/Cerrado (Open-Close Principle, LSP)

A continuación se muestra un ejemplo clásico de refactorización para cumplir con el Principio Abierto/Cerrado (OCP) en un escenario real: el cálculo de descuentos en un e-commerce. Inicialmente, el cálculo de descuento se implementa con condicionales, lo que obliga a modificar la clase cada vez que se agrega un nuevo tipo de descuento. Luego se refactoriza utilizando interfaces y clases concretas, de forma que se pueden añadir nuevos descuentos sin modificar el código existente.

### Versión sin aplicar OCP (Código original)
```java
public class DiscountCalculator {
    public double calculateDiscount(double price, String discountType) {
        double discount = 0;
        if ("PERCENTAGE".equalsIgnoreCase(discountType)) {
            discount = price * 0.1; // 10% de descuento
        } else if ("FLAT".equalsIgnoreCase(discountType)) {
            discount = 20.0; // Descuento fijo de 20 unidades monetarias
        }
        return discount;
    }
}

public class Main {
    public static void main(String[] args) {
        DiscountCalculator calculator = new DiscountCalculator();
        double discount1 = calculator.calculateDiscount(200, "PERCENTAGE");
        System.out.println("Descuento: " + discount1);
    }
}
```

### Versión refactorizada aplicando OCP
Se crea una interfaz `DiscountStrategy` y clases concretas para cada tipo de descuento. De esta forma, si se necesita agregar un nuevo tipo de descuento, basta con crear una nueva implementación sin modificar la lógica existente.

```java
// Interfaz que define la estrategia de descuento
public interface DiscountStrategy {
    double calculate(double price);
}

// Descuento basado en un porcentaje
public class PercentageDiscount implements DiscountStrategy {
    private double percentage;
    
    public PercentageDiscount(double percentage) {
        this.percentage = percentage;
    }
    
    @Override
    public double calculate(double price) {
        return price * percentage;
    }
}

// Descuento de monto fijo
public class FlatDiscount implements DiscountStrategy {
    private double flatAmount;
    
    public FlatDiscount(double flatAmount) {
        this.flatAmount = flatAmount;
    }
    
    @Override
    public double calculate(double price) {
        return flatAmount;
    }
}

// Contexto que utiliza la estrategia de descuento
public class DiscountCalculator {
    private DiscountStrategy strategy;
    
    public DiscountCalculator(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(DiscountStrategy strategy) {
        this.strategy = strategy;
    }
    
    public double calculate(double price) {
        return strategy.calculate(price);
    }
}

public class Main {
    public static void main(String[] args) {
        // Uso del descuento por porcentaje (10%)
        DiscountCalculator calculator = new DiscountCalculator(new PercentageDiscount(0.1));
        System.out.println("Descuento por porcentaje: " + calculator.calculate(200));

        // Cambio a descuento fijo
        calculator.setStrategy(new FlatDiscount(20));
        System.out.println("Descuento fijo: " + calculator.calculate(200));
    }
}
```

### Conclusión

En esta versión, la clase `DiscountCalculator` está abierta a la extensión mediante la inyección de diferentes estrategias de descuento, pero cerrada a modificaciones en su lógica interna. Esto cumple con el OCP y permite agregar nuevos descuentos (por ejemplo, descuentos por temporada, por fidelidad, etc.) sin alterar el código existente.
