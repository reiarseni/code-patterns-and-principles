## Composite Python

A continuación te muestro un ejemplo real y práctico del patron Composite usando Python en el que se modela la ejecución de tareas dentro de un ciclo de desarrollo (algo muy común en aplicaciones de gestión de proyectos, sistemas de automatización o pipelines de CI/CD). Este ejemplo ilustra cómo el patrón Composite te permite tratar de manera uniforme tanto tareas simples (acciones individuales) como tareas compuestas (subconjuntos de acciones) en una estructura jerárquica.

En aplicaciones reales, podrías tener tareas que representen operaciones de compilación, pruebas, despliegue, etc., y a menudo se agrupan en workflows complejos. Con el patrón Composite, puedes agregar, eliminar o ejecutar estas tareas de forma recursiva sin preocuparte si se trata de un objeto simple o de un conjunto de ellos.

```python
from abc import ABC, abstractmethod

# Componente base: define la interfaz común para tareas simples y compuestas
class Task(ABC):
    @abstractmethod
    def get_duration(self) -> int:
        """Devuelve la duración estimada de la tarea en minutos."""
        pass

    @abstractmethod
    def execute(self):
        """Ejecuta la tarea."""
        pass

# Componente hoja: representa una tarea simple sin sub-tareas
class SimpleTask(Task):
    def __init__(self, name: str, duration: int):
        self.name = name
        self.duration = duration

    def get_duration(self) -> int:
        return self.duration

    def execute(self):
        print(f"Ejecutando tarea simple: {self.name} (duración: {self.duration} minutos)")

# Componente compuesto: puede contener otras tareas, sean simples o compuestas
class CompositeTask(Task):
    def __init__(self, name: str):
        self.name = name
        self.tasks = []  # Lista de tareas que pueden ser SimpleTask o CompositeTask

    def add_task(self, task: Task):
        self.tasks.append(task)

    def remove_task(self, task: Task):
        self.tasks.remove(task)

    def get_duration(self) -> int:
        # Suma la duración de todas las tareas hijas
        return sum(task.get_duration() for task in self.tasks)

    def execute(self):
        print(f"\nIniciando tarea compuesta: {self.name} (duración total estimada: {self.get_duration()} minutos)")
        for task in self.tasks:
            task.execute()
        print(f"Tarea compuesta {self.name} completada.\n")

# Ejemplo de uso en un flujo de trabajo real de desarrollo
def main():
    # Tareas simples que se podrían encontrar en un ciclo de desarrollo
    implementar_funcionalidad = SimpleTask("Implementar funcionalidad X", 120)
    escribir_tests = SimpleTask("Escribir pruebas unitarias", 60)
    revision_codigo = SimpleTask("Revisión de código", 30)
    
    # Se agrupa el ciclo de desarrollo en una tarea compuesta
    ciclo_desarrollo = CompositeTask("Ciclo de Desarrollo")
    ciclo_desarrollo.add_task(implementar_funcionalidad)
    ciclo_desarrollo.add_task(escribir_tests)
    ciclo_desarrollo.add_task(revision_codigo)

    # Otra tarea simple, como el despliegue, que puede estar en otro componente del workflow
    despliegue = SimpleTask("Desplegar a producción", 20)

    # Se compone un proyecto completo que incluye el ciclo de desarrollo y el despliegue
    proyecto_completo = CompositeTask("Proyecto Completo")
    proyecto_completo.add_task(ciclo_desarrollo)
    proyecto_completo.add_task(despliegue)

    # Se ejecuta el proyecto completo de manera uniforme, sin diferenciar entre tareas simples y compuestas
    proyecto_completo.execute()

if __name__ == '__main__':
    main()
```

### Explicación

1. **Interfaz Común:**  
   La clase abstracta `Task` define los métodos que deben implementar tanto las tareas simples como las compuestas. Esto es esencial para que el cliente pueda tratar ambas de la misma forma.

2. **Tareas Simples:**  
   La clase `SimpleTask` implementa las acciones concretas: tiene un nombre y una duración. Su método `execute()` simula la ejecución de la tarea imprimiendo un mensaje.

3. **Tareas Compuestas:**  
   La clase `CompositeTask` mantiene una lista de tareas hijas. Permite agregar y remover tareas, y su método `execute()` recorre cada tarea ejecutándola. Además, el método `get_duration()` calcula la duración total sumando las duraciones de todas las tareas hijas.

4. **Uso en la Vida Real:**  
   Este patrón es muy útil para modelar procesos o workflows en sistemas de automatización, donde un proceso mayor se compone de múltiples subtareas. Al encapsular la complejidad en la estructura compuesta, el programador puede manipular el conjunto completo sin necesidad de distinguir entre tipos de tareas.

Este ejemplo demuestra cómo el patrón Composite se utiliza en aplicaciones profesionales para manejar jerarquías de objetos (en este caso, tareas) de manera uniforme y flexible, facilitando la extensión y el mantenimiento del código.
