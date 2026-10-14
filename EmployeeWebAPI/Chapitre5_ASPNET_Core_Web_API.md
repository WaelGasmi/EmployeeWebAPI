# Chapitre 5 : ASP.NET Core Web API

**Enseignant :** Malek Zribi

---

## Table des matières

1. [Introduction](#1-introduction)
2. [RESTful API](#2-restful-api)
3. [Conventions pour les Web API](#3-conventions-pour-les-web-api)
4. [Créer un projet Web API](#4-créer-un-projet-web-api)
5. [Création des classes du modèle](#5-création-des-classes-du-modèle)
6. [Installer les packages Entity Framework Core](#6-installer-les-packages-entity-framework-core)
7. [Créer la classe de contexte](#7-créer-la-classe-de-contexte)
8. [Configuration de la BD](#8-configuration-de-la-bd)
9. [EF Migration](#9-ef-migration)
10. [Repository Pattern](#10-repository-pattern)
11. [Les contrôleurs WebAPI](#11-les-contrôleurs-webapi)
12. [HTTP Status Codes](#12-http-status-codes)
13. [GetEmployee(id)](#13-getemployeeid)
14. [Méthode POST : créer un employé](#14-méthode-post--créer-un-employé)
15. [Validation de modèle](#15-validation-de-modèle-dans-aspnet-core-rest-api)
16. [Mettre à jour les données (PUT)](#16-mettre-à-jour-les-données-la-requête-http-put)
17. [Requête DELETE](#17-requête-delete-dans-aspnet-core-rest-api)
18. [Requête HTTP Search](#18-requête-http-search)
19. [Checklist du projet](#19-checklist-récapitulative-du-projet)

---

## 1. Introduction

- Le Framework web **ASP.NET Core MVC** permet de créer des **services web HTTP** accessibles depuis n'importe quel client : navigateurs, appareils mobiles, applications IoT et autres.
- C'est une plate-forme idéale pour créer des **services RESTful** sous .NET Core.

```
Web Application  ──HTTP Request──►                ◄──HTTP Request──  Mobile Application
                 ◄─HTTP Response Data (JSON/XML)─  Web API  ─HTTP Response Data (JSON/XML)─►
                                       ▲  │
                          HTTP Request │  │ HTTP Response Data (JSON/XML/Other)
                                       │  ▼
                              Win Form Application
```

---

## 2. RESTful API

**REST** = **RE**presentational **S**tate **T**ransfer
**API** = **A**pplication **P**rogramming **I**nterface

### Exemple Web API

- Une API REST permet aux applications d'**interagir** les unes avec les autres et d'**échanger des données**.
- Exemple : vous créez une application mobile ou Web qui doit afficher des données météorologiques (température, humidité, vitesse du vent, etc.).

```
Application (Web / Mobile)  ── Request  : City = London ──►  REST API
                            ◄─ Response : London Weather ──
```

---

## 3. Conventions pour les Web API

### 3.1 Ressource et URI

- Dans le contexte d'une API REST, une **ressource** est une entité de données telle que *Produit, Employé, Client, Commande*, etc.
- Une API REST qui fournit des données sur les employés rend la liste des employés disponible à l'**URI** (Uniform Resource Identifier) suivant :

```
http://localhost:65180/api/employees
 │         │              │    │
 │         │              │    └── Endpoint
 │         │              └─────── Optional
 │         └────────────────────── Domain Name
 └──────────────────────────────── Protocol
```

### 3.2 Les verbes HTTP

En plus de l'URI, nous devons également envoyer un **HTTP Verb** au serveur. C'est ce verbe qui indique à l'API **quoi faire** avec la ressource (créer, consulter, supprimer ou modifier un employé).

| HTTP Verb | Opération CRUD |
|-----------|----------------|
| `POST`    | Create         |
| `GET`     | Read           |
| `PUT` / `PATCH` | Update   |
| `DELETE`  | Delete         |

### 3.3 Combinaison URI + verbe HTTP

La combinaison de l'URI et du verbe HTTP, envoyée avec chaque requête, indique au serveur ce qu'il doit faire avec la ressource :

| URI                  | HTTP Verb | Résultat                      |
|----------------------|-----------|-------------------------------|
| `.../api/employees`   | `GET`     | Récupère la liste des employés |
| `.../api/employees/1` | `GET`     | Récupère l'employé avec Id = 1 |
| `.../api/employees`   | `POST`    | Crée un nouvel employé         |
| `.../api/employees/1` | `PUT`     | Met à jour l'employé avec Id = 1 |
| `.../api/employees/1` | `DELETE`  | Supprime l'employé avec Id = 1 |

---

## 4. Créer un projet Web API

### Étape 1 : Choisir le modèle de projet

Dans Visual Studio : **Créer un projet** → filtres `C#` / `Toutes les plateformes` / `Web` → choisir **API web ASP.NET Core** (modèle permettant de créer une application ASP.NET Core avec un exemple de contrôleur pour un service HTTP RESTful) → **Suivant**.

### Étape 2 : Configurer le nouveau projet

| Champ | Valeur |
|-------|--------|
| Nom du projet | `EmployeeWebAPI` |
| Emplacement | `C:\Users\USER\source\repos` |
| Nom de la solution | `EmployeeWebAPI` |

### Étape 3 : Informations supplémentaires

> ⚠️ **Vérifier que l'infrastructure est `.NET 8.0 (Prise en charge à long terme)`**, afin de pouvoir tester directement avec **Swagger** intégré dans le projet.

Options cochées dans la capture :

- Type d'authentification : **Aucun**
- ✅ Configurer pour HTTPS
- ☐ Activer la prise en charge du conteneur
- ✅ Activer la prise en charge d'OpenAPI
- ☐ N'utilisez pas d'instructions de niveau supérieur
- ✅ Utiliser des contrôleurs
- ☐ Inscrire dans l'orchestration de .NET Aspire

### Étape 4 : Structure du projet

```
Solution 'EmployeeWebAPI'
└── EmployeeWebAPI
    ├── Connected Services
    ├── Dépendances
    ├── Properties
    ├── Controllers
    │   └── WeatherForecastController.cs
    ├── appsettings.json
    ├── EmployeeWebAPI.http
    ├── Program.cs
    └── WeatherForecast.cs
```

### Étape 5 : Exécuter l'application

L'exécution ouvre **Swagger UI** (ex. `https://localhost:7262/swagger/index.html`) avec l'endpoint `GET /WeatherForecast`.

---

## 5. Création des classes du modèle

Pour pouvoir **réutiliser** les classes du modèle dans d'autres projets, créer dans la solution un nouveau projet de type **Bibliothèque de classes** nommé **`MesClasses`** et y ajouter les classes suivantes :

```csharp
using System.ComponentModel.DataAnnotations;

namespace MesClasses
{
    public class Employee
    {
        public int EmployeeId { get; set; }

        [Required]
        [StringLength(100, MinimumLength = 2)]
        public string FirstName { get; set; }

        [Required]
        public string LastName { get; set; }

        [Required]
        [EmailAddress]
        public string Email { get; set; }

        public DateTime DateOfBrith { get; set; }
        public Gender Gender { get; set; }
        public int DepartmentId { get; set; }
        public string PhotoPath { get; set; }
        public Department? Department { get; set; }
    }

    public enum Gender
    {
        Male,
        Female
    }
}
```

```csharp
namespace MesClasses
{
    public class Department
    {
        public int DepartmentId { get; set; }
        public string DepartmentName { get; set; }
    }
}
```

> 📝 Le nom de la propriété `DateOfBrith` (faute de frappe dans le cours) est utilisé partout dans le code et dans le JSON ; gardez la même orthographe dans tout le projet.

---

## 6. Installer les packages Entity Framework Core

Via **NuGet** (clic droit sur le projet → *Gérer les packages NuGet*), installer dans le projet `EmployeeWebAPI` (version **8.0.x**, compatible avec .NET 8) :

| Package |
|---------|
| `Microsoft.EntityFrameworkCore` |
| `Microsoft.EntityFrameworkCore.SqlServer` |
| `Microsoft.EntityFrameworkCore.Tools` |

Équivalent en ligne de commande :

```powershell
Install-Package Microsoft.EntityFrameworkCore -Version 8.0.15
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 8.0.15
Install-Package Microsoft.EntityFrameworkCore.Tools -Version 8.0.15
```

---

## 7. Créer la classe de contexte

1. Créer un dossier **`Models`** dans le projet `EmployeeWebAPI`.
2. Y ajouter la classe suivante :

```csharp
using Microsoft.EntityFrameworkCore;
using MesClasses;

public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }

    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
}
```

3. Les classes `Employee` et `Department` étant dans la bibliothèque `MesClasses`, **ajouter la référence au projet** :
   *Dépendances* → clic droit → **Ajouter une référence de projet...** → cocher `MesClasses`.
4. Ajouter la directive :

```csharp
using MesClasses;
```

---

## 8. Configuration de la BD

### 8.1 Chaîne de connexion (`appsettings.json`)

```json
"ConnectionStrings": {
  "DBConnection": "server=(localdb)\\MSSQLLocalDB;database=APIEmployeeDB;Trusted_Connection=true"
}
```

### 8.2 Inscrire le service de contexte (`Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DBConnection")));

var app = builder.Build();
```

---

## 9. EF Migration

Dans la **Package Manager Console**, exécuter les commandes de migration pour créer la base de données :

```powershell
Add-Migration InitialCreate
Update-Database
```

---

## 10. Repository Pattern

### 10.1 Interface `IEmployeeRepository`

Dans le dossier `Models`, ajouter un dossier **`Repositories`**, puis créer l'interface qui définit les opérations de base sur les employés :

```csharp
public interface IEmployeeRepository
{
    Task<IEnumerable<Employee>> GetEmployees();
    Task<Employee> GetEmployee(int employeeId);
    Task<Employee> AddEmployee(Employee employee);
    Task<Employee> UpdateEmployee(Employee employee);
    Task<Employee> DeleteEmployee(int employeeId);
    Task<Employee> GetEmployeeByEmail(string email);
}
```

> La méthode `Search(string name, Gender? gender)` sera ajoutée plus tard (voir section 18).

### 10.2 Implémentation SQL Server : `EmployeeRepository`

```csharp
using Microsoft.EntityFrameworkCore;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using MesClasses;

namespace EmployeeWebAPI.Models.Repositories
{
    public class EmployeeRepository : IEmployeeRepository
    {
        private readonly AppDbContext appDbContext;

        public EmployeeRepository(AppDbContext appDbContext)
        {
            this.appDbContext = appDbContext;
        }

        public async Task<IEnumerable<Employee>> GetEmployees()
        {
            return await appDbContext.Employees.ToListAsync();
        }

        public async Task<Employee> GetEmployee(int employeeId)
        {
            return await appDbContext.Employees
                .FirstOrDefaultAsync(e => e.EmployeeId == employeeId);
        }

        public async Task<Employee> AddEmployee(Employee employee)
        {
            var result = await appDbContext.Employees.AddAsync(employee);
            await appDbContext.SaveChangesAsync();
            return result.Entity;
        }

        public async Task<Employee> UpdateEmployee(Employee employee)
        {
            var result = await appDbContext.Employees
                .FirstOrDefaultAsync(e => e.EmployeeId == employee.EmployeeId);

            if (result != null)
            {
                result.FirstName = employee.FirstName;
                result.LastName = employee.LastName;
                result.Email = employee.Email;
                result.DateOfBrith = employee.DateOfBrith;
                result.Gender = employee.Gender;
                result.DepartmentId = employee.DepartmentId;
                result.PhotoPath = employee.PhotoPath;

                await appDbContext.SaveChangesAsync();
                return result;
            }

            return null;
        }

        public async Task<Employee> DeleteEmployee(int employeeId)
        {
            var result = await appDbContext.Employees
                .FirstOrDefaultAsync(e => e.EmployeeId == employeeId);

            if (result != null)
            {
                appDbContext.Employees.Remove(result);
                await appDbContext.SaveChangesAsync();
                return result;
            }

            return null;
        }

        public async Task<Employee> GetEmployeeByEmail(string email)
        {
            return await appDbContext.Employees
                .FirstOrDefaultAsync(e => e.Email == email);
        }
    }
}
```

### 10.3 Interface `IDepartmentRepository`

```csharp
public interface IDepartmentRepository
{
    IEnumerable<Department> GetDepartments();
    Department GetDepartment(int departmentId);
}
```

### 10.4 Implémentation SQL Server : `DepartmentRepository`

```csharp
public class DepartmentRepository : IDepartmentRepository
{
    private readonly AppDbContext appDbContext;

    public DepartmentRepository(AppDbContext appDbContext)
    {
        this.appDbContext = appDbContext;
    }

    public Department GetDepartment(int departmentId)
    {
        return appDbContext.Departments
            .FirstOrDefault(d => d.DepartmentId == departmentId);
    }

    public IEnumerable<Department> GetDepartments()
    {
        return appDbContext.Departments;
    }
}
```

### 10.5 Injection des services : classes Repository (`Program.cs`)

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DBConnection")));

builder.Services.AddScoped<IDepartmentRepository, DepartmentRepository>();
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();

var app = builder.Build();
```

### 10.6 Avantages du Repository Pattern

- Le code est plus **propre**, plus facile à **réutiliser** et à **maintenir**.
- Permet de créer des systèmes **faiblement couplés**.
- Exemple : pour utiliser **Oracle** au lieu de SQL Server, on implémente un `OracleRepository` qui sait lire et écrire dans la base Oracle, puis on l'enregistre dans le système d'**injection de dépendances**.
- Dans un projet de **test unitaire**, il est facile de remplacer un vrai repository par une **fausse implémentation** (mock).

---

## 11. Les contrôleurs WebAPI

- Un contrôleur WebAPI est une classe qui **hérite de `ControllerBase`**.
- `ControllerBase` se trouve dans l'espace de noms `Microsoft.AspNetCore.Mvc`.
- Pour une application **Web MVC**, on crée un contrôleur qui dérive de la classe `Controller`.

| Classe | Utilisation |
|--------|-------------|
| `Controller` | Web Applications (MVC) |
| `ControllerBase` | Web APIs |

### 11.1 Ajouter un contrôleur API

Dans le dossier **Controllers** → *Ajouter* → *Nouvel élément généré automatiquement* → catégorie **API** → **Contrôleur d'API - Vide** → nommer le contrôleur (ex. `EmployeesController`).

### 11.2 `EmployeesController` : squelette et injection de dépendance

```csharp
[Route("api/[controller]")]
[ApiController]
public class EmployeesController : ControllerBase
{
    // Injection de dépendance du service IEmployeeRepository
    private readonly IEmployeeRepository employeeRepository;

    public EmployeesController(IEmployeeRepository employeeRepository)
    {
        this.employeeRepository = employeeRepository;
    }

    // ... actions ci-dessous
}
```

### 11.3 Méthode d'action : `GetEmployees`

**GET** `/api/employees`

```csharp
[HttpGet]
public async Task<ActionResult> GetEmployees()
{
    try
    {
        return Ok(await employeeRepository.GetEmployees());
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error retrieving data from the database");
    }
}
```

---

## 12. HTTP Status Codes

Une fois le projet exécuté, on vérifie si l'API fonctionne correctement grâce aux **codes d'état HTTP**, qui fournissent au client le statut de la requête.

| Niveau | Code | Signification |
|--------|------|---------------|
| **200** | 200 | OK |
| | 201 | Created |
| | 204 | No Content |
| **400** | 400 | Bad Request |
| | 401 | Unauthorized |
| | 404 | Not Found |
| **500** | 500 | Internal Server Error |
| | 501 | Not Implemented |
| | 503 | Service Unavailable |

### Méthodes d'assistance ASP.NET Core

| HTTP Status Code | Helper Method |
|------------------|---------------|
| 200 | `Ok()` |
| 201 | `Created()` |
| 204 | `NoContent()` |
| 400 | `BadRequest()` |
| 401 | `Unauthorized()` |
| 404 | `NotFound()` |
| 500 | `StatusCode(500, ...)` |
| 501 | `StatusCode(501, ...)` |
| 503 | `StatusCode(503, ...)` |

### Test avec Postman

`GET https://localhost:44369/api/Employees` → **Status : 200 OK**, réponse JSON :

```json
[
  {
    "employeeId": 1,
    "firstName": "ZRIBI",
    "lastName": "MALEK",
    "email": "malek_zribi@yahoo.fr",
    "dateOfBrith": "1976-05-10T00:00:00",
    "gender": 1,
    "departmentId": 1,
    "photoPath": null
  },
  {
    "employeeId": 2,
    "firstName": "TRIKI",
    "lastName": "MOHAMED ALI",
    "email": "triki_ali@gmail.com",
    "dateOfBrith": "1988-05-22T00:00:00",
    "gender": 0,
    "departmentId": 2,
    "photoPath": null
  }
]
```

---

## 13. GetEmployee(id)

| URI | HTTP Verb | Résultat |
|-----|-----------|----------|
| `.../api/employees` | GET | Liste des employés |
| `.../api/employees/1` | GET | Employé avec Id = 1 |

```csharp
[HttpGet("{id:int}")]
public async Task<ActionResult<Employee>> GetEmployee(int id)
{
    try
    {
        var result = await employeeRepository.GetEmployee(id);

        if (result == null) return NotFound();

        return result;
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error retrieving data from the database");
    }
}
```

### Test avec Postman

`GET https://localhost:44369/api/Employees/1` → **200 OK**

```json
{
  "employeeId": 1,
  "firstName": "ZRIBI",
  "lastName": "MALEK",
  "email": "malek_zribi@yahoo.fr",
  "dateOfBrith": "1976-05-10T00:00:00",
  "gender": 1,
  "departmentId": 1,
  "photoPath": null
}
```

---

## 14. Méthode POST : créer un employé

```csharp
[HttpPost]
public async Task<ActionResult<Employee>> CreateEmployee([FromBody] Employee employee)
{
    try
    {
        if (employee == null)
            return BadRequest();

        var createdEmployee = await employeeRepository.AddEmployee(employee);

        return CreatedAtAction(nameof(GetEmployee),
            new { id = createdEmployee.EmployeeId }, createdEmployee);
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error creating new employee record");
    }
}
```

### Ce qui se produit lorsqu'une nouvelle ressource est créée

1. Renvoyer le **code d'état HTTP 201** pour indiquer que la ressource a été créée avec succès.
2. Renvoyer la **ressource nouvellement créée** (ici, le nouvel employé).
3. Ajouter un **en-tête `Location`** à la réponse : il spécifie l'URI de l'objet employé nouvellement créé.

La méthode **`CreatedAtAction`** réalise ces trois choses. On utilise l'opérateur **`nameof`** au lieu d'inclure le nom de la méthode (`GetEmployee`) dans une chaîne.

### Test avec Postman

`POST https://localhost:44369/api/Employees/` — *Body* → *raw* → *JSON* :

```json
{
  "firstName": "ZRIBI",
  "lastName": "MALEK",
  "email": "malek_zribi@yahoo.fr",
  "dateOfBrith": "1976-05-10",
  "gender": 1,
  "departmentId": 1,
  "photoPath": null
}
```

Réponse : **201 Created**

```json
{
  "employeeId": 3,
  "firstName": "ZRIBI",
  "lastName": "MALEK",
  "email": "malek_zribi@yahoo.fr",
  "dateOfBrith": "1976-05-10T00:00:00",
  "gender": 1,
  "departmentId": 1,
  "photoPath": null
}
```

---

## 15. Validation de modèle dans ASP.NET Core REST API

### 15.1 Attributs de validation intégrés

Ces attributs se trouvent dans l'espace de noms `System.ComponentModel.DataAnnotations`.

| Attribut | Rôle |
|----------|------|
| `Required` | Spécifie que le champ est obligatoire |
| `Range` | Spécifie la valeur minimale et maximale autorisée |
| `MinLength` | Spécifie la longueur minimale d'une chaîne |
| `MaxLength` | Spécifie la longueur maximale d'une chaîne |
| `Compare` | Compare 2 propriétés d'un modèle (ex. `Email` et `ConfirmEmail`) |
| `RegularExpression` | Valide si la valeur correspond au modèle de l'expression régulière |

### 15.2 Implémentation

Pour implémenter la validation, **décorer les propriétés** avec les attributs de validation. Exemple : `FirstName` est obligatoire, doit contenir **au minimum 2 caractères** et **ne pas dépasser 100 caractères**.

```csharp
public class Employee
{
    public int EmployeeId { get; set; }

    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string FirstName { get; set; }

    [Required]
    public string LastName { get; set; }

    [Required]
    public string Email { get; set; }

    public DateTime DateOfBrith { get; set; }
    public Gender Gender { get; set; }
    public int DepartmentId { get; set; }
    public string PhotoPath { get; set; }
}
```

### 15.3 Exemple de réponse en cas de données non conformes

Requête `POST` avec `"firstName": "B"` et sans `lastName` → **400 Bad Request** :

```json
{
  "status": 400,
  "traceId": "|871a158d-454303cd000a058f.",
  "errors": {
    "LastName": [
      "The LastName field is required."
    ],
    "FirstName": [
      "The field FirstName must be a string with a minimum length of 2 and a maximum length of 100."
    ]
  }
}
```

### 15.4 Erreurs de validation de modèle personnalisées

- Pour ajouter une erreur de validation personnalisée, utiliser la méthode **`AddModelError()`** de l'objet **`ModelState`**.
- Dans une API REST ASP.NET Core, **il n'est pas nécessaire de vérifier explicitement** si l'état du modèle est valide : la classe de contrôleur étant décorée avec l'attribut **`[ApiController]`**, elle vérifie automatiquement `ModelState` et renvoie **400 (Bad Request)** avec les erreurs de validation.

Modifions `CreateEmployee` pour **vérifier si l'email existe déjà**. D'abord, la méthode du repository (déjà vue en 10.2) :

```csharp
public async Task<Employee> GetEmployeeByEmail(string email)
{
    return await appDbContext.Employees
        .FirstOrDefaultAsync(e => e.Email == email);
}
```

Puis l'action du contrôleur :

```csharp
[HttpPost]
public async Task<ActionResult<Employee>> CreateEmployee(Employee employee)
{
    try
    {
        if (employee == null)
        {
            return BadRequest();
        }
        else
        // Add custom model validation error
        {
            var emp = await employeeRepository.GetEmployeeByEmail(employee.Email);

            if (emp != null)
            {
                ModelState.AddModelError("email", "Employee email already in use");
                return BadRequest(ModelState);
            }
            else
            {
                var createdEmployee = await employeeRepository.AddEmployee(employee);

                return CreatedAtAction(nameof(GetEmployee),
                    new { id = createdEmployee.EmployeeId }, createdEmployee);
            }
        }
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error retrieving data from the database");
    }
}
```

### 15.5 Test : email déjà existant

`POST` d'un employé avec un email existant → **400 Bad Request** :

```json
{
  "email": [
    "Employee email already in use"
  ]
}
```

---

## 16. Mettre à jour les données (la requête HTTP PUT)

- Pour mettre à jour un employé, ajouter une requête HTTP **PUT** et donner **l'ID de l'employé dans l'URL**.
- Le paramètre de `UpdateEmployee` doit être décoré avec `[FromBody]`, mais ce n'est **pas obligatoire** si l'attribut `[ApiController]` est présent sur le contrôleur.

| URI | HTTP Verb | Résultat |
|-----|-----------|----------|
| `.../api/employees` | GET | Liste des employés |
| `.../api/employees/1` | GET | Employé avec Id = 1 |
| `.../api/employees` | POST | Crée un employé |
| `.../api/employees/1` | PUT | Met à jour l'employé avec Id = 1 |

```csharp
[HttpPut("{id:int}")]
public async Task<ActionResult<Employee>> UpdateEmployee(int id, Employee employee)
{
    try
    {
        if (id != employee.EmployeeId)
            return BadRequest("Employee ID mismatch");

        var employeeToUpdate = await employeeRepository.GetEmployee(id);

        if (employeeToUpdate == null)
            return NotFound($"Employee with Id = {id} not found");

        return await employeeRepository.UpdateEmployee(employee);
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error updating data");
    }
}
```

### Test avec Postman

`PUT https://localhost:44369/api/Employees/3` — Body (JSON) :

```json
{
  "employeeId": 3,
  "firstName": "Akroute",
  "lastName": "Salah",
  "email": "ak_sal@yahoo.fr",
  "dateOfBrith": "1976-05-10T00:00:00",
  "gender": 1,
  "departmentId": 1,
  "photoPath": null
}
```

Réponse : **200 OK** avec l'employé mis à jour.

---

## 17. Requête DELETE dans ASP.NET Core REST API

Pour supprimer une ressource, envoyer une requête HTTP **DELETE** à l'URI `/api/Employees/{ID}`. L'ID de l'employé à supprimer doit être passé **dans l'URI**.

| URI | HTTP Verb | Résultat |
|-----|-----------|----------|
| `.../api/employees` | GET | Liste des employés |
| `.../api/employees` | POST | Crée un employé |
| `.../api/employees/1` | GET | Employé avec Id = 1 |
| `.../api/employees/1` | PUT | Met à jour l'employé avec Id = 1 |
| `.../api/employees/1` | **DELETE** | Supprime l'employé avec Id = 1 |

```csharp
[HttpDelete("{id:int}")]
public async Task<ActionResult<Employee>> DeleteEmployee(int id)
{
    try
    {
        var employeeToDelete = await employeeRepository.GetEmployee(id);

        if (employeeToDelete == null)
        {
            return NotFound($"Employee with Id = {id} not found");
        }

        return await employeeRepository.DeleteEmployee(id);
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error deleting data");
    }
}
```

### Test avec Postman

`DELETE https://localhost:44369/api/Employees/3` (sans body) → **200 OK**, l'employé supprimé est renvoyé :

```json
{
  "employeeId": 3,
  "firstName": "Akroute",
  "lastName": "Salah",
  "email": "ak_sal@yahoo.fr",
  "dateOfBrith": "1976-05-10T00:00:00",
  "gender": 1,
  "departmentId": 1,
  "photoPath": null
}
```

---

## 18. Requête HTTP Search

### 18.1 Ajouter la méthode `Search` dans l'interface du repository

```csharp
public interface IEmployeeRepository
{
    Task<IEnumerable<Employee>> GetEmployees();
    Task<Employee> GetEmployee(int employeeId);
    Task<Employee> AddEmployee(Employee employee);
    Task<Employee> UpdateEmployee(Employee employee);
    Task<Employee> DeleteEmployee(int employeeId);
    Task<Employee> GetEmployeeByEmail(string email);
    Task<IEnumerable<Employee>> Search(string name, Gender? gender);
}
```

### 18.2 Code de `Search` dans `EmployeeRepository`

```csharp
public async Task<IEnumerable<Employee>> Search(string name, Gender? gender)
{
    IQueryable<Employee> query = appDbContext.Employees;

    if (!string.IsNullOrEmpty(name))
    {
        query = query.Where(e => e.FirstName.Contains(name)
                              || e.LastName.Contains(name));
    }

    if (gender != null)
    {
        query = query.Where(e => e.Gender == gender);
    }

    return await query.ToListAsync();
}
```

### 18.3 Code de l'action `Search` dans `EmployeesController`

```csharp
[HttpGet("{search}")]
public async Task<ActionResult<IEnumerable<Employee>>> Search(string name, Gender? gender)
{
    try
    {
        var result = await employeeRepository.Search(name, gender);

        if (result.Any())
        {
            return Ok(result);
        }

        return NotFound();
    }
    catch (Exception)
    {
        return StatusCode(StatusCodes.Status500InternalServerError,
            "Error retrieving data from the database");
    }
}
```

### 18.4 Tests avec Postman

**Recherche avec résultat** : `GET https://localhost:44369/api/Employees/Search?name=zribi&gender=0` → **200 OK**

```json
[
  {
    "employeeId": 1,
    "firstName": "ZRIBI",
    "lastName": "MALEK",
    "email": "malek_zribi@yahoo.fr",
    "dateOfBrith": "1976-05-10T00:00:00",
    "gender": 0,
    "departmentId": 1,
    "photoPath": null
  }
]
```

**Recherche sans résultat** : `GET https://localhost:44369/api/Employees/Search?name=turki` → **404 Not Found**

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Not Found",
  "status": 404,
  "traceId": "|ba9c9e70-44871144cbef37ad."
}
```

---

## 19. Checklist récapitulative du projet

- [ ] Créer le projet **API web ASP.NET Core** (`EmployeeWebAPI`) en **.NET 8.0** avec OpenAPI/Swagger et contrôleurs.
- [ ] Créer la bibliothèque de classes **`MesClasses`** (`Employee`, `Department`, `Gender`) et l'ajouter en référence.
- [ ] Installer les packages **EF Core** (`EntityFrameworkCore`, `SqlServer`, `Tools`).
- [ ] Créer le dossier `Models` et la classe **`AppDbContext`**.
- [ ] Ajouter la chaîne de connexion `DBConnection` dans `appsettings.json`.
- [ ] Inscrire `AddDbContext` dans `Program.cs`.
- [ ] Exécuter `Add-Migration InitialCreate` puis `Update-Database`.
- [ ] Créer `Models/Repositories` : `IEmployeeRepository`, `EmployeeRepository`, `IDepartmentRepository`, `DepartmentRepository`.
- [ ] Enregistrer les repositories avec `AddScoped` dans `Program.cs`.
- [ ] Créer `EmployeesController` (`[Route("api/[controller]")]`, `[ApiController]`, hérite de `ControllerBase`).
- [ ] Implémenter les actions : `GetEmployees`, `GetEmployee(id)`, `CreateEmployee`, `UpdateEmployee`, `DeleteEmployee`, `Search`.
- [ ] Ajouter la validation (`[Required]`, `[StringLength]`, `AddModelError` pour l'email unique).
- [ ] Tester chaque endpoint avec **Swagger** ou **Postman**.

---

*Cours réalisé par Malek Zribi*
