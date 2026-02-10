# .NET Framework 4.8 to .NET 10.0 Migration - Execution Tasks

**Solution**: WebApiExample.sln  
**Branch**: upgrade-to-NET10-01  
**Strategy**: Bottom-Up (Dependency-First)  
**Target Framework**: .NET 10.0 (Long Term Support)

---

## Task Execution Dashboard

**Overall Progress**: 2/17 tasks complete (12%) ![12%](https://progress-bar.xyz/12)

### Tier Progress
- **Tier 1** (WebApiExample.Common): 1/3 tasks ???
- **Tier 2** (WebApiExample.DataStore): 0/4 tasks ????
- **Tier 3** (WebApiExample.WebApp): 0/7 tasks ???????
- **Tier 4** (WebApiExample.WebApp.Tests): 0/3 tasks ???

**Legend**: ? Not Started | ? In Progress | ? Complete | ? Failed | ? Skipped

---

## Prerequisites Validation

### [?] TASK-000: Validate Prerequisites *(Completed: 2026-02-09 20:50)*
**Estimated Time**: 15-30 minutes  
**Tier**: Pre-Migration

**Actions**:
- [?] (1) Verify .NET 10.0 SDK installed
  - Run: `dotnet --version`
  - Expected: Version 10.0.x or higher
  - If not installed: Download from https://dotnet.microsoft.com/download/dotnet/10.0
  
- [?] (2) Verify current branch
  - Run: `git branch --show-current`
  - Expected: `upgrade-to-NET10-01`
  - If wrong branch: Checkout correct branch
  
- [?] (3) Verify no pending changes
  - Run: `git status`
  - Expected: Clean working directory
  - If changes exist: Commit or stash them
  
- [?] (4) Verify solution builds on .NET Framework 4.8
  - Run: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Expected: Build succeeds (establish baseline)
  - Document: Any existing warnings or errors
  
- [?] (5) Install try-convert tool (if not already installed)
  - Run: `dotnet tool install -g try-convert`
  - Or update: `dotnet tool update -g try-convert`

**Validation**:
- ? .NET 10.0 SDK available
- ? On correct branch (upgrade-to-NET10-01)
- ? Clean working directory
- ? Baseline build successful
- ? try-convert tool ready

**On Failure**: Resolve prerequisites before proceeding to Tier 1

---

## Tier 1: WebApiExample.Common

**Objective**: Migrate foundation library to .NET 10.0, establish SDK-style pattern  
**Expected Duration**: 1-2 hours  
**Risk Level**: ?? Low

### [?] TASK-001: Convert WebApiExample.Common to SDK-Style Project *(Completed: 2026-02-09 21:07)*
**Estimated Time**: 15-30 minutes

**Actions**:
- [?] (1) Backup current project file
  - Copy: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj` ? `WebApiExample.Common.csproj.bak`
  
- [?] (2) Run try-convert
  - Command: `try-convert C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj`
  - Review: Conversion output and warnings
  
- [?] (3) Verify SDK-style project file created
  - Open: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj`
  - Check: Contains `<Project Sdk="Microsoft.NET.Sdk">`
  - Check: No `packages.config` file remains
  
- [?] (4) Verify all source files included
  - Build project: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj`
  - Check: All .cs files compiled (SDK-style includes by convention)

**Validation**:
- ? Project file is SDK-style format
- ? All source files included
- ? No packages.config file

**On Failure**: Restore from backup, attempt manual SDK-style conversion

---

### [ ] TASK-002: Update WebApiExample.Common Target Framework to .NET 10.0
**Estimated Time**: 5-10 minutes

**Actions**:
- [ ] (1) Update target framework in .csproj
  - Edit: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj`
  - Change: `<TargetFramework>net48</TargetFramework>` ? `<TargetFramework>net10.0</TargetFramework>`
  
- [ ] (2) Review AssemblyInfo.cs for duplicates
  - Check: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\Properties\AssemblyInfo.cs`
  - Remove: Duplicate attributes now in .csproj (AssemblyVersion, AssemblyFileVersion, etc.)
  - Keep: Custom attributes not supported in .csproj

**Validation**:
- ? TargetFramework is net10.0
- ? No duplicate AssemblyInfo attributes

---

### [ ] TASK-003: Validate Tier 1 Build and Integration
**Estimated Time**: 30-45 minutes

**Actions**:
- [ ] (1) Build WebApiExample.Common on .NET 10.0
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.Common\WebApiExample.Common.csproj`
  - Expected: Build succeeds with 0 errors, 0 warnings
  
- [ ] (2) Verify public API unchanged
  - Review: No breaking changes introduced
  - Check: Public classes, methods, properties accessible
  
- [ ] (3) Test consumer compatibility (higher tiers still on net48)
  - Build Tier 2 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  - Build Tier 3 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Build Tier 4 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Expected: All higher tiers still build successfully
  
- [ ] (4) Commit Tier 1 changes
  - Command:
    ```bash
    git add WebApiExample.Common\
    git commit -m "Tier 1: Migrate WebApiExample.Common to .NET 10.0
    
    - Convert to SDK-style project
    - Update target framework to net10.0
    - Validation: Builds successfully, no breaking changes"
    ```

**Validation**:
- ? WebApiExample.Common builds on .NET 10.0 with no errors/warnings
- ? No breaking API changes
- ? Higher tiers (still net48) can still build
- ? Changes committed to upgrade-to-NET10-01 branch

**Completion Criteria**: All Tier 1 validation checklist items complete  
**Next**: Proceed to Tier 2

---

## Tier 2: WebApiExample.DataStore

**Objective**: Migrate data access layer, validate Entity Framework 6.5.1 compatibility  
**Expected Duration**: 2-4 hours  
**Risk Level**: ?? Low

### [ ] TASK-004: Convert WebApiExample.DataStore to SDK-Style Project
**Estimated Time**: 15-30 minutes

**Actions**:
- [ ] (1) Backup current project file
  - Copy: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj` ? `WebApiExample.DataStore.csproj.bak`
  
- [ ] (2) Run try-convert
  - Command: `try-convert C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  
- [ ] (3) Verify SDK-style project file
  - Check: Contains `<Project Sdk="Microsoft.NET.Sdk">`
  - Check: PackageReference for EntityFramework
  - Check: ProjectReference to WebApiExample.Common preserved
  - Check: No packages.config file

**Validation**:
- ? Project file is SDK-style format
- ? Package references migrated
- ? Project reference to Common preserved

---

### [ ] TASK-005: Update WebApiExample.DataStore Target Framework and Packages
**Estimated Time**: 10-20 minutes

**Actions**:
- [ ] (1) Update target framework in .csproj
  - Edit: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  - Change: `<TargetFramework>net48</TargetFramework>` ? `<TargetFramework>net10.0</TargetFramework>`
  
- [ ] (2) Update EntityFramework package
  - Command: `dotnet add C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj package EntityFramework --version 6.5.1`
  - Or edit .csproj: `<PackageReference Include="EntityFramework" Version="6.5.1" />`
  
- [ ] (3) Restore packages
  - Command: `dotnet restore C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`

**Validation**:
- ? TargetFramework is net10.0
- ? EntityFramework 6.5.1 installed
- ? Package restore successful

---

### [ ] TASK-006: Verify Entity Framework 6.5.1 Compatibility
**Estimated Time**: 30-60 minutes

**Actions**:
- [ ] (1) Build project
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  - Expected: Build succeeds
  
- [ ] (2) Check for App.config/Web.config EF configuration
  - Review: Connection strings and EF configuration sections
  - Note: May need to migrate to code-based config later (Tier 3)
  
- [ ] (3) List EF migrations (if applicable)
  - Command: `dotnet ef migrations list --project C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  - Expected: Existing migrations listed without errors
  - If errors: Document for resolution
  
- [ ] (4) Test database connection (if database available)
  - Run smoke test: Verify DbContext can initialize
  - Check: Database.Exists() or similar check
  - Note: Full data access testing happens in Tier 3 integration

**Validation**:
- ? Project builds successfully
- ? EF migrations listed (if applicable)
- ? No EF compatibility errors

**On Issues**: Document EF compatibility problems; may need to remain on 6.4.4 or consider EF Core migration

---

### [ ] TASK-007: Validate Tier 2 Build and Integration
**Estimated Time**: 30-45 minutes

**Actions**:
- [ ] (1) Build WebApiExample.DataStore on .NET 10.0
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.DataStore\WebApiExample.DataStore.csproj`
  - Expected: Build succeeds with 0 errors, 0 warnings
  
- [ ] (2) Verify integration with Tier 1
  - Check: References WebApiExample.Common (net10.0) successfully
  - Check: No dependency conflicts
  
- [ ] (3) Test consumer compatibility (higher tiers still on net48)
  - Build Tier 3 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Build Tier 4 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Expected: Higher tiers still build successfully
  
- [ ] (4) Commit Tier 2 changes
  - Command:
    ```bash
    git add WebApiExample.DataStore\
    git commit -m "Tier 2: Migrate WebApiExample.DataStore to .NET 10.0
    
    - Convert to SDK-style project
    - Update target framework to net10.0
    - Upgrade EntityFramework 6.4.4 ? 6.5.1
    - Validation: Builds successfully, EF working, integrates with Tier 1"
    ```

**Validation**:
- ? WebApiExample.DataStore builds on .NET 10.0 with no errors/warnings
- ? EntityFramework 6.5.1 working
- ? Integration with Tier 1 successful
- ? Higher tiers (still net48) can still build
- ? Changes committed to upgrade-to-NET10-01 branch

**Completion Criteria**: All Tier 2 validation checklist items complete  
**Next**: Proceed to Tier 3

---

## Tier 3: WebApiExample.WebApp

**Objective**: Convert ASP.NET Web API to ASP.NET Core, modernize architecture  
**Expected Duration**: 16-32 hours  
**Risk Level**: ?? High

**?? CRITICAL**: This tier has the highest complexity. Break into daily milestones and validate frequently.

### [ ] TASK-008: Convert WebApiExample.WebApp to SDK-Style Web Project
**Estimated Time**: 30-90 minutes

**Actions**:
- [ ] (1) Backup entire WebApp project
  - Copy folder: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\` ? `WebApiExample.WebApp.backup\`
  
- [ ] (2) Run try-convert with web conversion flag
  - Command: `try-convert C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj --force-web-conversion`
  - Note: May require manual adjustments
  
- [ ] (3) Update project SDK to Microsoft.NET.Sdk.Web
  - Edit: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Ensure: `<Project Sdk="Microsoft.NET.Sdk.Web">`
  
- [ ] (4) Update target framework
  - Change: `<TargetFramework>net48</TargetFramework>` ? `<TargetFramework>net10.0</TargetFramework>`
  
- [ ] (5) Verify project references preserved
  - Check: ProjectReference to WebApiExample.Common
  - Check: ProjectReference to WebApiExample.DataStore

**Validation**:
- ? Project file is SDK-style Web format (Microsoft.NET.Sdk.Web)
- ? TargetFramework is net10.0
- ? Project references preserved

---

### [ ] TASK-009: Update Packages - Remove Incompatible ASP.NET Framework Packages
**Estimated Time**: 20-30 minutes

**Actions**:
- [ ] (1) Remove incompatible Microsoft.AspNet.* packages (8 packages)
  - Command:
    ```bash
    cd C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp
    dotnet remove package Microsoft.AspNet.Mvc
    dotnet remove package Microsoft.AspNet.Razor
    dotnet remove package Microsoft.AspNet.WebApi
    dotnet remove package Microsoft.AspNet.WebApi.Core
    dotnet remove package Microsoft.AspNet.WebApi.WebHost
    dotnet remove package Microsoft.AspNet.WebPages
    dotnet remove package Microsoft.CodeDom.Providers.DotNetCompilerPlatform
    dotnet remove package Microsoft.Web.Infrastructure
    ```
  
- [ ] (2) Remove incompatible bundling and DI packages
  - Command:
    ```bash
    dotnet remove package Microsoft.AspNet.Web.Optimization
    dotnet remove package Unity.WebAPI
    ```
  
- [ ] (3) Verify packages removed
  - Check .csproj: No PackageReference entries for removed packages

**Validation**:
- ? 10 incompatible packages removed from .csproj

---

### [ ] TASK-010: Update Packages - Add/Update Required Packages
**Estimated Time**: 20-30 minutes

**Actions**:
- [ ] (1) Update security vulnerability packages
  - Command:
    ```bash
    cd C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp
    dotnet add package Newtonsoft.Json --version 13.0.4
    ```
  - Note: bootstrap and jQuery are client-side, updated separately
  
- [ ] (2) Add ASP.NET Core Newtonsoft.Json support
  - Command: `dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson --version 10.0.0`
  
- [ ] (3) Update EntityFramework
  - Command: `dotnet add package EntityFramework --version 6.5.1`
  
- [ ] (4) Update System.Runtime.CompilerServices.Unsafe
  - Command: `dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2`
  
- [ ] (5) Restore packages
  - Command: `dotnet restore`
  
- [ ] (6) Run security scan
  - Command: `dotnet list package --vulnerable`
  - Expected: No vulnerabilities for Newtonsoft.Json

**Validation**:
- ? Newtonsoft.Json 13.0.4 installed
- ? Microsoft.AspNetCore.Mvc.NewtonsoftJson 10.0.0 installed
- ? EntityFramework 6.5.1 installed
- ? No security vulnerabilities detected

---

### [ ] TASK-011: Create Program.cs and Migrate Application Startup
**Estimated Time**: 60-120 minutes

**Actions**:
- [ ] (1) Create Program.cs in project root
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Program.cs`
  - Content template:
    ```csharp
    var builder = WebApplication.CreateBuilder(args);
    
    // Add services to the container
    builder.Services.AddControllers()
        .AddNewtonsoftJson(); // For Newtonsoft.Json compatibility
    
    // TODO: Add Entity Framework DbContext
    // builder.Services.AddDbContext<YourDbContext>(options =>
    //     options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    
    // TODO: Migrate Unity DI registrations
    // builder.Services.AddScoped<IService, ServiceImpl>();
    
    var app = builder.Build();
    
    // Configure middleware pipeline
    if (app.Environment.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    
    app.UseHttpsRedirection();
    app.UseStaticFiles(); // For wwwroot content
    app.UseRouting();
    app.UseAuthorization();
    app.MapControllers();
    
    app.Run();
    
    // Make Program class visible to tests
    public partial class Program { }
    ```
  
- [ ] (2) Review Global.asax.cs for application startup logic
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Global.asax.cs`
  - Identify: Application_Start, Application_End, and other lifecycle methods
  - Document: Logic to migrate to Program.cs
  
- [ ] (3) Migrate Application_Start logic to Program.cs
  - Move: WebApiConfig.Register calls ? attribute routing (handled by MapControllers)
  - Move: UnityConfig registrations ? builder.Services.AddXxx()
  - Move: Other initialization code ? appropriate locations in Program.cs
  
- [ ] (4) Remove App_Start folder
  - Delete: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\App_Start\` folder
  - Includes: BundleConfig.cs, WebApiConfig.cs, UnityConfig.cs (if present)
  
- [ ] (5) Remove Global.asax files
  - Delete: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Global.asax`
  - Delete: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Global.asax.cs`

**Validation**:
- ? Program.cs created with ASP.NET Core startup
- ? Application_Start logic migrated
- ? App_Start folder removed
- ? Global.asax files removed

---

### [ ] TASK-012: Create appsettings.json and Migrate Configuration
**Estimated Time**: 30-60 minutes

**Actions**:
- [ ] (1) Create appsettings.json in project root
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\appsettings.json`
  - Content template:
    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "ConnectionStrings": {
        "DefaultConnection": "TODO: Copy from Web.config"
      },
      "AppSettings": {
        "TODO": "Copy settings from Web.config appSettings section"
      },
      "AllowedHosts": "*"
    }
    ```
  
- [ ] (2) Review Web.config
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Web.config`
  - Extract: appSettings keys and values
  - Extract: connectionStrings
  
- [ ] (3) Migrate appSettings to appsettings.json
  - Copy settings from Web.config `<appSettings>` section
  - Structure as JSON in "AppSettings" section
  
- [ ] (4) Migrate connectionStrings to appsettings.json
  - Copy connection strings from Web.config
  - Add to "ConnectionStrings" section
  
- [ ] (5) Create appsettings.Development.json (optional)
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\appsettings.Development.json`
  - Content: Development-specific overrides
  
- [ ] (6) Update .gitignore (if needed)
  - Ensure: appsettings.Development.json ignored if contains secrets
  - Add: User secrets or environment variables for sensitive data

**Validation**:
- ? appsettings.json created
- ? All Web.config settings migrated
- ? Connection strings migrated
- ? No sensitive data in committed files

---

### [ ] TASK-013: Migrate Dependency Injection from Unity to Built-in DI
**Estimated Time**: 60-120 minutes

**Actions**:
- [ ] (1) Review Unity configuration
  - File: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\App_Start\UnityConfig.cs` (if exists)
  - Document: All container.RegisterType calls
  - Document: Service lifetimes (PerResolve ? Scoped, SingleInstance ? Singleton, etc.)
  
- [ ] (2) Migrate service registrations to Program.cs
  - In Program.cs, add service registrations:
    ```csharp
    // Example migrations:
    // Unity: container.RegisterType<IService, ServiceImpl>()
    // Built-in DI: builder.Services.AddScoped<IService, ServiceImpl>();
    
    // Unity: container.RegisterType<IOtherService, OtherServiceImpl>(new ContainerControlledLifetimeManager())
    // Built-in DI: builder.Services.AddSingleton<IOtherService, OtherServiceImpl>();
    ```
  
- [ ] (3) Update service lifetimes
  - PerResolve ? AddScoped
  - SingleInstance / ContainerControlledLifetime ? AddSingleton
  - Transient ? AddTransient
  
- [ ] (4) Register Entity Framework DbContext (if applicable)
  - Add to Program.cs:
    ```csharp
    builder.Services.AddDbContext<YourDbContext>(options =>
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    ```

**Validation**:
- ? All Unity registrations migrated to builder.Services
- ? Service lifetimes correctly mapped
- ? DbContext registered (if applicable)

---

### [ ] TASK-014: Update Controllers for ASP.NET Core
**Estimated Time**: 60-120 minutes

**Actions**:
- [ ] (1) Find all controller files
  - Location: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\Controllers\`
  - List: All *Controller.cs files
  
- [ ] (2) For each controller, update:
  
  **Base Class**:
  - Change: `public class XController : ApiController`
  - To: `public class XController : ControllerBase`
  
  **Add Attributes**:
  - Add to class: `[ApiController]`
  - Add to class: `[Route("api/[controller]")]`
  
  **Update Return Types**:
  - Change: `IHttpActionResult` ? `IActionResult`
  
  **Add HTTP Method Attributes**:
  - Add: `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpDelete]` to action methods
  
  **Update Result Creation**:
  - `Ok(value)` ? `Ok(value)` (same)
  - `Created(location, value)` ? `CreatedAtAction(actionName, routeValues, value)`
  - `BadRequest()` ? `BadRequest()` (same)
  - `NotFound()` ? `NotFound()` (same)
  
  **Update Namespaces**:
  - Remove: `using System.Web.Http;`
  - Add: `using Microsoft.AspNetCore.Mvc;`
  
- [ ] (3) Example controller conversion:
  ```csharp
  // BEFORE:
  public class ValuesController : ApiController
  {
      public IHttpActionResult Get()
      {
          return Ok(new[] { "value1", "value2" });
      }
  }
  
  // AFTER:
  [ApiController]
  [Route("api/[controller]")]
  public class ValuesController : ControllerBase
  {
      [HttpGet]
      public IActionResult Get()
      {
          return Ok(new[] { "value1", "value2" });
      }
  }
  ```

**Validation**:
- ? All controllers inherit from ControllerBase
- ? [ApiController] and [Route] attributes added
- ? Return types updated to IActionResult
- ? HTTP method attributes added
- ? Namespaces updated

---

### [ ] TASK-015: Update Static Content and Remove Bundling
**Estimated Time**: 45-90 minutes

**Actions**:
- [ ] (1) Create wwwroot folder
  - Location: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\wwwroot\`
  - Subfolders: `wwwroot\lib\`, `wwwroot\css\`, `wwwroot\js\`
  
- [ ] (2) Move static content to wwwroot
  - Move: `Scripts\` folder content ? `wwwroot\js\` or `wwwroot\lib\`
  - Move: `Content\` folder content ? `wwwroot\css\`
  - Move: Images, fonts, etc. ? `wwwroot\`
  
- [ ] (3) Update client-side libraries (security fixes)
  - **Bootstrap**: Update references from 3.3.7 to 5.3.8
    - Download: Bootstrap 5.3.8 from https://getbootstrap.com/
    - Place in: `wwwroot\lib\bootstrap\`
  - **jQuery**: Update from 3.3.1 to 3.7.1
    - Download: jQuery 3.7.1 from https://jquery.com/download/
    - Place in: `wwwroot\lib\jquery\`
  
- [ ] (4) Remove BundleConfig.cs
  - Already removed in TASK-011 (App_Start folder deletion)
  
- [ ] (5) Update HTML/Razor views (if applicable)
  - Find: All .cshtml files
  - Replace: `@Scripts.Render("~/bundles/jquery")` 
  - With: `<script src="~/lib/jquery/dist/jquery.min.js"></script>`
  - Replace: `@Styles.Render("~/Content/css")`
  - With: `<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />`
  
- [ ] (6) Update Bootstrap 3 to Bootstrap 5 classes (if HTML/views exist)
  - **Common changes**:
    - `.col-xs-*` ? `.col-*`
    - `.btn-default` ? `.btn-secondary`
    - `.pull-left` ? `.float-start`
    - `.pull-right` ? `.float-end`
    - `.panel` ? `.card`
  - Review: All HTML/Razor files for Bootstrap usage
  - Test: UI functionality after class updates

**Validation**:
- ? wwwroot folder created with static content
- ? Bootstrap 5.3.8 installed
- ? jQuery 3.7.1 installed
- ? Bundling references removed
- ? Static file references updated
- ? Bootstrap class updates applied (if applicable)

---

### [ ] TASK-016: Build and Fix Compilation Errors (Tier 3)
**Estimated Time**: 60-120 minutes

**Actions**:
- [ ] (1) Attempt build
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Expected: Compilation errors (normal at this stage)
  
- [ ] (2) Fix namespace errors
  - Find: `using System.Web.Http;` references
  - Replace: With `using Microsoft.AspNetCore.Mvc;`
  - Find: `using System.Net.Http.Formatting;`
  - Remove: Or replace with ASP.NET Core equivalents
  
- [ ] (3) Fix HttpContext.Current usage (if any)
  - Old: `HttpContext.Current.Request`
  - New: Inject `IHttpContextAccessor` and use `_httpContextAccessor.HttpContext.Request`
  
- [ ] (4) Fix configuration access
  - Old: `ConfigurationManager.AppSettings["key"]`
  - New: `_configuration["AppSettings:key"]` (inject IConfiguration)
  
- [ ] (5) Address other compilation errors
  - Use: Visual Studio Quick Actions for suggested fixes
  - Consult: .NET 10.0 breaking changes documentation if needed
  - Document: Unexpected errors for team awareness
  
- [ ] (6) Rebuild until successful
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Expected: Build succeeds with 0 errors
  - Accept: Warnings (review and document)

**Validation**:
- ? WebApiExample.WebApp builds successfully
- ? All compilation errors resolved
- ? Build warnings reviewed and documented

**On Persistent Errors**: Document blockers, seek team input, consider alternative approaches

---

### [ ] TASK-017: Test and Validate Tier 3 Application
**Estimated Time**: 90-180 minutes

**Actions**:
- [ ] (1) Run application
  - Command: `dotnet run --project C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Expected: Application starts without errors
  - Check: Console output for startup errors
  - Note: Port application is listening on (default: 5000/5001)
  
- [ ] (2) Test API endpoints
  - Use: Browser, Postman, or curl
  - Test: GET /api/[controller] for each controller
  - Test: POST, PUT, DELETE operations (if applicable)
  - Verify: Correct HTTP status codes (200, 201, 204, 404, etc.)
  - Verify: Response payloads are correct JSON
  
- [ ] (3) Test database connectivity (if applicable)
  - Check: Application can connect to database
  - Check: Entity Framework queries execute
  - Check: CRUD operations work
  
- [ ] (4) Verify dependency injection
  - Check: No DI resolution errors in console
  - Check: All services resolve correctly
  - Test: Controllers receive injected services
  
- [ ] (5) Test static file serving
  - Access: http://localhost:5000/index.html (if exists)
  - Check: CSS files load (Bootstrap styles applied)
  - Check: JavaScript files load (jQuery functional)
  - Check: Images and other static content accessible
  
- [ ] (6) Run security scan
  - Command: `dotnet list package --vulnerable --project C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Expected: No vulnerabilities
  - Verify: bootstrap 5.3.8, jQuery 3.7.1, Newtonsoft.Json 13.0.4 (secure versions)
  
- [ ] (7) Test consumer compatibility
  - Build Tier 4 (still net48): `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Expected: Tests project still builds (even though WebApp is now ASP.NET Core)
  
- [ ] (8) Document issues found
  - List: Any functional issues, bugs, or regressions
  - Prioritize: Critical vs. nice-to-have fixes
  - Fix: Critical issues before committing

**Validation**:
- ? Application starts successfully
- ? All API endpoints functional
- ? Database connectivity works (if applicable)
- ? Dependency injection works
- ? Static content serves correctly
- ? No security vulnerabilities
- ? No critical bugs or regressions

**On Failure**: Do NOT commit. Fix issues, re-test, then proceed.

---

### [ ] TASK-018: Commit Tier 3 Changes
**Estimated Time**: 15-20 minutes

**Actions**:
- [ ] (1) Review all changes
  - Command: `git status`
  - Command: `git diff`
  - Verify: All expected files changed
  
- [ ] (2) Stage changes
  - Command:
    ```bash
    git add WebApiExample.WebApp\
    ```
  
- [ ] (3) Commit with detailed message
  - Command:
    ```bash
    git commit -m "Tier 3: Migrate WebApiExample.WebApp to ASP.NET Core / .NET 10.0
    
    - Convert to SDK-style Web project (Microsoft.NET.Sdk.Web)
    - Update target framework to net10.0
    - ASP.NET Framework ? ASP.NET Core migration:
      * Replace Global.asax with Program.cs
      * Migrate Web.config to appsettings.json
      * Update controllers (ApiController ? ControllerBase + [ApiController])
      * Replace Unity with built-in DI
      * Remove bundling/minification, use static references
    - Package updates:
      * Remove 10 incompatible ASP.NET packages
      * Upgrade security vulnerabilities: bootstrap 5.3.8, jQuery 3.7.1, Newtonsoft.Json 13.0.4
      * Upgrade EntityFramework to 6.5.1
    - Validation: Application runs, all endpoints functional, no security vulnerabilities"
    ```

**Validation**:
- ? All Tier 3 changes committed
- ? Commit message is descriptive
- ? Ready to proceed to Tier 4

**Completion Criteria**: All Tier 3 validation checklist items complete  
**Next**: Proceed to Tier 4

---

## Tier 4: WebApiExample.WebApp.Tests

**Objective**: Migrate test project, validate entire solution  
**Expected Duration**: 4-8 hours  
**Risk Level**: ?? Medium

### [ ] TASK-019: Convert WebApiExample.WebApp.Tests to SDK-Style Project and Update Packages
**Estimated Time**: 30-45 minutes

**Actions**:
- [ ] (1) Backup test project
  - Copy: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj` ? `WebApiExample.WebApp.Tests.csproj.bak`
  
- [ ] (2) Run try-convert
  - Command: `try-convert C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  
- [ ] (3) Update project SDK and target framework
  - Edit: `C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Ensure: `<Project Sdk="Microsoft.NET.Sdk">`
  - Change: `<TargetFramework>net48</TargetFramework>` ? `<TargetFramework>net10.0</TargetFramework>`
  - Add: `<IsTestProject>true</IsTestProject>`
  
- [ ] (4) Remove incompatible packages (11 packages)
  - Command:
    ```bash
    cd C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests
    dotnet remove package Microsoft.AspNet.Mvc
    dotnet remove package Microsoft.AspNet.Razor
    dotnet remove package Microsoft.AspNet.WebApi.Core
    dotnet remove package Microsoft.AspNet.WebApi.WebHost
    dotnet remove package Microsoft.AspNet.WebPages
    dotnet remove package Microsoft.Web.Infrastructure
    dotnet remove package System.Buffers
    dotnet remove package System.Memory
    dotnet remove package System.Numerics.Vectors
    dotnet remove package System.Runtime.InteropServices.RuntimeInformation
    dotnet remove package System.Threading.Tasks.Extensions
    ```
  
- [ ] (5) Update packages
  - Command:
    ```bash
    dotnet add package Newtonsoft.Json --version 13.0.4
    dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2
    ```
  
- [ ] (6) Add ASP.NET Core testing package
  - Command: `dotnet add package Microsoft.AspNetCore.Mvc.Testing --version 10.0.0`
  
- [ ] (7) Restore packages
  - Command: `dotnet restore`

**Validation**:
- ? Project is SDK-style with TargetFramework net10.0
- ? IsTestProject property set
- ? 11 incompatible packages removed
- ? Newtonsoft.Json 13.0.4 installed
- ? Microsoft.AspNetCore.Mvc.Testing 10.0.0 installed

---

### [ ] TASK-020: Update Tests for ASP.NET Core
**Estimated Time**: 90-180 minutes

**Actions**:
- [ ] (1) Update test base classes (if any)
  - Find: Test base classes or helper classes
  - Update: For WebApplicationFactory pattern
  - Example:
    ```csharp
    public class ControllerTestBase : IClassFixture<WebApplicationFactory<Program>>
    {
        protected readonly WebApplicationFactory<Program> Factory;
        
        public ControllerTestBase(WebApplicationFactory<Program> factory)
        {
            Factory = factory;
        }
    }
    ```
  
- [ ] (2) Update unit tests (controller tests with mocks)
  - Find: Tests that manually instantiate controllers
  - Update result type assertions:
    - `OkNegotiatedContentResult<T>` ? `OkObjectResult`
    - Access value via `.Value` property instead of `.Content`
  - Example:
    ```csharp
    // BEFORE:
    var result = controller.Get() as OkNegotiatedContentResult<List<Product>>;
    result.Content.Count.ShouldBe(1);
    
    // AFTER:
    var result = controller.Get() as OkObjectResult;
    (result.Value as List<Product>).Count.ShouldBe(1);
    ```
  
- [ ] (3) Update integration tests (if any)
  - Replace: OWIN TestServer with WebApplicationFactory
  - Example:
    ```csharp
    // BEFORE:
    using (var server = TestServer.Create<Startup>())
    {
        var response = await server.HttpClient.GetAsync("/api/products");
    }
    
    // AFTER:
    public class ProductsIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
    {
        private readonly WebApplicationFactory<Program> _factory;
        
        public ProductsIntegrationTests(WebApplicationFactory<Program> factory)
        {
            _factory = factory;
        }
        
        [Fact]
        public async Task Get_ReturnsProducts()
        {
            var client = _factory.CreateClient();
            var response = await client.GetAsync("/api/products");
            // assertions
        }
    }
    ```
  
- [ ] (4) Update namespaces
  - Remove: `using System.Web.Http;`, `using System.Web.Http.Results;`
  - Add: `using Microsoft.AspNetCore.Mvc;`, `using Microsoft.AspNetCore.Mvc.Testing;`
  
- [ ] (5) Build and fix compilation errors
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Fix: Any namespace or type errors

**Validation**:
- ? Test base classes updated (if applicable)
- ? Unit test assertions updated
- ? Integration tests migrated to WebApplicationFactory
- ? Namespaces updated
- ? Project builds successfully

---

### [ ] TASK-021: Run Tests and Validate Tier 4
**Estimated Time**: 60-120 minutes

**Actions**:
- [ ] (1) Run all tests
  - Command: `dotnet test C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`
  - Expected: Tests discovered and executed
  - Note: Pass/fail count for comparison to baseline
  
- [ ] (2) Fix failing tests
  - Review: Test failures and errors
  - Categorize:
    - ASP.NET Core hosting changes
    - Result type differences
    - Test infrastructure issues
    - Actual bugs in migrated code
  - Fix: Each failure systematically
  - Re-run: After each fix
  
- [ ] (3) Verify test count matches baseline
  - Compare: Current test count to pre-migration baseline
  - Check: No tests accidentally removed or disabled
  
- [ ] (4) Run code coverage (optional)
  - Command: `dotnet test --collect:"XPlat Code Coverage"`
  - Compare: To baseline coverage percentage
  
- [ ] (5) Test full solution
  - Command: `dotnet test C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Expected: All tests pass across entire solution
  
- [ ] (6) Build full solution
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Expected: All 4 projects build successfully
  
- [ ] (7) Commit Tier 4 changes
  - Command:
    ```bash
    git add WebApiExample.WebApp.Tests\
    git add WebApiExample.WebApp\Program.cs  # If added 'public partial class Program'
    git commit -m "Tier 4: Migrate WebApiExample.WebApp.Tests to .NET 10.0
    
    - Convert to SDK-style test project
    - Update target framework to net10.0
    - Remove 11 incompatible packages
    - Upgrade Newtonsoft.Json to 13.0.4
    - Add Microsoft.AspNetCore.Mvc.Testing for ASP.NET Core testing
    - Update tests: WebApplicationFactory, OkObjectResult assertions
    - Add 'public partial class Program { }' to WebApp for test access
    - Validation: All tests pass, full solution migrated to .NET 10.0"
    ```

**Validation**:
- ? All tests discovered and execute on .NET 10.0
- ? All tests pass (or failures explained and documented)
- ? Test count matches baseline
- ? Full solution builds successfully
- ? Full solution tests pass
- ? Changes committed to upgrade-to-NET10-01 branch

**Completion Criteria**: All Tier 4 validation checklist items complete  
**Result**: **Migration Complete** - All 4 tiers migrated to .NET 10.0

---

## Post-Migration Validation

### [ ] TASK-022: Final Solution Validation
**Estimated Time**: 60-90 minutes

**Actions**:
- [ ] (1) Clean and rebuild entire solution
  - Command: `dotnet clean C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Command: `dotnet build C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Expected: Build succeeds, 0 errors, warnings reviewed
  
- [ ] (2) Run all tests
  - Command: `dotnet test C:\Repos\Demo2\net48-web-api-example\WebApiExample.sln`
  - Expected: All tests pass
  
- [ ] (3) Security scan for entire solution
  - Command: `dotnet list package --vulnerable`
  - Expected: No vulnerabilities in any project
  
- [ ] (4) Verify all target frameworks
  - Check each .csproj:
    - WebApiExample.Common: net10.0 ?
    - WebApiExample.DataStore: net10.0 ?
    - WebApiExample.WebApp: net10.0 ?
    - WebApiExample.WebApp.Tests: net10.0 ?
  
- [ ] (5) Test application end-to-end
  - Start: `dotnet run --project C:\Repos\Demo2\net48-web-api-example\WebApiExample.WebApp\WebApiExample.WebApp.csproj`
  - Test: All API endpoints
  - Test: Database operations
  - Test: Static content
  - Verify: No console errors
  
- [ ] (6) Performance smoke test (optional)
  - Measure: Response times for key endpoints
  - Compare: To .NET Framework 4.8 baseline (if available)
  - Expected: Equal or better performance

**Validation**:
- ? Full solution builds successfully
- ? All tests pass
- ? No security vulnerabilities
- ? All projects target net10.0
- ? Application functional
- ? Performance acceptable

---

### [ ] TASK-023: Update Documentation
**Estimated Time**: 30-45 minutes

**Actions**:
- [ ] (1) Update README.md
  - File: `C:\Repos\Demo2\net48-web-api-example\README.md`
  - Update: Build instructions for .NET 10.0
  - Update: Prerequisites (.NET 10.0 SDK required)
  - Update: Any deployment instructions
  
- [ ] (2) Create/Update CHANGELOG.md
  - File: `C:\Repos\Demo2\net48-web-api-example\CHANGELOG.md`
  - Add entry:
    ```markdown
    ## [2.0.0] - YYYY-MM-DD
    ### Changed
    - Migrated from .NET Framework 4.8 to .NET 10.0
    - Converted ASP.NET Web API to ASP.NET Core
    - Updated all projects to SDK-style format
    - Upgraded Entity Framework to 6.5.1
    - Replaced Unity DI with built-in ASP.NET Core DI
    - Updated Bootstrap from 3.3.7 to 5.3.8
    - Updated jQuery from 3.3.1 to 3.7.1
    - Updated Newtonsoft.Json from 11.0.1 to 13.0.4 (security fix)
    
    ### Removed
    - Global.asax application startup
    - Web.config configuration
    - ASP.NET bundling and minification
    - 21 incompatible .NET Framework packages
    
    ### Added
    - Program.cs for ASP.NET Core startup
    - appsettings.json for configuration
    - Built-in dependency injection
    - WebApplicationFactory for testing
    ```
  
- [ ] (3) Commit documentation updates
  - Command:
    ```bash
    git add README.md CHANGELOG.md .github\upgrades\
    git commit -m "docs: Update documentation for .NET 10.0 migration
    
    - Update README with .NET 10.0 build instructions
    - Add CHANGELOG entry for migration
    - Preserve assessment.md and plan.md for reference"
    ```

**Validation**:
- ? README.md updated
- ? CHANGELOG.md updated
- ? Documentation committed

---

## Migration Complete! ??

**Congratulations!** All 4 tiers have been successfully migrated from .NET Framework 4.8 to .NET 10.0.

### Summary of Achievements

? **Framework Migration Complete**
- All 4 projects now target .NET 10.0
- All projects converted to SDK-style format
- ASP.NET Web API ? ASP.NET Core (major architectural upgrade)

? **Security Vulnerabilities Resolved**
- bootstrap: 3.3.7 ? 5.3.8 ?
- jQuery: 3.3.1 ? 3.7.1 ?
- Newtonsoft.Json: 11.0.1 ? 13.0.4 ?

? **Package Updates Applied**
- EntityFramework: 6.4.4 ? 6.5.1
- 21 incompatible packages removed
- 2 new ASP.NET Core packages added

? **Architectural Modernization**
- Global.asax ? Program.cs
- Web.config ? appsettings.json
- Unity DI ? Built-in ASP.NET Core DI
- Bundling/minification ? Static references

? **Quality Assurance**
- All projects build successfully
- All tests pass
- No security vulnerabilities remain
- Application functional and tested

### Next Steps

**Review & Merge**:
1. Review all changes on branch `upgrade-to-NET10-01`
2. Create pull request to merge to `main`
3. Obtain stakeholder approvals
4. Merge with `--no-ff` to preserve tier commit history
5. Tag release: `v2.0.0-net10.0`

**Deployment**:
1. Update CI/CD pipelines for .NET 10.0
2. Update infrastructure (ensure .NET 10.0 runtime installed)
3. Deploy to staging environment
4. Run smoke tests
5. Deploy to production
6. Monitor application performance and errors

**Post-Deployment**:
1. Track performance metrics vs .NET Framework baseline
2. Monitor for errors or issues
3. Collect user feedback
4. Document lessons learned

---

## Execution Log

This section will be updated as tasks are executed. Progress is tracked via task checkboxes above.

**Migration Start**: [To be filled when execution begins]  
**Migration End**: [To be filled when execution completes]  
**Total Duration**: [To be calculated]

**Issues Encountered**: [To be documented during execution]

**Lessons Learned**: [To be documented post-migration]

---

**Plan Reference**: `.github\upgrades\plan.md`  
**Assessment Reference**: `.github\upgrades\assessment.md`
