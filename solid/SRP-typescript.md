## Principio de Responsabilidad Única (Unique Responsability Principle, SRP)

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

### Conclusión

En este ejemplo refactorizado, cada clase cumple una única responsabilidad, lo que facilita el mantenimiento, la escalabilidad y las pruebas unitarias, siguiendo el principio SRP.
