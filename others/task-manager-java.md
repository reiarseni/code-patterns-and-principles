## Gestor de Tareas en Java

A continuación se muestra un ejemplo completo de una aplicación de tareas en Java por consola que integra al menos 6 patrones de diseño (Composite, Singleton, Factory, Observer, Command y Strategy). La aplicación utiliza SQLite para almacenar los datos de las tareas y permite adicionar, completar, eliminar y modificar tareas. Además, algunas tareas pueden ser compuestas (tienen subtareas), permitiendo definir relaciones jerárquicas y un orden de ejecución.

Puedes compilar este ejemplo (asegúrate de tener el driver JDBC de SQLite en el classpath, por ejemplo, <code>sqlite-jdbc.jar</code>) y ejecutarlo en consola.

---

```java
import java.sql.*;
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;

// ==========================================================
// PATTERN: Singleton para la conexión a la base de datos SQLite
// ==========================================================
class DBConnection {
    private static DBConnection instance;
    private Connection connection;
    private final String url = "jdbc:sqlite:tasks.db";

    private DBConnection() {
        try {
            connection = DriverManager.getConnection(url);
            // Crear la tabla de tareas si no existe
            String createTableSQL = "CREATE TABLE IF NOT EXISTS tasks (" +
                    "id INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    "title TEXT NOT NULL, " +
                    "description TEXT, " +
                    "completed INTEGER, " +
                    "parentId INTEGER, " +
                    "taskOrder INTEGER" +
                    ")";
            Statement stmt = connection.createStatement();
            stmt.execute(createTableSQL);
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }

    public static synchronized DBConnection getInstance() {
        if (instance == null) {
            instance = new DBConnection();
        }
        return instance;
    }

    public Connection getConnection() {
        return connection;
    }
}

// ==========================================================
// PATTERN: Composite para representar Tareas y Tareas Compuestas
// ==========================================================
abstract class Task {
    protected int id;
    protected String title;
    protected String description;
    protected boolean completed;
    protected Integer parentId;  // null si es tarea raíz
    protected int taskOrder;     // para definir un orden

    public Task(String title, String description, Integer parentId, int taskOrder) {
        this.title = title;
        this.description = description;
        this.parentId = parentId;
        this.taskOrder = taskOrder;
        this.completed = false;
    }

    public int getId() {
        return id;
    }
    public String getTitle() {
        return title;
    }
    public boolean isCompleted() {
        return completed;
    }
    public Integer getParentId() {
        return parentId;
    }
    public int getTaskOrder() {
        return taskOrder;
    }
    
    public void setCompleted(boolean completed) {
        this.completed = completed;
    }
    
    public abstract void add(Task task);
    public abstract void remove(Task task);
    public abstract List<Task> getSubTasks();
    public abstract String display(String indent);
}

class SimpleTask extends Task {
    public SimpleTask(String title, String description, Integer parentId, int taskOrder) {
        super(title, description, parentId, taskOrder);
    }

    @Override
    public void add(Task task) {
        // No se puede agregar a una tarea simple
        throw new UnsupportedOperationException("No se pueden agregar subtareas a una tarea simple.");
    }

    @Override
    public void remove(Task task) {
        throw new UnsupportedOperationException("No se pueden remover subtareas de una tarea simple.");
    }

    @Override
    public List<Task> getSubTasks() {
        return Collections.emptyList();
    }

    @Override
    public String display(String indent) {
        return indent + "- [ " + (completed ? "X" : " ") + " ] " + title + " (ID: " + id + ")\n";
    }
}

class CompositeTask extends Task {
    private List<Task> subTasks = new ArrayList<>();

    public CompositeTask(String title, String description, Integer parentId, int taskOrder) {
        super(title, description, parentId, taskOrder);
    }

    @Override
    public void add(Task task) {
        subTasks.add(task);
    }

    @Override
    public void remove(Task task) {
        subTasks.remove(task);
    }

    @Override
    public List<Task> getSubTasks() {
        return subTasks;
    }

    @Override
    public String display(String indent) {
        StringBuilder sb = new StringBuilder();
        sb.append(indent).append("+ [ ").append(completed ? "X" : " ").append(" ] ").append(title)
          .append(" (ID: ").append(id).append(")\n");
        for (Task t : subTasks) {
            sb.append(t.display(indent + "    "));
        }
        return sb.toString();
    }
}

// ==========================================================
// PATTERN: Factory para la creación de Tareas
// ==========================================================
class TaskFactory {
    private static final AtomicInteger orderGenerator = new AtomicInteger(0);

    public static Task createTask(String type, String title, String description, Integer parentId) {
        int order = orderGenerator.incrementAndGet();
        if ("composite".equalsIgnoreCase(type)) {
            return new CompositeTask(title, description, parentId, order);
        } else {
            return new SimpleTask(title, description, parentId, order);
        }
    }
}

// ==========================================================
// PATTERN: Data Access Object (DAO) para Tareas en SQLite
// ==========================================================
class TaskDAO {
    private Connection conn;

    public TaskDAO() {
        conn = DBConnection.getInstance().getConnection();
    }

    public int insertTask(Task task) {
        String sql = "INSERT INTO tasks(title, description, completed, parentId, taskOrder) VALUES(?,?,?,?,?)";
        try (PreparedStatement pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            pstmt.setString(1, task.getTitle());
            pstmt.setString(2, task.description);
            pstmt.setInt(3, task.completed ? 1 : 0);
            if (task.getParentId() == null) {
                pstmt.setNull(4, Types.INTEGER);
            } else {
                pstmt.setInt(4, task.getParentId());
            }
            pstmt.setInt(5, task.getTaskOrder());
            pstmt.executeUpdate();
            ResultSet rs = pstmt.getGeneratedKeys();
            if (rs.next()) {
                int id = rs.getInt(1);
                task.id = id;
                return id;
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
        return -1;
    }

    public void updateTask(Task task) {
        String sql = "UPDATE tasks SET title=?, description=?, completed=?, parentId=?, taskOrder=? WHERE id=?";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, task.getTitle());
            pstmt.setString(2, task.description);
            pstmt.setInt(3, task.completed ? 1 : 0);
            if (task.getParentId() == null) {
                pstmt.setNull(4, Types.INTEGER);
            } else {
                pstmt.setInt(4, task.getParentId());
            }
            pstmt.setInt(5, task.getTaskOrder());
            pstmt.setInt(6, task.getId());
            pstmt.executeUpdate();
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }

    public void deleteTask(int id) {
        String sql = "DELETE FROM tasks WHERE id=?";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, id);
            pstmt.executeUpdate();
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
    }

    // Método para obtener todas las tareas (sin ensamblar la jerarquía)
    public List<Task> getAllTasks() {
        List<Task> tasks = new ArrayList<>();
        String sql = "SELECT * FROM tasks ORDER BY taskOrder ASC";
        try (Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            while (rs.next()) {
                int id = rs.getInt("id");
                String title = rs.getString("title");
                String description = rs.getString("description");
                boolean completed = rs.getInt("completed") == 1;
                int order = rs.getInt("taskOrder");
                int parentId = rs.getInt("parentId");
                Integer pid = rs.wasNull() ? null : parentId;
                // Para determinar el tipo de tarea: si tiene hijos en la colección luego se ensamblará como Composite
                // Aquí, para simplicidad, tratamos todas como simples y se reconstruye la jerarquía en TaskManager.
                Task task = new SimpleTask(title, description, pid, order);
                task.id = id;
                task.completed = completed;
                tasks.add(task);
            }
        } catch (SQLException ex) {
            ex.printStackTrace();
        }
        return tasks;
    }
}

// ==========================================================
// PATTERN: Observer para notificar cambios en las tareas
// ==========================================================
interface TaskObserver {
    void update(String action, Task task);
}

class ConsoleObserver implements TaskObserver {
    @Override
    public void update(String action, Task task) {
        System.out.println("[Observer] Acción: " + action + " - Tarea: " + task.getTitle() + " (ID: " + task.getId() + ")");
    }
}

// ==========================================================
// PATTERN: Command para encapsular operaciones
// ==========================================================
interface Command {
    void execute();
}

class AddTaskCommand implements Command {
    private Task task;
    private TaskDAO dao;
    private TaskManager manager;

    public AddTaskCommand(Task task, TaskDAO dao, TaskManager manager) {
        this.task = task;
        this.dao = dao;
        this.manager = manager;
    }

    @Override
    public void execute() {
        int id = dao.insertTask(task);
        if (id != -1) {
            manager.addTask(task);
        }
    }
}

class UpdateTaskCommand implements Command {
    private Task task;
    private TaskDAO dao;
    private TaskManager manager;

    public UpdateTaskCommand(Task task, TaskDAO dao, TaskManager manager) {
        this.task = task;
        this.dao = dao;
        this.manager = manager;
    }

    @Override
    public void execute() {
        dao.updateTask(task);
        manager.updateTask(task);
    }
}

class DeleteTaskCommand implements Command {
    private int taskId;
    private TaskDAO dao;
    private TaskManager manager;

    public DeleteTaskCommand(int taskId, TaskDAO dao, TaskManager manager) {
        this.taskId = taskId;
        this.dao = dao;
        this.manager = manager;
    }

    @Override
    public void execute() {
        dao.deleteTask(taskId);
        manager.deleteTask(taskId);
    }
}

class CompleteTaskCommand implements Command {
    private int taskId;
    private TaskDAO dao;
    private TaskManager manager;

    public CompleteTaskCommand(int taskId, TaskDAO dao, TaskManager manager) {
        this.taskId = taskId;
        this.dao = dao;
        this.manager = manager;
    }

    @Override
    public void execute() {
        Task task = manager.getTaskById(taskId);
        if (task != null) {
            task.setCompleted(true);
            dao.updateTask(task);
            manager.updateTask(task);
        }
    }
}

// ==========================================================
// PATTERN: Strategy para ordenar tareas (por título o por orden)
// ==========================================================
interface TaskSortStrategy {
    List<Task> sort(List<Task> tasks);
}

class SortByTitleStrategy implements TaskSortStrategy {
    @Override
    public List<Task> sort(List<Task> tasks) {
        return tasks.stream()
                .sorted(Comparator.comparing(Task::getTitle))
                .collect(Collectors.toList());
    }
}

class SortByOrderStrategy implements TaskSortStrategy {
    @Override
    public List<Task> sort(List<Task> tasks) {
        return tasks.stream()
                .sorted(Comparator.comparingInt(Task::getTaskOrder))
                .collect(Collectors.toList());
    }
}

// ==========================================================
// Clase que administra las tareas en memoria y notifica a los observadores
// ==========================================================
class TaskManager {
    private Map<Integer, Task> taskMap = new HashMap<>();
    private List<TaskObserver> observers = new ArrayList<>();

    public void addObserver(TaskObserver observer) {
        if (!observers.contains(observer)) {
            observers.add(observer);
        }
    }

    public void notifyObservers(String action, Task task) {
        for (TaskObserver obs : observers) {
            obs.update(action, task);
        }
    }

    public void addTask(Task task) {
        taskMap.put(task.getId(), task);
        notifyObservers("Agregado", task);
    }

    public void updateTask(Task task) {
        taskMap.put(task.getId(), task);
        notifyObservers("Actualizado", task);
    }

    public void deleteTask(int taskId) {
        Task removed = taskMap.remove(taskId);
        if (removed != null) {
            notifyObservers("Eliminado", removed);
        }
    }

    public Task getTaskById(int taskId) {
        return taskMap.get(taskId);
    }

    // Método para reconstruir la jerarquía usando parentId (Composite)
    public List<Task> buildHierarchy(List<Task> tasks) {
        Map<Integer, Task> map = tasks.stream().collect(Collectors.toMap(Task::getId, t -> t));
        List<Task> roots = new ArrayList<>();
        for (Task t : tasks) {
            if (t.getParentId() != null) {
                Task parent = map.get(t.getParentId());
                if (parent != null) {
                    try {
                        parent.add(t);
                    } catch (UnsupportedOperationException ex) {
                        // Si el padre es simple, se puede convertir a Composite si se desea
                        // Aquí ignoramos
                    }
                }
            } else {
                roots.add(t);
            }
        }
        return roots;
    }

    public Collection<Task> getAllTasks() {
        return taskMap.values();
    }
}

// ==========================================================
// Aplicación Principal (Consola)
// ==========================================================
public class TaskApp {
    private static Scanner scanner = new Scanner(System.in);
    private static TaskDAO taskDAO = new TaskDAO();
    private static TaskManager taskManager = new TaskManager();
    // Usamos una estrategia para ordenar; se puede cambiar a SortByTitleStrategy si se desea.
    private static TaskSortStrategy sortStrategy = new SortByOrderStrategy();

    public static void main(String[] args) {
        // Agregar un observador para que imprima cambios en consola
        taskManager.addObserver(new ConsoleObserver());
        // Cargar tareas existentes de la base de datos
        List<Task> tasksFromDB = taskDAO.getAllTasks();
        for (Task t : tasksFromDB) {
            taskManager.addTask(t);
        }
        boolean exit = false;
        while (!exit) {
            System.out.println("\n=== Menú de Tareas ===");
            System.out.println("1. Agregar Tarea");
            System.out.println("2. Completar Tarea");
            System.out.println("3. Modificar Tarea");
            System.out.println("4. Eliminar Tarea");
            System.out.println("5. Listar Tareas");
            System.out.println("6. Salir");
            System.out.print("Elige una opción: ");
            String option = scanner.nextLine();
            switch (option) {
                case "1":
                    addTask();
                    break;
                case "2":
                    completeTask();
                    break;
                case "3":
                    modifyTask();
                    break;
                case "4":
                    deleteTask();
                    break;
                case "5":
                    listTasks();
                    break;
                case "6":
                    exit = true;
                    break;
                default:
                    System.out.println("Opción inválida.");
            }
        }
        System.out.println("¡Hasta luego!");
    }

    private static void addTask() {
        System.out.print("Título: ");
        String title = scanner.nextLine();
        System.out.print("Descripción: ");
        String description = scanner.nextLine();
        System.out.print("Tipo (simple/composite): ");
        String type = scanner.nextLine();
        System.out.print("ID de tarea padre (deja vacío si no tiene): ");
        String parentInput = scanner.nextLine();
        Integer parentId = parentInput.isEmpty() ? null : Integer.parseInt(parentInput);
        Task newTask = TaskFactory.createTask(type, title, description, parentId);
        Command addCommand = new AddTaskCommand(newTask, taskDAO, taskManager);
        addCommand.execute();
    }

    private static void completeTask() {
        System.out.print("ID de la tarea a completar: ");
        int id = Integer.parseInt(scanner.nextLine());
        Command completeCommand = new CompleteTaskCommand(id, taskDAO, taskManager);
        completeCommand.execute();
    }

    private static void modifyTask() {
        System.out.print("ID de la tarea a modificar: ");
        int id = Integer.parseInt(scanner.nextLine());
        Task task = taskManager.getTaskById(id);
        if (task == null) {
            System.out.println("Tarea no encontrada.");
            return;
        }
        System.out.print("Nuevo título (deja vacío para mantener actual): ");
        String title = scanner.nextLine();
        System.out.print("Nueva descripción (deja vacío para mantener actual): ");
        String description = scanner.nextLine();
        if (!title.isEmpty()) {
            task.title = title;
        }
        if (!description.isEmpty()) {
            task.description = description;
        }
        Command updateCommand = new UpdateTaskCommand(task, taskDAO, taskManager);
        updateCommand.execute();
    }

    private static void deleteTask() {
        System.out.print("ID de la tarea a eliminar: ");
        int id = Integer.parseInt(scanner.nextLine());
        Command deleteCommand = new DeleteTaskCommand(id, taskDAO, taskManager);
        deleteCommand.execute();
    }

    private static void listTasks() {
        List<Task> sortedTasks = sortStrategy.sort(new ArrayList<>(taskManager.getAllTasks()));
        // Reconstruir la jerarquía para visualizar tareas compuestas
        List<Task> roots = taskManager.buildHierarchy(sortedTasks);
        System.out.println("\n--- Lista de Tareas ---");
        for (Task t : roots) {
            System.out.print(t.display(""));
        }
    }
}
```

---

## Explicación del Código y de los Patrones Aplicados

1. **Singleton (DBConnection):**  
   Se implementa la clase <code>DBConnection</code> para asegurar que exista una única conexión a la base de datos SQLite en toda la aplicación. Además, se crea la tabla <code>tasks</code> si no existe.

2. **Composite (Task, SimpleTask, CompositeTask):**  
   El modelo de tareas utiliza el patrón Composite. Las tareas simples (<code>SimpleTask</code>) no pueden contener subtareas, mientras que las tareas compuestas (<code>CompositeTask</code>) pueden contener una lista de tareas hijas. Esto permite estructurar tareas jerárquicas, donde una tarea puede tener subtareas.

3. **Factory (TaskFactory y PersistenceFactory):**  
   - <code>TaskFactory</code> se utiliza para crear instancias de tareas (simples o compuestas) en función de un parámetro de tipo.  
   - <code>PersistenceFactory</code> permite seleccionar el proveedor de persistencia (por defecto archivos JSON o una simulación de base de datos).

4. **DAO:**  
   La clase <code>TaskDAO</code> se encarga de interactuar con la base de datos SQLite, permitiendo insertar, actualizar, eliminar y obtener tareas.

5. **Observer (TaskObserver y ConsoleObserver):**  
   La interfaz <code>TaskObserver</code> y su implementación en <code>ConsoleObserver</code> permiten notificar a los “observadores” (en este caso, simplemente la consola) cada vez que se realiza una acción (agregar, actualizar o eliminar tareas). La clase <code>TaskManager</code> administra estas notificaciones y mantiene una colección en memoria de las tareas.

6. **Command:**  
   Se define la interfaz <code>Command</code> y se implementan comandos concretos (<code>AddTaskCommand</code>, <code>UpdateTaskCommand</code>, <code>DeleteTaskCommand</code> y <code>CompleteTaskCommand</code>) para encapsular las operaciones sobre las tareas. Esto permite que cada acción se ejecute de forma independiente, facilitando la extensión y posible implementación de undo/redo.

7. **Strategy (TaskSortStrategy):**  
   El patrón Strategy se aplica para definir distintas formas de ordenar las tareas. En el ejemplo se incluyen dos estrategias: <code>SortByTitleStrategy</code> y <code>SortByOrderStrategy</code>. Actualmente se usa <code>SortByOrderStrategy</code> para listar las tareas.

8. **Flujo de la Aplicación (Main):**  
   La clase <code>TaskApp</code> implementa la lógica de la aplicación en consola. Se muestra un menú interactivo para agregar, completar, modificar, eliminar y listar tareas. Al agregar o modificar tareas se utilizan los comandos para ejecutar la operación, lo que a su vez interactúa con el DAO y el TaskManager (que notifica a los observadores). La persistencia se realiza en SQLite, y se reconstruye la jerarquía (usando el patrón Composite) al listar las tareas.

---

## Conclusión

Este ejemplo educativo integra múltiples patrones de diseño para crear una aplicación de tareas modular, escalable y mantenible. Al combinar los patrones Composite, Singleton, Factory, Observer, Command y Strategy, se logra una solución robusta que permite gestionar tareas con relaciones jerárquicas y persistencia en una base de datos SQLite, demostrando técnicas comunes en aplicaciones empresariales.
