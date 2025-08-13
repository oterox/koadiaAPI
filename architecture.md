## 📁 ARCHITECTURE.md

# Arquitectura del Sistema
# Laravel Secure API

## 🏛️ Visión General de la Arquitectura

### Patrón Arquitectónico
**Arquitectura en Capas (Layered Architecture)** con elementos de **Repository Pattern** y **Service Layer Pattern**.

```
┌─────────────────────┐
│   API Routes        │ ← HTTP Requests
├─────────────────────┤
│   Controllers       │ ← Request handling
├─────────────────────┤
│   Form Requests     │ ← Input validation
├─────────────────────┤
│   Services          │ ← Business logic
├─────────────────────┤
│   Models/Eloquent   │ ← Data access
├─────────────────────┤
│   Database          │ ← Data persistence
└─────────────────────┘
```

## 🔄 Flujo de Request/Response

### Request Flow
1. **Route Matching:** Laravel router encuentra la ruta
2. **Middleware Pipeline:** Ejecuta middleware (auth, throttle, cors)
3. **Controller Method:** Recibe request validado
4. **Form Request:** Valida y sanitiza input
5. **Service Layer:** Ejecuta lógica de negocio
6. **Model Interaction:** Opera con base de datos
7. **Resource Transformation:** Formatea respuesta
8. **Response:** Retorna JSON estructurado

### Ejemplo Concreto - Crear Test Record
```
POST /api/test-records
    ↓
[auth:sanctum, throttle:60,1] middleware
    ↓
TestRecordController@store
    ↓
StoreTestRecordRequest (validation)
    ↓
TestRecordService@create
    ↓
TestRecord::create() + User relationship
    ↓
TestRecordResource::make()
    ↓
JSON Response
```

## 🏗️ Estructura de Directorios

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Api/
│   │   │   ├── AuthController.php
│   │   │   ├── UserController.php
│   │   │   ├── TestRecordController.php
│   │   │   └── RolePermissionController.php
│   │   └── Controller.php
│   ├── Requests/
│   │   ├── Auth/
│   │   ├── User/
│   │   └── TestRecord/
│   ├── Resources/
│   │   ├── UserResource.php
│   │   ├── TestRecordResource.php
│   │   └── RoleResource.php
│   ├── Middleware/
│   └── Kernel.php
├── Models/
│   ├── User.php
│   ├── TestRecord.php
│   └── Role.php (from Spatie)
├── Services/
│   ├── AuthService.php
│   ├── UserService.php
│   └── TestRecordService.php
├── Policies/
│   ├── UserPolicy.php
│   └── TestRecordPolicy.php
└── Providers/
```

## 🔐 Sistema de Autenticación y Autorización

### Flujo de Autenticación (Sanctum)
```
1. User login → Validate credentials
2. Generate personal access token
3. Return token to client  
4. Client includes token in Authorization header
5. Sanctum middleware validates token
6. Request proceeds with authenticated user
```

### Flujo de Autorización (Spatie Permissions)
```
1. Authenticated user attempts action
2. Policy/Gate checks user permissions
3. Query user roles and permissions
4. Allow/Deny based on rules
5. Continue or return 403 Forbidden
```

### Matriz de Permisos

| Rol     | Users CRUD | Test Records CRUD | Roles Management |
|---------|------------|-------------------|------------------|
| admin   | ✅ All      | ✅ All             | ✅ Full          |
| manager | ✅ Read/Update | ✅ All         | ❌ None          |
| user    | ✅ Own Profile | ✅ Own Records | ❌ None          |

## 💾 Diseño de Base de Datos

### Tablas Core
```sql
users
├── id (primary key)
├── name, email, password
├── email_verified_at
└── timestamps

test_records  
├── id (primary key)
├── user_id (foreign key → users.id)
├── title, description
├── status (enum)
├── priority (integer)
├── due_date, is_featured
├── metadata (json)
└── timestamps
```

### Tablas de Permisos (Spatie Package)
```sql
roles
permissions  
model_has_roles (polymorphic)
model_has_permissions (polymorphic)
role_has_permissions
```

### Relaciones Eloquent
```php
// User Model
public function testRecords() {
    return $this->hasMany(TestRecord::class);
}

public function roles() {
    return $this->belongsToMany(Role::class);
}

// TestRecord Model  
public function user() {
    return $this->belongsTo(User::class);
}
```

## 🔧 Servicios y Dependencias

### Service Container Bindings
```php
// AppServiceProvider
$this->app->bind(AuthServiceInterface::class, AuthService::class);
$this->app->bind(UserServiceInterface::class, UserService::class);
```

### Dependency Injection Pattern
```php
class TestRecordController extends Controller
{
    public function __construct(
        private TestRecordService $testRecordService,
        private AuthService $authService
    ) {}
}
```

## 📡 API Design Patterns

### RESTful Resource Design
```
GET    /api/test-records      → index()
POST   /api/test-records      → store()  
GET    /api/test-records/{id} → show()
PUT    /api/test-records/{id} → update()
DELETE /api/test-records/{id} → destroy()
```

### Consistent Response Structure
```json
{
  "success": boolean,
  "message": "string",
  "data": object|array,
  "meta": {
    "pagination": {},
    "filters": {},
    "total_count": integer
  },
  "errors": array // only on validation failures
}
```

### Error Handling Strategy
```php
// Custom Exception Handler
public function render($request, Throwable $exception)
{
    if ($request->expectsJson()) {
        return $this->handleApiException($request, $exception);
    }
    
    return parent::render($request, $exception);
}
```

## 🧪 Testing Architecture

### Test Structure
```
tests/
├── Feature/
│   ├── Auth/
│   │   ├── LoginTest.php
│   │   ├── RegistrationTest.php
│   │   └── PasswordResetTest.php
│   ├── Api/
│   │   ├── UserControllerTest.php
│   │   ├── TestRecordControllerTest.php
│   │   └── RolePermissionTest.php
│   └── TestCase.php
├── Unit/
│   ├── Models/
│   ├── Services/
│   └── Policies/
└── CreatesApplication.php
```

### Test Database Strategy
- **Separate test database:** `laravel_api_test`
- **Database transactions:** Each test in transaction
- **Factory pattern:** Consistent test data
- **Seeded permissions:** Standard roles/permissions

### Testing Utilities
```php
// Base test case with helpers
abstract class TestCase extends BaseTestCase
{
    protected function loginAs($user = null, $abilities = ['*'])
    {
        $user = $user ?: User::factory()->create();
        $token = $user->createToken('test', $abilities);
        
        $this->withHeaders([
            'Authorization' => 'Bearer ' . $token->plainTextToken
        ]);
        
        return $user;
    }
}
```

## 🐳 Infrastructure (Docker)

### Container Architecture
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Nginx     │───▶│ PHP-FPM     │───▶│   MySQL     │
│  (Web)      │    │ (Laravel)   │    │ (Database)  │
│   :7001     │    │    :9000    │    │   :3316     │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                          ▼
                   ┌─────────────┐    ┌─────────────┐
                   │   Redis     │    │  MailHog    │
                   │  (Cache)    │    │   (Mail)    │
                   │   :6379     │    │   :8025     │
                   └─────────────┘    └─────────────┘
```

### Network Communication
- **Internal network:** `laravel_network` (bridge driver)
- **Service discovery:** By container name
- **Port mapping:** Only necessary ports exposed to host

## 📊 Performance Considerations

### Caching Strategy
- **Redis:** Session, cache, and queue driver
- **Query caching:** For expensive database operations
- **Route caching:** In production environment
- **Config caching:** Optimized configuration loading

### Database Optimization
- **Indexes:** On foreign keys and search fields
- **Eager loading:** Prevent N+1 query problems
- **Pagination:** For large datasets
- **Connection pooling:** Efficient database connections

## 🚀 Deployment Strategy

### Environment Configuration
```
Development → Testing → Staging → Production
     ↓           ↓        ↓          ↓
  Docker     CI/CD    Docker    Kubernetes
  Compose    Tests    Swarm     /Docker
```

### Health Checks
```php
// Health check endpoint
Route::get('/health', function () {
    return response()->json([
        'status' => 'healthy',
        'database' => DB::connection()->getPdo() ? 'connected' : 'disconnected',
        'cache' => Cache::store('redis')->get('health_check') ? 'connected' : 'disconnected',
        'timestamp' => now()->toISOString()
    ]);
});