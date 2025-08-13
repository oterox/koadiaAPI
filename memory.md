## 📁 MEMORY.md

# Memory File - Laravel Secure API Project
# Contexto y Decisiones del Proyecto

## 🧠 Contexto del Proyecto

### Propósito
Este proyecto es una API REST desarrollada con Laravel 12 que sirve como backend seguro para aplicaciones que requieren autenticación robusta y sistema de permisos granulares.

### Decisiones Arquitectónicas Clave

#### 1. Framework y Versión
- **Laravel 12:** Versión más reciente con mejoras en performance y security
- **PHP 8.3:** Aprovecha las últimas características del lenguaje
- **MySQL 8.0:** Base de datos robusta y bien soportada

#### 2. Autenticación Elegida
- **Laravel Sanctum:** Preferido sobre JWT por simplicidad y integración nativa
- **Tokens SPA:** Para aplicaciones de página única
- **API Tokens:** Para integración con aplicaciones externas

#### 3. Sistema de Permisos
- **Spatie Laravel Permission:** Paquete maduro y bien mantenido
- **Roles jerárquicos:** admin > manager > user
- **Permisos granulares:** Por recurso y acción

#### 4. Containerización
- **Docker Compose:** Para desarrollo local consistente
- **Servicios separados:** PHP-FPM, Nginx, MySQL, Redis, MailHog
- **Volúmenes persistentes:** Para datos de desarrollo

### Patrones y Convenciones

#### Estructura de Respuestas API
```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": {},
  "meta": {
    "pagination": {},
    "filters_applied": {}
  }
}
```

#### Códigos de Estado HTTP
- 200: OK (GET, PUT exitosos)
- 201: Created (POST exitoso)
- 204: No Content (DELETE exitoso)
- 400: Bad Request (validación fallida)
- 401: Unauthorized (no autenticado)
- 403: Forbidden (no autorizado)
- 404: Not Found (recurso no existe)
- 422: Unprocessable Entity (validación específica)
- 500: Internal Server Error

#### Naming Conventions
- **Rutas API:** kebab-case (`/test-records`)
- **Campos JSON:** snake_case (`created_at`)
- **Clases PHP:** PascalCase (`TestRecord`)
- **Métodos:** camelCase (`getUserPermissions`)

## 🏗️ Arquitectura del Sistema

### Capas de la Aplicación
1. **Routes:** Definición de endpoints y middleware
2. **Controllers:** Lógica de presentación y validación inicial  
3. **Form Requests:** Validación robusta de inputs
4. **Services:** Lógica de negocio compleja
5. **Models:** Representación de datos y relaciones
6. **Resources:** Transformación de respuestas JSON

### Base de Datos - Relaciones Clave
```
users (1) -----> (n) test_records
users (n) <----> (n) roles (via model_has_roles)
roles (n) <----> (n) permissions (via role_has_permissions)
```

### Middleware Stack
1. **CORS:** Configurado para desarrollo y producción
2. **Throttle:** Rate limiting por rutas
3. **Auth:** Verificación de token válido
4. **Permission:** Verificación de permisos específicos

## 🔒 Consideraciones de Seguridad

### Vulnerabilidades Mitigadas
- **SQL Injection:** Eloquent ORM + Prepared statements
- **XSS:** Sanitización automática de inputs
- **CSRF:** No aplica para API REST stateless
- **Rate Limiting:** Protección contra ataques de fuerza bruta
- **Input Validation:** Form Requests exhaustivos

### Tokens y Sesiones
- **Expiración:** Tokens con TTL configurable
- **Revocación:** Capacidad de invalidar tokens
- **Scopes:** Diferentes niveles de acceso por token

## 🧪 Estrategia de Testing

### Test Database
- Base de datos separada (`laravel_api_test`)
- Transacciones para aislamiento entre tests
- Seeders específicos para testing

### Factories y Seeders
```php
// UserFactory con roles
User::factory()->withRole('admin')->create()
User::factory()->withPermissions(['users.read'])->create()

// TestRecordFactory con relaciones
TestRecord::factory()->for(User::factory())->create()
```

### Assertions Personalizados
- `assertApiResponse()`: Verifica estructura de respuesta
- `assertPermissionDenied()`: Verifica autorización
- `assertValidationErrors()`: Verifica validación

## 🐳 Docker Configuration

### Servicios y Puertos
- **Nginx:** 7001 (HTTP)
- **MySQL:** 3316 (Database)
- **Redis:** 6379 (Cache)
- **MailHog:** 8025 (Web UI), 1025 (SMTP)

### Volúmenes Importantes
- `./src:/var/www/html` (Código fuente)
- `mysql_data` (Persistencia BD)
- Logs de Nginx accesibles

### Variables de Entorno
```env
APP_ENV=local
DB_CONNECTION=mysql
DB_HOST=db
DB_DATABASE=laravel_api
CACHE_DRIVER=redis
REDIS_HOST=redis
MAIL_HOST=mailhog
```

## 🚨 Problemas Conocidos y Soluciones

### Performance
- **N+1 Queries:** Always use eager loading
- **Cache Estratégico:** Redis para queries frecuentes
- **Database Indexing:** Índices en campos de búsqueda

### Testing
- **Database Refresh:** Use RefreshDatabase trait
- **External APIs:** Mock servicios externos
- **File Uploads:** Use fake storage para tests

## 📋 Checklist de Calidad

### Antes de Deploy
- [ ] Todos los tests pasan
- [ ] Migraciones ejecutadas
- [ ] Seeders de producción listos  
- [ ] Variables de entorno configuradas
- [ ] SSL/HTTPS configurado
- [ ] Logs configurados correctamente
- [ ] Backups automatizados configurados

### Code Review
- [ ] Form Requests para validación
- [ ] Policies para autorización
- [ ] Resources para serialización
- [ ] Tests para nuevos endpoints
- [ ] Documentación actualizada