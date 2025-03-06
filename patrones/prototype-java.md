## Prototype en Java

Aquí tienes un ejemplo práctico y realista del patrón **Prototype** en Java, aplicado a la clonación de configuraciones de reportes en un sistema de generación de informes empresariales.  

---

### **Caso de Uso**  
En muchas aplicaciones empresariales, los reportes tienen múltiples configuraciones (formato, filtros, columnas, etc.), y generar estos objetos desde cero puede ser costoso. En lugar de crear cada configuración manualmente, podemos definir un **prototipo** y clonarlo para diferentes escenarios, personalizándolo según sea necesario.  

Este patrón es especialmente útil en sistemas de **BI (Business Intelligence)**, **generadores de reportes dinámicos**, y **plataformas de analítica de datos**.

---

### **Implementación en Java**  

```java
import java.util.ArrayList;
import java.util.List;

// Clase que representa la configuración de un reporte
class ReportConfig implements Cloneable {
    private String title;
    private String format;
    private List<String> columns;
    private boolean includeSummary;

    public ReportConfig(String title, String format) {
        this.title = title;
        this.format = format;
        this.columns = new ArrayList<>();
        this.includeSummary = false;
    }

    public void addColumn(String column) {
        columns.add(column);
    }

    public void setIncludeSummary(boolean includeSummary) {
        this.includeSummary = includeSummary;
    }

    @Override
    public ReportConfig clone() {
        try {
            ReportConfig clone = (ReportConfig) super.clone();
            clone.columns = new ArrayList<>(this.columns); // Copia profunda de la lista de columnas
            return clone;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException("Error al clonar la configuración del reporte", e);
        }
    }

    @Override
    public String toString() {
        return "ReportConfig{" +
                "title='" + title + '\'' +
                ", format='" + format + '\'' +
                ", columns=" + columns +
                ", includeSummary=" + includeSummary +
                '}';
    }
}

// Uso del patrón Prototype
public class PrototypeExample {
    public static void main(String[] args) {
        // Creamos un prototipo base para reportes en formato PDF
        ReportConfig prototypeReport = new ReportConfig("Sales Report", "PDF");
        prototypeReport.addColumn("Product");
        prototypeReport.addColumn("Revenue");
        prototypeReport.addColumn("Date");

        System.out.println("=== Prototipo Base ===");
        System.out.println(prototypeReport);

        // Clonamos el prototipo y personalizamos para un reporte de ventas mensual
        ReportConfig monthlyReport = prototypeReport.clone();
        monthlyReport.setIncludeSummary(true);
        monthlyReport.addColumn("Profit");

        System.out.println("\n=== Reporte de Ventas Mensual (Clonado y Modificado) ===");
        System.out.println(monthlyReport);

        // Clonamos otro reporte para el departamento de Finanzas con diferente formato
        ReportConfig financeReport = prototypeReport.clone();
        financeReport.setIncludeSummary(true);

        System.out.println("\n=== Reporte para Finanzas (Clonado y Modificado) ===");
        System.out.println(financeReport);

        // Verificamos que el prototipo original no haya sido afectado
        System.out.println("\n=== Verificación del Prototipo Original ===");
        System.out.println(prototypeReport);
    }
}
```

---

### **Explicación y Aplicación Real**
1. **Reutilización de Configuraciones:**  
   - En sistemas de generación de reportes empresariales, crear configuraciones desde cero puede ser tedioso.  
   - El **patrón Prototype** permite clonar un reporte base y modificarlo para diferentes escenarios.  

2. **Optimización de Recursos:**  
   - Se evita la recreación de objetos costosos (listas de columnas, formatos, opciones avanzadas).  
   - Se mejora la eficiencia, especialmente en sistemas que generan múltiples reportes dinámicos.  

3. **Clonación Segura:**  
   - Se usa `clone()` para duplicar la configuración base, asegurando que cada reporte sea independiente.  
   - Se hace una **copia profunda** de la lista de columnas para evitar modificaciones accidentales en el original.  

---

### **Ejemplo Real en la Industria**
Este enfoque se usa en **herramientas de BI**, **sistemas de generación de reportes en ERP**, y **aplicaciones de analítica de datos**. Empresas que trabajan con grandes volúmenes de datos (bancos, aseguradoras, retail) suelen aplicar el **patrón Prototype** para gestionar configuraciones predefinidas y personalizadas de reportes.  

Este patrón permite crear rápidamente variantes de un mismo reporte sin necesidad de escribir código adicional ni modificar la configuración base.
