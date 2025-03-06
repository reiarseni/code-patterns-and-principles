## Refactorización en C#

A continuación se muestra 5 ejemplos de refactoring en c#. Cada ejemplo presenta una versión "ANTES" y una versión "DESPUÉS" para ilustrar cómo mejorar el diseño o la legibilidad del código en situaciones reales de desarrollo.

---

```csharp
using System;

namespace RefactoringExamples
{
    // =====================================================
    // Tarea 1: Reemplazar Condicional con Polimorfismo
    // -----------------------------------------------------
    // Problema: Se utiliza un condicional para calcular el bono de un empleado según su tipo.
    // ANTES:
    class EmployeeBefore
    {
        public string Name { get; set; }
        public int Type { get; set; } // 1: Regular, 2: Manager, otros: Contrato
        public double Salary { get; set; }

        public double CalculateBonus()
        {
            if (Type == 1)
                return Salary * 0.1;
            else if (Type == 2)
                return Salary * 0.2;
            else
                return Salary * 0.05;
        }
    }

    // DESPUÉS: Se reemplaza el condicional con polimorfismo creando subclases para cada tipo.
    abstract class EmployeeAfter
    {
        public string Name { get; set; }
        public double Salary { get; set; }
        public abstract double CalculateBonus();
    }

    class RegularEmployee : EmployeeAfter
    {
        public override double CalculateBonus() => Salary * 0.1;
    }

    class Manager : EmployeeAfter
    {
        public override double CalculateBonus() => Salary * 0.2;
    }

    class ContractEmployee : EmployeeAfter
    {
        public override double CalculateBonus() => Salary * 0.05;
    }

    // =====================================================
    // Tarea 2: Introducir un Objeto Parámetro (Introduce Parameter Object)
    // -----------------------------------------------------
    // Problema: Un método recibe muchos parámetros relacionados entre sí.
    // ANTES:
    class OrderBefore
    {
        public double CalculateTotal(double price, double discount, double tax, double shipping)
        {
            return price - discount + tax + shipping;
        }
    }

    // DESPUÉS: Se agrupan los parámetros en un objeto.
    class OrderDetails
    {
        public double Price { get; set; }
        public double Discount { get; set; }
        public double Tax { get; set; }
        public double Shipping { get; set; }
    }

    class OrderAfter
    {
        public double CalculateTotal(OrderDetails details)
        {
            return details.Price - details.Discount + details.Tax + details.Shipping;
        }
    }

    // =====================================================
    // Tarea 3: Mover Método (Move Method)
    // -----------------------------------------------------
    // Problema: El método para calcular el descuento se encuentra en la clase Order,
    // cuando realmente depende de la información del Customer.
    // ANTES:
    class CustomerBefore
    {
        public string Name { get; set; }
        public int LoyaltyPoints { get; set; }
    }

    class OrderForMoveBefore
    {
        public CustomerBefore Customer { get; set; }
        public double TotalAmount { get; set; }

        public double CalculateDiscount()
        {
            // Cálculo de descuento basado en los puntos de lealtad del cliente
            if (Customer != null && Customer.LoyaltyPoints > 1000)
                return TotalAmount * 0.1;
            return 0;
        }
    }

    // DESPUÉS: Se mueve el método a la clase Customer, ya que es quien conoce su propia lógica.
    class CustomerAfter
    {
        public string Name { get; set; }
        public int LoyaltyPoints { get; set; }

        public double CalculateDiscount(double totalAmount)
        {
            if (LoyaltyPoints > 1000)
                return totalAmount * 0.1;
            return 0;
        }
    }

    class OrderForMoveAfter
    {
        public CustomerAfter Customer { get; set; }
        public double TotalAmount { get; set; }

        public double GetFinalAmount()
        {
            double discount = Customer.CalculateDiscount(TotalAmount);
            return TotalAmount - discount;
        }
    }

    // =====================================================
    // Tarea 4: Reemplazar Método por Propiedad (Replace Method with Property)
    // -----------------------------------------------------
    // Problema: Se utiliza un método para obtener el nombre completo, lo cual puede expresarse como propiedad.
    // ANTES:
    class PersonBefore
    {
        private string firstName;
        private string lastName;

        public PersonBefore(string first, string last)
        {
            firstName = first;
            lastName = last;
        }

        public string GetFullName()
        {
            return firstName + " " + lastName;
        }
    }

    // DESPUÉS: Se reemplaza el método por una propiedad de solo lectura.
    class PersonAfter
    {
        public string FirstName { get; set; }
        public string LastName { get; set; }

        public PersonAfter(string first, string last)
        {
            FirstName = first;
            LastName = last;
        }

        public string FullName => $"{FirstName} {LastName}";
    }

    // =====================================================
    // Tarea 5: Introducir el Patrón Null Object
    // -----------------------------------------------------
    // Problema: Se realizan comprobaciones nulas repetitivas para un Logger.
    // ANTES: Código que comprueba si Logger es nulo en cada uso.
    interface ILogger
    {
        void Log(string message);
    }

    class ConsoleLogger : ILogger
    {
        public void Log(string message)
        {
            Console.WriteLine("Log: " + message);
        }
    }

    class ProcessorBefore
    {
        public ILogger Logger { get; set; }  // Puede ser nulo

        public void Process()
        {
            if (Logger != null)
                Logger.Log("Process started.");

            // ... procesamiento ...

            if (Logger != null)
                Logger.Log("Process completed.");
        }
    }

    // DESPUÉS: Se introduce un NullLogger que implementa ILogger pero no realiza ninguna acción.
    class NullLogger : ILogger
    {
        public void Log(string message)
        {
            // No hace nada
        }
    }

    class ProcessorAfter
    {
        // Se asigna un NullLogger por defecto para evitar comprobaciones nulas.
        public ILogger Logger { get; set; } = new NullLogger();

        public void Process()
        {
            Logger.Log("Process started.");
            // ... procesamiento ...
            Logger.Log("Process completed.");
        }
    }

    // =====================================================
    // Función principal para demostrar todos los ejemplos
    // =====================================================
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("=== Tarea 1: Reemplazar Condicional con Polimorfismo ===");
            // ANTES:
            var empBefore = new EmployeeBefore { Name = "John", Type = 1, Salary = 1000 };
            Console.WriteLine($"EmployeeBefore bonus: {empBefore.CalculateBonus()}");

            // DESPUÉS:
            EmployeeAfter empAfter = new Manager { Name = "Alice", Salary = 2000 };
            Console.WriteLine($"EmployeeAfter bonus (Manager): {empAfter.CalculateBonus()}");

            Console.WriteLine("\n=== Tarea 2: Introducir un Objeto Parámetro ===");
            // ANTES:
            var orderBefore = new OrderBefore();
            double totalBefore = orderBefore.CalculateTotal(100, 10, 5, 15);
            Console.WriteLine($"Order total before: {totalBefore}");

            // DESPUÉS:
            var orderAfter = new OrderAfter();
            OrderDetails details = new OrderDetails { Price = 100, Discount = 10, Tax = 5, Shipping = 15 };
            double totalAfter = orderAfter.CalculateTotal(details);
            Console.WriteLine($"Order total after: {totalAfter}");

            Console.WriteLine("\n=== Tarea 3: Mover Método ===");
            // ANTES:
            var customerBefore = new CustomerBefore { Name = "Bob", LoyaltyPoints = 1500 };
            var orderMoveBefore = new OrderForMoveBefore { Customer = customerBefore, TotalAmount = 500 };
            double discountBefore = orderMoveBefore.CalculateDiscount();
            Console.WriteLine($"Discount before: {discountBefore}");

            // DESPUÉS:
            var customerAfter = new CustomerAfter { Name = "Bob", LoyaltyPoints = 1500 };
            var orderMoveAfter = new OrderForMoveAfter { Customer = customerAfter, TotalAmount = 500 };
            double finalAmountAfter = orderMoveAfter.GetFinalAmount();
            Console.WriteLine($"Final amount after discount: {finalAmountAfter}");

            Console.WriteLine("\n=== Tarea 4: Reemplazar Método por Propiedad ===");
            // ANTES:
            var personBefore = new PersonBefore("Jane", "Doe");
            Console.WriteLine($"Full name before: {personBefore.GetFullName()}");

            // DESPUÉS:
            var personAfter = new PersonAfter("Jane", "Doe");
            Console.WriteLine($"Full name after: {personAfter.FullName}");

            Console.WriteLine("\n=== Tarea 5: Introducir el Patrón Null Object ===");
            // ANTES:
            var processorBefore = new ProcessorBefore { Logger = new ConsoleLogger() };
            processorBefore.Process();

            // DESPUÉS:
            var processorAfter = new ProcessorAfter();
            // Si se desea un log real, se asigna un ConsoleLogger:
            processorAfter.Logger = new ConsoleLogger();
            processorAfter.Process();
        }
    }
}
```

---

### Explicación General

1. **Reemplazar Condicional con Polimorfismo:**  
   Se elimina el uso de condicionales en el método de cálculo de bono, delegando la lógica a clases específicas (RegularEmployee, Manager, ContractEmployee).

2. **Introducir un Objeto Parámetro:**  
   Se agrupan parámetros relacionados en la clase *OrderDetails* para simplificar la firma del método de cálculo del total de un pedido.

3. **Mover Método:**  
   La lógica para calcular el descuento se traslada de la clase *Order* a la clase *Customer*, asignando a cada quien la responsabilidad de lo que le compete.

4. **Reemplazar Método por Propiedad:**  
   Se sustituye un método simple (obtener el nombre completo) por una propiedad que mejora la legibilidad y el uso de la clase.

5. **Introducir el Patrón Null Object:**  
   En lugar de comprobar si el objeto Logger es nulo en cada uso, se implementa un *NullLogger* que no realiza ninguna acción, simplificando el código del método *Process*.
 en el día a día del programador.

Estos ejemplos son casos clásicos y prácticos de refactoring en aplicaciones profesionales, que ayudan a mejorar la mantenibilidad, extensibilidad y claridad del código