# RESTful API Guidelines

**Guía de Estilo para APIs RESTful v1.0**  
**API Factory**

---

## Metadatos del Documento

| Campo | Valor |
|-------|-------|
| **Versión** | 1.0 |
| **Fecha de Publicación** | Octubre 2025 |
| **Estado** | Activo |
| **Organización** | API Factory |
| **Repositorio** | github.com/apifactory-org/restful-api-guidelines |
| **Licencia** | MIT |
| **Contacto** | api-governance@apifactory.org |

---

## Descripción Ejecutiva

Esta guía establece estándares y mejores prácticas para el diseño, desarrollo e implementación de APIs RESTful. Proporciona convenciones, patrones arquitectónicos y guías operacionales para asegurar consistencia, escalabilidad e interoperabilidad en todos los servicios.

Este documento es mantenido como un proyecto de código abierto y puede ser adoptado, modificado y referenciado libremente.

---

## Índice

1. [Principios Fundamentales](#principios-fundamentales)
2. [Convenciones de Naming](#convenciones-de-naming)
3. [Estructura de Recursos](#estructura-de-recursos)
4. [Métodos HTTP Estándar](#métodos-http-estándar)
5. [Tags y OperationId](#tags-y-operationid)
6. [Respuestas y Paginación](#respuestas-y-paginación)
7. [Filtrado, Búsqueda y Ordenamiento](#filtrado-búsqueda-y-ordenamiento)
8. [Operaciones de Larga Duración](#operaciones-de-larga-duración)
9. [Operaciones Bulk](#operaciones-bulk)
10. [Versionado](#versionado)
11. [Códigos de Estado HTTP](#códigos-de-estado-http)
12. [Headers HTTP](#headers-http)
13. [Caching](#caching)
14. [Content Negotiation](#content-negotiation)
15. [Rate Limiting](#rate-limiting)
16. [Autenticación y Seguridad](#autenticación-y-seguridad)
17. [Campos Computados](#campos-computados)
18. [Idempotencia](#idempotencia)
19. [Validación Optimista (ETag)](#validación-optimista-etag)
20. [Manejo de Errores](#manejo-de-errores)
21. [Webhooks](#webhooks)
22. [Eventos en Tiempo Real (SSE)](#eventos-en-tiempo-real-sse)
23. [Partial Responses](#partial-responses)
24. [Deprecation](#deprecation)
25. [Documentación OpenAPI](#documentación-openapi)
26. [Checklist de Diseño](#checklist-de-diseño)

---

## Principios Fundamentales

Las APIs de API Factory deben seguir estos principios:

### 1. Resource-Oriented Design

Las APIs deben estar centradas en **recursos**, no en acciones. Los recursos son sustantivos que representan entidades del sistema.

**✅ Correcto:** `/workspaces`, `/files`, `/bundles`  
**❌ Incorrecto:** `/getWorkspaces`, `/deleteFile`, `/validateBundle`

### 2. Sin RPC (Remote Procedure Call)

No usar estilos RPC que expongan acciones como rutas. Las acciones deben expresarse mediante verbos HTTP.

**✅ Correcto:**
```
POST /workspaces/{id}:validate
GET /workspaces/{id}
```

**❌ Incorrecto:**
```
POST /workspaces/validate
GET /getWorkspace
```

### 3. REST Puro

Usar HTTP como aplicación, no como transporte. Los métodos HTTP deben corresponder con operaciones CRUD:

- `GET` → Leer
- `POST` → Crear
- `PATCH` → Actualizar parcialmente
- `DELETE` → Eliminar
- `HEAD` → Obtener metadatos

### 4. Recursos Primero

Los recursos son la unidad fundamental. Sub-recursos deben ser excepcionales y solo cuando tengan identidad propia.

```
# Sub-recurso válido (tiene identidad en contexto del padre)
GET /workspaces/{workspaceId}/files/{fileId}

# Pseudo-recurso de acción (usa colon)
POST /workspaces/{workspaceId}/bundles:validate
```

---

## Convenciones de Naming

### URLs

- **Minúsculas:** Todas las URLs deben estar en minúsculas
- **Plural:** Los recursos deben estar en plural
- **Guiones:** Usar guiones para separar palabras, nunca guiones bajos

**✅ Correcto:**
```
/v1/workspaces
/v1/workspace-templates
/v1/async-api-specs
```

**❌ Incorrecto:**
```
/v1/Workspaces          # Mayúscula
/v1/workspace           # Singular
/v1/workspace_templates # Guión bajo
/v1/workspaceTemplates  # camelCase
```

### Identificadores de Recursos

- **Slug format:** `[a-z0-9-]+` para IDs amigables
- **UUID:** Para identificadores internos/ocultos
- **Numéricos:** Solo cuando sea absolutamente necesario

**✅ Correcto:**
```
/workspaces/user-service
/workspaces/my-api-spec
```

**❌ Incorrecto:**
```
/workspaces/user_service
/workspaces/UserService
/workspaces/123
```

### Campos en JSON

- **camelCase:** Todos los nombres de campos deben estar en camelCase
- **Booleanos:** Prefijo `is` o `has` (ej: `isActive`, `hasErrors`)
- **Timestamps:** `createdAt`, `updatedAt`, `expiresAt`
- **Referencias:** Sufijo `Id` (ej: `workspaceId`, `userId`)

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  "isActive": true,
  "hasErrors": false,
  "createdAt": "2025-10-01T10:30:00Z",
  "updatedAt": "2025-10-01T10:30:00Z",
  "ownerId": "user-123"
}
```

---

## Estructura de Recursos

### Jerarquía de Recursos

Máximo 2-3 niveles de nesting. Evitar estructuras muy profundas.

**✅ Correcto:**
```
/workspaces/{workspaceId}/files/{fileId}
/users/{userId}/projects/{projectId}
```

**❌ Incorrecto:**
```
/users/{userId}/projects/{projectId}/tasks/{taskId}/subtasks/{subtaskId}/comments/{commentId}
```

### IDs Globalmente Únicos vs Contextuales

- **Globalmente únicos:** No necesitan el padre en la URL
- **Contextuales:** Necesitan el padre para identificarse

```
# Contexual: El archivo solo existe dentro del workspace
GET /workspaces/{workspaceId}/files/{fileId}

# Global: El usuario existe independientemente
GET /users/{userId}
```

### Estructura de Request/Response

#### Create (POST)

```http
POST /v1/workspaces
Content-Type: application/json

{
  "name": "User Service API",
  "slug": "user-service",
  "metadata": {
    "typeId": "asyncapi",
    "specVersion": "2.0.0",
    "entry": "api.yaml"
  }
}
```

Respuesta:
```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  "createdAt": "2025-10-01T10:30:00Z",
  "updatedAt": "2025-10-01T10:30:00Z",
  "status": "active"
}
```

#### Read (GET)

```http
GET /v1/workspaces/{workspaceId}
Accept: application/json
```

#### Update (PATCH)

Solo campos a actualizar:

```http
PATCH /v1/workspaces/{workspaceId}
Content-Type: application/json

{
  "name": "Nuevo nombre",
  "metadata": {
    "description": "Nueva descripción"
  }
}
```

#### Delete (DELETE)

```http
DELETE /v1/workspaces/{workspaceId}
```

---

## Tags y OperationId

### Tags en OpenAPI

Los tags organizan endpoints en categorías lógicas. Cada endpoint debe tener exactamente un tag.

```yaml
tags:
  - name: Workspaces
    description: Operaciones sobre workspaces
  - name: Files
    description: Operaciones sobre archivos
  - name: Bundles
    description: Operaciones sobre bundles resueltos

paths:
  /workspaces:
    get:
      tags: [Workspaces]  # Un solo tag
      summary: Listar workspaces
    post:
      tags: [Workspaces]
      summary: Crear workspace

  /workspaces/{id}/files:
    get:
      tags: [Files]
      summary: Listar archivos
```

**Reglas para Tags:**
- ✅ Usar PascalCase singular o plural consistente
- ✅ Máximo 20 tags por API
- ✅ Corresponden a los nombres de recursos o entidades
- ❌ Un tag por endpoint (no múltiples)

### OperationId

Identificador único para cada operación. Debe ser:
- Único en toda la API
- Descriptivo
- Formato: `{resource}#{action}`

```yaml
paths:
  /workspaces:
    get:
      operationId: workspaces#list
      summary: Listar workspaces
    post:
      operationId: workspaces#create
      summary: Crear workspace

  /workspaces/{id}:
    get:
      operationId: workspaces#getById
      summary: Obtener workspace por ID
    patch:
      operationId: workspaces#update
      summary: Actualizar workspace
    delete:
      operationId: workspaces#delete
      summary: Eliminar workspace

  /workspaces/{id}:validate:
    post:
      operationId: workspaces#validate
      summary: Validar workspace
```

**Convención de nombres:**
- `{resource}#list` → GET /recursos
- `{resource}#create` → POST /recursos
- `{resource}#getById` → GET /recursos/{id}
- `{resource}#update` → PATCH /recursos/{id}
- `{resource}#replace` → PUT /recursos/{id}
- `{resource}#delete` → DELETE /recursos/{id}
- `{resource}#{action}` → POST /recursos/{id}:action

---

## Métodos HTTP Estándar

| Método | Recurso | Propósito | Body | Idempotente | Response |
|--------|---------|----------|------|-------------|----------|
| **GET** | `/resources` | Listar | No | Sí | 200 + Array |
| **GET** | `/resources/{id}` | Obtener | No | Sí | 200 + Recurso |
| **HEAD** | `/resources/{id}` | Metadatos | No | Sí | 200 + Headers |
| **POST** | `/resources` | Crear | Sí | No | 201 + Recurso |
| **POST** | `/resources/{id}:action` | Acción | Opcional | Depende | 200 + Resultado |
| **PUT** | `/resources/{id}` | Reemplazar completo | Sí | Sí | 200 + Recurso |
| **PATCH** | `/resources/{id}` | Actualizar parcial | Sí | Sí | 200 + Recurso |
| **DELETE** | `/resources/{id}` | Eliminar | No | Sí | 204 |
| **OPTIONS** | `/resources/{id}` | Métodos permitidos | No | Sí | 200 + Allow |

### PUT vs PATCH

**PUT:** Reemplaza el recurso completo

```http
PUT /v1/workspaces/user-service
Content-Type: application/json

{
  "name": "New Name",
  "metadata": {
    "description": "New description"
  }
}
```

Campos omitidos se eliminan o resetean. Operación idempotente.

**PATCH:** Actualiza solo campos proporcionados

```http
PATCH /v1/workspaces/user-service
Content-Type: application/json

{
  "name": "New Name"
}
```

Otros campos permanecen sin cambios. Operación idempotente.

**Recomendación:** Preferir PATCH en la mayoría de casos.

### DELETE vs Soft Delete

**Hard Delete (permanente):**
```http
DELETE /v1/workspaces/user-service
→ 204 No Content (recurso eliminado permanentemente)
```

**Soft Delete (recomendado):**
```http
PATCH /v1/workspaces/user-service
{
  "status": "archived"
}
→ 200 OK (recurso archivado pero recuperable)
```

**Política:** Usar soft delete para datos críticos. Hard delete solo para admin.

---

## Operaciones de Larga Duración

Para operaciones que tardan más de 30 segundos, usar **Long-Running Operations**.

### Patrón de Long-Running Operation

```http
POST /v1/workspaces/{id}:validate
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json

{
  "name": "operations/validate-abc123",
  "done": false,
  "metadata": {
    "createTime": "2025-10-01T10:30:00Z",
    "updateTime": "2025-10-01T10:30:05Z"
  }
}
```

### Polling para Obtener Resultado

```http
GET /v1/operations/validate-abc123

HTTP/1.1 200 OK

{
  "name": "operations/validate-abc123",
  "done": true,
  "result": {
    "valid": true,
    "errors": [],
    "warnings": []
  }
}
```

**Respuesta mientras está en progreso:**
```json
{
  "name": "operations/validate-abc123",
  "done": false,
  "metadata": {
    "progress": 45
  }
}
```

**Estructura de Operation:**
```json
{
  "name": "operations/{operation-id}",
  "done": false,
  "result": {},        // Cuando done=true
  "error": {},         // Si hubo error
  "metadata": {}       // Progreso, timestamps, etc
}
```

---

## Operaciones Bulk

Para operaciones sobre múltiples recursos.

### Bulk Create

```http
POST /v1/workspaces:batchCreate
Content-Type: application/json

{
  "requests": [
    {
      "name": "Workspace 1",
      "slug": "workspace-1"
    },
    {
      "name": "Workspace 2",
      "slug": "workspace-2"
    }
  ]
}

HTTP/1.1 200 OK

{
  "responses": [
    {
      "workspaceId": "workspace-1",
      "status": 201
    },
    {
      "workspaceId": "workspace-2",
      "status": 201
    }
  ]
}
```

### Bulk Delete

```http
POST /v1/workspaces:batchDelete
Content-Type: application/json

{
  "ids": ["workspace-1", "workspace-2"]
}

HTTP/1.1 200 OK

{
  "results": [
    { "id": "workspace-1", "status": 204 },
    { "id": "workspace-2", "status": 204 }
  ]
}
```

**Endpoint estándar:** `POST /resources:batch{Action}`

---

## Caching

### Cache-Control Headers

```http
# Sin cache (sensible a cambios)
Cache-Control: no-cache, no-store, must-revalidate

# Cache 5 minutos
Cache-Control: public, max-age=300

# Cache solo para cliente (no intermediarios)
Cache-Control: private, max-age=3600

# Cache inmutable (no vuelve a validar)
Cache-Control: public, max-age=31536000, immutable
```

### ETag y Validación

```http
# Response inicial
GET /v1/workspaces/user-service
→ ETag: "abc123"

# Cliente envía If-None-Match
GET /v1/workspaces/user-service
If-None-Match: "abc123"

# Si no cambió
→ 304 Not Modified (sin body, sin transferencia de datos)

# Si cambió
→ 200 OK
   ETag: "def456"
```

### Estrategias de Caching

| Recurso | Estrategia |
|---------|-----------|
| Lista (GET /resources) | `public, max-age=60` (1 min) |
| Detalle (GET /resources/{id}) | `public, max-age=300` (5 min) |
| Estático | `public, max-age=31536000` (1 año) |
| Datos sensibles | `private, no-cache` |

---

## Content Negotiation

### Accept Header

```http
# Solicitar JSON
GET /v1/workspaces
Accept: application/json

# Solicitar YAML
GET /v1/workspaces
Accept: application/yaml

# Solicitar XML (si soportado)
GET /v1/workspaces
Accept: application/xml

# Multiple (prioridad por orden)
Accept: application/json, application/yaml;q=0.9
```

### Respuesta

```http
GET /v1/workspaces
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{ "items": [...] }
```

**Soportado obligatoriamente:** `application/json`  
**Recomendado:** `application/yaml`

---

## Partial Responses

Permitir al cliente seleccionar qué campos retornar.

### Query Parameter `fields`

```http
GET /v1/workspaces?fields=workspaceId,name,status

{
  "workspaceId": "user-service",
  "name": "User Service API",
  "status": "active"
}
```

Sin `createdAt`, `updatedAt`, `metadata`, etc.

### Nested Field Selection

```http
GET /v1/workspaces?fields=name,metadata(description,tags)

{
  "name": "User Service API",
  "metadata": {
    "description": "...",
    "tags": [...]
  }
}
```

---

## Webhooks

Para notificaciones push sobre eventos.

### Registro de Webhook

```http
POST /v1/webhooks
Content-Type: application/json

{
  "url": "https://tu-dominio.com/webhooks/apifactory",
  "events": [
    "workspace.created",
    "workspace.updated",
    "file.uploaded"
  ],
  "headers": {
    "X-Custom-Header": "value"
  }
}

HTTP/1.1 201 Created

{
  "webhookId": "hook-abc123",
  "status": "active",
  "createdAt": "2025-10-01T10:30:00Z"
}
```

### Envío de Webhook

```http
POST https://tu-dominio.com/webhooks/apifactory
Content-Type: application/json
X-Webhook-Signature: sha256=abc123...
X-Webhook-Id: evt-def456
X-Webhook-Timestamp: 1633081800

{
  "event": "workspace.created",
  "data": {
    "workspaceId": "user-service",
    "name": "User Service API"
  }
}
```

**Headers obligatorios:**
- `X-Webhook-Signature` (HMAC-SHA256 para validar)
- `X-Webhook-Id` (ID único del evento)
- `X-Webhook-Timestamp` (Unix timestamp)

---

## Deprecation

### Deprecar Campos

En OpenAPI, marcar campos obsoletos:

```yaml
properties:
  legacyField:
    type: string
    deprecated: true
    description: |
      **Deprecado desde v1.5**
      Use `newField` en su lugar.
  
  newField:
    type: string
    description: Reemplazo de legacyField
```

### Deprecar Endpoints

```yaml
paths:
  /v1/workspaces-old:
    deprecated: true
    get:
      summary: Obtener workspaces
      description: |
        ⚠️ **DEPRECADO** - Use `GET /v1/workspaces` en su lugar.
        Será removido el 2026-01-01.
```

### Política de Deprecation

1. Anunciar 6 meses antes de deprecación
2. Mantener 12 meses después de deprecación
3. Incluir en headers: `Deprecation: true`
4. Proporcionar alternativa clara

```http
HTTP/1.1 200 OK
Deprecation: true
Sunset: Fri, 01 Jan 2026 00:00:00 GMT
Link: </v2/workspaces>; rel="successor-version"
```



### Paginación por Offset

```http
GET /v1/workspaces?limit=20&offset=0

{
  "items": [
    { "workspaceId": "api-1", "name": "API 1" },
    { "workspaceId": "api-2", "name": "API 2" }
  ],
  "total": 100,
  "limit": 20,
  "offset": 0,
  "hasMore": true
}
```

**Parámetros estándar:**
- `limit` (default: 20, máx: 100)
- `offset` (default: 0)
- Respuesta incluye siempre: `total`, `limit`, `offset`, `hasMore`

### Paginación por Cursor

Para casos de alto rendimiento, usar cursor-based (opcional):

```http
GET /v1/workspaces?limit=20&cursor=abc123xyz

{
  "items": [...],
  "nextCursor": "def456xyz",
  "hasMore": true
}
```

### Valores Vacíos

Siempre retornar array vacío, nunca null:

```json
{
  "items": [],
  "total": 0,
  "hasMore": false
}
```

---

## Filtrado, Búsqueda y Ordenamiento

### Filtrado

Usar query parameters para filtrar:

```http
GET /v1/workspaces?status=active&typeId=asyncapi&visibility=public
```

Solo filtrar por campos que sean indexables en BD.

### Búsqueda

Parámetro `q` para búsqueda full-text:

```http
GET /v1/workspaces?q=user+service
```

Se busca en: nombre, descripción, tags.

### Ordenamiento

Parámetro `sort` con formato `[campo]` o `-[campo]` (- para descendente):

```http
GET /v1/workspaces?sort=-updatedAt,name

# Equivalente a: ORDER BY updatedAt DESC, name ASC
```

Campos ordenables: `createdAt`, `updatedAt`, `name`, `status`

---

## Versionado

### URL Versionado

La versión debe estar en la URL, no en headers:

```
https://api.apifactory.org/v1/workspaces
https://api.apifactory.org/v2/workspaces
```

**Estructura:**
```
https://{domain}/{version}/{recursos}
```

**Versiones:**
- `v1` (actual)
- `v2` (futuro)

**Nunca deprecar sin avisar:** Mínimo 12 meses de aviso antes de remover versión anterior.

---

## Códigos de Estado HTTP

### 2xx - Éxito

| Código | Uso |
|--------|-----|
| **200 OK** | Operación exitosa con contenido |
| **201 Created** | Recurso creado (incluir header `Location`) |
| **204 No Content** | Operación exitosa sin contenido (DELETE) |

### 3xx - Redirección

| Código | Uso |
|--------|-----|
| **304 Not Modified** | Cache válido (ETag coincide) |

### 4xx - Error del Cliente

| Código | Uso | Ejemplo |
|--------|-----|---------|
| **400 Bad Request** | Parámetros mal formados | `?limit=abc` |
| **401 Unauthorized** | No autenticado | Token expirado |
| **403 Forbidden** | Autenticado pero sin permisos | Acceso denegado |
| **404 Not Found** | Recurso no existe | `/workspaces/no-existe` |
| **409 Conflict** | Conflicto de estado | Recurso duplicado |
| **412 Precondition Failed** | ETag no coincide | If-Match falló |
| **422 Unprocessable Entity** | Validación fallida | Datos inválidos |
| **429 Too Many Requests** | Rate limit excedido | Reintentar después |

### 5xx - Error del Servidor

| Código | Uso |
|--------|-----|
| **500 Internal Server Error** | Error no controlado |
| **503 Service Unavailable** | Servicio temporalmente no disponible |

**Regla de oro:** Retornar el código más específico posible.

---

## Headers HTTP

### Request Headers

| Header | Descripción | Obligatorio |
|--------|-------------|-----------|
| `Authorization` | Token Bearer JWT | Sí (excepto public) |
| `Content-Type` | `application/json` | Para POST/PATCH/PUT |
| `Accept` | `application/json` | No (default) |
| `If-Match` | ETag para validación optimista | No |
| `Idempotency-Key` | UUID para idempotencia | No (recomendado) |
| `X-Commit-Message` | Mensaje de commit para cambios | No |

### Response Headers

| Header | Descripción |
|--------|-------------|
| `Location` | URL del recurso creado (201) |
| `ETag` | Identificador de versión del recurso |
| `X-RateLimit-Limit` | Límite de requests por ventana |
| `X-RateLimit-Remaining` | Requests restantes |
| `X-RateLimit-Reset` | Timestamp de reset |
| `Retry-After` | Segundos a esperar (429) |
| `Cache-Control` | Directivas de cache |
| `Content-Type` | `application/json` |

**Ejemplo de respuesta 201:**
```http
HTTP/1.1 201 Created
Location: /v1/workspaces/user-service
ETag: "abc123def456"
X-RateLimit-Remaining: 99
```

---

## Rate Limiting

### Límites Estándar

- **Usuarios autenticados:** 1000 requests/hora
- **Usuarios públicos:** 100 requests/hora
- **Burst:** 50 requests/minuto

### Headers de Rate Limit

En toda respuesta incluir:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1633081800
```

### Respuesta 429

```json
{
  "code": "TOO_MANY_REQUESTS",
  "message": "Has excedido el límite de requests",
  "details": {
    "limit": 100,
    "windowSeconds": 3600,
    "retryAfter": 1800
  }
}
```

---

## Autenticación y Seguridad

### Bearer Token (JWT)

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Scope de Token

Incluir en JWT:
- `user_id`
- `permissions` (list)
- `exp` (expiración)
- `iat` (creación)

### Headers de Seguridad

```http
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

### HTTPS

**Obligatorio** en producción. Redirigir HTTP a HTTPS.

---

## Campos Computados

Campos calculados por el servidor deben marcarse como `readOnly`:

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  
  "createdAt": "2025-10-01T10:30:00Z",  // readOnly
  "updatedAt": "2025-10-01T10:30:00Z",  // readOnly
  "sizeBytes": 1024,                     // readOnly
  "lastValidatedAt": "2025-10-01T11:00:00Z" // readOnly
}
```

**Estos campos:**
- NO acepta en requests
- Siempre se retornan en responses
- Se actualizan automáticamente

---

## Idempotencia

### Idempotency-Key Header

Para operaciones que crean recursos, el cliente puede enviar un UUID único:

```http
POST /v1/workspaces
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "name": "My Workspace",
  "slug": "my-workspace"
}
```

**Comportamiento:**
- Primera request: Crea recurso, retorna 201
- Segunda request con mismo Idempotency-Key: Retorna 200 + recurso existente
- Sin header: Retorna 409 si ya existe

### Operaciones Idempotentes

Siempre idempotentes:
- GET, HEAD
- PUT (reemplazo completo)
- DELETE (ya eliminado retorna 204)

Con precaución:
- POST (incluir Idempotency-Key)
- PATCH (puede no ser idempotente si usa operadores)

---

## Validación Optimista (ETag)

### ETag

Identificador de versión del recurso:

```http
GET /v1/workspaces/user-service

HTTP/1.1 200 OK
ETag: "abc123def456"

{
  "workspaceId": "user-service",
  "name": "User Service API"
}
```

### If-Match

Garantizar que el recurso no cambió antes de actualizar:

```http
PATCH /v1/workspaces/user-service
If-Match: "abc123def456"

{
  "name": "New Name"
}
```

**Respuesta si ETag no coincide:**
```http
HTTP/1.1 412 Precondition Failed

{
  "code": "PRECONDITION_FAILED",
  "message": "El recurso ha sido modificado",
  "details": {
    "expectedETag": "abc123def456",
    "actualETag": "xyz789"
  }
}
```

---

## Manejo de Errores

### Estructura de Error

```json
{
  "code": "VALIDATION_ERROR",
  "message": "El parámetro slug es inválido",
  "details": {
    "field": "slug",
    "reason": "Solo caracteres alfanuméricos y guiones",
    "pattern": "^[a-z0-9-]+$"
  },
  "requestId": "req-abc123def456"
}
```

### Códigos de Error Estándar

```
BAD_REQUEST
UNAUTHORIZED
FORBIDDEN
NOT_FOUND
CONFLICT
PRECONDITION_FAILED
TOO_MANY_REQUESTS
INTERNAL_SERVER_ERROR
SERVICE_UNAVAILABLE
VALIDATION_ERROR
```

### Validación en Request

Retornar 422 con detalles:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Validación fallida",
  "details": {
    "errors": [
      {
        "field": "name",
        "message": "El nombre es requerido"
      },
      {
        "field": "slug",
        "message": "Solo alfanuméricos y guiones"
      }
    ]
  }
}
```

### Incluir Request ID

Para debugging, incluir `requestId` en todos los errores:

```json
{
  "code": "INTERNAL_SERVER_ERROR",
  "message": "Error interno del servidor",
  "details": {},
  "requestId": "req-abc123-def456-xyz789"
}
```

Los logs deben indexarse por `requestId`.

---

## Eventos en Tiempo Real (SSE)

### Server-Sent Events

Para conexiones de larga duración, usar SSE:

```http
POST /v1/workspaces/{workspaceId}/events:subscribe
Content-Type: application/json

{
  "eventTypes": [
    "file.created",
    "file.updated",
    "validation.completed"
  ]
}
```

### Formato de Evento

```
id: evt-abc123
event: file.updated
timestamp: 2025-10-01T10:30:00Z
data: {"path":"src/api.yaml","sha":"abc123"}

id: evt-def456
event: validation.completed
timestamp: 2025-10-01T10:35:00Z
data: {"valid":true,"errors":[],"warnings":[]}
```

### Tipos de Eventos

```
file.created
file.updated
file.deleted
validation.completed
bundle.updated
resource.archived
```

---

## Documentación OpenAPI

### Estructura Mínima

```yaml
openapi: 3.0.3
info:
  title: API Name
  version: 0.1.0
  description: Brief description

servers:
  - url: https://api.apifactory.org/v1
    description: Production

tags:
  - name: Resources
    description: Resource operations

paths:
  /resources:
    get:
      tags: [Resources]
      summary: Listar recursos
      parameters:
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        "200":
          description: Lista de recursos
```

### Validaciones en Schemas

```yaml
schemas:
  Workspace:
    type: object
    properties:
      slug:
        type: string
        pattern: '^[a-z0-9-]+$'
        minLength: 1
        maxLength: 100
      name:
        type: string
        minLength: 1
        maxLength: 255
      isActive:
        type: boolean
```

---

## Checklist de Diseño

Usar esta lista antes de lanzar un endpoint:

- [ ] **URL:** Minúsculas, plural, RESTful (`/v1/resources`)
- [ ] **Métodos:** GET, POST, PATCH, DELETE, PUT (si aplica)
- [ ] **Tags:** Un tag único por endpoint
- [ ] **OperationId:** Único, formato `{resource}#{action}`
- [ ] **Códigos HTTP:** 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500, 503
- [ ] **Headers:** Content-Type, Authorization, ETag, Rate-Limit
- [ ] **Paginación:** limit, offset, total, hasMore (si aplica)
- [ ] **Filtrado:** Query params para filters (`?status=active`)
- [ ] **Búsqueda:** Parámetro `q` si es necesario
- [ ] **Ordenamiento:** Parámetro `sort` si es necesario
- [ ] **Errores:** Estructura consistente con code, message, details
- [ ] **RequestId:** Incluir en todos los errores
- [ ] **Validación:** Usar 422 para validaciones fallidas
- [ ] **ETag:** If-Match para operaciones de escritura
- [ ] **Idempotency:** Idempotency-Key para POSTs
- [ ] **Soft Delete:** Status=archived en lugar de DELETE físico
- [ ] **Campos Computed:** Marcar como readOnly
- [ ] **Long-running ops:** Si tarda >30s, usar pattern de operaciones
- [ ] **Bulk operations:** `POST /resources:batch{Action}` si aplica
- [ ] **Caching:** Cache-Control headers apropiados
- [ ] **Content-Type:** Soportar application/json y application/yaml
- [ ] **Partial responses:** Parámetro `fields` si aplica
- [ ] **Webhooks:** Si requiere notificaciones push
- [ ] **Deprecation:** Marcar si endpoint es legacy
- [ ] **Documentación:** OpenAPI completo con ejemplos
- [ ] **Rate Limiting:** Headers X-RateLimit-* presentes
- [ ] **CORS:** Headers de seguridad apropiados
- [ ] **Tests:** Casos de éxito y error documentados

---

## Referencias

- [Google API Design (AIPs)](https://google.aip.dev)
- [REST API Best Practices](https://restfulapi.net)
- [HTTP Status Codes](https://httpwg.org/specs/rfc9110.html)
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.3)

---

## Cómo Adoptar este Documento

### Para Proyectos Internos (API Factory)

1. Incluir referencia a esta especificación en los README.md de proyectos
2. Usar checklist de diseño antes de publicar endpoints
3. Validar contra OpenAPI schema

### Para Proyectos Externos

Este documento se publica bajo licencia MIT. Para adoptarlo:

```bash
# Clonar el repositorio
git clone https://github.com/apifactory-org/restful-api-guidelines.git

# O hacer fork en tu organización
# Ir a: https://github.com/apifactory-org/restful-api-guidelines/fork
```

### Referenciación

En tu proyecto:

```markdown
# Estándares de API

Este proyecto sigue las [RESTful API Guidelines](https://github.com/apifactory-org/restful-api-guidelines) de API Factory v1.0.
```

En comentarios de código:

```java
// Implementación conforme a: RESTful-Guidelines@v1.0#códigos-de-estado-http
// Ver: https://github.com/apifactory-org/restful-api-guidelines
```

---

## Contribuciones

Este documento es mantenido por la comunidad. Para contribuir:

1. **Fork** el repositorio
2. Crear rama: `git checkout -b feature/mejora-descripción`
3. Hacer cambios
4. Enviar **Pull Request** con:
   - Descripción clara de cambios
   - Sección afectada
   - Justificación

### Proceso de Review

- Mínimo 2 approvals del equipo de governance
- Discussion en issues antes de cambios mayores
- Versión se incrementa (MAJOR.MINOR.PATCH)

---

## Estructura del Repositorio

```
restful-api-guidelines/
├── README.md                 # Getting started
├── GUIDELINES.md             # Este documento (v1.0)
├── CHANGELOG.md              # Historial de versiones
├── examples/
│   ├── openapi.example.yaml  # Ejemplo de OpenAPI completo
│   ├── error-response.json   # Estructura de errores
│   └── pagination.json       # Ejemplo de paginación
├── templates/
│   ├── openapi-template.yaml # Template para nuevos proyectos
│   └── error-codes.yaml      # Códigos de error estándar
├── tests/
│   └── guidelines.test.js    # Tests de validación
└── LICENSE                   # MIT License
```

---

## Validación y Testing

### Linting de OpenAPI

```bash
npm install -g @openapitools/openapi-generator-cli

# Validar contra especificación
swagger-cli validate api.yaml
```

### Testing de Endpoints

```bash
# Usar los tests proporcionados
npm test

# O ejecutar contra tu API
npm run test -- --api-url https://tu-api.com
```

| Versión | Fecha | Cambios |
|---------|-------|---------|
| 1.0 | 2025-10-26 | Versión inicial |

---

**Última actualización:** Octubre 2025  
**Propietario:** API Factory  
**Contacto:** api-governance@apifactory.org