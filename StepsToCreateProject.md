# Create MicroService Project from scratch

### 1. Begin project

- Create a new project --> Blank Solution --> Next
- Provide name for project --> JewelsOnContainers --> Create
- Right-click on Solution -->
  - Add --> New Solution Folder --> Assign name "src"
  - // these are virtual folders
- Right-click on src folder -->
  - Add --> New Solution Folder --> Assign name "services"
- Right-click on src folder -->
  - Add --> New Solution Folder --> Assign name "web"

### 2. Create first microservice

- Right-click on services folder --> Add --> New Project -->
- Select template -> ASP.NET Core Web API
  - "A project template for creating an ASP.NET Core Application with an example controller for a RESTful Http Service. This template can also be used ASP.NET Core MVC Views and Controllers"
  - // Kal's video shows slightly different options
- Next --> Assign name "ProductCatalogAPI --> Next -->
- Target Framework - .NET Core 3.1 (I guess)
- Authentication --> None
- Select option --> Configure for Https
- Select option --> Enable Docker Support
- Select Linux from DropDown
- --> Create

### 3. Delete demo code

- Delete WeatherForecastController.cs
- Delete WeatherForecast.cs

### 4. Create Domain (also known as Model)

- Right-click ProductCatalogAPI -> Add -> New Folder --> Domain
- Right-click Domain -> Add -> Class -->

  - Assign name CatalogItem.cs --> Add

- Add public properties // Kal's example

  - int Id
  - string Name
  - string Description
  - decimal Price
  - string PictureUrl
  - int CatalogTypeId
  - int CatalogBrandId
  - // add virtual properties -
  - public virtual CatalogType CatalogType
  - public virtual CatalogBrand CatalogBrand

- Right-click Domain --> Add --> Class -->
  - Assign name CatalogBrand.cs --> Add
- Add public properties -->

  - int Id
  - string Brand

- Right-click Domain --> Add --> Class -->
  - Assign name CatalogType.cs --> Add
- Add public properties -->
  - int Id
  - string Brand

### 5. Add EntityFrameworkCore

- Right-click on src/services/ProductCatalogAPI --> Manage NuGet Packages
- New window --> Click Browse tab
- Install libraries for this project

  - 1. Microsoft.EntityFrameworkCore
  - 2. Microsoft.EntityFrameworkCore.Relational
  - 3. Microsoft.EntityFrameworkCore.SqlServer
  - 4. Microsoft.EntityFrameworkCore.Tools

- Set version to match .NET version for this project. Versions must match.

### 6. Create Data - CatalogContext.cs

- Right-click src/services/ProductCatalogAPI
  - Add --> New Folder --> Data
- Right-click Data folder
  - Add --> Class -->
  - Assign name CatalogContext.cs
  - inherits from DbContext
- Dependency injection -->
  - Pass parameter to constructor --> DbContextOptions options
  - Pass options to base class -->
  - public CatalogContext(DbContextOptions options) : base(options)
- Add public properties -->
  - DbSet\<CatalogType\> CatalogTypes
  - DbSet\<CatalogBrand\> CatalogBrands
  - DbSet\<CatalogItem\> Catalog
- Override method OnModelCreating
  - protected override void OnModelCreating(ModelBuilder modelBuilder)
  - Create rules for creating Db - required properties, max length, etc
  - Assign foreign keys - .HasForeignKey(c => c.CatalogTypeId);
  - It's complicated, see CatalogContext.cs and virtual properties in CatalogItem.cs

### 7. Inject dependencies in Startup.cs

- constructor injects configuration file -
- public Startup(IConfiguration configuration)
  // Configure services in Startup.cs
- Services are added in ConfigureServices method
- public void ConfigureServices(IServiceCollection services)
- services.AddControllers();
- Add DbContext (soon we inject connection string here)
- services.AddDbContext\<CatalogContext\>(options => options.UseSqlServer(/\* \*/));

### 8. Obtain connection string from Db

- Main menu --> View --> SqlServerObjectExplorer --> Opens in sidebar
- Right-click (localdb)MSSQLLocalDb(SQL Server 13.04001 etc)
- Right-click on SqlServer --> Properties --> Properties Panel -->
- General --> Connection string --> Copy to clipboard

### 9. Store connection string in appsettings.json

- Add a key value pair at the top of appsettings.json
- Paste the connection string from your clipboard as the value -
- "ConnectionString": "Data Source=(localdb)\\ etc"
- Example of complete connection string -
- "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=CatalogDb;Integrated Security=True;Connect Timeout=30"
- Value of "Initial Catalog" should be name you want to assign to this Db
- The rest of the auto-generated connection string can be deleted.

### 10. Provide reference to connection string in Startup.ConfigureServices() UseSqlServer()

- services.AddDbContext/<CatalogContext\>(options => options.UseSqlServer(Configuration["ConnectionString"]));

### 11. Build Db migrations

- Main menu --> View --> Other Windows --> Package Manager Console -->
- Opens Powershell window inside VS
  - Check that Powershell/Package Manger is running in the correct project -
  - In the dropdown, select ProductCatalogAPI
  - At the prompt, enter this command -
  - PM> Add-Migration anyname

### 12. Create CatalogSeed.cs to run Db migrations and to seed data into Db

- Right-click Data folder --> Add --> Class -->
- Assign name CatalogSeed.cs
- Make it a static class -
  - public static class CatalogSeed
    Add a static method - Seed()
- public static void Seed(CatalogContext context)
- Add code to execute migrations -- context.Database.Migrate();
- For seeding data, follow the example code in CatalogSeed.cs Seed()
- Warning - Data is not saved until --> context.SaveChanges();

### 13. Modify Program.cs for correct timing of Build() and Run()

- Migrations and seeding can't happen until Db is fully built which takes some time
- So we need to detect when Db is built and only then run migrations and seeding
- Inside Program.cs Main(), CreateHostBuilder(args).Build is saved to a variable -
- var host = CreateHostBuilder(args).Build();
- Then set up a way to watch for CatalogContext to finish -
  - Get a scope of all services running (there are a lot)
  - Find the service of CatalogContext type
  - Set a timer to notify when CatalogContext is finished
  - When Build() is finished, execute Seed() which runs migrations and seeding
  - When Seed() is finished, execute Run()
  - See Kal's code in Program.cs

### 14. Run the project!

- Main menu --> Run with IIS Express (not Docker yet)
- --> Click green Run button
- Browser tries to open up localhost://44399/WeatherForecast
- Page not available but web host is running!

### 15. Check to see if Db tables were created

- Main menu --> View --> SqlServerObjectExplorer -->
- Click list arrow caret next to (localdb)\MSSQLLocalDB...
- Right-click Databases folder --> Refresh
- Look for CatalogDb folder --> Tables folder -->
- Should see new tables -
  - dbo.Catalog
  - dbo.Brands
  - dbo.Types
- Right click on dbo. file --> View Data
  - Takes awhile to load
  - Then you can see table structure and data
