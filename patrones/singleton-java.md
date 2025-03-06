## Singleton en Java

A continuación se muestra un ejemplo práctico y profesional del patrón Singleton en Java, aplicado a un administrador de configuración (ConfigurationManager) que carga parámetros de un archivo de propiedades. Este tipo de clase es muy común en aplicaciones reales, ya que centraliza la configuración de la aplicación en una única instancia accesible desde cualquier parte del código, asegurando consistencia y evitando cargas redundantes del archivo de configuración.

---

### Ejemplo de Código

#### Archivo de propiedades (config.properties)
*(Colócalo en el directorio raíz o en el classpath de tu aplicación)*

```
# config.properties
db.url=jdbc:mysql://localhost:3306/miDB
db.user=root
db.password=secret
app.name=MiAplicacion
```

#### Clase ConfigurationManager (Singleton)

```java
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class ConfigurationManager {
    // La variable 'instance' es volatile para asegurar visibilidad en entornos multihilo.
    private static volatile ConfigurationManager instance;
    private Properties properties;

    // Constructor privado para evitar instanciación directa.
    private ConfigurationManager() {
        properties = new Properties();
        try (InputStream input = new FileInputStream("config.properties")) {
            properties.load(input);
            System.out.println("Configuración cargada correctamente.");
        } catch (IOException ex) {
            System.err.println("Error al cargar la configuración: " + ex.getMessage());
        }
    }

    // Método getInstance() implementado con doble verificación para asegurar seguridad en entornos concurrentes.
    public static ConfigurationManager getInstance() {
        if (instance == null) {
            synchronized (ConfigurationManager.class) {
                if (instance == null) {
                    instance = new ConfigurationManager();
                }
            }
        }
        return instance;
    }

    // Método para obtener un valor de configuración por clave.
    public String getProperty(String key) {
        return properties.getProperty(key);
    }
}
```

#### Clase de Ejemplo de Uso

```java
public class Application {
    public static void main(String[] args) {
        // Se obtiene la instancia única de ConfigurationManager.
        ConfigurationManager configManager = ConfigurationManager.getInstance();

        // Se acceden a los parámetros de configuración de forma centralizada.
        String dbUrl = configManager.getProperty("db.url");
        String dbUser = configManager.getProperty("db.user");
        String appName = configManager.getProperty("app.name");

        System.out.println("Aplicación: " + appName);
        System.out.println("Conectando a la base de datos en: " + dbUrl);
        System.out.println("Usuario: " + dbUser);

        // Ejemplo: en otra parte del código, se puede llamar a getInstance() sin volver a cargar la configuración.
        ConfigurationManager anotherReference = ConfigurationManager.getInstance();
        System.out.println("Acceso centralizado, misma instancia: " + (configManager == anotherReference));
    }
}
```

---

### Explicación

- **Centralización y Consistencia:**  
  El patrón Singleton se utiliza para asegurar que la configuración de la aplicación se cargue una sola vez. Esto es especialmente importante en aplicaciones empresariales donde múltiples componentes dependen de la misma configuración.

- **Doble Verificación y Seguridad en Entornos Multihilo:**  
  La implementación con doble verificación (double-checked locking) y la palabra clave `volatile` garantizan que la instancia se cree de manera segura incluso cuando varios hilos intentan acceder a ella simultáneamente.

- **Uso Profesional:**  
  Administrar la configuración a través de un Singleton simplifica la gestión de parámetros globales, evitando inconsistencias y sobrecargas en la carga de archivos de configuración. Este enfoque es muy común en aplicaciones empresariales, frameworks y sistemas de microservicios.

Este ejemplo muestra cómo aplicar el patrón Singleton en un caso real y útil en el día a día de un programador, permitiendo centralizar y reutilizar la configuración de una aplicación de manera segura y eficiente.
