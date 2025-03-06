## Builder en Python

A continuación se muestra un ejemplo práctico y profesional del patrón Builder en Python, aplicado a la construcción de una factura (Invoice) en un sistema de gestión comercial. En aplicaciones reales es común que los objetos complejos (como una factura) tengan muchos parámetros, algunos obligatorios y otros opcionales (por ejemplo, descuentos, impuestos, notas, etc.). Usar el patrón Builder permite construir dichos objetos de forma fluida y controlada, mejorando la legibilidad y facilitando la validación de campos obligatorios.

---

```python
from datetime import datetime
from typing import List, Optional

# Clase que representa un ítem de la factura.
class InvoiceItem:
    def __init__(self, description: str, quantity: int, unit_price: float):
        self.description = description
        self.quantity = quantity
        self.unit_price = unit_price

    def total(self) -> float:
        return self.quantity * self.unit_price

    def __str__(self):
        return f"{self.description}: {self.quantity} x ${self.unit_price:.2f} = ${self.total():.2f}"

# Producto final: Factura.
class Invoice:
    def __init__(
        self,
        invoice_number: str,
        date: datetime,
        customer: str,
        items: List[InvoiceItem],
        discount: float = 0.0,
        tax: float = 0.0,
        notes: Optional[str] = None,
    ):
        self.invoice_number = invoice_number
        self.date = date
        self.customer = customer
        self.items = items
        self.discount = discount
        self.tax = tax
        self.notes = notes

    def subtotal(self) -> float:
        return sum(item.total() for item in self.items)

    def total(self) -> float:
        subtotal = self.subtotal()
        subtotal_after_discount = subtotal - self.discount
        return subtotal_after_discount + (subtotal_after_discount * self.tax)

    def __str__(self):
        output = [
            f"Factura #{self.invoice_number}",
            f"Fecha: {self.date.strftime('%Y-%m-%d')}",
            f"Cliente: {self.customer}",
            "Ítems:"
        ]
        for item in self.items:
            output.append("  " + str(item))
        output.append(f"Subtotal: ${self.subtotal():.2f}")
        output.append(f"Descuento: ${self.discount:.2f}")
        output.append(f"Impuesto: {self.tax*100:.2f}%")
        output.append(f"Total: ${self.total():.2f}")
        if self.notes:
            output.append(f"Notas: {self.notes}")
        return "\n".join(output)

# Builder para construir la factura paso a paso.
class InvoiceBuilder:
    def __init__(self):
        self.invoice_number: Optional[str] = None
        self.date: Optional[datetime] = None
        self.customer: Optional[str] = None
        self.items: List[InvoiceItem] = []
        self.discount: float = 0.0
        self.tax: float = 0.0
        self.notes: Optional[str] = None

    def set_invoice_number(self, invoice_number: str) -> "InvoiceBuilder":
        self.invoice_number = invoice_number
        return self

    def set_date(self, date: datetime) -> "InvoiceBuilder":
        self.date = date
        return self

    def set_customer(self, customer: str) -> "InvoiceBuilder":
        self.customer = customer
        return self

    def add_item(self, description: str, quantity: int, unit_price: float) -> "InvoiceBuilder":
        self.items.append(InvoiceItem(description, quantity, unit_price))
        return self

    def set_discount(self, discount: float) -> "InvoiceBuilder":
        self.discount = discount
        return self

    def set_tax(self, tax: float) -> "InvoiceBuilder":
        self.tax = tax
        return self

    def set_notes(self, notes: str) -> "InvoiceBuilder":
        self.notes = notes
        return self

    def build(self) -> Invoice:
        # Validación de campos obligatorios.
        if not (self.invoice_number and self.date and self.customer):
            raise ValueError("El número de factura, la fecha y el cliente son obligatorios.")
        return Invoice(
            invoice_number=self.invoice_number,
            date=self.date,
            customer=self.customer,
            items=self.items,
            discount=self.discount,
            tax=self.tax,
            notes=self.notes
        )

# Ejemplo de uso en una aplicación profesional.
if __name__ == "__main__":
    builder = InvoiceBuilder()
    invoice = (
        builder
        .set_invoice_number("INV-2023001")
        .set_date(datetime.now())
        .set_customer("Acme Corp")
        .add_item("Servicio de Consultoría", 10, 150.0)
        .add_item("Soporte Técnico", 5, 100.0)
        .set_discount(50.0)
        .set_tax(0.07)  # 7% de impuesto.
        .set_notes("Pago neto a 30 días.")
        .build()
    )
    print(invoice)
```

---

### Explicación

- **Flexibilidad en la Construcción:**  
  El `InvoiceBuilder` permite configurar paso a paso cada atributo de la factura. Los métodos devuelven el mismo objeto builder (fluidez) para encadenar llamadas, lo que hace el código más legible y adaptable a diferentes requerimientos.

- **Validación de Campos Obligatorios:**  
  Al llamar al método `build()`, se verifica que se hayan establecido los campos esenciales (número de factura, fecha y cliente). Esto evita crear objetos inválidos y ayuda a detectar errores tempranamente.

- **Uso Profesional:**  
  En aplicaciones de gestión comercial o facturación, es común tener objetos complejos que requieren múltiples parámetros. El patrón Builder permite manejar esta complejidad de manera organizada, facilitando el mantenimiento y la extensión del código.

Este ejemplo demuestra cómo el patrón Builder puede aplicarse para construir objetos complejos de manera flexible y segura, una situación muy común en el día a día de un programador en aplicaciones profesionales.
