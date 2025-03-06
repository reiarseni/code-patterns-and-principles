## Principio de Inversion de Dependencias (Dependency Inversion Principle, DIP)

A continuación se muestra un ejemplo en PHP que ilustra cómo refactorizar una clase para cumplir con el Principio de Inversión de Dependencias (DIP). Usaremos el caso real del envío de notificaciones, donde inicialmente la clase depende directamente de una implementación concreta (por ejemplo, envío de emails) y, tras refactorizar, depende de una abstracción que permite inyectar diferentes mecanismos de envío (como SMS, push, etc.).

---

### Ejemplo Sin Aplicar DIP

En este ejemplo, la clase `Notification` crea una instancia directa de `EmailService`, lo que genera una dependencia fuerte de una implementación concreta:

```php
<?php
class EmailService {
    public function send(string $recipient, string $subject, string $message): void {
        // Simula el envío de un email
        echo "Enviando email a $recipient: $subject - $message\n";
    }
}

class Notification {
    public function sendNotification(object $user): void {
        // Dependencia directa de EmailService
        $emailService = new EmailService();
        $emailService->send($user->email, "Notificación", "Este es un mensaje de notificación.");
    }
}

// Uso del código sin DIP
$user = (object) ['email' => 'usuario@example.com'];
$notification = new Notification();
$notification->sendNotification($user);
?>
```

**Problema:**  
La clase `Notification` depende directamente de `EmailService`, lo que dificulta la extensión o sustitución del mecanismo de envío (por ejemplo, para enviar SMS o notificaciones push) y complica la realización de pruebas unitarias.

---

### Ejemplo Refactorizado Aplicando DIP

Para aplicar el DIP, se introduce una abstracción (interfaz) que define el comportamiento de envío de notificaciones. De esta forma, la clase `Notification` depende de la abstracción y no de implementaciones concretas:

```php
<?php
// Interfaz que define la abstracción para el envío de notificaciones
interface NotificationSender {
    public function send(string $recipient, string $subject, string $message): void;
}

// Implementación concreta para el envío de emails
class EmailNotificationSender implements NotificationSender {
    public function send(string $recipient, string $subject, string $message): void {
        echo "Enviando email a $recipient: $subject - $message\n";
    }
}

// Otra implementación concreta, por ejemplo, para enviar SMS
class SMSNotificationSender implements NotificationSender {
    public function send(string $recipient, string $subject, string $message): void {
        echo "Enviando SMS a $recipient: $subject - $message\n";
    }
}

// La clase Notification depende ahora de la abstracción NotificationSender
class Notification {
    private NotificationSender $sender;
    
    public function __construct(NotificationSender $sender) {
        $this->sender = $sender;
    }
    
    public function sendNotification(object $user): void {
        // Se utiliza la abstracción para enviar la notificación
        $this->sender->send($user->contact, "Notificación", "Este es un mensaje de notificación.");
    }
}

// Uso del código refactorizado con DIP

// Envío de notificación por email
$user = (object) ['contact' => 'usuario@example.com'];
$emailSender = new EmailNotificationSender();
$notification = new Notification($emailSender);
$notification->sendNotification($user);

// Envío de notificación por SMS
$smsSender = new SMSNotificationSender();
$notification = new Notification($smsSender);
$user->contact = '123456789'; // Simulando número de teléfono
$notification->sendNotification($user);
?>
```

**Explicación:**

- **Antes (Sin DIP):**  
  La clase `Notification` crea directamente una instancia de `EmailService`. Esto genera una dependencia fuerte de una implementación concreta, dificultando la extensión y el mantenimiento.

- **Después (Con DIP):**  
  La clase `Notification` depende de la abstracción `NotificationSender`. Se inyecta la implementación concreta (por ejemplo, `EmailNotificationSender` o `SMSNotificationSender`) mediante el constructor. Esto permite cambiar el mecanismo de envío sin modificar la clase `Notification`, promoviendo mayor flexibilidad, escalabilidad y facilidad para realizar pruebas.

### Conclusión

Este ejemplo demuestra cómo refactorizar un código para cumplir con el DIP, un patrón de diseño ampliamente utilizado en aplicaciones reales para mejorar la mantenibilidad y la extensibilidad del código.
