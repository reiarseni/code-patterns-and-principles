## Message Broker en Python

A continuación se muestra un ejemplo completo en Python que implementa un sistema de colas asíncronas y persistencia configurable, con múltiples consumidores y productores. El ejemplo integra varios patrones de diseño:

- **Singleton:** para la configuración global (por ejemplo, modo de persistencia y retardo por defecto).  
- **Factory:** para crear mensajes y para instanciar el proveedor de persistencia.  
- **Strategy:** para definir la estrategia de retardo en la entrega (constante o aleatoria).  
- **Observer:** para notificar a los productores (emisores) cuando sus mensajes han sido entregados.  
- **Facade:** el MessageBroker unifica la lógica de encolado, persistencia y notificación, ocultando la complejidad interna.

El sistema simula lo que hace un sistema de mensajería (tipo RabbitMQ) de forma simplificada: los productores publican mensajes en la cola y múltiples consumidores se suscriben a ella, procesando mensajes a su ritmo. Además, la persistencia por defecto es a archivos JSON en disco, pero se puede configurar para usar una base de datos.

Guarda el siguiente código en un archivo (por ejemplo, `async_message_queue.py`) y ejecútalo con Python 3.7+.

---

```python
import asyncio
import json
import os
import random
import time
import uuid

# ============================================================
# PATRÓN SINGLETON: Configuración Global
# ============================================================
class Config:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(Config, cls).__new__(cls)
            # Configuración global: modo de persistencia ("json" o "db") y retardo por defecto
            cls._instance.persistence_mode = "json"  # Por defecto, persistencia a archivos JSON
            cls._instance.default_delay = 2          # Retardo constante por defecto (segundos)
        return cls._instance

# ============================================================
# PATRÓN FACTORY: Proveedores de Persistencia
# ============================================================
class IPersistenceProvider:
    async def save_message(self, message):
        raise NotImplementedError

    async def load_messages(self) -> list:
        raise NotImplementedError

class JsonFilePersistence(IPersistenceProvider):
    def __init__(self, filename="messages.json"):
        self.filename = filename
        if not os.path.exists(self.filename):
            with open(self.filename, "w") as f:
                json.dump([], f)

    async def save_message(self, message):
        # Cargamos los mensajes existentes, añadimos el nuevo y guardamos.
        try:
            with open(self.filename, "r") as f:
                data = json.load(f)
        except Exception:
            data = []
        # Convertir el mensaje a diccionario
        data.append({
            "id": message.id,
            "content": message.content,
            "sender": message.sender,
            "recipient": message.recipient,
            "timestamp_sent": message.timestamp_sent,
            "timestamp_delivered": message.timestamp_delivered
        })
        with open(self.filename, "w") as f:
            json.dump(data, f)
        return True

    async def load_messages(self) -> list:
        with open(self.filename, "r") as f:
            data = json.load(f)
        return data

class DatabasePersistence(IPersistenceProvider):
    # Implementación simulada de persistencia a base de datos
    async def save_message(self, message):
        print(f"[DB Persistence] Guardando mensaje {message.id} en la base de datos.")
        return True

    async def load_messages(self) -> list:
        print("[DB Persistence] Cargando mensajes de la base de datos.")
        return []

class PersistenceFactory:
    @staticmethod
    def create_persistence_provider(mode: str) -> IPersistenceProvider:
        if mode == "json":
            return JsonFilePersistence()
        elif mode == "db":
            return DatabasePersistence()
        else:
            return JsonFilePersistence()

# ============================================================
# MODELO DE MENSAJE Y FACTORY (PATRÓN FACTORY)
# ============================================================
class Message:
    def __init__(self, content, sender, recipient):
        self.id = str(uuid.uuid4())
        self.content = content
        self.sender = sender
        self.recipient = recipient
        self.timestamp_sent = time.time()
        self.timestamp_delivered = None

    def __str__(self):
        return f"Message({self.id}) from {self.sender} to {self.recipient}: {self.content}"

class MessageFactory:
    @staticmethod
    def create_message(content, sender, recipient):
        return Message(content, sender, recipient)

# ============================================================
# PATRÓN STRATEGY: Estrategias para determinar el delay de entrega
# ============================================================
class DeliveryDelayStrategy:
    async def get_delay(self):
        raise NotImplementedError

class ConstantDelayStrategy(DeliveryDelayStrategy):
    def __init__(self, delay):
        self.delay = delay

    async def get_delay(self):
        return self.delay

class RandomDelayStrategy(DeliveryDelayStrategy):
    async def get_delay(self):
        return random.uniform(1, 5)

# ============================================================
# PATRÓN OBSERVER y FACÁDE: Broker de Mensajes
# ============================================================
class MessageBroker:
    def __init__(self, delay_strategy: DeliveryDelayStrategy, persistence_provider: IPersistenceProvider):
        self.queue = asyncio.Queue()  # Cola asíncrona para almacenar mensajes
        self.observers = []           # Lista de observadores (emisores) para notificaciones
        self.delay_strategy = delay_strategy
        self.persistence_provider = persistence_provider

    def register_observer(self, observer):
        if observer not in self.observers:
            self.observers.append(observer)

    def unregister_observer(self, observer):
        self.observers = [obs for obs in self.observers if obs != observer]

    async def notify_observers(self, message, delay):
        # Notificar a cada observador que el mensaje fue entregado, con el retardo
        for observer in self.observers:
            await observer.notify(message, delay)

    async def send_message(self, message: Message):
        # Persistir el mensaje al encolarlo y luego agregarlo a la cola
        await self.persistence_provider.save_message(message)
        await self.queue.put(message)
        print(f"[Broker] Mensaje encolado: {message}")

    async def process_messages(self, consumer_id):
        # Cada consumidor procesa mensajes de la cola a su ritmo
        while True:
            message = await self.queue.get()
            # Se obtiene el retardo según la estrategia (podría ser constante o aleatorio)
            delay = await self.delay_strategy.get_delay()
            await asyncio.sleep(delay)
            message.timestamp_delivered = time.time()
            delivery_delay = message.timestamp_delivered - message.timestamp_sent
            print(f"[Broker][{consumer_id}] Entregado {message} con retardo {delivery_delay:.2f} segundos")
            # Persistir el mensaje entregado (se podría actualizar el registro)
            await self.persistence_provider.save_message(message)
            # Notificar a los productores (observadores) que el mensaje fue entregado
            await self.notify_observers(message, delivery_delay)
            self.queue.task_done()

# ============================================================
# PRODUCTOR Y OBSERVADOR: Messenger
# ============================================================
class Messenger:
    def __init__(self, name, broker: MessageBroker):
        self.name = name
        self.broker = broker
        # Se registra como observador para recibir notificaciones de entrega
        self.broker.register_observer(self)

    async def send_message(self, content, recipient):
        message = MessageFactory.create_message(content, self.name, recipient)
        await self.broker.send_message(message)

    async def notify(self, message: Message, delay):
        # Solo notifica al emisor de su propio mensaje
        if message.sender == self.name:
            print(f"[Messenger: {self.name}] Tu mensaje {message.id} fue entregado con retardo {delay:.2f} segundos.")

# ============================================================
# EJECUCIÓN PRINCIPAL ASÍNCRONA
# ============================================================
async def main():
    # Inicializar configuración (Singleton)
    config = Config()
    # Seleccionar el proveedor de persistencia mediante la Factory
    persistence_provider = PersistenceFactory.create_persistence_provider(config.persistence_mode)
    # Seleccionar la estrategia de retardo: constante o aleatoria
    delay_strategy = ConstantDelayStrategy(config.default_delay)
    # Para usar retardo aleatorio, descomenta la siguiente línea:
    # delay_strategy = RandomDelayStrategy()

    # Crear el broker que unifica la lógica (Facade)
    broker = MessageBroker(delay_strategy, persistence_provider)

    # Crear múltiples consumidores que procesan mensajes de la cola a su ritmo
    consumer1 = asyncio.create_task(broker.process_messages("Consumer1"))
    consumer2 = asyncio.create_task(broker.process_messages("Consumer2"))

    # Crear productores (emisores) y registrarlos en el broker (Observer)
    alice = Messenger("Alice", broker)
    bob = Messenger("Bob", broker)

    # Los productores publican mensajes en la cola
    await alice.send_message("Hola Bob!", "Bob")
    await bob.send_message("Hola Alice, ¿cómo estás?", "Alice")
    await alice.send_message("Todo bien, gracias.", "Bob")
    await bob.send_message("Genial, seguimos en contacto.", "Alice")

    # Esperar para que los consumidores procesen todos los mensajes
    await asyncio.sleep(15)

    # Cancelar tareas de consumidores (para finalizar el ejemplo)
    consumer1.cancel()
    consumer2.cancel()

if __name__ == '__main__':
    asyncio.run(main())
```

---

## Explicación del Código y Conceptos en el Mundo Real

### Funcionamiento del Sistema

1. **Producción y Persistencia:**  
   - Los productores (instancias de `Messenger`) crean mensajes mediante la `MessageFactory` y los envían al `MessageBroker` con el método `send_message()`.  
   - Al encolar el mensaje, se persiste en disco usando el proveedor de persistencia configurado (por defecto, `JsonFilePersistence`).  
   - En un sistema real (como RabbitMQ), los mensajes se persisten para evitar pérdida en caso de fallos.

2. **Cola y Consumo:**  
   - El `MessageBroker` mantiene una cola asíncrona (`asyncio.Queue`) a la que se agregan los mensajes.  
   - Se inician múltiples tareas consumidoras (por ejemplo, "Consumer1" y "Consumer2") que se suscriben a la cola y extraen mensajes para procesarlos de forma independiente.  
   - Cada consumidor simula un retardo (definido mediante una estrategia de delay) antes de "entregar" el mensaje, emulando la latencia en la red o en el procesamiento.

3. **Notificación (Observer):**  
   - Una vez entregado un mensaje, el broker notifica a los productores (observadores) mediante el método `notify()`, indicando que su mensaje fue entregado y con qué retardo.  
   - Esto permite que los productores tengan retroalimentación sobre el estado de sus mensajes.

4. **Persistencia Configurable:**  
   - Gracias al patrón Factory y Singleton, la persistencia es configurable a través de la clase `Config`.  
   - Por defecto, se utiliza un proveedor que almacena los mensajes en un archivo JSON en disco, pero se puede cambiar fácilmente a otro proveedor (por ejemplo, `DatabasePersistence`) sin modificar la lógica principal.

### Aplicación en el Mundo Real

En sistemas reales de mensajería (como RabbitMQ, Apache Kafka o Amazon SQS):

- **Desacoplamiento:** Los productores publican mensajes sin preocuparse de quién los consume, y los consumidores se encargan de procesarlos a su ritmo.  
- **Persistencia y Fiabilidad:** Los mensajes se almacenan de forma persistente para garantizar la entrega incluso si ocurre un fallo en el sistema.  
- **Escalabilidad:** Es posible tener múltiples consumidores procesando la cola en paralelo, lo que permite distribuir la carga y mejorar el rendimiento.  
- **Notificación y Monitoreo:** Los sistemas suelen proveer mecanismos de retroalimentación, permitiendo a los productores saber cuándo y cómo fueron entregados los mensajes.

Este ejemplo educativo, utilizando patrones de diseño modernos, ilustra cómo se pueden combinar estas ideas para crear un sistema modular, escalable y fácil de mantener, similar en concepto a los sistemas de mensajería de nivel empresarial.
