## Command en Java

A continuación se muestra un ejemplo práctico  del patrón Command en Java, aplicado a un sistema de procesamiento de transacciones bancarias. En este caso, cada operación (depósito o retiro) se encapsula en un comando, lo que permite ejecutar, encolar e incluso deshacer (undo) transacciones. Este enfoque es común en aplicaciones financieras y sistemas que requieren auditoría o reversión de operaciones.

---

### Definición de la Interfaz Command

```java
// Comando: Define el contrato para ejecutar y deshacer operaciones.
public interface Command {
    void execute();
    void undo();
}
```

---

### Clase BankAccount (Receptor)

```java
// Receptor: Representa una cuenta bancaria con operaciones básicas.
public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        balance += amount;
        System.out.println("Depósito: " + amount + ". Nuevo saldo: " + balance);
    }

    public void withdraw(double amount) {
        if (amount > balance) {
            System.out.println("Fondos insuficientes para retirar: " + amount);
            return;
        }
        balance -= amount;
        System.out.println("Retiro: " + amount + ". Nuevo saldo: " + balance);
    }

    public double getBalance() {
        return balance;
    }
}
```

---

### Comandos Concretos

#### Comando para Depósito

```java
// Comando concreto para realizar un depósito.
public class DepositCommand implements Command {
    private final BankAccount account;
    private final double amount;

    public DepositCommand(BankAccount account, double amount) {
        this.account = account;
        this.amount = amount;
    }

    @Override
    public void execute() {
        account.deposit(amount);
    }

    @Override
    public void undo() {
        account.withdraw(amount);
    }
}
```

#### Comando para Retiro

```java
// Comando concreto para realizar un retiro.
public class WithdrawCommand implements Command {
    private final BankAccount account;
    private final double amount;

    public WithdrawCommand(BankAccount account, double amount) {
        this.account = account;
        this.amount = amount;
    }

    @Override
    public void execute() {
        account.withdraw(amount);
    }

    @Override
    public void undo() {
        account.deposit(amount);
    }
}
```

---

### Invocador: TransactionProcessor

```java
import java.util.Stack;

// Invocador: Procesa y almacena comandos para permitir operaciones de undo.
public class TransactionProcessor {
    private final Stack<Command> commandHistory = new Stack<>();

    public void executeCommand(Command command) {
        command.execute();
        commandHistory.push(command);
    }

    public void undoLastCommand() {
        if (!commandHistory.isEmpty()) {
            Command lastCommand = commandHistory.pop();
            lastCommand.undo();
        } else {
            System.out.println("No hay comandos para deshacer.");
        }
    }
}
```

---

### Ejemplo de Uso en una Aplicación

```java
public class Main {
    public static void main(String[] args) {
        // Crear una cuenta bancaria con saldo inicial.
        BankAccount account = new BankAccount(1000.0);
        TransactionProcessor processor = new TransactionProcessor();

        // Crear y ejecutar comandos de transacción.
        Command deposit100 = new DepositCommand(account, 100.0);
        Command withdraw50 = new WithdrawCommand(account, 50.0);

        // Ejecutar depósito y retiro.
        processor.executeCommand(deposit100);  // Saldo: 1100.0
        processor.executeCommand(withdraw50);    // Saldo: 1050.0

        // Deshacer el último comando (retiro).
        processor.undoLastCommand();             // Saldo: 1100.0

        // Deshacer el depósito.
        processor.undoLastCommand();             // Saldo: 1000.0
    }
}
```

---

### Explicación

- **Encapsulación de Operaciones:**  
  Cada transacción se encapsula en un objeto comando (DepositCommand o WithdrawCommand), permitiendo que las operaciones se traten como "ciertas acciones" que pueden ejecutarse y deshacerse.

- **Desacoplamiento:**  
  La cuenta bancaria (receptor) no conoce detalles de cómo se invocan sus métodos; el invocador (TransactionProcessor) gestiona el ciclo de vida de los comandos, facilitando la auditoría y la reversión de operaciones.

- **Manejo de Situaciones Comunes:**  
  Este patrón es útil en sistemas que requieren registro de operaciones (para auditorías o deshacer acciones), como aplicaciones financieras, de gestión de inventario o cualquier sistema transaccional, permitiendo manejar de forma estructurada y flexible las acciones del día a día.

Este ejemplo ilustra cómo el patrón Command permite construir un sistema de transacciones robusto y flexible, muy aplicable en escenarios reales de programación profesional.
