## Facade en Python

A continuación se muestra un ejemplo práctico del patrón Facade en Python aplicado a un sistema de generación y envío de informes. En aplicaciones profesionales es común tener que orquestar varios subsistemas (como acceso a datos, generación de informes y envío de emails) para realizar una tarea compleja. El Facade simplifica el uso de estos subsistemas, proporcionando una interfaz unificada para el usuario.

---

```python
import datetime

# Subsistema 1: Servicio de base de datos
class DatabaseService:
    def connect(self):
        # Simulación de conexión a base de datos.
        print("Conectando a la base de datos...")

    def query_data(self, query: str):
        # Simula la ejecución de una consulta y retorna datos de ejemplo.
        print(f"Ejecutando consulta: {query}")
        # Datos ficticios para el reporte
        return [
            {"id": 1, "name": "Producto A", "sales": 100},
            {"id": 2, "name": "Producto B", "sales": 150},
            {"id": 3, "name": "Producto C", "sales": 200},
        ]

# Subsistema 2: Generador de informes
class ReportGenerator:
    def generate_pdf(self, data: list) -> bytes:
        # Simula la generación de un PDF a partir de los datos.
        print("Generando informe en formato PDF...")
        report_content = "Reporte de Ventas\n\n"
        for item in data:
            report_content += f"ID: {item['id']} - {item['name']} - Ventas: {item['sales']}\n"
        report_content += f"\nGenerado el {datetime.datetime.now()}"
        # En un caso real, aquí se usaría una librería para generar PDFs y retornar un archivo binario.
        return report_content.encode("utf-8")

# Subsistema 3: Servicio de email
class EmailService:
    def send_email(self, recipient: str, subject: str, body: str, attachment: bytes):
        # Simula el envío de un email con un adjunto.
        print("Enviando email...")
        print(f"Para: {recipient}")
        print(f"Asunto: {subject}")
        print(f"Cuerpo: {body}")
        print(f"Adjunto (tamaño): {len(attachment)} bytes")
        # Aquí se integraría con un servidor SMTP o API de envío de emails.

# Facade que simplifica la generación y envío de informes.
class ReportFacade:
    def __init__(self):
        self.database = DatabaseService()
        self.report_generator = ReportGenerator()
        self.email_service = EmailService()

    def generate_and_send_report(self, recipient: str):
        # Paso 1: Conectar y obtener datos
        self.database.connect()
        data = self.database.query_data("SELECT * FROM ventas")

        # Paso 2: Generar el informe en PDF
        pdf_report = self.report_generator.generate_pdf(data)

        # Paso 3: Enviar el informe por email
        subject = "Reporte de Ventas"
        body = "Adjunto encontrarás el reporte de ventas generado recientemente."
        self.email_service.send_email(recipient, subject, body, pdf_report)

# Ejemplo de uso del Facade en una aplicación real.
if __name__ == "__main__":
    report_facade = ReportFacade()
    report_facade.generate_and_send_report("gerente@example.com")
```

---

### Explicación

- **Subsistemas Independientes:**  
  Cada clase (DatabaseService, ReportGenerator, EmailService) representa un componente especializado en la aplicación. Cada uno tiene su propia complejidad y lógica interna.

- **Facade (ReportFacade):**  
  La clase ReportFacade actúa como una interfaz unificada que coordina el proceso completo de generación y envío del informe. El usuario (o cliente) solo interactúa con el método `generate_and_send_report` sin preocuparse de la orquestación interna.

- **Beneficios Prácticos:**  
  - **Simplicidad:** El cliente no necesita conocer los detalles de conexión a la base de datos, generación de PDFs o envío de emails.
  - **Mantenibilidad:** Si se cambia algún subsistema (por ejemplo, se reemplaza el servicio de email), solo se debe modificar la implementación en el Facade.
  - **Reutilización:** Los subsistemas pueden ser reutilizados en otros contextos sin depender del proceso completo.

Este patrón es muy utilizado en aplicaciones profesionales para manejar tareas complejas de forma estructurada y desacoplada, facilitando la integración de múltiples servicios y reduciendo la complejidad del código cliente.
