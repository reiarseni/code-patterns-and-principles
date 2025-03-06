## Template Method e Python

A continuación se muestra un ejemplo práctico y profesional del patrón Template Method en Python, aplicado a un proceso de importación de datos desde diferentes fuentes (por ejemplo, archivos CSV y JSON). Este ejemplo es muy útil en aplicaciones reales donde se debe definir un flujo de trabajo común (conectar, leer, parsear, validar y guardar datos) pero que varía en algunos pasos según el tipo de fuente de datos. El Template Method permite definir el esqueleto del algoritmo en una clase base y delegar los pasos específicos a las subclases, facilitando la extensión y el mantenimiento del código.

---

```python
from abc import ABC, abstractmethod
import csv
import json

class DataImporter(ABC):
    """
    Clase abstracta que define el esqueleto del proceso de importación de datos.
    """
    def import_data(self, source: str) -> None:
        """
        Método plantilla (template method) que orquesta el proceso de importación.
        """
        self.connect(source)
        raw_data = self.read_data(source)
        data = self.parse_data(raw_data)
        if self.validate_data(data):
            self.save_data(data)
        else:
            print("Validación de datos fallida.")
        self.disconnect(source)
    
    def connect(self, source: str) -> None:
        print(f"Conectando a la fuente: {source}")
    
    @abstractmethod
    def read_data(self, source: str) -> str:
        """
        Lee los datos sin procesar desde la fuente.
        """
        pass
    
    @abstractmethod
    def parse_data(self, raw_data: str):
        """
        Parsea los datos sin procesar a una estructura de datos útil.
        """
        pass
    
    def validate_data(self, data) -> bool:
        """
        Valida los datos parseados. La implementación por defecto asume que los datos son válidos.
        """
        print("Validando datos...")
        return True  # En un escenario real se implementarían validaciones concretas.
    
    @abstractmethod
    def save_data(self, data) -> None:
        """
        Guarda los datos en el destino (por ejemplo, base de datos).
        """
        pass
    
    def disconnect(self, source: str) -> None:
        print(f"Desconectando de la fuente: {source}")

class CSVDataImporter(DataImporter):
    """
    Importador de datos para archivos CSV.
    """
    def read_data(self, source: str) -> str:
        print("Leyendo datos CSV...")
        with open(source, newline='', encoding='utf-8') as csvfile:
            return csvfile.read()
    
    def parse_data(self, raw_data: str):
        print("Parseando datos CSV...")
        # Usamos csv.reader para procesar el contenido CSV
        lines = raw_data.splitlines()
        reader = csv.reader(lines)
        return [row for row in reader]
    
    def save_data(self, data) -> None:
        print("Guardando datos CSV en la base de datos (simulado)...")
        for row in data:
            print(f"Inserción: {row}")

class JSONDataImporter(DataImporter):
    """
    Importador de datos para archivos JSON.
    """
    def read_data(self, source: str) -> str:
        print("Leyendo datos JSON...")
        with open(source, 'r', encoding='utf-8') as jsonfile:
            return jsonfile.read()
    
    def parse_data(self, raw_data: str):
        print("Parseando datos JSON...")
        return json.loads(raw_data)
    
    def save_data(self, data) -> None:
        print("Guardando datos JSON en la base de datos (simulado)...")
        print(f"Inserción: {data}")

# Ejemplo de uso en una aplicación real
if __name__ == "__main__":
    # Suponiendo que existen archivos 'data.csv' y 'data.json' en el mismo directorio.
    
    print("=== Importación de CSV ===")
    csv_importer = CSVDataImporter()
    csv_importer.import_data("data.csv")
    
    print("\n=== Importación de JSON ===")
    json_importer = JSONDataImporter()
    json_importer.import_data("data.json")
```

---

### Explicación

- **Clase Base con Template Method:**  
  La clase `DataImporter` define el método `import_data()`, que es el esqueleto del proceso de importación: conectar a la fuente, leer los datos sin procesar, parsearlos, validarlos, guardarlos y finalmente desconectar. Algunos de estos pasos se implementan por defecto (por ejemplo, `connect` y `disconnect`), mientras que otros se dejan como abstractos (`read_data`, `parse_data`, `save_data`) para que las subclases los implementen según el tipo de fuente.

- **Subclases Especializadas:**  
  - `CSVDataImporter` implementa la lectura y el parseo de archivos CSV utilizando el módulo `csv` de Python.
  - `JSONDataImporter` implementa la lectura y el parseo de archivos JSON utilizando el módulo `json`.

- **Aplicación Profesional:**  
  En aplicaciones reales, es común tener procesos de importación de datos desde diversas fuentes. El patrón Template Method permite definir un flujo de trabajo común y consistente, mientras se adapta a las particularidades de cada fuente sin duplicar lógica ni romper la estructura general del algoritmo.

Este ejemplo demuestra cómo aplicar el patrón Template Method para manejar procesos de importación de datos de manera estructurada y flexible, un escenario muy común en el desarrollo profesional de software.
