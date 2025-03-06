## Observer en PHP

A continuación se muestra un ejemplo práctico y profesional en PHP del patrón Observer aplicado a un sistema de notificación de eventos en una aplicación. En este caso, se utiliza un gestor de eventos que notifica a distintos observadores cuando ocurren eventos relevantes, como el registro de un usuario o la realización de un pedido.

---

```php
<?php

// Interfaz Observer: Define el método para recibir actualizaciones de eventos.
interface Observer {
    public function update(string $eventType, $data): void;
}

// Interfaz Subject: Define los métodos para adjuntar, remover y notificar observadores.
interface Subject {
    public function attach(Observer $observer): void;
    public function detach(Observer $observer): void;
    public function notify(string $eventType, $data = null): void;
}

// Clase concreta que actúa como gestor de eventos (Subject).
class EventManager implements Subject {
    private $observers = [];
    
    public function attach(Observer $observer): void {
        $this->observers[] = $observer;
    }
    
    public function detach(Observer $observer): void {
        foreach ($this->observers as $key => $obs) {
            if ($obs === $observer) {
                unset($this->observers[$key]);
            }
        }
        // Reindexar el array
        $this->observers = array_values($this->observers);
    }
    
    public function notify(string $eventType, $data = null): void {
        foreach ($this->observers as $observer) {
            $observer->update($eventType, $data);
        }
    }
}

// Observador para enviar notificaciones por email cuando se registra un usuario.
class EmailNotificationObserver implements Observer {
    public function update(string $eventType, $data): void {
        if ($eventType === 'user_registered') {
            echo "EmailNotificationObserver: Enviando email de bienvenida a {$data['email']}" . PHP_EOL;
        }
    }
}

// Observador para enviar notificaciones SMS cuando se realiza un pedido.
class SmsNotificationObserver implements Observer {
    public function update(string $eventType, $data): void {
        if ($eventType === 'order_placed') {
            echo "SmsNotificationObserver: Enviando SMS de confirmación para el pedido #{$data['order_id']}" . PHP_EOL;
        }
    }
}

// Observador para registrar en log cualquier evento que ocurra en la aplicación.
class LoggingObserver implements Observer {
    public function update(string $eventType, $data): void {
        echo "LoggingObserver: Evento '{$eventType}' ocurrido con datos: " . json_encode($data) . PHP_EOL;
    }
}

// Clase Application que simula operaciones y dispara eventos.
class Application {
    private $eventManager;
    
    public function __construct() {
        $this->eventManager = new EventManager();
    }
    
    public function getEventManager(): EventManager {
        return $this->eventManager;
    }
    
    // Simula el registro de un usuario.
    public function registerUser(array $userData): void {
        echo "Application: Registrando usuario {$userData['email']}" . PHP_EOL;
        // Notificar a los observadores sobre el evento de registro.
        $this->eventManager->notify('user_registered', $userData);
    }
    
    // Simula la realización de un pedido.
    public function placeOrder(array $orderData): void {
        echo "Application: Pedido realizado #{$orderData['order_id']}" . PHP_EOL;
        // Notificar a los observadores sobre el evento de pedido.
        $this->eventManager->notify('order_placed', $orderData);
    }
}

// Ejemplo de uso:

// Instanciar la aplicación y obtener el gestor de eventos.
$app = new Application();
$eventManager = $app->getEventManager();

// Adjuntar distintos observadores para manejar los eventos.
$eventManager->attach(new EmailNotificationObserver());
$eventManager->attach(new SmsNotificationObserver());
$eventManager->attach(new LoggingObserver());

// Simular el registro de un usuario.
$app->registerUser([
    'email' => 'usuario@example.com',
    'name'  => 'Juan Pérez'
]);

// Simular la realización de un pedido.
$app->placeOrder([
    'order_id' => 12345,
    'amount'   => 250.00
]);
```

---

### Explicación

- **EventManager (Subject):**  
  Gestiona la lista de observadores y notifica a cada uno de ellos cuando ocurre un evento. Esto permite desacoplar la lógica de disparo de eventos de la lógica de respuesta.

- **Observers:**  
  - **EmailNotificationObserver:** Envía un correo de bienvenida cuando se registra un usuario.  
  - **SmsNotificationObserver:** Envía un SMS cuando se realiza un pedido.  
  - **LoggingObserver:** Registra en log cualquier evento, lo que resulta muy útil para auditorías y debugging.

- **Application:**  
  Simula operaciones reales (registro de usuario y pedido) y utiliza el EventManager para notificar a los observadores correspondientes sin depender directamente de sus implementaciones.

Este ejemplo ilustra un caso de uso real en aplicaciones profesionales, donde el patrón Observer se emplea para gestionar notificaciones de eventos de forma flexible y escalable, permitiendo agregar o modificar comportamientos (como notificaciones o logging) sin alterar la lógica central de la aplicación.
