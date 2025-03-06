## Proxy en PHP

A continuación se muestra un ejemplo realista y profesional del patrón Proxy en PHP 8, aplicado a un caso de caching para un servicio de consulta de tipos de cambio. En muchas aplicaciones, especialmente aquellas que consumen APIs externas lentas o costosas, se utiliza un proxy para cachear resultados y evitar llamadas repetidas. Este ejemplo ilustra cómo encapsular la lógica de caching en un proxy, desacoplando la implementación real del servicio.

---

```php
<?php
declare(strict_types=1);

// Interfaz que define el contrato del servicio de tipos de cambio.
interface ExchangeRateServiceInterface {
    public function getExchangeRate(string $currency): float;
}

// Implementación real del servicio (por ejemplo, consumiendo una API externa).
class RealExchangeRateService implements ExchangeRateServiceInterface {
    public function getExchangeRate(string $currency): float {
        // Simulación de una llamada lenta a una API externa.
        echo "Consultando API remota para el tipo de cambio de {$currency}...\n";
        sleep(2); // Simula retardo en la respuesta.
        // Valores de ejemplo; en una aplicación real estos serían obtenidos dinámicamente.
        $rates = [
            'USD' => 1.0,
            'EUR' => 0.85,
            'GBP' => 0.75,
        ];
        return $rates[$currency] ?? 1.0;
    }
}

// Proxy que agrega caching al servicio de tipos de cambio.
class ExchangeRateServiceProxy implements ExchangeRateServiceInterface {
    private RealExchangeRateService $realService;
    private array $cache = []; // Almacena los tipos de cambio cacheados.

    public function __construct() {
        $this->realService = new RealExchangeRateService();
    }

    public function getExchangeRate(string $currency): float {
        // Si el tipo de cambio ya está en caché, se devuelve sin realizar una nueva consulta.
        if (isset($this->cache[$currency])) {
            echo "Obteniendo tipo de cambio de {$currency} desde la caché...\n";
            return $this->cache[$currency];
        }
        // Si no está en caché, se consulta el servicio real y se guarda el resultado.
        $rate = $this->realService->getExchangeRate($currency);
        $this->cache[$currency] = $rate;
        return $rate;
    }
}

// Ejemplo de uso en una aplicación profesional.
function main(): void {
    $exchangeService = new ExchangeRateServiceProxy();

    // Primera consulta: se realiza la llamada real y se cachea el resultado.
    echo "Tipo de cambio EUR: " . $exchangeService->getExchangeRate('EUR') . "\n";
    // Segunda consulta con el mismo parámetro: se obtiene de la caché, sin retardo.
    echo "Tipo de cambio EUR: " . $exchangeService->getExchangeRate('EUR') . "\n";
    // Consulta para otra moneda.
    echo "Tipo de cambio GBP: " . $exchangeService->getExchangeRate('GBP') . "\n";
}

main();
```

---

### Explicación

- **Interfaz Común:**  
  Se define `ExchangeRateServiceInterface` para que tanto el servicio real como el proxy implementen el mismo contrato, facilitando su intercambiabilidad.

- **Servicio Real:**  
  `RealExchangeRateService` simula una llamada a una API externa lenta (usando `sleep(2)`) y retorna tipos de cambio predefinidos. En una aplicación real, esta clase se encargaría de realizar la consulta a la API.

- **Proxy con Caching:**  
  `ExchangeRateServiceProxy` encapsula una instancia de `RealExchangeRateService` y almacena en un arreglo (`$cache`) los resultados obtenidos. Antes de realizar una llamada real, verifica si el resultado ya está en caché, evitando así retardo y consumo innecesario de recursos.

- **Uso Profesional:**  
  En aplicaciones reales, es muy común utilizar proxies para mejorar el rendimiento y la escalabilidad. Por ejemplo, al consultar servicios externos o bases de datos, se puede evitar la sobrecarga de llamadas repetitivas mediante caching, lo que mejora la experiencia del usuario y reduce costos operativos.

Este ejemplo muestra cómo el patrón Proxy se puede aplicar para gestionar situaciones comunes del día a día, como el caching de resultados de servicios externos, un caso clásico en aplicaciones profesionales con PHP 8.
