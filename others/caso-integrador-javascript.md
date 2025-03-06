## Caso Integrador en JavaScript

A continuación se muestra un archivo HTML completo que integra varios patrones de diseño:  
- Se usan patrones de diseño **Strategy** (para obtener datos desde la API con 3 estrategias diferentes), **Observer** (para notificar a dos componentes de UI que deben actualizarse), **Singleton** (para la configuración global), **Builder** (para construir configuraciones complejas de filtrado) y **Factory** (para la creación centralizada de estrategias).  
- Además, se incluye un detallado “guía” en la propia página explicando cada uno de estos patrones y cómo se integran en el ejemplo.

Puedes guardar este contenido en un archivo (por ejemplo, `guia_patrones.html`) y abrirlo en tu navegador para ver tanto la aplicación en funcionamiento como la guía explicativa.

---

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Guía de Patrones de Diseño en JavaScript Moderno</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      line-height: 1.6;
    }
    h1, h2, h3 {
      color: #333;
    }
    .strategy-btn {
      margin-right: 10px;
      padding: 8px 12px;
      cursor: pointer;
      background-color: #007BFF;
      color: white;
      border: none;
      border-radius: 4px;
    }
    .strategy-btn:hover {
      background-color: #0056b3;
    }
    input {
      padding: 8px;
      width: 300px;
      margin-bottom: 20px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 20px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 8px;
      text-align: left;
    }
    .user-card {
      border: 1px solid #ccc;
      padding: 10px;
      margin: 5px;
      border-radius: 5px;
      display: inline-block;
      width: 200px;
      vertical-align: top;
    }
    .section-guide {
      background: #f7f7f7;
      padding: 15px;
      margin-bottom: 20px;
      border: 1px solid #ddd;
    }
    .section-code {
      background: #e7f5ff;
      padding: 15px;
      margin-bottom: 20px;
      border: 1px solid #b3e5ff;
    }
  </style>
</head>
<body>
  <h1>Guía de Patrones de Diseño en JavaScript Moderno</h1>
  
  <div class="section-guide">
    <h2>Introducción</h2>
    <p>
      En esta guía se integra un ejemplo real que combina múltiples patrones de diseño:
    </p>
    <ul>
      <li><strong>Strategy:</strong> Permite definir distintas estrategias para obtener datos desde una API (usando <em>async/await</em>, <em>Promises.then</em> y una versión formal de <code>fetch</code>).</li>
      <li><strong>Observer:</strong> Notifica a dos componentes de la UI (una tabla y una lista de tarjetas) cuando los datos han sido obtenidos y filtrados.</li>
      <li><strong>Singleton:</strong> Asegura que la configuración global (por ejemplo, la URL de la API) sea única en toda la aplicación.</li>
      <li><strong>Builder:</strong> Facilita la creación escalonada de configuraciones complejas (por ejemplo, para filtrar resultados).</li>
      <li><strong>Factory:</strong> Centraliza la creación de las estrategias de obtención de datos.</li>
    </ul>
  </div>

  <div class="section-guide">
    <h2>Estructura de la Aplicación</h2>
    <p>
      La aplicación cuenta con:
    </p>
    <ul>
      <li>Un conjunto de botones que permiten cambiar la estrategia de obtención de datos.</li>
      <li>Un campo de búsqueda que filtra los datos obtenidos.</li>
      <li>Dos áreas de visualización: una en forma de tabla y otra como lista de tarjetas.</li>
      <li>Un patrón Observer que actualiza ambas áreas de la UI cada vez que se obtienen o filtran datos.</li>
    </ul>
  </div>

  <div class="section-guide">
    <h2>Explicación de los Patrones Utilizados</h2>
    <h3>1. Strategy</h3>
    <p>
      Se definen tres estrategias para obtener datos desde la API:
      <em>AsyncAwaitFetcher</em> (usando async/await), <em>PromiseThenFetcher</em> (usando <code>.then()</code>) y <em>FormalFetchFetcher</em> (con comprobaciones formales de la respuesta). La clase <code>DataFetcher</code> actúa como contexto y permite cambiar la estrategia en tiempo real.
    </p>
    <h3>2. Observer</h3>
    <p>
      El patrón Observer se implementa mediante la clase <code>DataSubject</code>. Esta clase permite suscribir múltiples funciones (observadores) que se notificarán cuando los datos se actualicen. En nuestro ejemplo, las funciones <code>displayResultsTable</code> y <code>displayResultsCardList</code> se suscriben para que la UI se actualice.
    </p>
    <h3>3. Singleton</h3>
    <p>
      La clase <code>Config</code> implementa el patrón Singleton para garantizar que la configuración global (como la URL de la API) se mantenga única en toda la aplicación.
    </p>
    <h3>4. Builder</h3>
    <p>
      El patrón Builder se muestra con <code>FilterConfigBuilder</code>, que permite construir un objeto de configuración para el filtrado de datos de forma escalonada.
    </p>
    <h3>5. Factory</h3>
    <p>
      La clase <code>FetchStrategyFactory</code> centraliza la creación de las estrategias de obtención de datos. Esto facilita la selección y extensión de estrategias en el futuro.
    </p>
  </div>

  <div class="section-guide">
    <h2>Uso e Interacción</h2>
    <p>
      Al iniciar la aplicación, se obtienen los datos de una API pública (<code>jsonplaceholder.typicode.com/users</code>). El usuario puede:
    </p>
    <ul>
      <li>Cambiar la estrategia de obtención de datos mediante los botones.</li>
      <li>Escribir en el campo de búsqueda para filtrar los usuarios en tiempo real.</li>
      <li>Ver los resultados actualizados simultáneamente en una tabla y en una lista de tarjetas.</li>
    </ul>
  </div>

  <!-- Área de interacción y visualización -->
  <div>
    <div>
      <button id="btn-async" class="strategy-btn">Async/Await</button>
      <button id="btn-then" class="strategy-btn">Promises.then</button>
      <button id="btn-formal" class="strategy-btn">Fetch Formal</button>
    </div>
    <br>
    <input type="text" id="searchInput" placeholder="Buscar usuario por nombre...">
  </div>

  <h2>Resultados en Tabla</h2>
  <div id="resultsTable"></div>
  
  <h2>Resultados en Lista de Tarjetas</h2>
  <div id="resultsCardList"></div>

  <!-- Script con la implementación de los patrones y la lógica de la aplicación -->
  <script>
    /*******************************
     * PATRÓN SINGLETON: Configuración Global
     *******************************/
    class Config {
      constructor() {
        if (Config.instance) {
          return Config.instance;
        }
        // Configuración global única
        this.API_URL = 'https://jsonplaceholder.typicode.com/users';
        Config.instance = this;
      }
      setAPIUrl(url) {
        this.API_URL = url;
      }
      getAPIUrl() {
        return this.API_URL;
      }
    }
    const config = new Config();

    /*******************************
     * PATRÓN BUILDER: Configuración de Filtrado
     *******************************/
    class FilterConfig {
      constructor() {
        this.searchTerm = '';
        this.sortBy = null;
        this.limit = null;
      }
    }

    class FilterConfigBuilder {
      constructor() {
        this.config = new FilterConfig();
      }
      setSearchTerm(term) {
        this.config.searchTerm = term;
        return this;
      }
      setSortBy(field) {
        this.config.sortBy = field;
        return this;
      }
      setLimit(limit) {
        this.config.limit = limit;
        return this;
      }
      build() {
        return this.config;
      }
    }
    // Ejemplo de uso (actualmente no se utiliza directamente en el filtrado)
    const filterConfig = new FilterConfigBuilder()
      .setSearchTerm('')
      .setSortBy('name')
      .setLimit(10)
      .build();

    /*******************************
     * PATRÓN FACTORY: Creación de Estrategias de Fetch
     *******************************/
    class FetchStrategyFactory {
      static createStrategy(type) {
        switch(type) {
          case 'async': return new AsyncAwaitFetcher();
          case 'then': return new PromiseThenFetcher();
          case 'formal': return new FormalFetchFetcher();
          default: return new AsyncAwaitFetcher();
        }
      }
    }

    /*******************************
     * PATRÓN STRATEGY: Estrategias para Obtener Datos
     *******************************/
    class IFetchStrategy {
      fetchData(url) {
        throw new Error("El método fetchData() debe ser implementado.");
      }
    }

    // Estrategia 1: Async/Await
    class AsyncAwaitFetcher extends IFetchStrategy {
      async fetchData(url) {
        try {
          const response = await fetch(url);
          if (!response.ok) throw new Error("Error en la respuesta");
          const data = await response.json();
          return data;
        } catch (error) {
          console.error("Error en Async/Await:", error);
          return [];
        }
      }
    }

    // Estrategia 2: Promises.then
    class PromiseThenFetcher extends IFetchStrategy {
      fetchData(url) {
        return fetch(url)
          .then(response => {
            if (!response.ok) throw new Error("Error en la respuesta");
            return response.json();
          })
          .then(data => data)
          .catch(error => {
            console.error("Error en Promises.then:", error);
            return [];
          });
      }
    }

    // Estrategia 3: Fetch Formal (con comprobación formal)
    class FormalFetchFetcher extends IFetchStrategy {
      fetchData(url) {
        return new Promise((resolve, reject) => {
          fetch(url)
            .then(response => {
              if (response.ok) return response.json();
              else reject("Error: " + response.status);
            })
            .then(data => resolve(data))
            .catch(error => {
              console.error("Error en Fetch Formal:", error);
              resolve([]);
            });
        });
      }
    }

    // Contexto para el patrón Strategy
    class DataFetcher {
      constructor(strategy) {
        this.strategy = strategy;
      }
      setStrategy(strategy) {
        this.strategy = strategy;
      }
      fetch(url) {
        return this.strategy.fetchData(url);
      }
    }

    /*******************************
     * PATRÓN OBSERVER: Notificación a Componentes de la UI
     *******************************/
    class DataSubject {
      constructor() {
        this.observers = [];
      }
      subscribe(observer) {
        if (!this.observers.includes(observer)) {
          this.observers.push(observer);
        }
      }
      unsubscribe(observer) {
        this.observers = this.observers.filter(obs => obs !== observer);
      }
      notify(data) {
        this.observers.forEach(observer => observer(data));
      }
    }
    const dataSubject = new DataSubject();
    // Suscribimos las funciones que actualizarán la UI:
    dataSubject.subscribe(displayResultsTable);
    dataSubject.subscribe(displayResultsCardList);

    /*******************************
     * FUNCIONES DE FILTRADO DE DATOS
     *******************************/
    // Filtrado usando map y filter
    function filterUsersWithMapFilter(users, searchTerm) {
      const simplifiedUsers = users.map(user => ({
        id: user.id,
        name: user.name,
        email: user.email
      }));
      return simplifiedUsers.filter(user =>
        user.name.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }

    // Filtrado usando ciclo while y comprobaciones manuales
    function filterUsersWithWhile(users, searchTerm) {
      let filtered = [];
      let i = 0;
      while (i < users.length) {
        const user = users[i];
        if (user.name.toLowerCase().includes(searchTerm.toLowerCase())) {
          filtered.push({
            id: user.id,
            name: user.name,
            email: user.email
          });
        }
        i++;
      }
      return filtered;
    }

    /*******************************
     * FUNCIONES PARA MOSTRAR RESULTADOS (OBSERVADORES)
     *******************************/
    // Muestra los datos en forma de tabla
    function displayResultsTable(users) {
      const tableDiv = document.getElementById('resultsTable');
      tableDiv.innerHTML = '';
      if (users.length === 0) {
        tableDiv.innerHTML = '<p>No se encontraron usuarios.</p>';
        return;
      }
      const table = document.createElement('table');
      const headerRow = document.createElement('tr');
      headerRow.innerHTML = `<th>ID</th><th>Nombre</th><th>Email</th>`;
      table.appendChild(headerRow);
      users.forEach(user => {
        const row = document.createElement('tr');
        row.innerHTML = `<td>${user.id}</td><td>${user.name}</td><td>${user.email}</td>`;
        table.appendChild(row);
      });
      tableDiv.appendChild(table);
    }

    // Muestra los datos en forma de lista de tarjetas
    function displayResultsCardList(users) {
      const cardDiv = document.getElementById('resultsCardList');
      cardDiv.innerHTML = '';
      if (users.length === 0) {
        cardDiv.innerHTML = '<p>No se encontraron usuarios.</p>';
        return;
      }
      users.forEach(user => {
        const card = document.createElement('div');
        card.className = 'user-card';
        card.innerHTML = `<strong>${user.name}</strong><br>${user.email}`;
        cardDiv.appendChild(card);
      });
    }

    /*******************************
     * CONFIGURACIÓN INICIAL Y MANEJO DE EVENTOS
     *******************************/
    // Se obtiene la URL de la API desde el Singleton Config.
    const API_URL = config.getAPIUrl();
    // Instanciamos el DataFetcher con la estrategia por defecto (Async/Await)
    const fetcher = new DataFetcher(new AsyncAwaitFetcher());
    let allUsers = [];

    // Función que carga los datos, filtra según el término de búsqueda y notifica a los observadores.
    async function loadDataAndNotify() {
      allUsers = await fetcher.fetch(API_URL);
      const searchTerm = document.getElementById('searchInput').value.trim();
      // Se puede alternar entre filtrado con map/filter o con while
      // const filtered = searchTerm ? filterUsersWithMapFilter(allUsers, searchTerm) : allUsers;
      const filtered = searchTerm ? filterUsersWithWhile(allUsers, searchTerm) : allUsers;
      dataSubject.notify(filtered);
    }

    // Manejo del input de búsqueda en tiempo real
    document.getElementById('searchInput').addEventListener('input', loadDataAndNotify);

    // Botones para cambiar la estrategia de obtención de datos utilizando la Factory
    document.getElementById('btn-async').addEventListener('click', async () => {
      fetcher.setStrategy(FetchStrategyFactory.createStrategy('async'));
      await loadDataAndNotify();
    });
    document.getElementById('btn-then').addEventListener('click', async () => {
      fetcher.setStrategy(FetchStrategyFactory.createStrategy('then'));
      await loadDataAndNotify();
    });
    document.getElementById('btn-formal').addEventListener('click', async () => {
      fetcher.setStrategy(FetchStrategyFactory.createStrategy('formal'));
      await loadDataAndNotify();
    });

    // Cargar datos iniciales al iniciar la aplicación
    loadDataAndNotify();
  </script>
</body>
</html>
```

---

### Explicación Detallada

1. **Singleton (Clase Config):**  
   - La clase <code>Config</code> se asegura de que solo exista una instancia en toda la aplicación.  
   - Se utiliza para almacenar la URL de la API y otros parámetros globales.  
   - Ejemplo de uso: <code>config.getAPIUrl()</code> obtiene la URL configurada.

2. **Builder (FilterConfigBuilder):**  
   - Permite construir un objeto de configuración para el filtrado de datos de forma modular.  
   - En el ejemplo se configura un objeto <code>FilterConfig</code> con parámetros como <em>searchTerm</em>, <em>sortBy</em> y <em>limit</em>.

3. **Factory (FetchStrategyFactory):**  
   - Centraliza la creación de instancias de estrategias de obtención de datos (Async/Await, Promises.then, Fetch Formal).  
   - Facilita la extensión futura y la selección automática de la estrategia deseada.

4. **Strategy (IFetchStrategy y sus Implementaciones):**  
   - Se definen tres estrategias para obtener datos desde la API:  
     <code>AsyncAwaitFetcher</code> (usando async/await),  
     <code>PromiseThenFetcher</code> (usando .then()),  
     <code>FormalFetchFetcher</code> (con comprobaciones formales).  
   - La clase <code>DataFetcher</code> utiliza la estrategia actual para obtener datos, permitiendo cambiarla dinámicamente.

5. **Observer (DataSubject):**  
   - Permite suscribir múltiples funciones que se notificarán cuando se actualicen los datos.  
   - En este caso, las funciones <code>displayResultsTable</code> y <code>displayResultsCardList</code> se suscriben para actualizar la UI (tabla y lista de tarjetas).

6. **Filtrado y Visualización:**  
   - Se implementan dos técnicas de filtrado (una con <code>map/filter</code> y otra con un ciclo <code>while</code>) para procesar los usuarios obtenidos según el término de búsqueda ingresado.  
   - La función <code>loadDataAndNotify</code> carga los datos desde la API, filtra según el input del usuario y notifica a los observadores para que actualicen la vista.

7. **Interacción:**  
   - Los botones permiten cambiar la estrategia de obtención de datos en tiempo real, utilizando la fábrica para crear la estrategia deseada.  
   - El campo de búsqueda actualiza automáticamente la visualización de los resultados en ambas áreas (tabla y tarjetas).

Esta guía integra de forma elegante varios patrones de diseño para crear una aplicación modular, escalable y fácil de mantener en JavaScript moderno. ¡Explora, modifica y disfruta de la flexibilidad que ofrecen estos patrones en tus proyectos diarios!
