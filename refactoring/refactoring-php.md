## Refactorización en PHP

Se muestran 5 tareas de refactoring aplicadas a situaciones comunes en el desarrollo de software. Cada sección incluye el código “ANTES” (con un problema de diseño, legibilidad o duplicación) y la versión “DESPUÉS” refactorizada para mejorar la mantenibilidad.

---

```php
<?php
/**
 * Este archivo contiene 5 ejemplos de refactoring en PHP aplicados a situaciones reales.
 * Cada sección muestra el código "ANTES" y su versión "DESPUÉS".
 */

// ===============================
// Tarea 1: Extraer Función (Extract Method)
// -------------------------------
// Problema: La función de procesamiento de pedidos tiene la lógica del cálculo total dentro del bucle,
// lo que dificulta su mantenimiento y pruebas.
// ANTES:
function processOrder($order) {
    $total = 0;
    foreach ($order as $item) {
        // Cálculo del descuento según la cantidad
        if ($item['quantity'] > 10) {
            $discount = $item['price'] * 0.1;
        } else {
            $discount = 0;
        }
        $total += ($item['price'] - $discount) * $item['quantity'];
    }
    return $total;
}

// DESPUÉS:
function calculateItemTotal($item) {
    // Extrae la lógica de cálculo de cada ítem
    $discount = ($item['quantity'] > 10) ? $item['price'] * 0.1 : 0;
    return ($item['price'] - $discount) * $item['quantity'];
}

function processOrderRefactored($order) {
    $total = 0;
    foreach ($order as $item) {
        $total += calculateItemTotal($item);
    }
    return $total;
}

// ===============================
// Tarea 2: Renombrar Variable (Rename Variable)
// -------------------------------
// Problema: Uso de nombres poco descriptivos que dificultan la comprensión del código.
// ANTES:
function calculateArea($r) {
    return 3.14159 * $r * $r;
}

// DESPUÉS:
function calculateCircleArea($radius) {
    $pi = 3.14159;
    return $pi * $radius * $radius;
}

// ===============================
// Tarea 3: Eliminar Código Duplicado (Remove Duplication)
// -------------------------------
// Problema: Validaciones similares se repiten en distintas funciones.
// ANTES:
function validateUsername($username) {
    if ($username === null || trim($username) === '') {
        throw new Exception("Username cannot be empty");
    }
    return true;
}

function validateEmail($email) {
    if ($email === null || trim($email) === '') {
        throw new Exception("Email cannot be empty");
    }
    return true;
}

// DESPUÉS:
function validateNonEmpty($value, $fieldName) {
    if ($value === null || trim($value) === '') {
        throw new Exception("$fieldName cannot be empty");
    }
    return true;
}

function validateUsernameRefactored($username) {
    return validateNonEmpty($username, "Username");
}

function validateEmailRefactored($email) {
    return validateNonEmpty($email, "Email");
}

// ===============================
// Tarea 4: Reemplazar Números Mágicos (Replace Magic Numbers with Constants)
// -------------------------------
// Problema: Uso directo de números en la lógica, lo que dificulta su interpretación.
// ANTES:
function calculateShippingCost($weight) {
    if ($weight <= 5) {
        return 10;
    } elseif ($weight <= 20) {
        return 20;
    } else {
        return 50;
    }
}

// DESPUÉS:
define('LOW_WEIGHT_LIMIT', 5);
define('MID_WEIGHT_LIMIT', 20);
define('LOW_COST', 10);
define('MID_COST', 20);
define('HIGH_COST', 50);

function calculateShippingCostRefactored($weight) {
    if ($weight <= LOW_WEIGHT_LIMIT) {
        return LOW_COST;
    } elseif ($weight <= MID_WEIGHT_LIMIT) {
        return MID_COST;
    } else {
        return HIGH_COST;
    }
}

// ===============================
// Tarea 5: Simplificar Condicional (Simplify Conditional Logic)
// -------------------------------
// Problema: Uso de condicionales anidados que dificultan la lectura del código.
// ANTES:
function getDiscount($customer) {
    $discount = 0;
    if ($customer !== null) {
        if ($customer['loyalty_years'] > 5) {
            $discount = 0.15;
        } else {
            if ($customer['has_coupon']) {
                $discount = 0.10;
            } else {
                $discount = 0.05;
            }
        }
    }
    return $discount;
}

// DESPUÉS:
function getDiscountRefactored($customer) {
    if ($customer === null) {
        return 0;
    }
    if ($customer['loyalty_years'] > 5) {
        return 0.15;
    }
    if ($customer['has_coupon']) {
        return 0.10;
    }
    return 0.05;
}

// ===============================
// Función principal para demostrar los ejemplos
// ===============================
function main() {
    echo "=== Tarea 1: Extraer Función ===\n";
    $order = [
        ['price' => 100, 'quantity' => 5],
        ['price' => 50, 'quantity' => 12],
        ['price' => 20, 'quantity' => 1]
    ];
    echo "Total (ANTES): " . processOrder($order) . "\n";
    echo "Total (DESPUÉS): " . processOrderRefactored($order) . "\n\n";

    echo "=== Tarea 2: Renombrar Variable ===\n";
    $r = 7;
    echo "Área del círculo (ANTES): " . calculateArea($r) . "\n";
    echo "Área del círculo (DESPUÉS): " . calculateCircleArea($r) . "\n\n";

    echo "=== Tarea 3: Eliminar Código Duplicado ===\n";
    try {
        echo "Validando username (DESPUÉS): " . (validateUsernameRefactored("   user123   ") ? "OK" : "Error") . "\n";
        echo "Validando email (DESPUÉS): " . (validateEmailRefactored("user@example.com") ? "OK" : "Error") . "\n";
        // Descomenta la siguiente línea para ver la excepción:
        // validateUsernameRefactored("");
    } catch (Exception $e) {
        echo "Error: " . $e->getMessage() . "\n";
    }
    echo "\n";

    echo "=== Tarea 4: Reemplazar Números Mágicos ===\n";
    $weight = 15;
    echo "Costo de envío (ANTES): " . calculateShippingCost($weight) . "\n";
    echo "Costo de envío (DESPUÉS): " . calculateShippingCostRefactored($weight) . "\n\n";

    echo "=== Tarea 5: Simplificar Condicional ===\n";
    $customer1 = ['loyalty_years' => 6, 'has_coupon' => false];
    $customer2 = ['loyalty_years' => 2, 'has_coupon' => true];
    $customer3 = ['loyalty_years' => 2, 'has_coupon' => false];
    echo "Descuento para customer1 (ANTES): " . getDiscount($customer1) . "\n";
    echo "Descuento para customer1 (DESPUÉS): " . getDiscountRefactored($customer1) . "\n";
    echo "Descuento para customer2 (ANTES): " . getDiscount($customer2) . "\n";
    echo "Descuento para customer2 (DESPUÉS): " . getDiscountRefactored($customer2) . "\n";
    echo "Descuento para customer3 (ANTES): " . getDiscount($customer3) . "\n";
    echo "Descuento para customer3 (DESPUÉS): " . getDiscountRefactored($customer3) . "\n";
}

main();
?>
```

---

### Explicación General

1. **Extraer Función:**  
   Separamos la lógica de cálculo del total de cada ítem en la función `calculateItemTotal`, de modo que la función `processOrderRefactored` quede más clara y modular.

2. **Renombrar Variable:**  
   Cambiamos el nombre de la variable `$r` por `$radius` en la función que calcula el área del círculo, lo que mejora la legibilidad del código.

3. **Eliminar Código Duplicado:**  
   Centralizamos la validación de campos no vacíos en la función `validateNonEmpty` y luego la reutilizamos en funciones específicas para username y email, eliminando duplicación.

4. **Reemplazar Números Mágicos:**  
   Se definen constantes descriptivas para los límites de peso y costos de envío, haciendo el código más autoexplicativo y fácil de modificar.

5. **Simplificar Condicional:**  
   Se reestructura la función `getDiscount` eliminando condicionales anidados innecesarios y usando retornos tempranos, lo que mejora la claridad.

Este ejemplo demuestra cómo aplicar refactorizaciones comunes en PHP para mejorar la calidad, mantenibilidad y legibilidad del código en situaciones del día a día en el desarrollo profesional.
