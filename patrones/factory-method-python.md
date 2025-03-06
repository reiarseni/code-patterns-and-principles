## Factory Method en Python

A continuación se muestra un ejemplo en Python del patrón Factory Method aplicado a la creación de analizadores (parsers) de archivos. Este es un caso real y clásico en aplicaciones que deben procesar múltiples formatos (CSV, JSON, XML), permitiendo la extensión sin modificar el código cliente.

---

### Ejemplo del Patrón Factory Method

```python
from abc import ABC, abstractmethod

# Producto abstracto: define la interfaz para los parsers.
class FileParser(ABC):
    @abstractmethod
    def parse(self, file_path: str) -> None:
        pass

# Productos concretos
class CSVParser(FileParser):
    def parse(self, file_path: str) -> None:
        # Lógica de parsing para archivos CSV.
        print(f"Parsing CSV file: {file_path}")
        # Aquí iría la implementación real para leer y procesar el archivo CSV.

class JSONParser(FileParser):
    def parse(self, file_path: str) -> None:
        # Lógica de parsing para archivos JSON.
        print(f"Parsing JSON file: {file_path}")
        # Aquí iría la implementación real para leer y procesar el archivo JSON.

class XMLParser(FileParser):
    def parse(self, file_path: str) -> None:
        # Lógica de parsing para archivos XML.
        print(f"Parsing XML file: {file_path}")
        # Aquí iría la implementación real para leer y procesar el archivo XML.

# Clase Fábrica que implementa el Factory Method.
class ParserFactory:
    @staticmethod
    def get_parser(file_type: str) -> FileParser:
        if file_type.lower() == 'csv':
            return CSVParser()
        elif file_type.lower() == 'json':
            return JSONParser()
        elif file_type.lower() == 'xml':
            return XMLParser()
        else:
            raise ValueError(f"Parser for file type '{file_type}' not implemented.")

# Ejemplo de uso del Factory Method.
if __name__ == "__main__":
    # Supongamos que el tipo de archivo se determina en tiempo de ejecución.
    file_type = 'json'  # Puede ser 'csv', 'json' o 'xml'
    file_path = "data.json"
    
    # Se utiliza la fábrica para obtener el parser adecuado sin conocer su implementación concreta.
    parser = ParserFactory.get_parser(file_type)
    parser.parse(file_path)
```

---

### Explicación

- **Producto abstracto:**  
  La clase `FileParser` define la interfaz común que deben implementar todos los parsers.

- **Productos concretos:**  
  Se implementan clases como `CSVParser`, `JSONParser` y `XMLParser`, cada una con su lógica específica para procesar un tipo de archivo.

- **Factory Method:**  
  La clase `ParserFactory` contiene el método estático `get_parser` que, según el parámetro recibido (por ejemplo, 'csv', 'json' o 'xml'), crea y retorna una instancia del parser adecuado.

- **Uso práctico:**  
  Al separar la creación de objetos de su uso, se facilita la extensión (por ejemplo, agregando un nuevo parser para otro formato) sin modificar el código cliente. Este patrón es común en aplicaciones que deben soportar múltiples formatos de datos y es un ejemplo clásico del Factory Method en la vida real.
