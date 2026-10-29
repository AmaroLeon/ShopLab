# ShopLab

Proyecto de aprendizaje: una tienda con clientes, productos y pedidos.
Sirve para practicar arquitectura por capas (Clean Architecture), CQRS,
autenticación con claims, middleware, gateway y tests con xUnit y Reqnroll.

## Tecnologías (Tech stack)

- .NET 9
- ASP.NET Core Web API (controllers)
- xUnit

## Estructura

```
src/Api/
  ShopLab.Api             Host: controllers, middleware, configuración (Program.cs)
  ShopLab.Application     Casos de uso: commands, queries, handlers, DTOs, interfaces
  ShopLab.Domain          Entidades y reglas de negocio (no depende de nadie)
  ShopLab.Infrastructure  Acceso a datos: EF Core y repositorios
tests/                    Tests unitarios (xUnit), un proyecto por capa
docs/                     Plan, progreso y comandos
```

## Cómo compilar

```powershell
dotnet build
```