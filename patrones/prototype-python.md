## Prototype en Python

A continuación se muestra un ejemplo práctico y profesional del patrón Prototype en Python, aplicado a la clonación de un portafolio financiero en un sistema de gestión de inversiones. En aplicaciones reales es común que la creación o configuración de objetos complejos (con estructuras anidadas y datos interrelacionados) sea costosa o requiera una configuración inicial, por lo que clonar un objeto prototipo y luego modificarlo resulta muy eficiente y flexible.

En este ejemplo, creamos un portafolio con una lista de inversiones, y mediante el método `clone()` (basado en `copy.deepcopy`) obtenemos una copia exacta del portafolio. Luego, modificamos la copia sin afectar el objeto original, demostrando cómo el patrón Prototype permite replicar y personalizar objetos de manera eficiente.

---

```python
import copy

class Investment:
    def __init__(self, symbol: str, shares: int, price: float):
        self.symbol = symbol
        self.shares = shares
        self.price = price

    def __str__(self):
        return f"{self.symbol}: {self.shares} shares @ ${self.price:.2f}"

class FinancialPortfolio:
    def __init__(self, name: str):
        self.name = name
        self.investments = []  # Lista de objetos Investment
        self.risk_level = None

    def add_investment(self, investment: Investment):
        self.investments.append(investment)

    def set_risk_level(self, risk: str):
        self.risk_level = risk

    def clone(self):
        """
        Método Prototype: retorna una copia profunda del objeto,
        clonando toda su estructura interna.
        """
        return copy.deepcopy(self)

    def __str__(self):
        investments_str = "\n".join(str(inv) for inv in self.investments)
        return (
            f"Portfolio: {self.name}\n"
            f"Risk Level: {self.risk_level}\n"
            f"Investments:\n{investments_str}"
        )

# Ejemplo de uso profesional
if __name__ == "__main__":
    # Creación del portafolio prototipo con inversiones iniciales.
    prototype_portfolio = FinancialPortfolio("Prototype Portfolio")
    prototype_portfolio.add_investment(Investment("AAPL", 50, 150.0))
    prototype_portfolio.add_investment(Investment("GOOGL", 10, 2800.0))
    prototype_portfolio.set_risk_level("Medium")

    print("=== Portafolio Original ===")
    print(prototype_portfolio)

    # Clonar el portafolio para crear una nueva versión y modificarla.
    cloned_portfolio = prototype_portfolio.clone()
    cloned_portfolio.name = "Cloned Portfolio"
    # Modificar la cantidad de acciones en una inversión para simular una reconfiguración
    if cloned_portfolio.investments:
        cloned_portfolio.investments[0].shares = 100

    print("\n=== Portafolio Clonado y Modificado ===")
    print(cloned_portfolio)

    print("\n=== Verificación del Portafolio Original (sin cambios) ===")
    print(prototype_portfolio)
```

---

### Explicación

- **Centralización de la Configuración:**  
  El portafolio financiero es un objeto complejo con varias inversiones y atributos (como nivel de riesgo). En escenarios reales, configurar este objeto desde cero puede ser costoso o propenso a errores.  
- **Clonación y Personalización:**  
  Con el método `clone()`, se obtiene una copia exacta del portafolio original utilizando `copy.deepcopy`. Esto permite modificar la copia (por ejemplo, cambiar el número de acciones o el nombre) sin afectar la configuración inicial del prototipo.  
- **Uso Profesional:**  
  Este patrón es muy útil en sistemas de gestión de inversiones, simuladores financieros o cualquier aplicación donde se necesite replicar configuraciones complejas de manera eficiente. Permite gestionar diferentes escenarios o estrategias sin recrear objetos desde cero, facilitando la experimentación y la personalización en tiempo real.

El ejemplo ilustra cómo el patrón Prototype puede resolver problemas comunes en el desarrollo profesional, al permitir clonar y adaptar objetos complejos de forma rápida y segura.
