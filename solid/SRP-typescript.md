## Principio de Responsabilidad Única (Unique Responsability Principle, SRP)

### Definición

**"Una clase debe tener una única razón para cambiar, es decir, debe tener una única responsabilidad."**

### Explicación

- **Responsabilidad Única**: Cada módulo, clase o función debe centrarse en una única tarea o responsabilidad. Esto significa que debe manejar una sola parte del funcionamiento del programa y esa parte debe estar encapsulada en la clase.

- **Razón para Cambiar**: Si una clase tiene más de una responsabilidad, se vuelve más difícil de mantener y de modificar, ya que un cambio en una responsabilidad puede afectar a las demás. Esto hace que el software sea más propenso a errores y más difícil de probar.

### Ejemplo

A continuación se muestra un ejemplo práctico en TypeScript donde se refactoriza un proceso de registro de usuario para cumplir con el Principio de Responsabilidad Única (SRP). En el ejemplo original, una única clase se encarga de validar, guardar y enviar una notificación por correo, mientras que en la versión refactorizada cada responsabilidad se delega en una clase específica.

### Antes del refactor (violación de SRP) 
```typescript
class UserRegistration {
  registerUser(user: { email: string; name: string }): void {
    // Validación simple
    if (!user.email.includes('@')) {
      throw new Error("Invalid email");
    }
    // Simulación de guardar en base de datos
    console.log("Saving user to the database:", user);
    // Simulación de envío de correo de confirmación
    console.log("Sending confirmation email to:", user.email);
  }
}

const user = { email: "example@example.com", name: "John Doe" };
const registration = new UserRegistration();
registration.registerUser(user);
```

### Después del refactor (aplicando SRP)
En esta versión, cada clase tiene una única responsabilidad:
- **UserValidator:** Se encarga de validar los datos del usuario.  
- **UserRepository:** Maneja el almacenamiento del usuario.  
- **EmailService:** Envía el correo de confirmación.  
- **UserRegistrationService:** Coordina el proceso de registro utilizando las clases anteriores.

```typescript
interface IUser {
  email: string;
  name: string;
}

class UserValidator {
  validate(user: IUser): void {
    if (!user.email.includes('@')) {
      throw new Error("Invalid email");
    }
    // Se pueden agregar otras validaciones aquí.
  }
}

class UserRepository {
  save(user: IUser): void {
    // Simulación de guardar en la base de datos.
    console.log("Saving user to the database:", user);
  }
}

class EmailService {
  sendConfirmation(user: IUser): void {
    // Simulación del envío de correo electrónico.
    console.log("Sending confirmation email to:", user.email);
  }
}

class UserRegistrationService {
  private validator: UserValidator;
  private repository: UserRepository;
  private emailService: EmailService;

  constructor(
    validator: UserValidator,
    repository: UserRepository,
    emailService: EmailService
  ) {
    this.validator = validator;
    this.repository = repository;
    this.emailService = emailService;
  }

  registerUser(user: IUser): void {
    this.validator.validate(user);
    this.repository.save(user);
    this.emailService.sendConfirmation(user);
  }
}

// Ejemplo de uso:
const user: IUser = { email: "example@example.com", name: "John Doe" };

const validator = new UserValidator();
const repository = new UserRepository();
const emailService = new EmailService();

const registrationService = new UserRegistrationService(validator, repository, emailService);
registrationService.registerUser(user);
```

### Conclusión del ejemplo

En este ejemplo refactorizado, cada clase cumple una única responsabilidad, lo que facilita el mantenimiento, la escalabilidad y las pruebas unitarias, siguiendo el principio SRP.

### Beneficios

1. **Mantenibilidad**: Al tener clases con responsabilidades claras y únicas, es más fácil comprender el código y realizar cambios sin riesgo de afectar otras funcionalidades.

2. **Reusabilidad**: Clases diseñadas con SRP son más fáciles de reutilizar en diferentes contextos, ya que están enfocadas en una única tarea.

3. **Escalabilidad**: Facilita la adición de nuevas funcionalidades al sistema, ya que al agregar clases que manejen nuevas responsabilidades no se alteran las existentes.

### Conclusión

El SRP es fundamental para crear un código limpio y organizado, facilitando la evolución del software y su comprensión por parte de otros desarrolladores. ¿Te gustaría saber más sobre cómo aplicar este principio en un contexto específico?