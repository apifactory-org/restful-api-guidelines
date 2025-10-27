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
7. [Estrategias de Paginación](#estrategias-de-paginación)
8. [Filtrado, Búsqueda y Ordenamiento](#filtrado-búsqueda-y-ordenamiento)
9. [Consultas de Rango de Tiempo](#consultas-de-rango-de-tiempo)
10. [Operaciones de Larga Duración](#operaciones-de-larga-duración)
11. [Operaciones Bulk](#operaciones-bulk)
12. [Resource Expansion](#resource-expansion)
13. [Partial Responses](#partial-responses)
14. [Respuestas Polimórficas](#respuestas-polimórficas)
15. [HATEOAS](#hateoas)
16. [Versionado](#versionado)
17. [Estrategias de Versionado](#estrategias-de-versionado)
18. [Compatibilidad Hacia Atrás](#compatibilidad-hacia-atrás)
19. [Deprecation](#deprecation)
20. [Códigos de Estado HTTP](#códigos-de-estado-http)
21. [RFC 7807 - Problem Details](#rfc-7807---problem-details)
22. [Manejo de Errores](#manejo-de-errores)
23. [Headers HTTP](#headers-http)
24. [Conditional Requests](#conditional-requests)
25. [Caching](#caching)
26. [Content Negotiation](#content-negotiation)
27. [Compression](#compression)
28. [Streaming y Large Files](#streaming-y-large-files)
29. [Range Requests](#range-requests)
30. [Limits de Tamaño](#limits-de-tamaño)
31. [Query Complexity](#query-complexity)
32. [Rate Limiting](#rate-limiting)
33. [Estrategias de Rate Limiting](#estrategias-de-rate-limiting)
34. [Retry Strategies](#retry-strategies)
35. [Timeout Strategies](#timeout-strategies)
36. [Circuit Breaker](#circuit-breaker)
37. [Health Checks](#health-checks)
38. [Autenticación y Seguridad](#autenticación-y-seguridad)
39. [CORS](#cors)
40. [CSRF Protection](#csrf-protection)
41. [Request Signing](#request-signing)
42. [Campos Computados](#campos-computados)
43. [Idempotencia](#idempotencia)
44. [Validación Optimista (ETag)](#validación-optimista-etag)
45. [Webhooks](#webhooks)
46. [Eventos en Tiempo Real (SSE)](#eventos-en-tiempo-real-sse)
47. [Observabilidad](#observabilidad)
48. [Audit Logging](#audit-logging)
49. [Structured Logging](#structured-logging)
50. [Metrics y Monitoring](#metrics-y-monitoring)
51. [Distributed Tracing](#distributed-tracing)
52. [Data Retention](#data-retention)
53. [Richardson Maturity Model](#richardson-maturity-model)
54. [Documentación OpenAPI](#documentación-openapi)
55. [Checklist de Diseño](#checklist-de-diseño)

---

## Principios Fundamentales

Las APIs de API Factory deben seguir estos principios:

### 1. Resource-Oriented Design

Las APIs deben estar centradas en **recursos**, no en acciones.

**✅ Correcto:** `/workspaces`, `/files`, `/bundles`  
**❌ Incorrecto:** `/getWorkspaces`, `/deleteFile`, `/validateBundle`

### 2. Sin RPC (Remote Procedure Call)

No usar estilos RPC que expongan acciones como rutas.

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

Usar HTTP como aplicación, no como transporte.

### 4. Recursos Primero

Los recursos son la unidad fundamental.

---

## Convenciones de Naming

### URLs

- **Minúsculas:** Todas las URLs deben estar en minúsculas
- **Plural:** Los recursos deben estar en plural
- **Guiones:** Usar guiones para separar palabras

**✅ Correcto:**
```
/v1/workspaces
/v1/workspace-templates
/v1/async-api-specs
```

**❌ Incorrecto:**
```
/v1/Workspaces
/v1/workspace
/v1/workspace_templates
```

### Identificadores de Recursos

- **Slug format:** `[a-z0-9-]+` para IDs amigables
- **UUID:** Para identificadores internos
- **Numéricos:** Solo cuando sea necesario

### Campos en JSON

- **camelCase:** Todos los campos en camelCase
- **Booleanos:** Prefijo `is` o `has`
- **Timestamps:** `createdAt`, `updatedAt`, `expiresAt`
- **Referencias:** Sufijo `Id`

---

## Estructura de Recursos

### Jerarquía de Recursos

Máximo 2-3 niveles de nesting.

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

- **Globalmente únicos:** No necesitan el padre en URL
- **Contextuales:** Necesitan el padre para identificarse

```
# Contextual
GET /workspaces/{workspaceId}/files/{fileId}

# Global
GET /users/{userId}
```

---

## Métodos HTTP Estándar

| Método | Recurso | Propósito | Body | Idempotente | Response |
|--------|---------|----------|------|-------------|----------|
| **GET** | `/resources` | Listar | No | Sí | 200 + Array |
| **GET** | `/resources/{id}` | Obtener | No | Sí | 200 + Recurso |
| **HEAD** | `/resources/{id}` | Metadatos | No | Sí | 200 + Headers |
| **POST** | `/resources` | Crear | Sí | No | 201 + Recurso |
| **POST** | `/resources/{id}:action` | Acción | Opcional | Depende | 200 + Resultado |
| **PUT** | `/resources/{id}` | Reemplazar | Sí | Sí | 200 + Recurso |
| **PATCH** | `/resources/{id}` | Actualizar | Sí | Sí | 200 + Recurso |
| **DELETE** | `/resources/{id}` | Eliminar | No | Sí | 204 |
| **OPTIONS** | `/resources/{id}` | Métodos permitidos | No | Sí | 200 + Allow |

### PUT vs PATCH

**PUT:** Reemplaza el recurso completo. Idempotente.

```http
PUT /v1/workspaces/user-service
{ "name": "New Name", "metadata": {} }
```

**PATCH:** Actualiza solo campos proporcionados. Idempotente.

```http
PATCH /v1/workspaces/user-service
{ "name": "New Name" }
```

**Recomendación:** Preferir PATCH.

---

## Tags y OperationId

### Tags en OpenAPI

```yaml
tags:
  - name: Workspaces
    description: Operaciones sobre workspaces
  - name: Files
    description: Operaciones sobre archivos

paths:
  /workspaces:
    get:
      tags: [Workspaces]
```

Un solo tag por endpoint.

### OperationId

Formato: `{resource}#{action}`

```yaml
operationId: workspaces#list      # GET /workspaces
operationId: workspaces#create    # POST /workspaces
operationId: workspaces#getById   # GET /workspaces/{id}
operationId: workspaces#update    # PATCH /workspaces/{id}
operationId: workspaces#replace   # PUT /workspaces/{id}
operationId: workspaces#delete    # DELETE /workspaces/{id}
operationId: workspaces#validate  # POST /workspaces/{id}:validate
```

---

## Respuestas y Paginación

### Estructura Estándar

```json
{
  "items": [
    { "id": "1", "name": "Item 1" },
    { "id": "2", "name": "Item 2" }
  ],
  "total": 100,
  "limit": 20,
  "offset": 0,
  "hasMore": true
}
```

### Parámetros de Paginación

```http
GET /v1/workspaces?limit=20&offset=0

{
  "items": [...],
  "total": 100,
  "limit": 20,
  "offset": 0,
  "hasMore": true
}
```

- `limit` (default: 20, máx: 100)
- `offset` (default: 0)

### Valores Vacíos

Retornar siempre array vacío, nunca null:

```json
{
  "items": [],
  "total": 0,
  "hasMore": false
}
```

---

## Estrategias de Paginación

### Offset-Based Pagination

Simple pero lento con datasets grandes.

```http
GET /v1/workspaces?limit=20&offset=40
```

**Problema:** Offset requiere contar todos los documentos anteriores.

### Cursor-Based Pagination

Más eficiente para datos grandes.

```http
GET /v1/workspaces?limit=20&cursor=abc123xyz

{
  "items": [...],
  "nextCursor": "def456xyz",
  "prevCursor": "xyz789abc",
  "hasMore": true
}
```

El cursor es opaco para el cliente (puede ser Base64, JWT, etc).

### Keyset Pagination (Search-After)

Para datasets muy grandes con sorting.

```http
GET /v1/workspaces?limit=20&lastId=workspace-20&lastDate=2025-10-01T10:30:00Z

# Retorna 20 workspaces después de workspace-20 en orden de fecha
```

**Recomendación:** Offset para <10k items, Cursor para >10k items.

---

## Filtrado, Búsqueda y Ordenamiento

### Filtrado

```http
GET /v1/workspaces?status=active&typeId=asyncapi&visibility=public
```

### Búsqueda Full-Text

```http
GET /v1/workspaces?q=user+service

# Busca en: nombre, descripción, tags
```

### Ordenamiento

```http
GET /v1/workspaces?sort=-updatedAt,name

# ORDER BY updatedAt DESC, name ASC
```

Campos ordenables: `createdAt`, `updatedAt`, `name`, `status`

---

## Consultas de Rango de Tiempo

### Range Queries

```http
# Workspaces creados entre dos fechas
GET /v1/workspaces?createdAfter=2025-01-01T00:00:00Z&createdBefore=2025-12-31T23:59:59Z

# Última actualización en últimos 7 días
GET /v1/workspaces?updatedAfter=2025-09-24T00:00:00Z
```

### ISO 8601 Timestamps

Todos los timestamps deben usar ISO 8601:

```
2025-10-01T10:30:00Z
2025-10-01T10:30:00.123Z
2025-10-01T10:30:00+02:00
```

Preferencia: `Z` (UTC).

---

## Operaciones de Larga Duración

Para operaciones que tardan más de 30 segundos, usar **Long-Running Operations**.

### Iniciar Operación

```http
POST /v1/workspaces/{id}:validate

HTTP/1.1 200 OK

{
  "name": "operations/validate-abc123",
  "done": false,
  "metadata": {
    "createTime": "2025-10-01T10:30:00Z"
  }
}
```

### Polling

```http
GET /v1/operations/validate-abc123

{
  "name": "operations/validate-abc123",
  "done": true,
  "result": {
    "valid": true,
    "errors": []
  }
}
```

### En Progreso

```json
{
  "name": "operations/validate-abc123",
  "done": false,
  "metadata": {
    "progress": 45,
    "eta": "2025-10-01T10:35:00Z"
  }
}
```

---

## Operaciones Bulk

### Batch Create

```http
POST /v1/workspaces:batchCreate

{
  "requests": [
    { "name": "Workspace 1", "slug": "workspace-1" },
    { "name": "Workspace 2", "slug": "workspace-2" }
  ]
}

HTTP/1.1 200 OK

{
  "responses": [
    { "workspaceId": "workspace-1", "status": 201 },
    { "workspaceId": "workspace-2", "status": 201 }
  ]
}
```

### Batch Delete

```http
POST /v1/workspaces:batchDelete

{
  "ids": ["workspace-1", "workspace-2"]
}

{
  "results": [
    { "id": "workspace-1", "status": 204 },
    { "id": "workspace-2", "status": 204 }
  ]
}
```

Endpoint: `POST /resources:batch{Action}`

---

## Resource Expansion

Incluir relacionados con `?include` para evitar N+1.

### Petición

```http
GET /v1/workspaces/user-service?include=metadata,files
```

### Respuesta

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  "metadata": {
    "description": "...",
    "tags": [...]
  },
  "files": [
    { "path": "api.yaml", "sha": "abc123" },
    { "path": "schemas.yaml", "sha": "def456" }
  ]
}
```

### Nested Expansion

```http
GET /v1/workspaces?include=files(path,sha)

# Solo retorna path y sha de files, no todo
```

---

## Partial Responses

Seleccionar qué campos retornar.

### Query Parameter `fields`

```http
GET /v1/workspaces?fields=workspaceId,name,status

{
  "workspaceId": "user-service",
  "name": "User Service API",
  "status": "active"
}
```

Sin `createdAt`, `updatedAt`, etc.

---

## Respuestas Polimórficas

Un endpoint retorna diferentes tipos de recursos.

```http
GET /v1/search?q=user-service

{
  "items": [
    {
      "type": "workspace",
      "data": {
        "workspaceId": "user-service",
        "name": "User Service API"
      }
    },
    {
      "type": "file",
      "data": {
        "path": "api.yaml",
        "content": "..."
      }
    }
  ]
}
```

Usar `type` discriminator field.

---

## HATEOAS

Hypermedia As The Engine Of Application State. Incluir links para navegabilidad.

### Response con Links

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  "_links": {
    "self": {
      "href": "/v1/workspaces/user-service"
    },
    "files": {
      "href": "/v1/workspaces/user-service/files"
    },
    "update": {
      "href": "/v1/workspaces/user-service",
      "method": "PATCH"
    },
    "delete": {
      "href": "/v1/workspaces/user-service",
      "method": "DELETE"
    }
  }
}
```

O en Link header (RFC 8288):

```http
Link: </v1/workspaces/user-service>; rel="self"
Link: </v1/workspaces/user-service/files>; rel="files"
```

**Nivel 3 de Richardson Maturity Model.**

---

## Versionado

### URL Versionado

```
https://api.apifactory.org/v1/workspaces
https://api.apifactory.org/v2/workspaces
```

Estructura: `https://{domain}/{version}/{recursos}`

---

## Estrategias de Versionado

### 1. URL Versioning (Recomendado)

```
GET /v1/workspaces  →  2024 behavior
GET /v2/workspaces  →  2025 behavior
```

✅ Explícito, fácil de entender  
❌ URLs duplicadas

### 2. Header Versioning

```http
GET /workspaces
API-Version: 1
```

✅ URLs limpias  
❌ Menos discernible

### 3. Media-Type Versioning

```http
GET /workspaces
Accept: application/vnd.apifactory.v1+json
```

✅ Arquitectura REST pura  
❌ Complejo

### 4. Query Parameter Versioning

```
GET /workspaces?version=1
```

❌ Ambiguo, no recomendado

**Recomendación:** URL Versioning.

---

## Compatibilidad Hacia Atrás

### Agregar Campos

Siempre seguro. Clientes viejos los ignoran.

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  "newField": "value"  // Los clientes viejos lo ignoran
}
```

### Remover Campos

**Nunca remover.** Marcar como deprecated y mantener 12 meses.

### Cambiar Estructura

❌ No hacer breaking changes.

**Si es necesario:** Crear nueva versión (v2).

---

## Deprecation

### Deprecar Campos

```yaml
properties:
  legacyField:
    type: string
    deprecated: true
    description: |
      **Deprecado desde v1.5**
      Use `newField` en su lugar.
      Será removido el 2026-10-01.
```

### Deprecar Endpoints

```yaml
paths:
  /v1/workspaces-old:
    deprecated: true
    get:
      summary: Obtener workspaces
      description: |
        ⚠️ **DEPRECADO** - Use `GET /v2/workspaces`
        Será removido el 2026-01-01.
```

### Headers de Deprecation

```http
Deprecation: true
Sunset: Fri, 01 Jan 2026 00:00:00 GMT
Link: </v2/workspaces>; rel="successor-version"
```

**Política:** 6 meses de aviso, 12 meses de mantenimiento, después remover.

---

## Códigos de Estado HTTP

### 2xx - Éxito

| Código | Uso |
|--------|-----|
| **200 OK** | Operación exitosa con contenido |
| **201 Created** | Recurso creado (incluir Location) |
| **204 No Content** | Operación exitosa sin contenido |
| **206 Partial Content** | Range request (bytes X-Y/Total) |

### 3xx - Redirección

| Código | Uso |
|--------|-----|
| **304 Not Modified** | Cache válido (ETag coincide) |
| **307 Temporary Redirect** | Redirect (mantiene método) |
| **308 Permanent Redirect** | Redirect permanente |

### 4xx - Error del Cliente

| Código | Uso |
|--------|-----|
| **400 Bad Request** | Parámetros mal formados |
| **401 Unauthorized** | No autenticado |
| **403 Forbidden** | Sin permisos |
| **404 Not Found** | Recurso no existe |
| **409 Conflict** | Conflicto de estado |
| **412 Precondition Failed** | ETag no coincide |
| **413 Payload Too Large** | Body muy grande |
| **414 URI Too Long** | URL muy larga |
| **415 Unsupported Media Type** | Content-Type no soportado |
| **422 Unprocessable Entity** | Validación fallida |
| **429 Too Many Requests** | Rate limit excedido |

### 5xx - Error del Servidor

| Código | Uso |
|--------|-----|
| **500 Internal Server Error** | Error no controlado |
| **501 Not Implemented** | Funcionalidad no implementada |
| **503 Service Unavailable** | Servicio no disponible |
| **504 Gateway Timeout** | Timeout del gateway |

---

## RFC 7807 - Problem Details

Formato estándar para errores HTTP.

### Estructura

```json
{
  "type": "https://example.com/errors/validation-error",
  "title": "Validación fallida",
  "status": 422,
  "detail": "El parámetro slug es inválido",
  "instance": "/v1/workspaces/user-service",
  "errors": [
    {
      "field": "slug",
      "message": "Solo alfanuméricos y guiones",
      "type": "pattern-mismatch"
    }
  ]
}
```

### Headers

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/problem+json
```

---

## Manejo de Errores

### Estructura Consistente

```json
{
  "type": "https://api.apifactory.org/errors/conflict",
  "title": "Recurso Duplicado",
  "status": 409,
  "detail": "El workspace con slug 'user-service' ya existe",
  "instance": "/v1/workspaces",
  "requestId": "req-abc123-def456",
  "timestamp": "2025-10-01T10:30:00Z"
}
```

### Códigos de Error

```
validation_error
conflict
not_found
unauthorized
forbidden
rate_limit_exceeded
internal_error
service_unavailable
```

---

## Headers HTTP

### Request Headers

| Header | Descripción | Obligatorio |
|--------|-------------|-----------|
| `Authorization` | Bearer JWT | Sí (excepto public) |
| `Content-Type` | application/json | Para POST/PATCH/PUT |
| `Accept` | application/json | No (default) |
| `If-Match` | ETag para validación | No |
| `If-None-Match` | ETag para cache | No |
| `If-Modified-Since` | Date para cache | No |
| `If-Unmodified-Since` | Date para validación | No |
| `Idempotency-Key` | UUID para idempotencia | No (recomendado) |
| `Range` | bytes=0-1023 | Para streaming |
| `Accept-Encoding` | gzip, deflate, br | No (default) |
| `Accept-Language` | en, es | Para i18n |
| `User-Agent` | Client info | Recomendado |
| `X-Request-Id` | Request UUID | No |
| `X-Forwarded-For` | IP original | No (proxies) |

### Response Headers

| Header | Descripción |
|--------|-------------|
| `Location` | URL del recurso creado (201) |
| `ETag` | Identificador de versión |
| `Last-Modified` | Fecha de última modificación |
| `Cache-Control` | Directivas de cache |
| `X-RateLimit-Limit` | Límite de requests |
| `X-RateLimit-Remaining` | Requests restantes |
| `X-RateLimit-Reset` | Timestamp de reset |
| `Retry-After` | Segundos a esperar (429) |
| `Allow` | Métodos permitidos (OPTIONS) |
| `Content-Type` | Tipo de contenido |
| `Content-Length` | Tamaño del body |
| `Content-Encoding` | gzip, deflate, br |
| `Content-Range` | bytes X-Y/Total (206) |
| `Vary` | Headers que afectan cache |
| `Access-Control-*` | CORS headers |
| `Deprecation` | true si está deprecado |
| `Sunset` | Fecha de retiro |
| `Link` | Enlaces a recursos relacionados |
| `X-Request-Id` | Request UUID para tracking |

---

## Conditional Requests

### If-Match (Validación Optimista)

```http
PATCH /v1/workspaces/user-service
If-Match: "abc123def456"

→ 200 OK (si ETag coincide)
→ 412 Precondition Failed (si no coincide)
```

### If-None-Match (Cached)

```http
GET /v1/workspaces/user-service
If-None-Match: "abc123def456"

→ 200 OK (si cambió)
→ 304 Not Modified (si no cambió)
```

### If-Modified-Since (Cached por Fecha)

```http
GET /v1/workspaces/user-service
If-Modified-Since: Mon, 01 Oct 2025 10:30:00 GMT

→ 200 OK (si cambió después)
→ 304 Not Modified (si no cambió)
```

### If-Unmodified-Since (Validación por Fecha)

```http
DELETE /v1/workspaces/user-service
If-Unmodified-Since: Mon, 01 Oct 2025 10:30:00 GMT

→ 204 No Content (si no cambió)
→ 412 Precondition Failed (si cambió)
```

---

## Caching

### Cache-Control Headers

```http
# Sin cache (sensible a cambios)
Cache-Control: no-cache, no-store, must-revalidate

# Cache 5 minutos
Cache-Control: public, max-age=300

# Cache solo para cliente
Cache-Control: private, max-age=3600

# Cache inmutable
Cache-Control: public, max-age=31536000, immutable
```

### Directivas

- `public` - Cualquiera puede cachear
- `private` - Solo cliente cachea
- `no-cache` - Revalidar antes de usar
- `no-store` - No cachear
- `max-age=N` - Válido por N segundos
- `s-maxage=N` - Proxy cachea N segundos
- `must-revalidate` - Usar cache si disponible
- `immutable` - Nunca cambiar

### Estrategias

| Recurso | Estrategia |
|---------|-----------|
| Lista (GET /recursos) | `public, max-age=60` |
| Detalle (GET /{id}) | `public, max-age=300` |
| Estático | `public, max-age=31536000, immutable` |
| Datos sensibles | `private, no-cache` |

---

## Content Negotiation

### Accept Header

```http
GET /v1/workspaces
Accept: application/json

GET /v1/workspaces
Accept: application/yaml

GET /v1/workspaces
Accept: application/json, application/yaml;q=0.9
```

### Respuesta

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{ "items": [...] }
```

**Obligatorio:** `application/json`  
**Recomendado:** `application/yaml`

---

## Compression

### Soportar Compresión

```http
GET /v1/workspaces
Accept-Encoding: gzip, deflate, br

HTTP/1.1 200 OK
Content-Encoding: gzip
Transfer-Encoding: chunked
```

Algoritmos soportados:
- `gzip` (obligatorio)
- `deflate` (recomendado)
- `br` (Brotli, opcional)

### Threshold

Comprimir solo si:
- Tamaño > 1KB
- Content-Type es comprimible (JSON, XML, text)

---

## Streaming y Large Files

### Descarga Streaming

```http
GET /v1/workspaces/user-service/files/api.yaml

HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 1048576
Transfer-Encoding: chunked

[archivo en chunks]
```

### Carga Streaming (Chunked)

```http
PUT /v1/workspaces/user-service/files/api.yaml
Transfer-Encoding: chunked
Content-Type: application/yaml

5\r\n
hello\r\n
5\r\n
world\r\n
0\r\n
\r\n
```

### Upload Resumible

Usar **Range Requests** para uploads resumibles.

---

## Range Requests

Permitir descargas resumibles y streaming de video.

### Petición

```http
GET /v1/files/{fileId}
Range: bytes=0-1023

HTTP/1.1 206 Partial Content
Content-Range: bytes 0-1023/10240
Content-Length: 1024

[bytes 0-1023]
```

### Múltiples Rangos

```http
Range: bytes=0-1023, 2048-3071

HTTP/1.1 206 Partial Content
Content-Type: multipart/byteranges; boundary=abc123

--abc123
Content-Type: application/octet-stream
Content-Range: bytes 0-1023/10240

[bytes 0-1023]
--abc123
Content-Type: application/octet-stream
Content-Range: bytes 2048-3071/10240

[bytes 2048-3071]
--abc123--
```

### Header `Accept-Ranges`

```http
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 10240
```

---

## Limits de Tamaño

### Request Size Limits

```http
# Si body > 100MB
POST /v1/workspaces/user-service/files
Content-Length: 104857600

HTTP/1.1 413 Payload Too Large

{
  "type": "https://api.apifactory.org/errors/payload-too-large",
  "title": "Payload Too Large",
  "status": 413,
  "detail": "El archivo no puede exceder 100MB",
  "maxSize": 104857600,
  "receivedSize": 209715200
}
```

### Default Limits

- **JSON Payload:** 10MB
- **File Upload:** 100MB
- **URL Length:** 8KB
- **Header Size:** 32KB

---

## Query Complexity

### Prevenir DoS

Limitar complejidad de queries para evitar abuso.

### Puntos de Complejidad

```
GET /v1/workspaces?fields=...&include=...

# Cada sub-include suma puntos
# 1 punto = 1 recurso
# Max 100 puntos por query
```

### Validación

```http
GET /v1/workspaces?include=files(content),metadata(tags(history))

# Calcula complejidad: 1 + (1 * 5) + (1 * 10) + 10 = 27 puntos (OK)

GET /v1/workspaces?include=files(content(history(metadata(tags))))

# Calcula: 1 + 50 + 50 + 50 = 151 puntos (ERROR)

HTTP/1.1 422 Unprocessable Entity

{
  "type": "query-complexity-exceeded",
  "complexity": 151,
  "maxAllowed": 100
}
```

---

## Rate Limiting

### Headers Estándar

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1633081800
```

### Límites Estándar

- **Autenticados:** 1000 req/hora
- **Públicos:** 100 req/hora
- **Burst:** 50 req/minuto

### Respuesta 429

```json
{
  "type": "https://api.apifactory.org/errors/too-many-requests",
  "title": "Too Many Requests",
  "status": 429,
  "detail": "Has excedido el límite de requests",
  "retryAfter": 1800
}
```

Con header:
```http
Retry-After: 1800
```

---

## Estrategias de Rate Limiting

### Token Bucket

Permite burst de requests.

```
- Bucket: 100 tokens
- Refill: 10 tokens/segundo
- Burst: Hasta 100 requests inmediatos
```

### Sliding Window

Más preciso pero requiere más memoria.

```
Finestra: 1 hora (3600 segundos)
Límite: 1000 requests por ventana deslizante
```

### Fixed Window

Simple pero permite abuse en bordes.

```
Ventana: 1 hora (0:00-1:00, 1:00-2:00)
Límite: 1000 requests por ventana
```

**Recomendación:** Token Bucket o Sliding Window.

---

## Retry Strategies

### Cuando Reintentar

**Reintentar (idempotente):**
- 408 Request Timeout
- 429 Too Many Requests
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout

**No reintentar:**
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found

### Exponential Backoff with Jitter

```
Intento 1: wait 1 + random(0-1) segundos
Intento 2: wait 2 + random(0-2) segundos
Intento 3: wait 4 + random(0-4) segundos
Intento 4: wait 8 + random(0-8) segundos
```

**Pseudo-código:**

```python
base_wait = 2 ** attempt  # Exponencial
jitter = random(0, base_wait)
wait_time = base_wait + jitter
max_wait = 60  # Cap en 60 segundos
```

### Máximo de Reintentos

- Operaciones idempotentes (GET, PUT, DELETE): 3-5 reintentos
- Operaciones no idempotentes (POST): 0-1 reintento
- Timeout: 3 reintentos

---

## Timeout Strategies

### Server-Side Timeouts

```
GET /v1/workspaces (lista): 10 segundos
GET /v1/workspaces/{id} (detalle): 5 segundos
POST /v1/workspaces (create): 30 segundos
Long-running operation: Configurable (default 300)
```

### Client-Side Timeouts

```
Connection timeout: 5 segundos
Read timeout: 30 segundos
Total timeout: 60 segundos
```

### Respuesta Timeout

```http
HTTP/1.1 504 Gateway Timeout

{
  "type": "gateway-timeout",
  "title": "Request Timeout",
  "status": 504,
  "detail": "La solicitud tardó más de 30 segundos"
}
```

---

## Circuit Breaker

Patrón para manejar fallos en cascada.

### Estados

```
CLOSED (normal)
  ↓ (después de N fallos)
OPEN (rechazar requests)
  ↓ (después de X segundos)
HALF_OPEN (permitir 1 request de prueba)
  ↓ (si falla)
OPEN
  ↓ (si éxito)
CLOSED
```

### Configuración

```
failureThreshold: 5        # Fallos antes de abrir
successThreshold: 2        # Éxitos para cerrar
timeout: 60                # Segundos para half-open
```

### Respuesta Circuit Abierto

```http
HTTP/1.1 503 Service Unavailable

{
  "type": "service-unavailable",
  "title": "Circuit Breaker Open",
  "status": 503,
  "detail": "El servicio está temporalmente no disponible",
  "retryAfter": 60
}
```

---

## Health Checks

### Liveness (¿Está vivo?)

```http
GET /health/live

HTTP/1.1 200 OK

{
  "status": "UP",
  "timestamp": "2025-10-01T10:30:00Z"
}
```

Si servicio está muerto: `503 Service Unavailable`

### Readiness (¿Está listo?)

```http
GET /health/ready

{
  "status": "UP",
  "checks": {
    "database": "UP",
    "cache": "UP",
    "messageQueue": "DEGRADED"
  }
}
```

Puede retornar 503 si no está listo.

### Estructura

```json
{
  "status": "UP|DOWN|DEGRADED",
  "timestamp": "2025-10-01T10:30:00Z",
  "components": {
    "database": {
      "status": "UP",
      "responseTime": 5
    },
    "cache": {
      "status": "UP",
      "responseTime": 1
    }
  },
  "uptime": 3600000
}
```

---

## Autenticación y Seguridad

### Bearer Token (JWT)

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

JWT debe incluir:
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
Content-Security-Policy: default-src 'self'
```

### HTTPS Obligatorio

Producción: HTTPS solo.  
Desarrollo: HTTP permitido.

---

## CORS

### Preflight Request

```http
OPTIONS /v1/workspaces
Origin: https://client.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type

HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://client.example.com
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

### Response Headers

```http
Access-Control-Allow-Origin: https://client.example.com
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

### Configuración

**Desarrollo:** `Access-Control-Allow-Origin: *`

**Producción:** Lista específica de dominios.

---

## CSRF Protection

### SameSite Cookie

```http
Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly
```

Opciones:
- `Strict` - No enviar en requests cross-site
- `Lax` - Enviar solo en GET cross-site
- `None` - Enviar siempre (requiere Secure)

### Token CSRF

```http
POST /v1/workspaces
X-CSRF-Token: abc123def456

# Server valida token antes de procesar
```

---

## Request Signing

### HMAC-SHA256

Para APIs de servidor a servidor.

```http
POST /v1/webhooks
X-Webhook-Signature: sha256=hmacsha256(body, secret)
X-Webhook-Timestamp: 1633081800

# Client valida:
# 1. Timestamp reciente (< 5 minutos)
# 2. Signature válida
```

**Pseudo-código:**

```python
import hmac
import hashlib

def verify_signature(body, signature, secret):
    expected = hmac.new(
        secret.encode(),
        body.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(signature, expected)
```

---

## Campos Computados

Campos calculados por servidor. Marcar como `readOnly`:

```json
{
  "workspaceId": "user-service",
  "name": "User Service API",
  
  "createdAt": "2025-10-01T10:30:00Z",
  "updatedAt": "2025-10-01T10:30:00Z",
  "sizeBytes": 1024
}
```

En OpenAPI:

```yaml
properties:
  createdAt:
    type: string
    format: date-time
    readOnly: true
```

---

## Idempotencia

### Idempotency-Key

```http
POST /v1/workspaces
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000

{
  "name": "My Workspace"
}
```

**Comportamiento:**
- Primera request: Crea, retorna 201
- Segunda con mismo key: Retorna 200 + recurso existente

### Sin Header

```http
POST /v1/workspaces
# Sin Idempotency-Key

# Mismo request dos veces
→ 201 Created (primera)
→ 409 Conflict (segunda, ya existe)
```

---

## Validación Optimista (ETag)

### If-Match

```http
PATCH /v1/workspaces/user-service
If-Match: "abc123def456"

→ 200 OK (si ETag coincide)
→ 412 Precondition Failed (si no coincide)
```

### If-None-Match

```http
GET /v1/workspaces/user-service
If-None-Match: "abc123def456"

→ 200 OK (cambió)
→ 304 Not Modified (sin cambios)
```

---

## Webhooks

### Registro

```http
POST /v1/webhooks

{
  "url": "https://tu-dominio.com/webhooks",
  "events": ["workspace.created", "file.updated"],
  "headers": {
    "X-Custom-Header": "value"
  }
}
```

### Envío

```http
POST https://tu-dominio.com/webhooks
X-Webhook-Signature: sha256=abc123...
X-Webhook-Id: evt-abc123
X-Webhook-Timestamp: 1633081800

{
  "event": "workspace.created",
  "data": { "workspaceId": "user-service" }
}
```

---

## Eventos en Tiempo Real (SSE)

### Suscripción

```http
POST /v1/workspaces/{id}/events:subscribe

{
  "eventTypes": ["file.created", "validation.completed"]
}
```

### Stream

```
id: evt-abc123
event: file.created
timestamp: 2025-10-01T10:30:00Z
data: {"path":"api.yaml"}

id: evt-def456
event: validation.completed
timestamp: 2025-10-01T10:35:00Z
data: {"valid":true}
```

---

## Observabilidad

### Principios

1. **Loggeable:** Todo debe ser loggeable
2. **Monitoreable:** Métricas de todo
3. **Traceable:** Request-Id en todo
4. **Debuggeable:** Información suficiente en logs

---

## Audit Logging

### Qué Loggear

```
- Quién: user_id
- Qué: operación (CREATE, UPDATE, DELETE)
- Cuándo: timestamp
- Dónde: resource_id
- Por qué: reason (si aplica)
- Resultado: success/failure
```

### Estructura

```json
{
  "timestamp": "2025-10-01T10:30:00Z",
  "actor": "user-123",
  "action": "UPDATE",
  "resource": "workspaces/user-service",
  "changes": {
    "name": { "before": "Old", "after": "New" }
  },
  "result": "SUCCESS",
  "requestId": "req-abc123"
}
```

### Retención

- **Operaciones críticas:** 7 años
- **Operaciones normales:** 1 año
- **Logs de debug:** 30 días

---

## Structured Logging

### Formato

```json
{
  "timestamp": "2025-10-01T10:30:00.123Z",
  "level": "INFO",
  "logger": "workspaces.service",
  "message": "Workspace created",
  "requestId": "req-abc123",
  "userId": "user-123",
  "workspaceId": "user-service",
  "duration": 145,
  "status": 201
}
```

### Campos Requeridos

- `timestamp` (ISO 8601)
- `level` (DEBUG, INFO, WARN, ERROR)
- `message` (texto corto)
- `requestId` (UUID)
- Contexto específico (user_id, resource_id, etc)

---

## Metrics y Monitoring

### SLO/SLA

```
Availabilidad: 99.95% (target)
Latencia P50: < 100ms
Latencia P99: < 500ms
Error rate: < 0.1%
```

### Métricas Clave

```
http_requests_total
http_request_duration_seconds
http_errors_total
cache_hit_rate
database_query_duration
```

### Alertas

```
ErrorRate > 1% por 5 minutos
P99Latency > 1 segundo
Availability < 99.9%
```

---

## Distributed Tracing

### Trace-Id

Incluir en todos los logs y responses:

```http
X-Trace-Id: 550e8400-e29b-41d4-a716-446655440000
X-Span-Id: abcdef123456
X-Parent-Span-Id: xyz789abc
```

### Estructura

```json
{
  "traceId": "550e8400-e29b-41d4-a716-446655440000",
  "spans": [
    {
      "spanId": "abcdef123456",
      "parentSpanId": null,
      "operationName": "POST /v1/workspaces",
      "startTime": 1633081800000,
      "duration": 145,
      "tags": {
        "http.method": "POST",
        "http.status": 201
      }
    }
  ]
}
```

---

## Data Retention

### Políticas

```
Datos activos:     Sin límite
Datos archivados:  1 año
Datos borrados:    30 días (recoverable)
Logs de acceso:    90 días
Audit logs:        7 años
Backups:           30 días
```

### GDPR Compliance

- Right to deletion: Borrar en < 30 días
- Right to access: Exportar datos en 30 días
- Right to rectification: Corregir datos
- Data portability: Formato estándar (JSON)

---

## Richardson Maturity Model

Niveles de madurez REST:

### Nivel 0: Swamp of POX

```
POST /api
{
  "operation": "getWorkspace",
  "workspaceId": "user-service"
}
```

❌ Sin estructura REST.

### Nivel 1: Resources

```
GET /api/workspaces/user-service
POST /api/workspaces
```

✅ Recursos identificados.

### Nivel 2: HTTP Verbs

```
GET /workspaces
POST /workspaces
PATCH /workspaces/{id}
DELETE /workspaces/{id}
```

✅ Métodos HTTP correctos, códigos de estado.

### Nivel 3: HATEOAS

```json
{
  "workspaceId": "user-service",
  "_links": {
    "self": { "href": "/workspaces/user-service" },
    "files": { "href": "/workspaces/user-service/files" }
  }
}
```

✅ Hypermedia como motor de estado.

**Recomendación:** API Factory alcanza Nivel 2-3.

---

## Documentación OpenAPI

### Estructura Mínima

```yaml
openapi: 3.0.3
info:
  title: Workspace API
  version: 1.0.0
  description: API para gestionar workspaces

servers:
  - url: https://api.apifactory.org/v1

tags:
  - name: Workspaces
    description: Operaciones sobre workspaces

paths:
  /workspaces:
    get:
      operationId: workspaces#list
      tags: [Workspaces]
      summary: Listar workspaces
      responses:
        "200":
          description: Lista de workspaces
```

### Validaciones

```yaml
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
```

---

## Checklist de Diseño

Antes de lanzar un endpoint:

- [ ] **URL:** Minúsculas, plural, RESTful
- [ ] **Métodos:** Correctos (GET, POST, PATCH, DELETE, PUT)
- [ ] **Tags:** Un tag único
- [ ] **OperationId:** Formato `{resource}#{action}`
- [ ] **Códigos HTTP:** 200, 201, 204, 206, 304, 4xx, 5xx
- [ ] **RFC 7807:** Error format
- [ ] **Headers:** Content-Type, Auth, ETag, Rate-Limit, CORS
- [ ] **Paginación:** limit, offset, total, hasMore
- [ ] **Filtrado:** Query params
- [ ] **Búsqueda:** Parámetro q
- [ ] **Ordenamiento:** Parámetro sort
- [ ] **Resource Expansion:** ?include param
- [ ] **Partial Responses:** ?fields param
- [ ] **Respuestas Polimórficas:** type discriminator
- [ ] **HATEOAS:** Links en responses
- [ ] **Versionado:** /v1 en URL
- [ ] **Compatibilidad:** No breaking changes
- [ ] **Deprecation:** Si deprecated
- [ ] **Conditional Requests:** If-Match, If-None-Match
- [ ] **ETag:** For cache validation
- [ ] **Caching:** Cache-Control headers
- [ ] **Compression:** gzip support
- [ ] **Streaming:** Para archivos grandes
- [ ] **Range Requests:** 206 Partial Content
- [ ] **Size Limits:** Validar payloads
- [ ] **Query Complexity:** Limitar complejidad
- [ ] **Rate Limiting:** Headers presentes
- [ ] **Retry:** Estrategia definida
- [ ] **Timeout:** Configurado
- [ ] **Circuit Breaker:** Si aplica
- [ ] **Health Checks:** /health/live, /health/ready
- [ ] **CORS:** Configurado
- [ ] **CSRF:** Protection si necesario
- [ ] **Request Signing:** Si es servidor-a-servidor
- [ ] **Idempotency:** Key para POSTs
- [ ] **Soft Delete:** Status=archived
- [ ] **Campos Computed:** readOnly
- [ ] **Long-Running Ops:** Si tarda >30s
- [ ] **Bulk Operations:** batch endpoints
- [ ] **Webhooks:** Si notificaciones push
- [ ] **SSE:** Si real-time
- [ ] **Request-Id:** En todo
- [ ] **Audit Logging:** Cambios registrados
- [ ] **Structured Logging:** JSON
- [ ] **Metrics:** Observables
- [ ] **Tracing:** Trace-Id
- [ ] **Documentación:** OpenAPI completo
- [ ] **Ejemplos:** Casos de uso
- [ ] **Tests:** Éxito y error
- [ ] **Security Review:** ✅

---

## Cómo Adoptar este Documento

### Para Proyectos Internos (API Factory)

1. Incluir referencia en README.md
2. Usar checklist antes de publicar
3. Validar OpenAPI schema

### Para Proyectos Externos

```bash
git clone https://github.com/apifactory-org/restful-api-guidelines.git
cd restful-api-guidelines
# Adaptar a tu organización
```

### Referenciación

```markdown
Este proyecto sigue las [RESTful API Guidelines](https://github.com/apifactory-org/restful-api-guidelines) de API Factory v1.0.
```

---

## Contribuciones

### Proceso

1. Fork repositorio
2. Rama: `git checkout -b feature/mejora`
3. Cambios
4. PR con descripción y justificación

### Aprobación

- Mínimo 2 approvals
- Discussion en issues para cambios mayores

---

## Estructura del Repositorio

```
restful-api-guidelines/
├── README.md
├── GUIDELINES.md (este documento)
├── CHANGELOG.md
├── examples/
│   ├── openapi-complete.yaml
│   ├── error-response.json
│   └── pagination.json
├── templates/
│   └── openapi-template.yaml
├── tests/
│   └── guidelines.test.js
└── LICENSE (MIT)
```

---

## Validación

### Linting OpenAPI

```bash
npm install -g @openapitools/openapi-generator-cli
swagger-cli validate api.yaml
```

### Testing

```bash
npm test
npm run test -- --api-url https://api.example.com
```

---

## Normatividad

### RFCs Aplicables

- RFC 7230-7235 - HTTP/1.1 Semantics
- RFC 9110 - HTTP Semantics
- RFC 3986 - URI
- RFC 7807 - Problem Details
- RFC 8288 - Web Linking

### Estándares

- OpenAPI 3.0.3
- Google AIPs (general scope)
- JSON Schema Draft 7

---

## Versionado de la Especificación

**Semantic Versioning:** MAJOR.MINOR.PATCH

- **MAJOR:** Breaking changes
- **MINOR:** Nuevas características compatibles
- **PATCH:** Correcciones

**Política:** 12 meses de deprecation antes de remover.

---

## Historial de Cambios

| Versión | Fecha | Descripción |
|---------|-------|-------------|
| **1.0** | 2025-10-26 | Especificación inicial completa |

---

## Información de Contacto

**Equipo de Governance de API Factory**

- Email: api-governance@apifactory.org
- GitHub: github.com/apifactory-org/restful-api-guidelines
- Docs: docs.apifactory.org

---

## Licencia

```
MIT License

Copyright (c) 2025 API Factory

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

---

**Documento de Guía de Estilo | RESTful API Guidelines v1.0 | API Factory**  
**Clasificación:** Público | Última revisión: Octubre 2025