# ShopLab – Progreso

## Estado

- **Fase actual:** 0 – Estructura (sin lógica)
- **Última sesión:** 2026-10-29 – creado el plan (`docs/PLAN.md`).

## Fase 0 – Checklist

- [x] Fijar el SDK en `global.json` → **9.0.320** (decisión: .NET 9, no se instala .NET 10 para no afectar a Chorus; migrar a .NET 10 más adelante → escribir ADR 0001)
- [x] Crear `ShopLab/` e iniciar Git
- [x] Crear `ShopLab.sln` (formato .sln clásico)
- [x] Crear proyectos de `src/Api` y añadirlos a la solución (Api = Web API con `--use-controllers`; Application, Domain, Infrastructure = class library)
- [x] Borrar los `Class1.cs` y `UnitTest1.cs` de ejemplo (desde VS). `WeatherForecast` se borra más adelante
- [x] Crear proyectos de `tests/` (MSTest): Domain.Tests, Application.Tests, Api.Tests, y añadirlos a la solución
- [x] Poner las referencias según la regla de dependencias (verificadas en los 7 `.csproj`)
- [x] `.gitignore`, `.editorconfig`, `Directory.Build.props` (en `Solution Items`)
- [ ] `Directory.Packages.props`, `README.md`
- [x] La solución compila vacía (`dotnet build` → succeeded)
- [ ] Primer commit
- [ ] Crear el repositorio en GitHub, conectarlo y hacer push

(Gateway, Web y Specs se crean en sus fases.)

## Decisiones tomadas

| Fecha | Decisión | Motivo | ADR pendiente |
|---|---|---|---|
| 2026-10-29 | .NET 9 (SDK 9.0.320) | No instalar .NET 10 para no cambiar el SDK que usa Chorus. Migrar a .NET 10 (LTS) más adelante | 0001 |
| 2026-10-29 | Tests con **xUnit** (v2), no MSTest | Ya lo conozco; es el más usado en .NET y en entrevistas; Reqnroll lo soporta (`Reqnroll.xUnit`) | 0002 |

> 📝 **Nota (fase 6):** además de xUnit, hacer **un proyecto pequeño de MSTest** con los tests de alguna clase,
> como ejemplo comparativo (`[TestClass]`/`[TestMethod]`/`[TestInitialize]` vs `[Fact]`/constructor/`IDisposable`).

## Próxima sesión

Empezar la checklist de la fase 0.

## Notas y dudas

- **Infrastructure (fase 1–2):** clases `internal` (`CustomerRepository`, `ShopLabDbContext`…); solo `AddInfrastructure()` `public`.
  Así la referencia Api → Infrastructure no es "una ventana" a EF. Reforzar con tests de arquitectura (fase 6).
