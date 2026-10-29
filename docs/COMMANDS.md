# ShopLab – Comandos ejecutados

> Registro de los comandos que voy ejecutando, en orden, con qué hace cada uno.
> Todos se ejecutan desde la raíz del repositorio: `C:\Datos\Repos\Personal\Net\ShopLab`

## Fase 0 – Estructura

### 2026-10-29

```powershell
dotnet --version
```
Muestra el SDK de .NET más alto instalado → `9.0.320`.

```powershell
dotnet new globaljson --sdk-version 9.0.320
```
Crea `global.json`, que fija el SDK que usa este repositorio (aunque haya otros instalados).

```powershell
git init
```
Inicia el repositorio Git (crea la carpeta `.git`).

```powershell
dotnet new sln -n ShopLab
```
Crea la solución `ShopLab.sln`. `sln` es la plantilla; `-n` el nombre (sin extensión).

```powershell
dotnet new classlib -n ShopLab.Domain -o src/Api/ShopLab.Domain
```
Crea la biblioteca de clases `ShopLab.Domain` (net9.0). `-o` indica la carpeta donde se crea.
Después: borrar el `Class1.cs` de ejemplo.

```powershell
dotnet sln ShopLab.sln add src/Api/ShopLab.Domain/ShopLab.Domain.csproj
```
Añade el proyecto a la solución. Crea solas las carpetas de solución `src` → `Api`, igual que en disco.

```powershell
dotnet new classlib -n ShopLab.Application -o src/Api/ShopLab.Application
dotnet sln ShopLab.sln add src\Api\ShopLab.Application\ShopLab.Application.csproj
```
Lo mismo para `ShopLab.Application`. En Windows valen `/` y `\` en las rutas.

```powershell
dotnet new classlib -n ShopLab.Infrastructure -o src/Api/ShopLab.Infrastructure
dotnet sln ShopLab.sln add src/Api/ShopLab.Infrastructure/ShopLab.Infrastructure.csproj
```
Lo mismo para `ShopLab.Infrastructure`.
⚠️ Error típico: la ruta del `.csproj` lleva el nombre dos veces (carpeta + fichero):
`src/Api/ShopLab.Infrastructure.csproj` ❌ → `src/Api/ShopLab.Infrastructure/ShopLab.Infrastructure.csproj` ✅

```powershell
dotnet new webapi -n ShopLab.Api -o src/Api/ShopLab.Api --use-controllers
dotnet sln ShopLab.sln add src/Api/ShopLab.Api/ShopLab.Api.csproj
```
Crea la Web API. `--use-controllers`: con controllers (en .NET 9 la plantilla usa Minimal APIs por defecto).
Trae un ejemplo (`WeatherForecast`) que se borra más adelante.

```powershell
dotnet new xunit -n ShopLab.Domain.Tests -o tests/ShopLab.Domain.Tests
dotnet sln ShopLab.sln add tests/ShopLab.Domain.Tests/ShopLab.Domain.Tests.csproj
dotnet new xunit -n ShopLab.Application.Tests -o tests/ShopLab.Application.Tests
dotnet sln ShopLab.sln add tests/ShopLab.Application.Tests/ShopLab.Application.Tests.csproj
dotnet new xunit -n ShopLab.Api.Tests -o tests/ShopLab.Api.Tests
dotnet sln ShopLab.sln add tests/ShopLab.Api.Tests/ShopLab.Api.Tests.csproj
```
Proyectos de tests con **xUnit** (el plan decía MSTest; ver decisión en PROGRESS.md). Traen un `UnitTest1.cs` de ejemplo que se borra.

⚠️ Errores que tuve y cómo los arreglé:
- **`-n` y `-o` no coinciden** (`-n ShopLab.Application.Tests -o tests/ShopLab.Api.Tests`): el proyecto se crea con un nombre en una carpeta de otro. Arreglo: borrar la carpeta y crearlo de nuevo.
- **Carpeta mal escrita** (`ShopLabDomain.Tests`, sin punto) y ya añadida a la solución.
  Lo que hice: editar `ShopLab.sln` a mano, quitar el proyecto y su carpeta de solución de todas
  las secciones (`Project`, configuraciones, `NestedProjects`) y volver a crear los tests.
  Alternativa con comandos (más segura):
  ```powershell
  dotnet sln ShopLab.sln remove tests/ShopLabDomain.Tests/ShopLab.Domain.Tests.csproj
  # renombrar la carpeta a ShopLab.Domain.Tests
  dotnet sln ShopLab.sln add tests/ShopLab.Domain.Tests/ShopLab.Domain.Tests.csproj
  ```
- Truco: revisar siempre que `-n` y la última parte de `-o` sean **idénticos**.

## Fase 0 – Referencias entre proyectos

```powershell
dotnet add .\tests\ShopLab.Api.Tests\ShopLab.Api.Tests.csproj reference .\src\Api\ShopLab.Api\ShopLab.Api.csproj
dotnet list .\tests\ShopLab.Api.Tests\ShopLab.Api.Tests.csproj reference
```
```powershell
dotnet add .\tests\ShopLab.Application.Tests\ShopLab.Application.Tests.csproj reference .\src\Api\ShopLab.Application\ShopLab.Application.csproj
dotnet list .\tests\ShopLab.Application.Tests\ShopLab.Application.Tests.csproj reference
```

```powershell
dotnet add .\tests\ShopLab.Domain.Tests\ShopLab.Domain.Tests.csproj reference .\src\Api\ShopLab.Domain\ShopLab.Domain.csproj
dotnet list .\tests\ShopLab.Domain.Tests\ShopLab.Domain.Tests.csproj reference
```

```powershell
dotnet add .\src\Api\ShopLab.Application\ShopLab.Application.csproj reference .\src\Api\ShopLab.Domain\ShopLab.Domain.csproj
dotnet add .\src\Api\ShopLab.Infrastructure\ShopLab.Infrastructure.csproj reference .\src\Api\ShopLab.Application\ShopLab.Application.csproj
dotnet add .\src\Api\ShopLab.Api\ShopLab.Api.csproj reference .\src\Api\ShopLab.Infrastructure\ShopLab.Infrastructure.csproj
dotnet add .\src\Api\ShopLab.Api\ShopLab.Api.csproj reference .\src\Api\ShopLab.Application\ShopLab.Application.csproj
dotnet list .\src\Api\ShopLab.Api\ShopLab.Api.csproj reference
```
Referencias entre capas (la flecha = "conoce a"):
- `Application → Domain`: los casos de uso usan las entidades.
- `Infrastructure → Application`: implementa las interfaces (`ICustomerRepository`) que define Application (Dependency Inversion).
- `Api → Infrastructure`: **solo** para registrarla en `Program.cs` (*composition root*). Reja: en Infrastructure todo `internal` salvo `AddInfrastructure()`.
- `Api → Application`: los controllers llaman a los handlers. Compilaría sin ella (referencias transitivas), pero se pone explícita: lo que se usa directamente, se referencia directamente.

`add … reference`: el primer proyecto **referencia al** segundo (añade un `<ProjectReference>` en su `.csproj`).
`list … reference`: muestra las referencias de un proyecto, para comprobarlo.

## Fase 0 – Ficheros raíz

```powershell
dotnet new gitignore
git status
```
Crea el `.gitignore` oficial de .NET (excluye `bin/`, `obj/`, `.vs/`). `git status` lo comprueba: no deben salir esas carpetas.

```powershell
git branch --show-current
```
Muestra la rama actual. (`git branch` no muestra nada hasta el primer commit.)

```powershell
dotnet new editorconfig
```
Crea el `.editorconfig`: reglas de estilo y nombres que VS aplica y avisa al escribir.
VS lo aplica aunque no esté en la solución. Para verlo en VS: clic derecho en la solución → *Add → Existing Item* → queda en `Solution Items` (ahí también `global.json`, `.gitignore`, `Directory.Build.props`).

```powershell
dotnet new buildprops
```
Crea `Directory.Build.props`: propiedades comunes que heredan **todos** los `.csproj` de debajo (como una clase base).
Moví ahí `TargetFramework` (`net9.0`), `ImplicitUsings` y `Nullable`, y las quité de los 7 `.csproj`.
Cada `.csproj` se queda solo con lo suyo: `Sdk` (Api = `Microsoft.NET.Sdk.Web`), referencias, paquetes y, en tests, `IsPackable`.
⚠️ `Directory.Build.props` no lleva `Sdk`; los `.csproj` **sí** (sin él no compilan).

```powershell
dotnet build
```
Compila toda la solución → `Build succeeded`. Compila en orden de dependencias: Domain primero, Api.Tests al final.

```powershell
dotnet new packagesprops
```
Crea `Directory.Packages.props` (*Central Package Management*): `ManagePackageVersionsCentrally = true` + las versiones, una vez.
- En el props: `<PackageVersion Include="xunit" Version="2.9.2" />` → **qué versión**.
- En cada `.csproj`: `<PackageReference Include="xunit" />` → **que lo usa** (sin `Version`).
⚠️ Errores que tuve:
- **NU1008**: un `.csproj` todavía tenía `Version="..."` (la Api). Con gestión central, la versión solo va en el props.
- **CS0400 `Xunit` not found**: borré la línea `<PackageReference>` entera en vez de solo `Version="..."`. Hacen falta las dos piezas.

```powershell
dotnet new nugetconfig
```
Crea `nuget.config` en la raíz. `<clear />` quita los orígenes globales (Artifactory de la empresa) **solo para ShopLab** y deja solo nuget.org.
Quita el aviso **NU1507** ("2 package sources") y hace que el repo no dependa del servidor de la empresa.
⚠️ Es `dotnet new nugetconfig`, no `dotnet nugetconfig`.

```powershell
dotnet build
```
→ `Build succeeded`, 0 avisos.

`README.md`: creado en VS (*Solution Items → Add → New Item*).
⚠️ VS lo guardó en ANSI y las tildes salían rotas. Arreglo: *File → Save As → flecha de Save → Save with Encoding → UTF-8 without signature (65001)*.

## Fase 0 – Primer commit

```powershell
git config user.name "Amaro Leon"
git config user.email "amaro_1982@outlook.es"
```
Identidad **solo para este repo** (sin `--global`; se guarda en `.git/config`). Chorus sigue con la de la empresa.

```powershell
git add .
git status
```
Prepara todos los ficheros. Comprobar que no salen `bin/`, `obj/`, `.vs/` ni `*.csproj.user`.

```powershell
git commit -m "Create initial solution architecture"
git branch -m master main
git log --oneline
```
Primer commit → `eb31014`. Renombra la rama a `main` (lo habitual hoy y lo que usa GitHub).
Mensajes de commit: inglés, imperativo, mayúscula inicial, sin abreviaturas, < 50 caracteres.
Prueba: *"If applied, this commit will ___"*. Orden en inglés: lo principal al final (*solution architecture*).
