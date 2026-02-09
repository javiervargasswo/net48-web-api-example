# .NET Framework 4.8 to .NET 10.0 Migration - Execution Tasks

**Solution**: WebApiExample.sln  
**Branch**: upgrade-to-NET10  
**Strategy**: Bottom-Up (Dependency-First), Tier-by-Tier Migration

---

## Progress Dashboard

**Overall Progress**: 1/29 tasks completed (3%) ![3%](https://progress-bar.xyz/3)

**Tier Status**:
- [?] Prerequisites & Validation (1/3)
- [ ] Tier 1: WebApiExample.Common (0/5)
- [ ] Tier 2: WebApiExample.DataStore (0/6)
- [ ] Tier 3: WebApiExample.WebApp (0/12)
- [ ] Tier 4: WebApiExample.WebApp.Tests (0/6)
- [ ] Final Validation (0/2)

---

## Task List

### Phase 0: Prerequisites & Validation

#### [?] TASK-001: Verify .NET 10.0 SDK Installation *(Completed: 2026-02-09 15:31)*
**Description**: Ensure .NET 10.0 SDK is installed and accessible

**Actions**:
- [?] (1) Run command: `dotnet --version`
- [?] (2) Verify version is 10.0.x or higher
- [?] (3) Run command: `dotnet --list-runtimes`
- [?] (4) Verify Microsoft.NETCore.App 10.0.x is listed

**Expected Outcome**: .NET 10.0 SDK confirmed available

**References**: Plan §Testing & Validation Strategy

---

#### [ ] TASK-002: Verify Git Working Directory Clean
**Description**: Ensure no uncommitted changes before migration

**Actions**:
- [ ] (1) Run command: `git status`
- [ ] (2) Verify output shows "working tree clean" or "nothing to commit"
- [ ] (3) If uncommitted changes exist, commit or stash them
- [ ] (4) Verify current branch is `upgrade-to-NET10`

**Expected Outcome**: Clean working directory on upgrade-to-NET10 branch

**References**: Plan §Source Control Strategy

---

#### [ ] TASK-003: Create Pre-Migration Backup Commit
**Description**: Create safety commit before starting migration

**Actions**:
- [ ] (1) Ensure all current work committed
- [ ] (2) Create tag: `git tag -a pre-migration-net48 -m "Pre-migration state: .NET Framework 4.8"`
- [ ] (3) Push tag: `git push origin pre-migration-net48`
- [ ] (4) Verify tag created: `git tag -l`

**Expected Outcome**: Backup tag created for rollback safety

**References**: Plan §Source Control Strategy > Backup & Safety

---

### Phase 1: Tier 1 - WebApiExample.Common

#### [ ] TASK-004: Convert WebApiExample.Common to SDK-Style
**Description**: Convert legacy project format to modern SDK-style

**Actions**:
- [ ] (1) Navigate to WebApiExample.Common directory
- [ ] (2) Run conversion tool: `try-convert -p WebApiExample.Common.csproj`
- [ ] (3) Review generated .csproj file
- [ ] (4) Verify all 3 files included in project
- [ ] (5) Build to verify: `dotnet build`

**Expected Outcome**: Project converted to SDK-style, builds on current framework (net48)

**References**: Plan §Tier 1: WebApiExample.Common > Step 2

---

#### [ ] TASK-005: Update WebApiExample.Common Target Framework to net10.0
**Description**: Change target framework from net48 to net10.0

**Actions**:
- [ ] (1) Open WebApiExample.Common\WebApiExample.Common.csproj
- [ ] (2) Change `<TargetFramework>net48</TargetFramework>` to `<TargetFramework>net10.0</TargetFramework>`
- [ ] (3) Save file
- [ ] (4) Run: `dotnet restore`
- [ ] (5) Run: `dotnet build`

**Expected Outcome**: Project builds successfully on net10.0 with 0 errors, 0 warnings

**References**: Plan §Tier 1: WebApiExample.Common > Step 3

---

#### [ ] TASK-006: Validate WebApiExample.Common Migration
**Description**: Verify Tier 1 migration successful

**Actions**:
- [ ] (1) Verify build output in bin\Debug\net10.0\
- [ ] (2) Check all public types accessible (no API changes)
- [ ] (3) Verify no build warnings
- [ ] (4) Run: `dotnet build --configuration Release`

**Expected Outcome**: All validation checks pass

**References**: Plan §Tier 1: WebApiExample.Common > Step 7

---

#### [ ] TASK-007: Commit Tier 1 Completion
**Description**: Commit WebApiExample.Common migration

**Actions**:
- [ ] (1) Stage changes: `git add .`
- [ ] (2) Commit: `git commit -m "Tier 1: Migrated WebApiExample.Common to .NET 10.0"`
- [ ] (3) Push: `git push origin upgrade-to-NET10`
- [ ] (4) Verify commit created: `git log -1`

**Expected Outcome**: Tier 1 changes committed and pushed

**References**: Plan §Source Control Strategy > Commit Strategy

---

#### [ ] TASK-008: Tier 1 Gate Check
**Description**: Final validation before proceeding to Tier 2

**Actions**:
- [ ] (1) WebApiExample.Common builds on net10.0 ?
- [ ] (2) No errors or warnings ?
- [ ] (3) Changes committed to Git ?
- [ ] (4) Ready to proceed to Tier 2 ?

**Expected Outcome**: All gate checks pass, cleared to proceed

**References**: Plan §Migration Strategy > Tier Completion Criteria

---

### Phase 2: Tier 2 - WebApiExample.DataStore

#### [ ] TASK-009: Convert WebApiExample.DataStore to SDK-Style
**Description**: Convert legacy project format to modern SDK-style

**Actions**:
- [ ] (1) Navigate to WebApiExample.DataStore directory
- [ ] (2) Run conversion: `try-convert -p WebApiExample.DataStore.csproj`
- [ ] (3) Verify all 3 files included
- [ ] (4) Verify EntityFramework package reference present
- [ ] (5) Verify reference to WebApiExample.Common preserved
- [ ] (6) Build to verify: `dotnet build`

**Expected Outcome**: Project converted to SDK-style, builds on net48

**References**: Plan §Tier 2: WebApiExample.DataStore > Step 2

---

#### [ ] TASK-010: Update WebApiExample.DataStore Target Framework to net10.0
**Description**: Change target framework to net10.0

**Actions**:
- [ ] (1) Open WebApiExample.DataStore\WebApiExample.DataStore.csproj
- [ ] (2) Change `<TargetFramework>` to `net10.0`
- [ ] (3) Save file
- [ ] (4) Run: `dotnet restore`

**Expected Outcome**: Framework updated, packages restored

**References**: Plan §Tier 2: WebApiExample.DataStore > Step 3

---

#### [ ] TASK-011: Update EntityFramework Package
**Description**: Upgrade EntityFramework from 6.4.4 to 6.5.1

**Actions**:
- [ ] (1) Update package: `dotnet add package EntityFramework --version 6.5.1`
- [ ] (2) Or edit .csproj directly: `<PackageReference Include="EntityFramework" Version="6.5.1" />`
- [ ] (3) Run: `dotnet restore`
- [ ] (4) Verify EF 6.5.1 in packages

**Expected Outcome**: EntityFramework 6.5.1 installed

**References**: Plan §Tier 2: WebApiExample.DataStore > Step 4, Plan §Package Update Reference

---

#### [ ] TASK-012: Build and Validate WebApiExample.DataStore
**Description**: Verify Tier 2 migration successful

**Actions**:
- [ ] (1) Run: `dotnet build`
- [ ] (2) Verify 0 errors, 0 warnings
- [ ] (3) Verify reference to WebApiExample.Common (net10.0) resolved
- [ ] (4) Verify EntityFramework 6.5.1 referenced

**Expected Outcome**: Build successful with all validations passed

**References**: Plan §Tier 2: WebApiExample.DataStore > Step 7

---

#### [ ] TASK-013: Commit Tier 2 Completion
**Description**: Commit WebApiExample.DataStore migration

**Actions**:
- [ ] (1) Stage changes: `git add .`
- [ ] (2) Commit: `git commit -m "Tier 2: Migrated WebApiExample.DataStore to .NET 10.0, upgraded EF to 6.5.1"`
- [ ] (3) Push: `git push origin upgrade-to-NET10`

**Expected Outcome**: Tier 2 changes committed and pushed

**References**: Plan §Source Control Strategy

---

#### [ ] TASK-014: Tier 2 Gate Check
**Description**: Final validation before proceeding to Tier 3

**Actions**:
- [ ] (1) WebApiExample.DataStore builds on net10.0 ?
- [ ] (2) EntityFramework 6.5.1 installed ?
- [ ] (3) Integration with Tier 1 (Common) verified ?
- [ ] (4) Changes committed ?
- [ ] (5) Ready to proceed to Tier 3 ?

**Expected Outcome**: All gate checks pass

**References**: Plan §Migration Strategy > Tier Completion Criteria

---

### Phase 3: Tier 3 - WebApiExample.WebApp (HIGH COMPLEXITY)

#### [ ] TASK-015: Convert WebApiExample.WebApp to SDK-Style Web Project
**Description**: Convert WAP to SDK-style Web project

**Actions**:
- [ ] (1) Navigate to WebApiExample.WebApp directory
- [ ] (2) Run conversion: `try-convert -p WebApiExample.WebApp.csproj`
- [ ] (3) Change SDK to Web: `<Project Sdk="Microsoft.NET.Sdk.Web">`
- [ ] (4) Verify project references to Common and DataStore preserved
- [ ] (5) Build to verify: `dotnet build` (still on net48 temporarily)

**Expected Outcome**: Project converted to SDK-style Web project

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 2

---

#### [ ] TASK-016: Update WebApiExample.WebApp Target Framework to net10.0
**Description**: Change target framework to net10.0

**Actions**:
- [ ] (1) Open WebApiExample.WebApp\WebApiExample.WebApp.csproj
- [ ] (2) Change `<TargetFramework>` to `net10.0`
- [ ] (3) Add properties: `<Nullable>enable</Nullable>` and `<ImplicitUsings>enable</ImplicitUsings>`
- [ ] (4) Save file
- [ ] (5) Run: `dotnet restore`

**Expected Outcome**: Framework updated, ASP.NET Core framework automatically referenced

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 3

---

#### [ ] TASK-017: Remove Incompatible Packages from WebApiExample.WebApp
**Description**: Remove packages built into ASP.NET Core or incompatible

**Actions**:
- [ ] (1) Remove from .csproj: Microsoft.AspNet.Mvc
- [ ] (2) Remove: Microsoft.AspNet.Razor
- [ ] (3) Remove: Microsoft.AspNet.WebApi
- [ ] (4) Remove: Microsoft.AspNet.WebApi.Core
- [ ] (5) Remove: Microsoft.AspNet.WebApi.WebHost
- [ ] (6) Remove: Microsoft.AspNet.WebPages
- [ ] (7) Remove: Microsoft.CodeDom.Providers.DotNetCompilerPlatform
- [ ] (8) Remove: Microsoft.Web.Infrastructure
- [ ] (9) Remove: Microsoft.AspNet.Web.Optimization
- [ ] (10) Remove: Unity.WebAPI
- [ ] (11) Remove: Antlr (verify not used first)
- [ ] (12) Save .csproj
- [ ] (13) Run: `dotnet restore`

**Expected Outcome**: Incompatible packages removed

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 4, Plan §Package Update Reference

---

#### [ ] TASK-018: Update Security-Critical Packages in WebApiExample.WebApp
**Description**: Fix security vulnerabilities (CRITICAL)

**Actions**:
- [ ] (1) Update bootstrap: `dotnet add package bootstrap --version 5.3.8`
- [ ] (2) Update jQuery: `dotnet add package jQuery --version 3.7.1`
- [ ] (3) Update Newtonsoft.Json: `dotnet add package Newtonsoft.Json --version 13.0.4`
- [ ] (4) Update System.Runtime.CompilerServices.Unsafe: `dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2`
- [ ] (5) Run: `dotnet restore`
- [ ] (6) Verify packages updated in .csproj

**Expected Outcome**: All security vulnerabilities addressed

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 4, Plan §Risk Management > Security Vulnerabilities

---

#### [ ] TASK-019: Create Program.cs for ASP.NET Core
**Description**: Replace Global.asax with Program.cs

**Actions**:
- [ ] (1) Create new file: WebApiExample.WebApp\Program.cs
- [ ] (2) Add basic ASP.NET Core Web API template code (see Plan §Tier 3 Step 5)
- [ ] (3) Include: builder.Services.AddControllers()
- [ ] (4) Include: app.MapControllers()
- [ ] (5) Include: app.UseStaticFiles() for static content
- [ ] (6) Save file

**Expected Outcome**: Program.cs created with ASP.NET Core initialization

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 5

---

#### [ ] TASK-020: Migrate Unity DI to Built-in DI
**Description**: Replace Unity container with Microsoft.Extensions.DependencyInjection

**Actions**:
- [ ] (1) Review App_Start\UnityConfig.cs for all registrations
- [ ] (2) Map each Unity registration to built-in DI in Program.cs
- [ ] (3) Add services to builder.Services (AddScoped, AddSingleton, AddTransient)
- [ ] (4) Delete App_Start\UnityConfig.cs
- [ ] (5) Verify all controllers use constructor injection

**Expected Outcome**: DI migrated to built-in container

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 7

---

#### [ ] TASK-021: Migrate Web.config to appsettings.json
**Description**: Move configuration from Web.config to appsettings.json

**Actions**:
- [ ] (1) Create appsettings.json in project root
- [ ] (2) Create appsettings.Development.json
- [ ] (3) Migrate <appSettings> to JSON format
- [ ] (4) Migrate <connectionStrings> to JSON
- [ ] (5) Add Logging configuration
- [ ] (6) Update code using ConfigurationManager to use IConfiguration
- [ ] (7) Keep minimal Web.config for IIS only (if needed)

**Expected Outcome**: Configuration migrated to appsettings.json

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 9

---

#### [ ] TASK-022: Update All Controllers to ASP.NET Core
**Description**: Migrate controllers from ApiController to ControllerBase

**Actions**:
- [ ] (1) For each controller in Controllers\ folder:
  - Change `using System.Web.Http` to `using Microsoft.AspNetCore.Mvc`
  - Change base class from `ApiController` to `ControllerBase`
  - Add `[Route("api/[controller]")]` attribute
  - Add `[ApiController]` attribute
  - Change `IHttpActionResult` to `IActionResult`
  - Add `[HttpGet]`, `[HttpPost]`, etc. attributes to actions
- [ ] (2) Save all controller files
- [ ] (3) Delete Global.asax and Global.asax.cs

**Expected Outcome**: All controllers migrated to ASP.NET Core

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 10

---

#### [ ] TASK-023: Migrate Static Files to wwwroot
**Description**: Move static files to ASP.NET Core convention

**Actions**:
- [ ] (1) Create wwwroot folder in project root
- [ ] (2) Move Scripts\ content to wwwroot\lib\ or wwwroot\js\
- [ ] (3) Move Content\ CSS to wwwroot\css\
- [ ] (4) Update HTML/Razor views with new paths
- [ ] (5) Delete App_Start\BundleConfig.cs
- [ ] (6) Ensure app.UseStaticFiles() in Program.cs

**Expected Outcome**: Static files in wwwroot, direct HTML references working

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 8

---

#### [ ] TASK-024: Build WebApiExample.WebApp
**Description**: Attempt first build and resolve compilation errors

**Actions**:
- [ ] (1) Run: `dotnet build`
- [ ] (2) Address namespace errors (System.Web.* ? Microsoft.AspNetCore.*)
- [ ] (3) Fix type errors (ApiController ? ControllerBase)
- [ ] (4) Fix configuration access errors
- [ ] (5) Rebuild iteratively until 0 errors

**Expected Outcome**: Project builds successfully on net10.0

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 12

---

#### [ ] TASK-025: Test WebApiExample.WebApp Runtime
**Description**: Start application and test API endpoints

**Actions**:
- [ ] (1) Run: `dotnet run`
- [ ] (2) Verify application starts without exceptions
- [ ] (3) Test each API endpoint (GET, POST, PUT, DELETE)
- [ ] (4) Verify JSON serialization works
- [ ] (5) Verify database connectivity (if applicable)
- [ ] (6) Verify static files serve correctly
- [ ] (7) Check for runtime errors in logs
- [ ] (8) Stop application (Ctrl+C)

**Expected Outcome**: All endpoints functional, no runtime errors

**References**: Plan §Tier 3: WebApiExample.WebApp > Step 13

---

#### [ ] TASK-026: Commit Tier 3 Completion
**Description**: Commit WebApiExample.WebApp migration

**Actions**:
- [ ] (1) Stage changes: `git add .`
- [ ] (2) Commit with detailed message (see Plan §Source Control Strategy for template)
- [ ] (3) Include summary: SDK conversion, ASP.NET Core migration, security fixes, package updates
- [ ] (4) Push: `git push origin upgrade-to-NET10`

**Expected Outcome**: Tier 3 changes committed

**References**: Plan §Source Control Strategy > Commit Strategy

---

#### [ ] TASK-027: Tier 3 Gate Check
**Description**: Final validation before proceeding to Tier 4

**Actions**:
- [ ] (1) WebApiExample.WebApp builds on net10.0 ?
- [ ] (2) Application starts and runs ?
- [ ] (3) All API endpoints tested ?
- [ ] (4) Security vulnerabilities fixed ?
- [ ] (5) Integration with Tier 1 & 2 verified ?
- [ ] (6) Changes committed ?
- [ ] (7) Ready to proceed to Tier 4 ?

**Expected Outcome**: All gate checks pass

**References**: Plan §Migration Strategy > Tier Completion Criteria

---

### Phase 4: Tier 4 - WebApiExample.WebApp.Tests

#### [ ] TASK-028: Convert WebApiExample.WebApp.Tests to SDK-Style
**Description**: Convert legacy test project to SDK-style

**Actions**:
- [ ] (1) Navigate to WebApiExample.WebApp.Tests directory
- [ ] (2) Run conversion: `try-convert -p WebApiExample.WebApp.Tests.csproj`
- [ ] (3) Verify all 4 test files included
- [ ] (4) Verify references to Common, DataStore, WebApp preserved
- [ ] (5) Build to verify: `dotnet build`

**Expected Outcome**: Project converted to SDK-style

**References**: Plan §Tier 4: WebApiExample.WebApp.Tests > Step 2

---

#### [ ] TASK-029: Update WebApiExample.WebApp.Tests Target Framework to net10.0
**Description**: Change target framework to net10.0

**Actions**:
- [ ] (1) Open WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj
- [ ] (2) Change `<TargetFramework>` to `net10.0`
- [ ] (3) Save file
- [ ] (4) Run: `dotnet restore`

**Expected Outcome**: Framework updated

**References**: Plan §Tier 4: WebApiExample.WebApp.Tests > Step 3

---

#### [ ] TASK-030: Update and Remove Packages in WebApiExample.WebApp.Tests
**Description**: Fix security vulnerability and remove incompatible packages

**Actions**:
- [ ] (1) Update Newtonsoft.Json: `dotnet add package Newtonsoft.Json --version 13.0.4`
- [ ] (2) Update System.Runtime.CompilerServices.Unsafe: `dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2`
- [ ] (3) Remove from .csproj: Microsoft.AspNet.Mvc, Microsoft.AspNet.Razor, Microsoft.AspNet.WebApi.Core, Microsoft.AspNet.WebApi.WebHost, Microsoft.AspNet.WebPages
- [ ] (4) Remove: Microsoft.Web.Infrastructure, System.Buffers, System.Memory, System.Numerics.Vectors, System.Runtime.InteropServices.RuntimeInformation, System.Threading.Tasks.Extensions
- [ ] (5) Run: `dotnet restore`

**Expected Outcome**: Security fix applied, incompatible packages removed

**References**: Plan §Tier 4: WebApiExample.WebApp.Tests > Step 4, Plan §Package Update Reference

---

#### [ ] TASK-031: Update Test Code for ASP.NET Core
**Description**: Update using directives and test assertions

**Actions**:
- [ ] (1) For each test file:
  - Remove: `using System.Web.Http`, `using System.Web.Http.Results`
  - Add (if needed): `using Microsoft.AspNetCore.Mvc`
  - Update controller references (ApiController ? ControllerBase)
  - Update assertion types (IHttpActionResult ? IActionResult, OkNegotiatedContentResult ? OkObjectResult)
- [ ] (2) Save all test files

**Expected Outcome**: Test code updated for ASP.NET Core

**References**: Plan §Tier 4: WebApiExample.WebApp.Tests > Step 6

---

#### [ ] TASK-032: Build and Run Tests
**Description**: Build test project and execute all tests

**Actions**:
- [ ] (1) Run: `dotnet build`
- [ ] (2) Verify 0 errors, 0 warnings
- [ ] (3) Run: `dotnet test`
- [ ] (4) Verify all tests discovered
- [ ] (5) Verify all tests pass (target: 100%)
- [ ] (6) If failures, investigate and fix (update tests or fix code)

**Expected Outcome**: All tests pass

**References**: Plan §Tier 4: WebApiExample.WebApp.Tests > Step 8

---

#### [ ] TASK-033: Commit Tier 4 Completion
**Description**: Commit WebApiExample.WebApp.Tests migration

**Actions**:
- [ ] (1) Stage changes: `git add .`
- [ ] (2) Commit: `git commit -m "Tier 4: Migrated WebApiExample.WebApp.Tests to .NET 10.0

All projects successfully migrated from .NET Framework 4.8 to .NET 10.0.
All security vulnerabilities addressed. All tests passing."`
- [ ] (3) Push: `git push origin upgrade-to-NET10`

**Expected Outcome**: Tier 4 changes committed, migration complete

**References**: Plan §Source Control Strategy

---

### Phase 5: Final Validation

#### [ ] TASK-034: Full Solution Build and Test
**Description**: Validate entire solution migrated successfully

**Actions**:
- [ ] (1) Run: `dotnet build WebApiExample.sln --configuration Release`
- [ ] (2) Verify: Build succeeded, 0 errors, 0 warnings
- [ ] (3) Run: `dotnet test WebApiExample.sln --configuration Release`
- [ ] (4) Verify: All tests pass (100%)
- [ ] (5) Run: `dotnet list package --vulnerable --include-transitive`
- [ ] (6) Verify: No vulnerabilities found
- [ ] (7) Verify all projects show net10.0: `grep -r "<TargetFramework>" --include="*.csproj"`

**Expected Outcome**: Full solution builds, all tests pass, no vulnerabilities

**References**: Plan §Success Criteria > Verification Commands

---

#### [ ] TASK-035: Create Migration Completion Tag
**Description**: Tag successful migration completion

**Actions**:
- [ ] (1) Create tag: `git tag -a v2.0.0-net10.0 -m "Migrated to .NET 10.0 - Full solution on net10.0, all security vulnerabilities addressed"`
- [ ] (2) Push tag: `git push origin v2.0.0-net10.0`
- [ ] (3) Verify tag: `git tag -l`
- [ ] (4) Prepare for merge to main (if ready)

**Expected Outcome**: Migration tagged, ready for merge/deployment

**References**: Plan §Source Control Strategy > Post-Merge, Plan §Success Criteria

---

## Execution Log

*This section will be updated as tasks are executed*

---

## Notes

- **High-Risk Tier**: Tier 3 (WebApiExample.WebApp) is marked as HIGH COMPLEXITY
- **Security Critical**: Tasks 018 and 030 address security vulnerabilities - MUST be completed
- **Dependencies**: Each tier must complete before next tier starts (strict ordering)
- **Rollback**: Use `git reset --hard <commit>` to rollback to any tier if needed
- **Estimated Effort**: Tier 3 represents ~55% of total migration effort

**Total Tasks**: 35  
**Estimated Duration**: Tier 1 (short), Tier 2 (short-medium), Tier 3 (long), Tier 4 (medium), Validation (short)

