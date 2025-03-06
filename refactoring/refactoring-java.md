## Refactorización en Java

A continuación se muestra 5 ejemplos de refactoring aplicados a situaciones comunes en el desarrollo de software. Cada ejemplo incluye la versión "ANTES" (con código que presenta algún problema de diseño o legibilidad) y la versión "DESPUÉS" (refactorizada para mejorar claridad, evitar duplicación o facilitar el mantenimiento).

---

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""

Este archivo contiene 5 ejemplos de refactoring en Python aplicados a situaciones comunes en aplicaciones reales.
Cada sección muestra el código "ANTES" y su versión "DESPUÉS".
"""

# ===============================
# Tarea 1: Extraer Función (Extract Method)
# -------------------------------
# Problema: La función de procesamiento de pedidos tiene lógica interna para calcular el total,
# lo que la hace difícil de mantener y probar.
# Antes:
def process_order(order):
    total = 0
    for item in order:
        # Cálculo del descuento según cantidad
        if item['quantity'] > 10:
            discount = item['price'] * 0.1
        else:
            discount = 0
        total += (item['price'] - discount) * item['quantity']
    return total

# Después:
def calculate_item_total(item):
    """Extrae la lógica de cálculo del total de cada ítem."""
    discount = item['price'] * 0.1 if item['quantity'] > 10 else 0
    return (item['price'] - discount) * item['quantity']

def process_order_refactored(order):
    """Procesa el pedido calculando el total a partir de cada ítem."""
    total = sum(calculate_item_total(item) for item in order)
    return total

# ===============================
# Tarea 2: Renombrar Variable (Rename Variable)
# -------------------------------
# Problema: Uso de nombres poco descriptivos que dificultan la comprensión del código.
# Antes:
def calculate_area(r):
    return 3.14159 * r * r

# Después:
def calculate_circle_area(radius):
    PI = 3.14159
    return PI * radius * radius

# ===============================
# Tarea 3: Eliminar Código Duplicado (Remove Duplication)
# -------------------------------
# Problema: Validaciones similares para diferentes campos se repiten en múltiples funciones.
# Antes:
def validate_username(username):
    if username is None or username.strip() == "":
        raise ValueError("Username cannot be empty")
    return True

def validate_email(email):
    if email is None or email.strip() == "":
        raise ValueError("Email cannot be empty")
    return True

# Después:
def validate_non_empty(value, field_name):
    """Valida que un campo no esté vacío."""
    if value is None or value.strip() == "":
        raise ValueError(f"{field_name} cannot be empty")
    return True

def validate_username_refactored(username):
    return validate_non_empty(username, "Username")

def validate_email_refactored(email):
    return validate_non_empty(email, "Email")

# ===============================
# Tarea 4: Reemplazar Números Mágicos (Replace Magic Numbers with Constants)
# -------------------------------
# Problema: Uso directo de números en la lógica que hace difícil entender su significado.
# Antes:
def calculate_shipping_cost(weight):
    if weight <= 5:
        return 10
    elif weight <= 20:
        return 20
    else:
        return 50

# Después:
LOW_WEIGHT_LIMIT = 5
MID_WEIGHT_LIMIT = 20
LOW_COST = 10
MID_COST = 20
HIGH_COST = 50

def calculate_shipping_cost_refactored(weight):
    if weight <= LOW_WEIGHT_LIMIT:
        return LOW_COST
    elif weight <= MID_WEIGHT_LIMIT:
        return MID_COST
    else:
        return HIGH_COST

# ===============================
# Tarea 5: Simplificar Condicional (Simplify Conditional Logic)
# -------------------------------
# Problema: Uso de condicionales anidados que dificultan la lectura.
# Antes:
def get_discount(customer):
    discount = 0
    if customer is not None:
        if customer['loyalty_years'] > 5:
            discount = 0.15
        else:
            if customer['has_coupon']:
                discount = 0.10
            else:
                discount = 0.05
    return discount

# Después:
def get_discount_refactored(customer):
    if customer is None:
        return 0
    if customer['loyalty_years'] > 5:
        return 0.15
    if customer['has_coupon']:
        return 0.10
    return 0.05

# ===============================
# Función principal para demostrar los ejemplos
# ===============================
def main():
    print("=== Tarea 1: Extraer Función ===")
    order = [
        {'price': 100, 'quantity': 5},
        {'price': 50, 'quantity': 12},
        {'price': 20, 'quantity': 1}
    ]
    print("Total (ANTES):", process_order(order))
    print("Total (DESPUÉS):", process_order_refactored(order))

    print("\n=== Tarea 2: Renombrar Variable ===")
    r = 7
    print("Área del círculo (ANTES):", calculate_area(r))
    print("Área del círculo (DESPUÉS):", calculate_circle_area(r))

    print("\n=== Tarea 3: Eliminar Código Duplicado ===")
    try:
        print("Validando username (DESPUÉS):", validate_username_refactored("   user123   "))
        print("Validando email (DESPUÉS):", validate_email_refactored("user@example.com"))
        # Descomenta la siguiente línea para ver la excepción:
        # validate_username_refactored("")
    except ValueError as e:
        print("Error:", e)

    print("\n=== Tarea 4: Reemplazar Números Mágicos ===")
    weight = 15
    print("Costo de envío (ANTES):", calculate_shipping_cost(weight))
    print("Costo de envío (DESPUÉS):", calculate_shipping_cost_refactored(weight))

    print("\n=== Tarea 5: Simplificar Condicional ===")
    customer1 = {'loyalty_years': 6, 'has_coupon': False}
    customer2 = {'loyalty_years': 2, 'has_coupon': True}
    customer3 = {'loyalty_years': 2, 'has_coupon': False}
    print("Descuento para customer1 (ANTES):", get_discount(customer1))
    print("Descuento para customer1 (DESPUÉS):", get_discount_refactored(customer1))
    print("Descuento para customer2 (ANTES):", get_discount(customer2))
    print("Descuento para customer2 (DESPUÉS):", get_discount_refactored(customer2))
    print("Descuento para customer3 (ANTES):", get_discount(customer3))
    print("Descuento para customer3 (DESPUÉS):", get_discount_refactored(customer3))

if __name__ == '__main__':
    main()
```

---

### Explicación General

1. **Extraer Función:**  
   Separamos la lógica de cálculo de cada ítem en una función independiente (`calculate_item_total`), haciendo el código más modular y fácil de testear.

2. **Renombrar Variable:**  
   Cambiamos nombres poco descriptivos (como `r`) por nombres más claros (`radius`), lo cual mejora la legibilidad y la comprensión del propósito de la variable.

3. **Eliminar Código Duplicado:**  
   Se centraliza la validación de cadenas no vacías en una función común (`validate_non_empty`), evitando duplicación y facilitando el mantenimiento.

4. **Reemplazar Números Mágicos:**  
   Se reemplazan números literales por constantes con nombres significativos, de manera que sea evidente el significado de cada valor en la lógica del envío.

5. **Simplificar Condicional:**  
   Se reestructura la lógica condicional eliminando anidamientos innecesarios mediante cláusulas de guarda, lo que hace el código más claro y directo.

Este archivo ejemplifica cómo aplicar refactorizaciones comunes para mejorar la calidad y mantenibilidad del código en situaciones reales.
