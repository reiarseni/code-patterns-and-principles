## Builder en Java

A continuación se muestra un ejemplo práctico y profesional del patrón Builder en Java, aplicado a la creación de un objeto complejo (por ejemplo, un equipo de cómputo) en el que existen parámetros obligatorios y opcionales. Este patrón es muy útil en aplicaciones reales donde se debe construir un objeto con muchos atributos, permitiendo configurar solo los parámetros necesarios y utilizando valores por defecto para los demás, lo cual mejora la legibilidad y evita constructores gigantes.

---

### Código de Ejemplo

```java
// Clase que representa un equipo de cómputo.
public class Computer {
    // Atributos obligatorios
    private final String CPU;
    private final String RAM;
    // Atributos opcionales
    private final String storage;
    private final String GPU;
    private final boolean bluetoothEnabled;
    private final boolean wiFiEnabled;

    // Constructor privado que recibe el builder
    private Computer(Builder builder) {
        this.CPU = builder.CPU;
        this.RAM = builder.RAM;
        this.storage = builder.storage;
        this.GPU = builder.GPU;
        this.bluetoothEnabled = builder.bluetoothEnabled;
        this.wiFiEnabled = builder.wiFiEnabled;
    }

    // Getters
    public String getCPU() { return CPU; }
    public String getRAM() { return RAM; }
    public String getStorage() { return storage; }
    public String getGPU() { return GPU; }
    public boolean isBluetoothEnabled() { return bluetoothEnabled; }
    public boolean isWiFiEnabled() { return wiFiEnabled; }

    @Override
    public String toString() {
        return "Computer [CPU=" + CPU + ", RAM=" + RAM + ", Storage=" + storage 
            + ", GPU=" + GPU + ", Bluetooth=" + bluetoothEnabled + ", WiFi=" + wiFiEnabled + "]";
    }

    // Clase Builder estática anidada
    public static class Builder {
        // Parámetros obligatorios
        private final String CPU;
        private final String RAM;
        // Parámetros opcionales con valores por defecto
        private String storage = "256GB SSD";
        private String GPU = "Integrated";
        private boolean bluetoothEnabled = false;
        private boolean wiFiEnabled = false;

        // Constructor del Builder con los parámetros obligatorios
        public Builder(String CPU, String RAM) {
            this.CPU = CPU;
            this.RAM = RAM;
        }

        // Métodos para configurar parámetros opcionales, retornando el Builder para encadenar llamadas.
        public Builder storage(String storage) {
            this.storage = storage;
            return this;
        }

        public Builder GPU(String GPU) {
            this.GPU = GPU;
            return this;
        }

        public Builder enableBluetooth(boolean enabled) {
            this.bluetoothEnabled = enabled;
            return this;
        }

        public Builder enableWiFi(boolean enabled) {
            this.wiFiEnabled = enabled;
            return this;
        }

        // Método build que crea la instancia final de Computer.
        public Computer build() {
            return new Computer(this);
        }
    }
}
```

### Ejemplo de Uso en la Aplicación

```java
public class Main {
    public static void main(String[] args) {
        // Construcción de un equipo de alto rendimiento para gaming con todos los parámetros configurados.
        Computer gamingComputer = new Computer.Builder("Intel Core i9", "32GB")
            .storage("1TB NVMe SSD")
            .GPU("NVIDIA RTX 3080")
            .enableBluetooth(true)
            .enableWiFi(true)
            .build();
        System.out.println("Gaming Computer: " + gamingComputer);

        // Construcción de un equipo para oficina usando solo los parámetros obligatorios y los valores por defecto.
        Computer officeComputer = new Computer.Builder("Intel Core i5", "16GB")
            .build();
        System.out.println("Office Computer: " + officeComputer);
    }
}
```

---

### Explicación

- **Flexibilidad y Claridad:**  
  El patrón Builder permite crear instancias de objetos complejos sin tener que pasar múltiples parámetros en un único constructor. Se evita la sobrecarga de constructores y se mejora la legibilidad del código.

- **Configuración Condicional:**  
  Al utilizar el Builder, se pueden establecer solo aquellos parámetros que son relevantes para cada caso, mientras que el resto toma valores por defecto. Esto es muy común en aplicaciones donde los objetos tienen muchas opciones de configuración.

- **Inmutabilidad:**  
  Una vez creado el objeto, sus atributos son finales, lo que garantiza que su estado no se modifique después de la construcción. Esto es útil para la seguridad y consistencia del estado en aplicaciones profesionales.

Este ejemplo es representativo de un uso real del patrón Builder en Java, facilitando la construcción de objetos complejos de forma legible y mantenible, una situación frecuente en el desarrollo de aplicaciones profesionales.
