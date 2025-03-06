## Principio de Sustitución de Liskov (Liskov Substitution Principle, LSP)


El **Principio de Sustitución de Liskov** (Liskov Substitution Principle, LSP) es uno de los cinco principios SOLID del diseño orientado a objetos. Este principio establece una guía sobre cómo las subclases deben comportarse en relación con sus superclases. 

(LSP) establece que los objetos de una clase derivada deben poder ser reemplazados por objetos de la clase base sin alterar el correcto funcionamiento del programa. Es decir, si una clase es un subtipo de otra, debe ser capaz de ser utilizada de manera intercambiable con la clase base, sin causar errores o comportamientos inesperados.

### Definición

**"Los objetos de una clase derivada deben poder ser sustituidos por objetos de la clase base sin alterar el correcto funcionamiento del programa."**

---

## Ejemplo Violando LSP

A continuación se muestra un ejemplo en Java que ilustra un caso práctico real de refactorización para cumplir con el Principio de Sustitución de Liskov (LSP). Se parte de un diseño que viola LSP (donde algunas subclases modifican el comportamiento esperado) y se refactoriza para lograr una jerarquía de clases coherente y sustituible.

Imagina que tenemos una clase base `Bird` que define el método `fly()`. Sin embargo, no todas las aves pueden volar (por ejemplo, el avestruz), lo que nos lleva a que la subclase `Pinguino` sobrescriba el método lanzando una excepción:

```java
public class Bird {
    public void fly() {
        System.out.println("Flying");
    }
}

public class Sparrow extends Bird {
    // Hereda correctamente el método fly()
}

public class Pinguino extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Pinguinos can't fly");
    }
}

public class Main {
    public static void main(String[] args) {
        Bird sparrow = new Sparrow();
        sparrow.fly(); // Funciona correctamente: "Flying"

        Bird pinguino = new Pinguino();
        // Esta llamada lanzará una excepción en tiempo de ejecución:
        pinguino.fly();
    }
}
```

**Problema:**  
El uso de `Pinguino` como una instancia de `Bird` rompe el contrato implícito de que todas las aves pueden volar. Esto es una violación de LSP, ya que se espera que cualquier subclase de `Bird` pueda sustituir a su padre sin causar errores inesperados.

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
public class Pinguino extends Bird {
    @Override
    public void eat() {
        System.out.println("Pinguino is eating");
    }
    
    @Override
    public void sleep() {
        System.out.println("Pinguino is sleeping");
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

        Bird pinguino = new Pinguino();
        pinguino.eat();
        pinguino.sleep();
        // Como Ostrich no implementa Flyable, evitamos llamar a fly()
        if (pinguino instanceof Flyable) {
            ((Flyable) pinguino).fly();
        } else {
            System.out.println("Pinguino cannot fly");
        }
    }
}
```

### Conclusión del ejemplo

Este ejemplo muestra cómo refactorizar un código violatorio del LSP en un escenario real, mejorando la calidad del diseño y la robustez del sistema en aplicaciones reales.

**Beneficios del ejemplo refactor:**

- **Cumplimiento de LSP:**  
  Las subclases de `Bird` ya no deben cumplir un contrato que no pueden sostener (como volar), ya que la habilidad de volar se ha delegado a la interfaz `Flyable`.

- **Mayor claridad y cohesión:**  
  Cada clase e interfaz tiene una responsabilidad bien definida. Las aves que pueden volar implementan `Flyable`, mientras que las que no pueden, simplemente heredan la funcionalidad común de `Bird`.

- **Facilidad de mantenimiento y extensión:**  
  Si se agregan nuevas funcionalidades o tipos de aves, el sistema es más flexible y las modificaciones se realizan sin alterar la estructura existente.

---

### En Resumen:
- **Intercambiabilidad**: Las instancias de clases derivadas deben poder sustituir a instancias de su clase base.
- **Contracto**: Las clases derivadas deben cumplir las expectativas establecidas por la clase base, como invariantes, precondiciones y postcondiciones.

Este principio ayuda a garantizar que la herencia se utilice correctamente y a fomentar el diseño orientado a objetos robusto y mantenible.

### Explicación

1. **Intercambiabilidad**: Esto significa que, si tienes una clase base y varias clases derivadas, deberías poder usar una instancia de una clase derivada en lugar de una instancia de la clase base sin que esto cause errores o altere el funcionamiento del programa.

2. **Contratos y Comportamiento**: Las clases derivadas deben cumplir con las expectativas establecidas por la clase base. Esto incluye:
   - **Invariantes**: Las propiedades que deben ser verdaderas para los objetos.
   - **Precondiciones**: Las condiciones que deben ser verdaderas antes de ejecutar un método.
   - **Postcondiciones**: Las condiciones que deben ser verdaderas después de ejecutar un método.
   
   Esto significa que cualquier clase derivada no puede reforzar las precondiciones de sus métodos en comparación con los métodos de la clase base, ni debilitar las postcondiciones.

### Beneficios

- **Consistencia y Predecibilidad**: El LSP asegura que las clases derivadas se comporten de manera predecible y consistente, lo que facilita la comprensión y el uso de las jerarquías de clases.
- **Mantenibilidad**: Al seguir correctamente el principio, se facilita la extensión y modificación del software porque no necesitarás cambiar el código que trabaja con la clase base.
- **Polimorfismo**: Permite un uso efectivo del polimorfismo, donde las clases pueden ser utilizadas de manera intercambiable en diferentes contextos.

### Conclusión

El Principio de Sustitución de Liskov es fundamental en la programación orientada a objetos, ya que promueve el uso adecuado de la herencia y asegura que las subclases sean intercambiables con sus superclases, mejorando la mantenibilidad, flexibilidad y claridad del código. Si tienes más preguntas o necesitas ejemplos adicionales sobre el LSP, estaré encantado de ayudarte.
