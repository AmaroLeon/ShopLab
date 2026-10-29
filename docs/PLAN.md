# ShopLab – Plan del proyecto

> Contexto para Claude (y para mí). Leer este fichero y `PROGRESS.md` al empezar cada sesión.

## Objetivo

Proyecto de aprendizaje **pequeño pero con piezas no triviales**, inspiradas en lo que hay en Chorus
(Lab Automation): gateway, middleware, autenticación con claims, autorización con policies,
ProblemDetails, CQRS, tests con Reqnroll.

Sirve para practicar el **balotario 1 (Senior Developer)** con código real, mientras estudio el
**balotario 2 (Engineering Lead / TPM)**.

## Reglas de trabajo

1. **Claude no escribe código.** Dice qué toca hacer, explica por qué, señala el fichero de Chorus
   de referencia y revisa lo que yo escribo.
2. **Yo escribo todo el código.** Si no sé algo, lo vemos juntos (explicación, no solución hecha).
3. Revisión de cada paso: nombres en inglés, comentarios XML, SOLID, patrones, casos límite.
4. Un paso cada vez. Sesiones cortas (20–40 min).
5. Al terminar cada sesión se actualiza `PROGRESS.md`.

## Ubicación

- Local: `C:\Datos\Repos\Personal\Net\ShopLab`
- Remoto: mi GitHub personal (se conecta al final de la fase 0).

## Arquitectura

```
[Frontend .NET → luego Angular] ──► [Gateway (YARP)] ──► [API (.NET)] ──► [Base de datos (EF Core)]
                                     = "nginx"            controllers + CQRS
                                     rutas, CORS,         middleware, auth, filtros
                                     cabecera de gateway
```

Regla de dependencias (Clean Architecture):

```
Api ──► Application ──► Domain
 │            ▲
 └──► Infrastructure ─┘   (Infrastructure implementa interfaces definidas en Application)
```

Domain no depende de nadie (D de SOLID: Dependency Inversion).

## Estructura de carpetas

```
ShopLab/
├─ README.md
├─ .gitignore
├─ .editorconfig
├─ global.json                     versión del SDK
├─ Directory.Build.props           propiedades comunes
├─ Directory.Packages.props        versiones centralizadas (como Chorus)
├─ ShopLab.sln
├─ docs/
│  ├─ PLAN.md                      este fichero
│  ├─ PROGRESS.md                  dónde vamos
│  └─ adr/                         decisiones de arquitectura (ADR)
├─ src/
│  ├─ Gateway/ShopLab.Gateway/     proxy YARP (= nginx)
│  ├─ Api/
│  │  ├─ ShopLab.Api/              host: controllers, middleware, filtros, auth
│  │  ├─ ShopLab.Application/      commands, queries, handlers, DTOs, mappers
│  │  ├─ ShopLab.Domain/           entidades y reglas
│  │  └─ ShopLab.Infrastructure/   EF Core, repositorios
│  └─ Web/
│     ├─ ShopLab.Web/              frontend .NET
│     └─ shoplab-angular/          frontend Angular (después)
└─ tests/
   ├─ ShopLab.Domain.Tests/
   ├─ ShopLab.Application.Tests/
   ├─ ShopLab.Api.Tests/
   └─ ShopLab.Api.Specs/           Reqnroll
```

## Dominio

- **Customer**: cliente y usuario que hace login.
- **Product**: producto.
- **Order** + **OrderLine**: pedido y sus líneas (relación pedido–producto). Un cliente tiene muchos pedidos.

## Piezas a aprender (y referencia en Chorus)

| Pieza | Qué haremos | Referencia en Chorus | Balotario 1 |
|---|---|---|---|
| Middleware | `GatewayGateMiddleware`: sin cabecera de gateway / versión no soportada → corta; si no, `next(context)` | `ExternalApiGateMiddleware` | Middleware |
| Autenticación con claims | Login → token con claims (`CustomerId`, `Role`) | `ExternalApiAuthenticationHandler` (`CreateLabAutomationPrincipal`) | Authentication vs Authorization, JWT |
| Autorización | Policy + `IAuthorizationRequirement` + `AuthorizationHandler` | `ExternalApiAuthorizationHandler`, `ExternalApiRequirement` | Authorization |
| Respuestas HTTP | 200, 201 (`Created`), 400, 401, 403, 404, 409 como ProblemDetails con `correlationId` | `ExternalApiProblem`, `ExternalApiProblemDetails`, `ExternalApiControllerBase` | Códigos HTTP |
| Filtros MVC | Filtro de validación → 400 ProblemDetails | `AddExternalApiModelStateProblems` | Filter vs Middleware |
| DTOs y mappers | Contrato separado del dominio, mapeo explícito | `ToSummary`, `ToContractRunOrder` | DTO / Mapper |
| CQRS | Commands y queries separados (decidir: a mano o MediatR) | — (nuevo) | CQRS, MediatR |
| EF Core | Proyecciones `Select`, `AsSplitQuery`, paginación con orden estable | `GetExperimentSummaries` | Include vs Select, N+1 |
| DI | Lifetimes bien elegidos | `AddExternalApiInfrastructure` | Transient/Scoped/Singleton |
| Opciones y secretos | `IOptions<T>`, nada de claves en código | `ProtectedFileExternalApiSecrets` | Configuración |
| Feature flag | Activar/desactivar pedidos → 503 | `ExternalApiAvailability` | Feature flags (balotario 2) |
| Gateway + CORS | YARP con tabla de rutas, reescritura y CORS | `nginx.conf`, `routes.conf` | Arquitectura |
| Unit tests | Handlers y middleware (MSTest + Moq) | Tests de los filtros OpenAPI (PR 48576) | Testing |
| Reqnroll | Features con etiquetas y hooks que arrancan el gateway | `LabAutomationGatewayHooks`, `LabAutomationGateway` (PR 48066) | Testing / BDD |
| SOLID y patrones | Repository, Strategy (precio), Decorator (logging de handlers) | — | SOLID, Patrones |

## Nomenclatura

| Cosa | Convención | Ejemplo |
|---|---|---|
| Proyectos | `ShopLab.<Capa>` | `ShopLab.Application` |
| Commands | Verbo + sustantivo + `Command` | `PlaceOrderCommand`, `RegisterCustomerCommand` |
| Queries | `Get` + qué + `Query` | `GetCustomerOrdersQuery` |
| Handlers | Nombre + `Handler` | `PlaceOrderCommandHandler` |
| DTO salida | `...Response` / `...Summary` | `OrderSummary` |
| DTO entrada | `...Request` | `PlaceOrderRequest` |
| Interfaces | `I` + nombre | `ICustomerRepository` |
| Métodos async | Terminan en `Async` | `GetByIdAsync` |
| Tests | `Método_Escenario_Resultado` | `Handle_UnknownProduct_ReturnsNotFound` |
| Ramas | `feature/<fase>-<tema>` | `feature/phase1-customer-login` |
| Commits | Imperativo, en inglés | `Add customer login with claims` |

## Fases

| Fase | Contenido | Resultado |
|---|---|---|
| 0 | Solución, proyectos, referencias, ficheros raíz, Git, GitHub | Compila vacío y está en GitHub |
| 1 | Customer + login con claims | Login devuelve token; `GET /customers/me` 200/401 |
| 2 | Products + Orders con CQRS, DTOs, mappers, 200/201/400/404/409 | Endpoints con relaciones |
| 3 | Middleware gate, filtro de validación, ProblemDetails con `correlationId` | Errores uniformes |
| 4 | Policy + requirement, feature flag | 403 y 503 donde toca |
| 5 | Gateway YARP + CORS | El frontend solo habla con el gateway |
| 6 | Unit tests → Reqnroll (hook que arranca el gateway) | Suite de tests |
| 7 | Frontend .NET → Angular | Login, productos, pedidos |

## Decisiones pendientes

1. CQRS a mano (interfaces propias) o con MediatR. Recomendación: a mano primero.
2. Autenticación: JWT propio o ASP.NET Core Identity.
3. Frontend .NET: Razor Pages o Blazor.
4. Base de datos: SQLite o SQL Server LocalDB.

Cada decisión tomada se apunta como ADR en `docs/adr/`.
