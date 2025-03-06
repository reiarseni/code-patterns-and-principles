## Imágenes y Debounce Filter

A continuación se muestra un ejemplo completo en HTML, CSS y JavaScript que utiliza la API pública de Pixabay para traer imágenes y mostrarlas en "cards" estilizadas. En este ejemplo se incorpora:

- **Debouncing configurable:** El usuario puede modificar el delay (en milisegundos) mediante un input.
- **Uso de patrones de diseño:**  
  - **Strategy Pattern:** En la clase `Debouncer` para encapsular la lógica del debouncing.  
  - **Singleton Pattern:** En la clase `PixabayService` para asegurar una única instancia del servicio de llamadas a la API.  
  - **Mediator/Controller Pattern:** En la clase `SearchController` para coordinar la interacción entre la vista, el debouncing y el servicio de API.  

Reemplaza `"YOUR_API_KEY"` por tu clave de API válida de Pixabay.

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Buscador de Imágenes con Pixabay</title>
  <style>
    /* Estilos básicos para la disposición de las cards */
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
    }
    .search-container {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-bottom: 20px;
    }
    #searchBox, #delayInput {
      padding: 10px;
      font-size: 16px;
    }
    .cards-container {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
      gap: 15px;
    }
    .card {
      box-shadow: 0 2px 5px rgba(0,0,0,0.2);
      border-radius: 5px;
      overflow: hidden;
      background-color: #fff;
    }
    .card img {
      width: 100%;
      height: auto;
      display: block;
    }
    .card-content {
      padding: 10px;
    }
  </style>
</head>
<body>
  <div class="search-container">
    <input type="text" id="searchBox" placeholder="Buscar imágenes...">
    <input type="number" id="delayInput" placeholder="Delay (ms)" value="500">
  </div>
  <div class="cards-container" id="cardsContainer"></div>
  
  <script>
    // Debouncer: Encapsula la lógica de debouncing (Strategy Pattern)
    class Debouncer {
      constructor(delay = 500) {
        this.delay = delay;
        this.timeoutId = null;
      }
      
      setDelay(delay) {
        this.delay = delay;
      }
      
      debounce(func) {
        return (...args) => {
          if (this.timeoutId) clearTimeout(this.timeoutId);
          this.timeoutId = setTimeout(() => {
            func(...args);
          }, this.delay);
        };
      }
    }
    
    // Servicio de Pixabay (Singleton Pattern)
    class PixabayService {
      constructor() {
        if (PixabayService.instance) {
          return PixabayService.instance;
        }
        this.apiKey = 'YOUR_API_KEY'; // Reemplaza con tu clave de API de Pixabay
        this.baseUrl = 'https://pixabay.com/api/';
        PixabayService.instance = this;
      }
      
      async fetchImages(query) {
        const url = `${this.baseUrl}?key=${this.apiKey}&q=${encodeURIComponent(query)}&image_type=photo`;
        try {
          const response = await fetch(url);
          const data = await response.json();
          return data.hits;
        } catch (error) {
          console.error('Error al traer imágenes:', error);
          return [];
        }
      }
    }
    
    // Vista: Encargada de renderizar las imágenes en cards
    class ImageView {
      constructor(containerId) {
        this.container = document.getElementById(containerId);
      }
      
      render(images) {
        this.container.innerHTML = '';
        if (images.length === 0) {
          this.container.innerHTML = '<p>No se encontraron imágenes.</p>';
          return;
        }
        images.forEach(image => {
          const card = document.createElement('div');
          card.className = 'card';
          
          const img = document.createElement('img');
          img.src = image.webformatURL;
          img.alt = image.tags;
          
          const content = document.createElement('div');
          content.className = 'card-content';
          content.innerHTML = `<p><strong>${image.user}</strong></p><p>Likes: ${image.likes}</p>`;
          
          card.appendChild(img);
          card.appendChild(content);
          this.container.appendChild(card);
        });
      }
    }
    
    // Controlador: Coordina la interacción entre la vista, el debouncer y el servicio (Mediator Pattern)
    class SearchController {
      constructor(searchInputId, delayInputId, view) {
        this.searchInput = document.getElementById(searchInputId);
        this.delayInput = document.getElementById(delayInputId);
        this.view = view;
        this.pixabayService = new PixabayService();
        this.debouncer = new Debouncer(parseInt(this.delayInput.value));
        
        // Asignación de eventos
        this.searchInput.addEventListener('input', this.handleSearch.bind(this));
        this.delayInput.addEventListener('change', this.handleDelayChange.bind(this));
      }
      
      handleDelayChange() {
        const newDelay = parseInt(this.delayInput.value);
        if (!isNaN(newDelay) && newDelay >= 0) {
          this.debouncer.setDelay(newDelay);
        }
      }
      
      handleSearch() {
        // Se envuelve la llamada con el debouncer
        const debouncedFetch = this.debouncer.debounce(async () => {
          const query = this.searchInput.value.trim();
          if (query.length > 0) {
            const images = await this.pixabayService.fetchImages(query);
            this.view.render(images);
          } else {
            this.view.render([]);
          }
        });
        debouncedFetch();
      }
    }
    
    // Inicialización de la aplicación
    document.addEventListener('DOMContentLoaded', () => {
      const imageView = new ImageView('cardsContainer');
      new SearchController('searchBox', 'delayInput', imageView);
    });
  </script>
</body>
</html>
```

### Explicación del Código

- **HTML & CSS:**  
  Se definen dos inputs: uno para el término de búsqueda y otro para el delay configurable. Las imágenes se muestran en un contenedor que usa un grid para crear un layout de cards con estilos simples.

- **Clase Debouncer:**  
  Permite encapsular la lógica del debouncing. La función `debounce` limpia el timeout anterior y programa la ejecución de la función pasada tras el delay configurado.

- **Clase PixabayService:**  
  Implementa el patrón Singleton para gestionar las peticiones a la API de Pixabay. La función `fetchImages` construye la URL, llama a la API y retorna las imágenes.

- **Clase ImageView:**  
  Se encarga de renderizar en el DOM las imágenes obtenidas, creando "cards" con la imagen, nombre del usuario y número de likes.

- **Clase SearchController:**  
  Actúa como mediador coordinando eventos: actualiza el delay cuando el usuario lo cambia y ejecuta la búsqueda (con debouncing) cuando se ingresa un término.

Esta estructura modular y orientada a objetos permite un código más mantenible y escalable, aplicando varios patrones de diseño que ayudan a separar responsabilidades y a mejorar la claridad del flujo de la aplicación.
