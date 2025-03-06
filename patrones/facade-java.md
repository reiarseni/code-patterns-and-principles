## Facade en Java 

A continuación se muestra un ejemplo práctico y profesional del patrón Facade en Java aplicado al procesamiento de un pedido en un sistema de e-commerce. En aplicaciones reales es común tener que interactuar con varios subsistemas (como pago, inventario, envío y notificaciones) para completar una operación compleja. El Facade simplifica la interacción del cliente con estos subsistemas, ofreciendo una interfaz unificada para realizar la operación de manera sencilla.

---

### Código de Ejemplo

#### Subsistema 1: Servicio de Pago

```java
public interface PaymentService {
    boolean processPayment(double amount);
}

public class PaymentServiceImpl implements PaymentService {
    @Override
    public boolean processPayment(double amount) {
        // Lógica para procesar el pago (simulación)
        System.out.println("Procesando el pago de $" + amount);
        return true; // Se asume que el pago es exitoso
    }
}
```

#### Subsistema 2: Servicio de Inventario

```java
public interface InventoryService {
    boolean checkStock(String productId, int quantity);
    void updateInventory(String productId, int quantity);
}

public class InventoryServiceImpl implements InventoryService {
    @Override
    public boolean checkStock(String productId, int quantity) {
        // Lógica para verificar el stock (simulación)
        System.out.println("Verificando stock del producto " + productId + " para cantidad: " + quantity);
        return true; // Se asume que hay stock suficiente
    }
    
    @Override
    public void updateInventory(String productId, int quantity) {
        // Lógica para actualizar el inventario (simulación)
        System.out.println("Actualizando inventario: reduciendo " + quantity + " unidades del producto " + productId);
    }
}
```

#### Subsistema 3: Servicio de Envío

```java
public interface ShippingService {
    String shipOrder(String productId, int quantity);
}

public class ShippingServiceImpl implements ShippingService {
    @Override
    public String shipOrder(String productId, int quantity) {
        // Lógica para enviar el pedido (simulación)
        String trackingNumber = "TRACK" + System.currentTimeMillis();
        System.out.println("Enviando pedido del producto " + productId + " (cantidad: " + quantity + "). Número de seguimiento: " + trackingNumber);
        return trackingNumber;
    }
}
```

#### Subsistema 4: Servicio de Notificaciones

```java
public interface NotificationService {
    void sendNotification(String message);
}

public class NotificationServiceImpl implements NotificationService {
    @Override
    public void sendNotification(String message) {
        // Lógica para enviar notificaciones (simulación)
        System.out.println("Enviando notificación: " + message);
    }
}
```

#### Facade: OrderFacade

La clase `OrderFacade` coordina la interacción entre todos los subsistemas para procesar un pedido de manera simple.

```java
public class OrderFacade {
    private PaymentService paymentService;
    private InventoryService inventoryService;
    private ShippingService shippingService;
    private NotificationService notificationService;
    
    public OrderFacade() {
        // En un escenario real, estos servicios podrían inyectarse mediante un framework de inyección de dependencias
        this.paymentService = new PaymentServiceImpl();
        this.inventoryService = new InventoryServiceImpl();
        this.shippingService = new ShippingServiceImpl();
        this.notificationService = new NotificationServiceImpl();
    }
    
    public void placeOrder(String productId, int quantity, double amount) {
        System.out.println("Iniciando proceso de pedido para el producto: " + productId);
        
        // 1. Verificar el stock disponible
        if (!inventoryService.checkStock(productId, quantity)) {
            System.out.println("Stock insuficiente para el producto: " + productId);
            return;
        }
        
        // 2. Procesar el pago
        if (!paymentService.processPayment(amount)) {
            System.out.println("El procesamiento del pago falló para el producto: " + productId);
            return;
        }
        
        // 3. Actualizar el inventario
        inventoryService.updateInventory(productId, quantity);
        
        // 4. Enviar el pedido y obtener el número de seguimiento
        String trackingNumber = shippingService.shipOrder(productId, quantity);
        
        // 5. Notificar al cliente
        notificationService.sendNotification("Pedido realizado con éxito. Número de seguimiento: " + trackingNumber);
        
        System.out.println("Proceso de pedido completado para el producto: " + productId);
    }
}
```

#### Ejemplo de Uso en la Aplicación

```java
public class Main {
    public static void main(String[] args) {
        OrderFacade orderFacade = new OrderFacade();
        // Procesar un pedido para el producto "PROD001", con 2 unidades y un monto total de 99.99
        orderFacade.placeOrder("PROD001", 2, 99.99);
    }
}
```

---

### Explicación

- **Desacoplamiento:**  
  La clase `OrderFacade` oculta la complejidad de interactuar con varios servicios (pago, inventario, envío y notificaciones). El cliente solo llama a un método (`placeOrder`) sin preocuparse de los detalles internos.

- **Simplicidad y Mantenibilidad:**  
  Si en el futuro se requiere modificar alguno de los servicios (por ejemplo, cambiar el proveedor de pago), solo se deberá actualizar la implementación correspondiente sin afectar la lógica central del procesamiento de pedidos.

- **Reutilización:**  
  Los servicios individuales pueden ser utilizados de manera independiente en otros contextos, y la fachada puede extenderse o modificarse según las necesidades del negocio.

Este ejemplo demuestra cómo aplicar el patrón Facade en un escenario real de e-commerce, coordinando múltiples subsistemas para resolver una tarea compleja de manera sencilla y eficiente, una situación común en el día a día de un programador en aplicaciones profesionales.
