# Aplicaciones Distribuidas
# Semana 1 - Creación del Microservicio `MicroClientes.API`

**Carrera:** Desarrollo de Software  
**Asignatura:** Aplicaciones Distribuidas  
**Docente:** Ing. Giovanny Cholca MSc.  

---

# 1. Objetivo del laboratorio

En este laboratorio se creará un microservicio llamado `MicroClientes.API` utilizando Visual Studio Community y ASP.NET Core Web API.

El proyecto permitirá administrar clientes mediante un CRUD completo:

- Consultar todos los clientes.
- Consultar un cliente por su identificador.
- Registrar un cliente.
- Actualizar un cliente.
- Eliminar lógicamente un cliente.

La información se almacenará en SQL Server utilizando Entity Framework Core con el enfoque Code First.

---

# 2. Arquitectura que se construirá

```text
Swagger / Postman
        |
ClientesController
        |
IClienteRepository
        |
ClienteRepository
        |
AppDbContext
        |
Entity Framework Core
        |
SQL Server
```

---

# 3. Requisitos previos

Antes de iniciar, verificar que se encuentren instaladas las siguientes herramientas:

- Visual Studio Community.
- SDK de .NET 8 o superior.
- SQL Server o SQL Server ejecutándose en Docker.
- Docker Desktop, si se utilizará SQL Server en contenedor.
- Postman, opcional.

Verificar la instalación de .NET desde PowerShell o CMD:

```bash
dotnet --version
```

Verificar que Docker se encuentre funcionando:

```bash
docker ps
```

---

# 4. Crear la solución y el proyecto en Visual Studio Community

## 4.1 Abrir Visual Studio

1. Iniciar **Visual Studio Community**.
2. Seleccionar **Crear un proyecto**.
3. En el buscador escribir:

```text
ASP.NET Core Web API
```

4. Seleccionar la plantilla **ASP.NET Core Web API**.
5. Presionar **Siguiente**.

---

## 4.2 Configurar el proyecto

Ingresar la siguiente información:

| Campo | Valor |
|---|---|
| Nombre del proyecto | `MicroClientes.API` |
| Nombre de la solución | `MicroClientes` |
| Ubicación | Carpeta de trabajo del estudiante |

Ejemplo de ubicación:

```text
C:\Desarrollo\AplicacionesDistribuidas\MicroClientes
```

Activar la opción:

```text
Colocar la solución y el proyecto en el mismo directorio
```

Presionar **Siguiente**.

---

## 4.3 Configurar ASP.NET Core

Seleccionar las siguientes opciones:

| Opción | Valor recomendado |
|---|---|
| Framework | `.NET 8.0` |
| Tipo de autenticación | Ninguno |
| Configurar para HTTPS | Activado |
| Habilitar OpenAPI | Activado |
| Usar controladores | Activado |
| No usar instrucciones de nivel superior | Desactivado |

> Si el equipo únicamente tiene instalado .NET 10, se puede seleccionar .NET 10. Los paquetes de Entity Framework Core deben coincidir con la versión principal del proyecto.

Presionar **Crear**.

---

# 5. Ejecutar el proyecto inicial

Presionar:

```text
F5
```

o utilizar el botón de ejecución de Visual Studio.

Swagger debe abrirse en una dirección similar a:

```text
https://localhost:7000/swagger
```

El número del puerto puede variar en cada equipo.

Si el proyecto contiene el ejemplo `WeatherForecast`, se puede eliminar porque no será utilizado.

Eliminar, si existen:

```text
Controllers/WeatherForecastController.cs
WeatherForecast.cs
```

---

# 6. Crear la estructura de carpetas

En el Explorador de soluciones, hacer clic derecho sobre el proyecto `MicroClientes.API`, seleccionar **Agregar > Nueva carpeta** y crear:

```text
Controllers
Data
DTOs
Entities
Repositories
```

La estructura esperada será:

```text
MicroClientes.API
|
|-- Controllers
|-- Data
|-- DTOs
|-- Entities
|-- Repositories
|-- Program.cs
|-- appsettings.json
```

---

# 7. Instalar paquetes de Entity Framework Core

## 7.1 Abrir el administrador de paquetes NuGet

1. Clic derecho sobre el proyecto.
2. Seleccionar **Administrar paquetes NuGet**.
3. Ingresar en la pestaña **Examinar**.
4. Instalar los siguientes paquetes:

```text
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools
Microsoft.EntityFrameworkCore.Design
```

## 7.2 Función de cada paquete

| Paquete | Función |
|---|---|
| `Microsoft.EntityFrameworkCore.SqlServer` | Permite conectar Entity Framework Core con SQL Server. |
| `Microsoft.EntityFrameworkCore.Tools` | Permite ejecutar migraciones desde la consola de Visual Studio. |
| `Microsoft.EntityFrameworkCore.Design` | Proporciona herramientas necesarias para crear migraciones y generar la base de datos. |

> No instalar versiones incompatibles. Por ejemplo, para un proyecto .NET 8 se recomienda Entity Framework Core 8.

---

# 8. Crear la entidad base

Crear el archivo:

```text
Entities/EntityBase.cs
```

Agregar el siguiente código:

```csharp
using System.ComponentModel.DataAnnotations;

namespace MicroClientes.API.Entities;

public abstract class EntityBase
{
    [Key]
    public int Id { get; set; }

    public bool Estado { get; set; } = true;

    public DateTime FechaIngreso { get; set; } = DateTime.Now;

    public DateTime? FechaModificacion { get; set; }
}
```

## Explicación

- `abstract`: evita crear objetos directamente de `EntityBase`.
- `Id`: representa la clave primaria.
- `Estado`: permite manejar eliminación lógica.
- `FechaIngreso`: registra cuándo se creó el registro.
- `FechaModificacion`: registra la última actualización.

La entidad `Cliente` heredará estas propiedades.

---

# 9. Crear la entidad Cliente

Crear el archivo:

```text
Entities/Cliente.cs
```

Agregar:

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MicroClientes.API.Entities;

[Table("Clientes")]
public class Cliente : EntityBase
{
    [Required]
    [StringLength(100)]
    public string Nombre { get; set; } = string.Empty;

    [Required]
    [StringLength(100)]
    public string Apellido { get; set; } = string.Empty;

    [Required]
    [StringLength(10, MinimumLength = 10)]
    public string Cedula { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    [StringLength(150)]
    public string Email { get; set; } = string.Empty;

    [StringLength(20)]
    public string? Telefono { get; set; }

    [Column(TypeName = "date")]
    public DateTime FechaNacimiento { get; set; }
}
```

## Explicación de las anotaciones

| Anotación | Función |
|---|---|
| `[Table("Clientes")]` | Define el nombre de la tabla en SQL Server. |
| `[Required]` | Indica que el campo es obligatorio. |
| `[StringLength]` | Define la longitud máxima y, cuando corresponde, mínima. |
| `[EmailAddress]` | Valida que el texto tenga formato de correo electrónico. |
| `[Column(TypeName = "date")]` | Guarda únicamente la fecha, sin hora. |

---

# 10. Crear los DTO de Cliente

Un DTO es un objeto utilizado para recibir o devolver información a través de la API.

No se recomienda exponer directamente la entidad porque:

- Evita enviar campos internos.
- Reduce el acoplamiento entre la base de datos y la API.
- Permite aplicar validaciones diferentes.
- Mejora la seguridad y el mantenimiento.

## 10.1 DTO para crear un cliente

Crear:

```text
DTOs/ClienteCreateDto.cs
```

```csharp
using System.ComponentModel.DataAnnotations;

namespace MicroClientes.API.DTOs;

public class ClienteCreateDto
{
    [Required]
    [StringLength(100)]
    public string Nombre { get; set; } = string.Empty;

    [Required]
    [StringLength(100)]
    public string Apellido { get; set; } = string.Empty;

    [Required]
    [StringLength(10, MinimumLength = 10)]
    public string Cedula { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    [StringLength(150)]
    public string Email { get; set; } = string.Empty;

    [StringLength(20)]
    public string? Telefono { get; set; }

    [Required]
    public DateTime FechaNacimiento { get; set; }
}
```

---

## 10.2 DTO para actualizar un cliente

Crear:

```text
DTOs/ClienteUpdateDto.cs
```

```csharp
using System.ComponentModel.DataAnnotations;

namespace MicroClientes.API.DTOs;

public class ClienteUpdateDto
{
    [Required]
    [StringLength(100)]
    public string Nombre { get; set; } = string.Empty;

    [Required]
    [StringLength(100)]
    public string Apellido { get; set; } = string.Empty;

    [Required]
    [StringLength(10, MinimumLength = 10)]
    public string Cedula { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    [StringLength(150)]
    public string Email { get; set; } = string.Empty;

    [StringLength(20)]
    public string? Telefono { get; set; }

    [Required]
    public DateTime FechaNacimiento { get; set; }

    public bool Estado { get; set; } = true;
}
```

---

## 10.3 DTO de respuesta

Crear:

```text
DTOs/ClienteResponseDto.cs
```

```csharp
namespace MicroClientes.API.DTOs;

public class ClienteResponseDto
{
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public string Apellido { get; set; } = string.Empty;
    public string NombreCompleto { get; set; } = string.Empty;
    public string Cedula { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public string? Telefono { get; set; }
    public DateTime FechaNacimiento { get; set; }
    public bool Estado { get; set; }
    public DateTime FechaIngreso { get; set; }
    public DateTime? FechaModificacion { get; set; }
}
```

---

# 11. Crear AppDbContext

Crear:

```text
Data/AppDbContext.cs
```

```csharp
using Microsoft.EntityFrameworkCore;
using MicroClientes.API.Entities;

namespace MicroClientes.API.Data;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }

    public DbSet<Cliente> Clientes => Set<Cliente>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<Cliente>()
            .HasIndex(cliente => cliente.Cedula)
            .IsUnique();

        modelBuilder.Entity<Cliente>()
            .HasIndex(cliente => cliente.Email)
            .IsUnique();
    }
}
```

## Explicación

- `DbContext`: representa la conexión con la base de datos.
- `DbSet<Cliente>`: representa la tabla `Clientes`.
- `HasIndex(...).IsUnique()`: evita registrar cédulas o correos duplicados.

---

# 12. Configurar la cadena de conexión

Abrir:

```text
appsettings.json
```

Agregar la sección `ConnectionStrings`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1434;Database=MicroClientesDB;User Id=sa;Password=adminAppDist2025#;TrustServerCertificate=True;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## Consideraciones

- `localhost,1434`: servidor y puerto de SQL Server.
- `MicroClientesDB`: nombre de la base de datos.
- `sa`: usuario administrador.
- `Password`: contraseña configurada en Docker.
- `TrustServerCertificate=True`: permite conexión local sin un certificado confiable.

Si SQL Server está instalado directamente en Windows, la cadena puede cambiar.

Ejemplo con autenticación de Windows:

```json
"DefaultConnection": "Server=localhost;Database=MicroClientesDB;Trusted_Connection=True;TrustServerCertificate=True;"
```

---

# 13. Registrar Entity Framework Core en Program.cs

Abrir:

```text
Program.cs
```

Reemplazar o ajustar el contenido:

```csharp
using Microsoft.EntityFrameworkCore;
using MicroClientes.API.Data;
using MicroClientes.API.Repositories;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddScoped<IClienteRepository, ClienteRepository>();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

---

# 14. Crear el Repository

## 14.1 Crear la interfaz

Crear:

```text
Repositories/IClienteRepository.cs
```

```csharp
using MicroClientes.API.Entities;

namespace MicroClientes.API.Repositories;

public interface IClienteRepository
{
    Task<List<Cliente>> ObtenerTodosAsync();
    Task<Cliente?> ObtenerPorIdAsync(int id);
    Task<Cliente?> ObtenerPorCedulaAsync(string cedula);
    Task<bool> ExisteEmailAsync(string email, int? idExcluir = null);
    Task<Cliente> CrearAsync(Cliente cliente);
    Task<bool> ActualizarAsync(Cliente cliente);
    Task<bool> EliminarAsync(Cliente cliente);
}
```

---

## 14.2 Crear la implementación

Crear:

```text
Repositories/ClienteRepository.cs
```

```csharp
using Microsoft.EntityFrameworkCore;
using MicroClientes.API.Data;
using MicroClientes.API.Entities;

namespace MicroClientes.API.Repositories;

public class ClienteRepository : IClienteRepository
{
    private readonly AppDbContext _context;

    public ClienteRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<List<Cliente>> ObtenerTodosAsync()
    {
        return await _context.Clientes
            .AsNoTracking()
            .OrderBy(cliente => cliente.Apellido)
            .ThenBy(cliente => cliente.Nombre)
            .ToListAsync();
    }

    public async Task<Cliente?> ObtenerPorIdAsync(int id)
    {
        return await _context.Clientes
            .FirstOrDefaultAsync(cliente => cliente.Id == id);
    }

    public async Task<Cliente?> ObtenerPorCedulaAsync(string cedula)
    {
        return await _context.Clientes
            .AsNoTracking()
            .FirstOrDefaultAsync(cliente => cliente.Cedula == cedula);
    }

    public async Task<bool> ExisteEmailAsync(string email, int? idExcluir = null)
    {
        return await _context.Clientes.AnyAsync(cliente =>
            cliente.Email == email &&
            (!idExcluir.HasValue || cliente.Id != idExcluir.Value));
    }

    public async Task<Cliente> CrearAsync(Cliente cliente)
    {
        _context.Clientes.Add(cliente);
        await _context.SaveChangesAsync();
        return cliente;
    }

    public async Task<bool> ActualizarAsync(Cliente cliente)
    {
        _context.Clientes.Update(cliente);
        return await _context.SaveChangesAsync() > 0;
    }

    public async Task<bool> EliminarAsync(Cliente cliente)
    {
        cliente.Estado = false;
        cliente.FechaModificacion = DateTime.Now;

        _context.Clientes.Update(cliente);
        return await _context.SaveChangesAsync() > 0;
    }
}
```

## Explicación

El repositorio concentra el acceso a la base de datos y evita escribir consultas directamente en el controlador.

- `AsNoTracking()`: mejora el rendimiento en consultas de solo lectura.
- `OrderBy`: ordena los clientes.
- `SaveChangesAsync()`: guarda los cambios en SQL Server.
- `EliminarAsync()`: realiza una eliminación lógica, cambiando `Estado` a `false`.

---

# 15. Crear el controlador y las API CRUD

Crear:

```text
Controllers/ClientesController.cs
```

Agregar:

```csharp
using Microsoft.AspNetCore.Mvc;
using MicroClientes.API.DTOs;
using MicroClientes.API.Entities;
using MicroClientes.API.Repositories;

namespace MicroClientes.API.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ClientesController : ControllerBase
{
    private readonly IClienteRepository _repository;

    public ClientesController(IClienteRepository repository)
    {
        _repository = repository;
    }

    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<ClienteResponseDto>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<ClienteResponseDto>>> ObtenerTodos()
    {
        var clientes = await _repository.ObtenerTodosAsync();

        var respuesta = clientes.Select(MapearResponseDto);

        return Ok(respuesta);
    }

    [HttpGet("{id:int}")]
    [ProducesResponseType(typeof(ClienteResponseDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<ClienteResponseDto>> ObtenerPorId(int id)
    {
        var cliente = await _repository.ObtenerPorIdAsync(id);

        if (cliente is null)
        {
            return NotFound(new
            {
                mensaje = $"No se encontró el cliente con Id {id}."
            });
        }

        return Ok(MapearResponseDto(cliente));
    }

    [HttpPost]
    [ProducesResponseType(typeof(ClienteResponseDto), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status409Conflict)]
    public async Task<ActionResult<ClienteResponseDto>> Crear(
        [FromBody] ClienteCreateDto dto)
    {
        var clienteConCedula = await _repository.ObtenerPorCedulaAsync(dto.Cedula);

        if (clienteConCedula is not null)
        {
            return Conflict(new
            {
                mensaje = "Ya existe un cliente registrado con esa cédula."
            });
        }

        if (await _repository.ExisteEmailAsync(dto.Email))
        {
            return Conflict(new
            {
                mensaje = "Ya existe un cliente registrado con ese correo electrónico."
            });
        }

        var cliente = new Cliente
        {
            Nombre = dto.Nombre.Trim(),
            Apellido = dto.Apellido.Trim(),
            Cedula = dto.Cedula.Trim(),
            Email = dto.Email.Trim().ToLowerInvariant(),
            Telefono = dto.Telefono?.Trim(),
            FechaNacimiento = dto.FechaNacimiento,
            Estado = true,
            FechaIngreso = DateTime.Now
        };

        await _repository.CrearAsync(cliente);

        var respuesta = MapearResponseDto(cliente);

        return CreatedAtAction(
            nameof(ObtenerPorId),
            new { id = cliente.Id },
            respuesta);
    }

    [HttpPut("{id:int}")]
    [ProducesResponseType(typeof(ClienteResponseDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status409Conflict)]
    public async Task<ActionResult<ClienteResponseDto>> Actualizar(
        int id,
        [FromBody] ClienteUpdateDto dto)
    {
        var cliente = await _repository.ObtenerPorIdAsync(id);

        if (cliente is null)
        {
            return NotFound(new
            {
                mensaje = $"No se encontró el cliente con Id {id}."
            });
        }

        var clienteConCedula = await _repository.ObtenerPorCedulaAsync(dto.Cedula);

        if (clienteConCedula is not null && clienteConCedula.Id != id)
        {
            return Conflict(new
            {
                mensaje = "La cédula ya pertenece a otro cliente."
            });
        }

        if (await _repository.ExisteEmailAsync(dto.Email, id))
        {
            return Conflict(new
            {
                mensaje = "El correo electrónico ya pertenece a otro cliente."
            });
        }

        cliente.Nombre = dto.Nombre.Trim();
        cliente.Apellido = dto.Apellido.Trim();
        cliente.Cedula = dto.Cedula.Trim();
        cliente.Email = dto.Email.Trim().ToLowerInvariant();
        cliente.Telefono = dto.Telefono?.Trim();
        cliente.FechaNacimiento = dto.FechaNacimiento;
        cliente.Estado = dto.Estado;
        cliente.FechaModificacion = DateTime.Now;

        await _repository.ActualizarAsync(cliente);

        return Ok(MapearResponseDto(cliente));
    }

    [HttpDelete("{id:int}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> Eliminar(int id)
    {
        var cliente = await _repository.ObtenerPorIdAsync(id);

        if (cliente is null)
        {
            return NotFound(new
            {
                mensaje = $"No se encontró el cliente con Id {id}."
            });
        }

        if (!cliente.Estado)
        {
            return BadRequest(new
            {
                mensaje = "El cliente ya se encuentra inactivo."
            });
        }

        await _repository.EliminarAsync(cliente);

        return NoContent();
    }

    private static ClienteResponseDto MapearResponseDto(Cliente cliente)
    {
        return new ClienteResponseDto
        {
            Id = cliente.Id,
            Nombre = cliente.Nombre,
            Apellido = cliente.Apellido,
            NombreCompleto = $"{cliente.Nombre} {cliente.Apellido}",
            Cedula = cliente.Cedula,
            Email = cliente.Email,
            Telefono = cliente.Telefono,
            FechaNacimiento = cliente.FechaNacimiento,
            Estado = cliente.Estado,
            FechaIngreso = cliente.FechaIngreso,
            FechaModificacion = cliente.FechaModificacion
        };
    }
}
```

---

# 16. Endpoints creados

| Método | Endpoint | Acción | Código esperado |
|---|---|---|---|
| GET | `/api/clientes` | Obtener todos los clientes | `200 OK` |
| GET | `/api/clientes/{id}` | Obtener un cliente por Id | `200 OK` o `404 Not Found` |
| POST | `/api/clientes` | Crear un cliente | `201 Created` |
| PUT | `/api/clientes/{id}` | Actualizar un cliente | `200 OK` |
| DELETE | `/api/clientes/{id}` | Desactivar un cliente | `204 No Content` |

---

# 17. Crear la migración y la base de datos

## 17.1 Abrir la consola de administración de paquetes

En Visual Studio seleccionar:

```text
Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes
```

Verificar que el proyecto predeterminado sea:

```text
MicroClientes.API
```

## 17.2 Crear la migración

Ejecutar:

```powershell
Add-Migration InitialCreate
```

Entity Framework Core creará la carpeta:

```text
Migrations
```

## 17.3 Crear o actualizar la base de datos

Ejecutar:

```powershell
Update-Database
```

Este comando:

1. Se conecta a SQL Server.
2. Crea la base de datos `MicroClientesDB` si no existe.
3. Crea la tabla `Clientes`.
4. Crea los índices únicos de cédula y correo.
5. Crea la tabla interna `__EFMigrationsHistory`.

---

# 18. Ejecutar y probar la API en Swagger

Presionar `F5`.

Swagger mostrará el grupo:

```text
Clientes
```

---

## 18.1 Probar POST - Crear cliente

Endpoint:

```http
POST /api/clientes
```

JSON de prueba:

```json
{
  "nombre": "Carlos",
  "apellido": "Pérez",
  "cedula": "1723456789",
  "email": "carlos.perez@email.com",
  "telefono": "0999999999",
  "fechaNacimiento": "1995-08-15"
}
```

Respuesta esperada:

```http
201 Created
```

Ejemplo:

```json
{
  "id": 1,
  "nombre": "Carlos",
  "apellido": "Pérez",
  "nombreCompleto": "Carlos Pérez",
  "cedula": "1723456789",
  "email": "carlos.perez@email.com",
  "telefono": "0999999999",
  "fechaNacimiento": "1995-08-15T00:00:00",
  "estado": true,
  "fechaIngreso": "2026-07-11T20:00:00",
  "fechaModificacion": null
}
```

---

## 18.2 Probar GET - Obtener todos

```http
GET /api/clientes
```

Respuesta esperada:

```http
200 OK
```

---

## 18.3 Probar GET - Obtener por Id

```http
GET /api/clientes/1
```

Respuesta esperada:

```http
200 OK
```

Si no existe:

```http
404 Not Found
```

---

## 18.4 Probar PUT - Actualizar cliente

```http
PUT /api/clientes/1
```

JSON:

```json
{
  "nombre": "Carlos Andrés",
  "apellido": "Pérez López",
  "cedula": "1723456789",
  "email": "carlos.andres@email.com",
  "telefono": "0988888888",
  "fechaNacimiento": "1995-08-15",
  "estado": true
}
```

Respuesta esperada:

```http
200 OK
```

---

## 18.5 Probar DELETE - Desactivar cliente

```http
DELETE /api/clientes/1
```

Respuesta esperada:

```http
204 No Content
```

El registro no se elimina físicamente. El campo `Estado` cambia a `false`.

---

# 19. Códigos HTTP utilizados

| Código | Uso en el proyecto |
|---|---|
| `200 OK` | Consulta o actualización exitosa. |
| `201 Created` | Cliente creado correctamente. |
| `204 No Content` | Cliente desactivado correctamente. |
| `400 Bad Request` | Datos inválidos o cliente ya inactivo. |
| `404 Not Found` | No se encontró el cliente. |
| `409 Conflict` | Cédula o correo duplicado. |
| `500 Internal Server Error` | Error inesperado del servidor. |

---

# 20. Verificar los datos en SQL Server

Ejecutar en SQL Server Management Studio, Azure Data Studio o la herramienta utilizada:

```sql
USE MicroClientesDB;
GO

SELECT
    Id,
    Nombre,
    Apellido,
    Cedula,
    Email,
    Telefono,
    FechaNacimiento,
    Estado,
    FechaIngreso,
    FechaModificacion
FROM Clientes;
GO
```

---

# 21. Errores frecuentes

## Error de conexión con SQL Server

Revisar:

- Que el contenedor esté iniciado.
- Que el puerto sea correcto.
- Que el usuario y contraseña sean correctos.
- Que la cadena contenga `TrustServerCertificate=True`.

Verificar Docker:

```bash
docker ps
```

---

## Error al ejecutar Add-Migration

Verificar:

- Que el proyecto predeterminado sea `MicroClientes.API`.
- Que `Microsoft.EntityFrameworkCore.Tools` esté instalado.
- Que el proyecto compile correctamente.

Compilar:

```text
Compilar > Compilar solución
```

---

## Swagger no muestra el controlador

Revisar que `Program.cs` contenga:

```csharp
builder.Services.AddControllers();
app.MapControllers();
```

Y que el controlador tenga:

```csharp
[ApiController]
[Route("api/[controller]")]
```

---

## Error por cédula o correo duplicado

La base de datos tiene índices únicos. La API responderá:

```http
409 Conflict
```

Se debe utilizar otra cédula y otro correo.

---

# 22. Estructura final del proyecto

```text
MicroClientes
|
|-- MicroClientes.API
    |
    |-- Controllers
    |   |-- ClientesController.cs
    |
    |-- Data
    |   |-- AppDbContext.cs
    |
    |-- DTOs
    |   |-- ClienteCreateDto.cs
    |   |-- ClienteUpdateDto.cs
    |   |-- ClienteResponseDto.cs
    |
    |-- Entities
    |   |-- EntityBase.cs
    |   |-- Cliente.cs
    |
    |-- Migrations
    |
    |-- Repositories
    |   |-- IClienteRepository.cs
    |   |-- ClienteRepository.cs
    |
    |-- appsettings.json
    |-- Program.cs
    |-- MicroClientes.API.csproj
```

---

# 23. Resultado esperado

Al finalizar el laboratorio, el estudiante debe tener:

- Una solución llamada `MicroClientes`.
- Un proyecto ASP.NET Core Web API llamado `MicroClientes.API`.
- Una entidad `Cliente` con validaciones.
- DTO separados para creación, actualización y respuesta.
- Un `AppDbContext` conectado a SQL Server.
- Un repositorio para acceder a la base de datos.
- Un controlador con CRUD completo.
- Una base de datos creada mediante migraciones.
- Los endpoints probados desde Swagger.

---

# 24. Actividad de comprobación

Cada estudiante debe registrar al menos tres clientes y presentar capturas de:

1. Proyecto ejecutándose en Visual Studio.
2. Swagger mostrando los endpoints.
3. Ejecución exitosa de `POST`.
4. Ejecución exitosa de `GET`.
5. Ejecución exitosa de `PUT`.
6. Ejecución exitosa de `DELETE`.
7. Tabla `Clientes` creada en SQL Server.

