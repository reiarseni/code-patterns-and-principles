## Strategy en PHP

A continuación se muestra un ejemplo en PHP utilizando el patrón Strategy aplicado a un caso práctico muy común: métodos de pago en una aplicación de e-commerce. En este ejemplo, la estrategia permite elegir entre diferentes métodos de pago (por ejemplo, tarjeta de crédito o PayPal) de forma intercambiable, lo que es un caso clásico de uso del patrón Strategy.

```php
<?php

// Interfaz que define la estrategia de pago
interface PaymentStrategy {
    public function pay(float $amount): void;
}

// Estrategia concreta: Pago con tarjeta de crédito
class CreditCardPayment implements PaymentStrategy {
    private string $cardNumber;
    private string $cardHolder;
    private string $expiryDate;
    private string $cvv;

    public function __construct(string $cardNumber, string $cardHolder, string $expiryDate, string $cvv) {
        $this->cardNumber = $cardNumber;
        $this->cardHolder = $cardHolder;
        $this->expiryDate = $expiryDate;
        $this->cvv = $cvv;
    }

    public function pay(float $amount): void {
        echo "Se han pagado $$amount usando Tarjeta de Crédito.\n";
    }
}

// Estrategia concreta: Pago con PayPal
class PaypalPayment implements PaymentStrategy {
    private string $email;
    private string $password;

    public function __construct(string $email, string $password) {
        $this->email = $email;
        $this->password = $password;
    }

    public function pay(float $amount): void {
        echo "Se han pagado $$amount usando PayPal.\n";
    }
}

// Contexto que utiliza una estrategia de pago
class PaymentContext {
    private PaymentStrategy $paymentStrategy;

    public function __construct(PaymentStrategy $paymentStrategy) {
        $this->paymentStrategy = $paymentStrategy;
    }

    public function setPaymentStrategy(PaymentStrategy $paymentStrategy): void {
        $this->paymentStrategy = $paymentStrategy;
    }

    public function executePayment(float $amount): void {
        $this->paymentStrategy->pay($amount);
    }
}

// Ejemplo de uso:
// Inicialmente, se paga usando tarjeta de crédito
$creditCardPayment = new CreditCardPayment("1234567890123456", "Juan Pérez", "12/23", "123");
$context = new PaymentContext($creditCardPayment);
$context->executePayment(100.0);

// Cambiamos la estrategia para pagar usando PayPal
$paypalPayment = new PaypalPayment("juan@example.com", "secret");
$context->setPaymentStrategy($paypalPayment);
$context->executePayment(200.0);
?>
```

En este ejemplo, la clase `PaymentContext` es responsable de ejecutar el pago, pero delega la lógica de pago a la estrategia que se le asigne, ya sea `CreditCardPayment` o `PaypalPayment`. Esto permite cambiar la implementación de pago en tiempo de ejecución sin modificar el contexto, siguiendo el principio de “abierto/cerrado” y promoviendo un diseño flexible y escalable.
