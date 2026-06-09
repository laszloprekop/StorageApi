# StorageApi

A small ASP.NET Core Web API for a warehouse/inventory system (*Lagersystem*), built with Entity Framework Core (**Code First**).

This is Exercise 1 from the *Web-API / Entity Framework Core* module of the Lexicon LTU VT-2026 - Fullstack .NET Developer course (teacher: Michael Svensson).

## Tech stack

- **.NET 10** (`net10.0`)
- **ASP.NET Core Web API** (controllers)
- **Entity Framework Core 10** (Code First + migrations)
- **SQL Server** (LocalDB)
- **OpenAPI** enabled in development

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download)
- SQL Server LocalDB (ships with Visual Studio)
- [`dotnet-ef`](https://learn.microsoft.com/ef/core/cli/dotnet) CLI tool (for running migrations outside Visual Studio):
  ```bash
  dotnet tool install --global dotnet-ef
  ```
- [Postman](https://www.postman.com/downloads/) for testing endpoints 

## Getting started

```bash
# 1. Restore dependencies
dotnet restore

# 2. Create / update the database from the migrations
dotnet ef database update

# 3. Run the API
dotnet run
```

The connection string lives in `appsettings.json` under `ConnectionStrings:StorageContext`
(defaults to `(localdb)\mssqllocaldb`, database `StorageContext`).

> In Visual Studio, the equivalents are `Add-Migration <Name>` and `Update-Database` in the Package Manager Console.

## Project structure

| Path | Purpose |
|------|---------|
| `Models/Product.cs` | The `Product` entity |
| `Data/StorageContext.cs` | The `DbContext` (`DbSet<Product> Product`) |
| `Controllers/ProductsController.cs` | Scaffolded CRUD controller (renamed from `StorageController`) |
| `Migrations/` | EF Core migrations (`Init`) |
| `Program.cs` | App startup + DI registration of `StorageContext` |

## Data model

`Product`:

| Field | Type |
|-------|------|
| `Id` | `int` |
| `Name` | `string` |
| `Price` | `int` |
| `Category` | `string` |
| `Shelf` | `string` |
| `Count` | `int` |
| `Description` | `string` |

## API endpoints

The controller class is `ProductsController`, so with `[Route("api/[controller]")]` the base route is **`/api/Products`**.

| Method | Route | Description |
|--------|-------|-------------|
| `GET` | `/api/Products` | Get all products |
| `GET` | `/api/Products/{id}` | Get one product by id |
| `POST` | `/api/Products` | Create a product (returns **201** + `Location`) |
| `PUT` | `/api/Products/{id}` | Update a product (**204**) |
| `DELETE` | `/api/Products/{id}` | Delete a product (**204**) |

---

## Reflection - After Scaffold

### 🇬🇧 English

**What the scaffold gave us:**

- A full CRUD controller: two `GetProduct` methods (all, and by id), plus `PostProduct`, `PutProduct`, `DeleteProduct` and a little `ProductExists` helper.
- `StorageContext` gets injected through the constructor (DI), and `_context.Product` is our `DbSet<Product>`. Everything goes through it: `ToListAsync()`, `FindAsync(id)`, `Add()`, `Remove()` and `SaveChangesAsync()`.
- The return types map straight onto HTTP status codes: `CreatedAtAction` gives back 201 Created with a `Location` header pointing at `GetProduct`, `NotFound()` is 404, `BadRequest()` is 400, and `NoContent()` is 204.

**Three things that tripped me up:**

1. The scaffold never actually calls `Ok()`. The GET just returns the entity and you get an implicit 200.
2. The controller came out as `StorageController`, not `ProductsController`, so the route was `/api/Storage`. I kept hitting `/api/products` in Postman and getting 404 before it clicked. Renaming the controller sorted it out.
3. `Product` still has plain `string` fields with no null handling or validation.

**The big takeaway:** EF Core plus the scaffold writes a huge amount of code for you (the whole CRUD layer and the database wiring, basically for free). And that is exactly why you have to read it. When something breaks, the "magic" is hard to debug if you have no idea what it generated.

**The setup was a little adventurous:** at one point Visual Studio just fell apart, only about a third of the menu items showed up, NuGet packages went missing, and I got all sorts of build errors. LLM helped a lot with putting it back together, and now everything works as expected apart from the odd crash. The one annoying thing: the Package Manager Console cmdlets (`Add-Migration`, `Update-Database`) are VS-specific, so I couldn't write the project in Rider, had to spin up VS in a Windows box.

### 🇸🇪 Svenska

**Vad scaffolden gav oss:**

- En komplett CRUD-controller: två `GetProduct` (alla / via id), `PostProduct`, `PutProduct`, `DeleteProduct` och hjälpmetoden `ProductExists`.
- `StorageContext` injiceras via konstruktorn (DI), och `_context.Product` är vår `DbSet<Product>`. All dataåtkomst går genom den: `ToListAsync()`, `FindAsync(id)`, `Add()`, `Remove()` och `SaveChangesAsync()`.
- Resultattyperna: `CreatedAtAction` ger 201 Created + en `Location`-header som pekar på `GetProduct`; `NotFound()` = 404, `BadRequest()` = 400, `NoContent()` = 204.

**Tre saker jag fastnade på:**

1. Scaffolden använder faktiskt inte `Ok()`. GET returnerar entiteten direkt (implicit 200).
2. Scaffolden döpte controllern till `StorageController`, inte `ProductsController`, så endpointen blev `/api/Storage`. Mina anrop mot `/api/products` i Postman gav 404 tills jag förstod varför. Jag döpte om controllern, och då stämde det.
3. `Product` har `string`-fält utan nullbarhet eller validering ännu.

**Viktigaste lärdomen:** EF Core + scaffold tar bort enormt mycket kod (hela CRUD + DB-kopplingen "gratis"), men det är just därför man måste läsa den. "Magin" är svår att felsöka om man inte vet vad som hänt.

**Det var lite äventyrligt,** eftersom Visual Studio gick sönder, bara en tredjedel av menyalternativen dök upp, NuGet-paket försvann och alla möjliga byggfel uppstod. LLM hjälpte mycket till med återställningen, och nu fungerar allt som förväntat, förutom enstaka krascher. Det enda irriterande: cmdlets för pakethanterarkonsolen (`Add-Migration`, `Update-Database`) är VS-specifika, så jag kunde inte skriva projektet i Rider, utan var tvungen att dra igång VS på en Windows-burk.
