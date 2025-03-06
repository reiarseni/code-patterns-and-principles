## Principio de Sustitución de Liskov (Liskov Substitution Principle, LSP)

A continuación se muestra un ejemplo en Java que ilustra un caso práctico real de refactorización para cumplir con el Principio de Sustitución de Liskov (LSP). Se parte de un diseño que viola LSP (donde algunas subclases modifican el comportamiento esperado) y se refactoriza para lograr una jerarquía de clases coherente y sustituible.

---

## Ejemplo Violando LSP

Imagina que tenemos una clase base `Bird` que define el método `fly()`. Sin embargo, no todas las aves pueden volar (por ejemplo, el avestruz), lo que nos lleva a que la subclase `Ostrich` sobrescriba el método lanzando una excepción:

```java
public class Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

public class Sparrow extends Bird {
    // Hereda correctamente el método fly()
}

public class Ostrich extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Ostriches can't fly");
    }
}

public class Main {
    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        sparrow.fly(); // Funciona correctamente: "Flying"

        Bird ostrich = new Ostrich();
        // Esta llamada lanzará una excepción en tiempo de ejecución:
        ostrich.fly();
    }
}
```

**Problema:**  
El uso de `Ostrich` como una instancia de `Bird` rompe el contrato implícito de que todas las aves pueden volar. Esto es una violación de LSP, ya que se espera que cualquier subclase de `Bird` pueda sustituir a su padre sin causar errores inesperados.

---

## Ejemplo Refactorizado Cumpliendo LSP

Para resolver el problema, separamos la responsabilidad de volar en una interfaz independiente. Así, solo las aves que pueden volar implementan esa interfaz, mientras que las que no pueden volar se limitan a heredar comportamientos comunes (como comer o dormir).

```java
// Clase base con comportamientos comunes a todas las aves.
public abstract class Bird {
    public abstract void eat();
    public abstract void sleep();
}

// Interfaz que define la habilidad de volar.
public interface Flyable {
    void fly();
}

// Clase concreta para aves que pueden volar.
public class Sparrow extends Bird implements Flyable {
    @Override
    public void eat() {
        System.out.println("Sparrow is eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Sparrow is sleeping");
    }
    
    @Override
    public void fly() {
        System.out.println("Sparrow is flying");
    }
}

// Clase concreta para aves que no pueden volar.
public class Ostrich extends Bird {
    @Override
    public void eat() {
        System.out.println("Ostrich is eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Ostrich is sleeping");
    }
}

public class Main {
    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        sparrow.eat();
        sparrow.sleep();
        // Verificamos si la instancia es voladora antes de llamar a fly()
        if (sparrow instanceof Flyable) {
            ((Flyable) sparrow).fly();
        }

        Bird ostrich = new Ostrich();
        ostrich.eat();
        ostrich.sleep();
        // Como Ostrich no implementa Flyable, evitamos llamar a fly()
        if (ostrich instanceof Flyable) {
            ((Flyable) ostrich).fly();
        } else {
            System.out.println("Ostrich cannot fly");
        }
    }
}
```

**Beneficios del refactor:**

- **Cumplimiento de LSP:**  
  Las subclases de `Bird` ya no deben cumplir un contrato que no pueden sostener (como volar), ya que la habilidad de volar se ha delegado a la interfaz `Flyable`.

- **Mayor claridad y cohesión:**  
  Cada clase e interfaz tiene una responsabilidad bien definida. Las aves que pueden volar implementan `Flyable`, mientras que las que no pueden, simplemente heredan la funcionalidad común de `Bird`.

- **Facilidad de mantenimiento y extensión:**  
  Si se agregan nuevas funcionalidades o tipos de aves, el sistema es más flexible y las modificaciones se realizan sin alterar la estructura existente.

---

### Conclusión

Este ejemplo muestra cómo refactorizar un código violatorio del LSP en un escenario real, mejorando la calidad del diseño y la robustez del sistema en aplicaciones reales.
