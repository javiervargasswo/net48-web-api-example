# .NET Framework 4.8 to .NET 10.0 Migration Plan

## Table of Contents

- [Executive Summary](#executive-summary)
- [Migration Strategy](#migration-strategy)
- [Detailed Dependency Analysis](#detailed-dependency-analysis)
- [Project-by-Project Migration Plans](#project-by-project-migration-plans)
  - [Tier 1: WebApiExample.Common](#tier-1-webapiexamplecommon)
  - [Tier 2: WebApiExample.DataStore](#tier-2-webapiexampledatastore)
  - [Tier 3: WebApiExample.WebApp](#tier-3-webapiexamplewebapp)
  - [Tier 4: WebApiExample.WebApp.Tests](#tier-4-webapiexamplewebapptests)
- [Package Update Reference](#package-update-reference)
- [Breaking Changes Catalog](#breaking-changes-catalog)
- [Risk Management](#risk-management)
- [Testing & Validation Strategy](#testing--validation-strategy)
- [Complexity & Effort Assessment](#complexity--effort-assessment)
- [Source Control Strategy](#source-control-strategy)
- [Success Criteria](#success-criteria)

---

## Executive Summary

### Scenario Overview

This plan details the migration of the WebApiExample solution from **.NET Framework 4.8** to **.NET 10.0 (Long Term Support)**. The solution consists of 4 projects with a clear dependency hierarchy, totaling 4,176 lines of code across 70 files.

### Scope

**Projects to Migrate:**
- WebApiExample.Common (61 LOC, 0 dependencies) - Shared models and utilities
- WebApiExample.DataStore (110 LOC, 1 dependency) - Data access layer using Entity Framework
- WebApiExample.WebApp (3,612 LOC, 2 dependencies) - ASP.NET Web API application
- WebApiExample.WebApp.Tests (393 LOC, 3 dependencies) - xUnit test project

**Current State:** All projects target .NET Framework 4.8 using legacy non-SDK-style project files

**Target State:** All projects targeting .NET 10.0 using modern SDK-style project files

### Complexity Assessment

**Discovered Metrics:**
- Total Projects: 4
- Total LOC: 4,176
- Dependency Depth: 3 levels
- High-Risk Projects: 1 (WebApiExample.WebApp)
- Total Package Updates: 14 out of 46 packages
- Security Vulnerabilities: 2 (bootstrap 3.3.7, jQuery 3.3.1)
- Breaking Package Changes: 8 incompatible packages requiring replacement

**Classification: Medium Complexity**

This solution qualifies as medium complexity due to:
- Small project count (4 projects)
- Manageable dependency depth (3 levels)
- One high-risk project (WebApp with complex ASP.NET MVC/WebAPI patterns)
- Security vulnerabilities requiring immediate attention
- Significant architectural changes needed (Global.asax ? Program.cs, bundling/minification)

### Critical Issues

?? **Security Vulnerabilities**
- `bootstrap` 3.3.7 ? 5.3.8 (known CVEs)
- `jQuery` 3.3.1 ? 3.7.1 (known CVEs)

?? **Incompatible Packages**
- Microsoft.AspNet.* packages (replaced by framework)
- Unity.WebAPI (requires alternative DI approach)
- Microsoft.AspNet.Web.Optimization (bundling replacement needed)

?? **Architectural Changes**
- Global.asax.cs application initialization ? Program.cs with minimal hosting model
- System.Web.Optimization bundling ? Link tags or build-time bundling
- ASP.NET MVC/WebAPI ? ASP.NET Core MVC/Controllers

### Recommended Approach

**Bottom-Up Incremental Migration** (dependency-first strategy)

Migrate projects sequentially from leaf nodes upward through the dependency chain:
1. **Tier 1**: Common (no dependencies)
2. **Tier 2**: DataStore (depends on Common)
3. **Tier 3**: WebApp (depends on Common, DataStore)
4. **Tier 4**: Tests (depends on all)

Each tier is fully upgraded, tested, and stabilized before proceeding to the next, ensuring dependencies are always on the same or newer framework version than consumers.

### Iteration Strategy

Using **phase-based iterations** - one detail iteration per tier, treating each tier as a milestone with distinct upgrade phases (preparation, update, testing, stabilization).

## Migration Strategy

### Approach: Bottom-Up Incremental Migration

**Selected Strategy: Bottom-Up (Dependency-First)**

This strategy upgrades projects sequentially starting from leaf nodes (projects with no dependencies) and progressing upward through the dependency chain to the main application and tests. Each tier is fully upgraded, tested, and stabilized before moving to the next.

### Rationale for Bottom-Up Approach

**Why Bottom-Up is Ideal for This Solution:**

1. **Clear Dependency Hierarchy** (4 distinct tiers)
   - Well-defined layers: Common ? DataStore ? WebApp ? Tests
   - No circular dependencies
   - Natural progression from simple to complex

2. **Medium Complexity** (4 projects, 1 high-risk)
   - Small enough to complete in phases
   - Large enough to benefit from incremental approach
   - High-risk project (WebApp) isolated to single tier

3. **Risk Mitigation**
   - Each tier builds on stable, already-upgraded foundation
   - Can validate tier completion before proceeding
   - Issues isolated to current tier, not cascading
   - Security vulnerabilities addressed in natural dependency order

4. **No Multi-Targeting Complexity**
   - Dependencies always on same or newer framework than consumers
   - Avoid complex TargetFrameworks conditions
   - Simpler package resolution

### Migration Phases

Each tier follows a consistent 4-phase workflow:

**Phase Structure per Tier:**

1. **Preparation** (Assessment)
   - Review tier's current state
   - Verify dependencies are stable (lower tiers complete)
   - Identify tier-specific risks and package requirements
   - Prepare migration checklist

2. **Update** (Transformation)
   - Convert project(s) to SDK-style format
   - Update TargetFramework to net10.0
   - Update/replace/remove NuGet packages
   - Fix compilation errors from package/API changes

3. **Testing** (Validation)
   - Build tier projects successfully
   - Run unit tests (if tier contains tests)
   - Run integration tests with lower tiers
   - Validate higher tiers (still on old framework) not affected

4. **Stabilization** (Checkpoint)
   - Address discovered issues
   - Document lessons learned
   - Verify tier completion criteria met
   - Mark tier ready for next phase

### Tier Execution Sequence

**Strict Ordering (Cannot Skip):**

```
Tier 1: Common
  ? (validate before proceeding)
Tier 2: DataStore  
  ? (validate before proceeding)
Tier 3: WebApp
  ? (validate before proceeding)
Tier 4: WebApp.Tests
  ? (validate complete)
? Migration Complete
```

**Tier Completion Criteria (Gate for Next Tier):**
- ? All projects in tier build without errors
- ? All projects in tier build without warnings
- ? All tests in tier pass (if applicable)
- ? No package dependency conflicts
- ? Integration with lower tiers validated
- ? Higher tiers (on old framework) still function

### Between-Tier Validation

After each tier completes, verify:

1. **Current Tier Health**
   - All tier projects build clean (no errors/warnings)
   - All tier tests pass
   - No security vulnerabilities in tier packages

2. **Backward Compatibility**
   - Lower tiers still build and pass tests
   - No regressions introduced

3. **Forward Compatibility**
   - Higher tiers (still on net48) can reference upgraded tier
   - Solution still builds as a whole (mixed framework versions)

### Parallel vs Sequential Within Tiers

**For This Solution:**

- **Tier 1**: 1 project ? Sequential (only one project)
- **Tier 2**: 1 project ? Sequential (only one project)
- **Tier 3**: 1 project ? Sequential (only one project)
- **Tier 4**: 1 project ? Sequential (only one project)

All tiers contain single projects, so no parallelization opportunities within tiers. Migration is strictly sequential tier-by-tier.

### Strategy-Specific Considerations for .NET Framework ? .NET Core

**Bottom-Up Benefits for Framework Migration:**

1. **Entity Framework Compatibility** (Tier 2)
   - DataStore migrates before WebApp
   - EF6 compatibility issues discovered early
   - WebApp can assume stable data layer

2. **Bundling/Optimization Changes** (Tier 3)
   - System.Web.Optimization incompatibility isolated to WebApp tier
   - Can address architectural change in focused phase
   - Common/DataStore already stable

3. **Dependency Injection Migration** (Tier 3)
   - Unity.WebAPI incompatibility addressed in WebApp tier
   - Can migrate to ASP.NET Core DI in isolated context
   - Lower tiers don't need DI changes

4. **Test Isolation** (Tier 4)
   - All application tiers stable before test migration
   - Test failures clearly attributable to test changes, not app changes
   - Can update test patterns/frameworks independently

## Detailed Dependency Analysis

### Dependency Graph Structure

The solution has a clear 4-tier dependency structure with no circular dependencies:

```
Tier 4: [WebApp.Tests]
         ?
Tier 3: [WebApp]
         ?
Tier 2: [DataStore]
         ?
Tier 1: [Common]
```

**Tier Breakdown:**

**Tier 1 (Leaf Nodes - No Dependencies):**
- `WebApiExample.Common` - Shared models and utilities (61 LOC)
  - No internal project dependencies
  - No NuGet packages
  - Lowest risk, simplest migration

**Tier 2 (Depends Only on Tier 1):**
- `WebApiExample.DataStore` - Data access layer (110 LOC)
  - Depends on: Common
  - Packages: EntityFramework 6.4.4
  - Low complexity, Entity Framework compatibility check needed

**Tier 3 (Application Layer):**
- `WebApiExample.WebApp` - ASP.NET Web API application (3,612 LOC)
  - Depends on: Common, DataStore
  - 19 package updates/changes required
  - Highest complexity due to ASP.NET Framework ? ASP.NET Core conversion
  - Architectural changes: Global.asax ? Program.cs, bundling replacement

**Tier 4 (Test Layer):**
- `WebApiExample.WebApp.Tests` - Test project (393 LOC)
  - Depends on: Common, DataStore, WebApp
  - 14 package updates required
  - Must migrate after all dependencies complete

### Project Groupings by Migration Phase

**Phase 1 - Tier 1: Foundation Layer**
- Projects: WebApiExample.Common
- Rationale: Leaf node with zero dependencies, provides foundational models
- Risk: Low (no external packages, simple code)

**Phase 2 - Tier 2: Data Access Layer**
- Projects: WebApiExample.DataStore
- Rationale: Depends only on stable Tier 1, isolated data access logic
- Risk: Low-Medium (Entity Framework 6 compatibility with .NET Core)

**Phase 3 - Tier 3: Application Layer**
- Projects: WebApiExample.WebApp
- Rationale: Main application consuming lower tiers
- Risk: High (complex ASP.NET Framework ? Core conversion, many package changes)

**Phase 4 - Tier 4: Test Layer**
- Projects: WebApiExample.WebApp.Tests
- Rationale: Tests depend on all application projects
- Risk: Medium (package updates, test framework compatibility)

### Critical Path

The critical path follows the dependency chain strictly:

**Common ? DataStore ? WebApp ? WebApp.Tests**

- Cannot skip tiers (e.g., cannot upgrade WebApp before DataStore)
- Each tier builds on the stable foundation of previous tiers
- Testing is cumulative (validate current tier + integration with lower tiers)

### Dependency Validation Rules

1. **Tier 1 must complete** before Tier 2 begins
2. **Tier 2 must complete** before Tier 3 begins
3. **Tier 3 must complete** before Tier 4 begins
4. **Completion criteria**: Project builds, tests pass, no errors/warnings
5. **No multi-targeting**: Each tier fully migrates to net10.0 before next tier starts

## Project-by-Project Migration Plans

### Tier 1: WebApiExample.Common

#### Current State

- **Target Framework:** net48
- **SDK-style:** False (legacy project format)
- **Project Type:** ClassLibrary
- **Dependencies:** 0 (leaf node - no project dependencies)
- **Dependants:** 3 (DataStore, WebApp, WebApp.Tests)
- **Files:** 3
- **Lines of Code:** 61
- **Packages:** 0
- **Risk Level:** ?? Low

#### Target State

- **Target Framework:** net10.0
- **SDK-style:** True
- **Package Updates:** None required (no packages currently)

#### Migration Steps

##### 1. Prerequisites

- ? No dependencies to wait for (leaf node)
- ? .NET 10.0 SDK installed
- ? Working on branch: `upgrade-to-NET10`

##### 2. Project File Conversion

**Convert to SDK-style:**
- Use automated conversion tool or manual conversion
- Replace verbose legacy .csproj with SDK-style format
- Update TargetFramework element: `<TargetFramework>net10.0</TargetFramework>`
- Remove unnecessary elements (PropertyGroups, references to System assemblies now implicit)

**Expected Changes:**
```xml
<!-- Before: Legacy project file (~100+ lines) -->
<!-- After: SDK-style project file (~5-10 lines) -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>
</Project>
```

##### 3. Package Updates

**No package updates required** - project has zero NuGet dependencies.

##### 4. Expected Breaking Changes

**None anticipated** - this is a simple class library with shared models and utilities. No APIs expected to break based on assessment.

**Potential Areas to Review:**
- Any usage of .NET Framework-specific APIs (unlikely in a common library)
- Serialization patterns (if models use attributes)

##### 5. Code Modifications

**Expected:** Minimal to zero code changes

**Review Areas:**
- Validate all type definitions compile on net10.0
- Check for any .NET Framework-specific attributes
- Verify no System.Web or Framework-specific references

**API Compatibility:** Assessment shows 10 APIs analyzed, all compatible.

##### 6. Testing Strategy

**Build Validation:**
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] No package restore issues

**Compatibility Testing:**
- [ ] All public types accessible
- [ ] No API changes in public surface
- [ ] Serialization/deserialization works (if applicable)

**Integration Testing:**
- [ ] Dependent projects (DataStore, WebApp, WebApp.Tests) still reference correctly
- [ ] Solution builds with mixed frameworks (Common on net10.0, others on net48)

##### 7. Validation Checklist

- [ ] SDK-style conversion successful
- [ ] TargetFramework set to net10.0
- [ ] Build succeeds with zero errors
- [ ] Build succeeds with zero warnings
- [ ] No package dependency conflicts
- [ ] All APIs remain compatible
- [ ] Dependant projects can still reference this project
- [ ] Git commit created with clear message

#### Tier 1 Completion Criteria

? **Ready to proceed to Tier 2 when:**
- WebApiExample.Common builds cleanly on net10.0
- No errors or warnings
- Dependent projects (still on net48) can reference upgraded Common
- Changes committed to source control

---

### Tier 2: WebApiExample.DataStore

#### Current State

- **Target Framework:** net48
- **SDK-style:** False (legacy project format)
- **Project Type:** ClassLibrary
- **Dependencies:** 1 (WebApiExample.Common - on net10.0 after Tier 1)
- **Dependants:** 2 (WebApp, WebApp.Tests)
- **Files:** 3
- **Lines of Code:** 110
- **Packages:** 1 (EntityFramework 6.4.4)
- **Risk Level:** ?? Medium (Entity Framework compatibility)

#### Target State

- **Target Framework:** net10.0
- **SDK-style:** True
- **Package Updates:** EntityFramework 6.4.4 ? 6.5.1

#### Migration Steps

##### 1. Prerequisites

- ? **Tier 1 Complete:** WebApiExample.Common on net10.0 and stable
- ? .NET 10.0 SDK installed
- ? Working on branch: `upgrade-to-NET10`

##### 2. Project File Conversion

**Convert to SDK-style:**
- Use automated conversion tool or manual conversion
- Update TargetFramework: `<TargetFramework>net10.0</TargetFramework>`
- Simplify project structure

**Expected Changes:**
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>
  
  <ItemGroup>
    <ProjectReference Include="..\WebApiExample.Common\WebApiExample.Common.csproj" />
  </ItemGroup>
  
  <ItemGroup>
    <PackageReference Include="EntityFramework" Version="6.5.1" />
  </ItemGroup>
</Project>
```

##### 3. Package Updates

| Package | Current Version | Target Version | Reason |
|---------|----------------|----------------|---------|
| EntityFramework | 6.4.4 | 6.5.1 | Recommended upgrade for better .NET Core compatibility |

**Update Process:**
- Update EntityFramework package reference to 6.5.1
- Restore packages
- Verify no package conflicts

**Important Notes:**
- Entity Framework 6.x is supported on .NET Core/.NET 5+ but not actively developed
- EF6 has limitations on .NET Core (some providers, features differ from .NET Framework)
- Long-term recommendation: consider migrating to EF Core (out of scope for this migration)

##### 4. Expected Breaking Changes

**Entity Framework 6 on .NET Core Considerations:**

- **Database Providers:** Ensure SQL Server provider (or other) is .NET Core compatible
- **Configuration:** App.config-based EF configuration not supported (must use code-based configuration)
- **Connection Strings:** May need to move from app.config to appsettings.json or code
- **Migrations:** EF6 migrations still supported but tooling may differ

**Likely Required Changes:**
- If using app.config for EF configuration ? Migrate to code-based configuration (DbConfiguration)
- If using connection strings from app.config ? Pass connection strings via code or appsettings.json

##### 5. Code Modifications

**Expected Code Changes:**

1. **DbContext Configuration:**
   - Ensure DbContext uses code-based configuration
   - Connection strings passed via constructor or configured in code

2. **Provider Registration:**
   - Verify SQL Server provider (or other) registered correctly
   - May need explicit provider configuration

**Example Pattern:**
```csharp
// If currently using app.config, migrate to:
public class MyDbContext : DbContext
{
    public MyDbContext(string connectionString) 
        : base(connectionString)
    {
    }
    
    // Explicit configuration if needed
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configuration here
    }
}
```

##### 6. Testing Strategy

**Build Validation:**
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] EntityFramework 6.5.1 package restores successfully
- [ ] No package dependency conflicts

**Entity Framework Testing:**
- [ ] DbContext can be instantiated
- [ ] Database connection succeeds
- [ ] Basic CRUD operations work
- [ ] Migrations apply (if used)
- [ ] Queries return expected results

**Integration Testing:**
- [ ] Common project reference works correctly
- [ ] Dependent projects (WebApp, WebApp.Tests) can still reference DataStore
- [ ] Solution builds with mixed frameworks

##### 7. Validation Checklist

- [ ] SDK-style conversion successful
- [ ] TargetFramework set to net10.0
- [ ] EntityFramework updated to 6.5.1
- [ ] Build succeeds with zero errors
- [ ] Build succeeds with zero warnings
- [ ] Entity Framework operations functional
- [ ] Database connectivity verified
- [ ] No package dependency conflicts
- [ ] Common project reference intact
- [ ] Git commit created with clear message

#### Tier 2 Completion Criteria

? **Ready to proceed to Tier 3 when:**
- WebApiExample.DataStore builds cleanly on net10.0
- EntityFramework 6.5.1 package installed and functional
- Database operations validated
- No errors or warnings
- Dependent projects (still on net48) can reference upgraded DataStore
- Changes committed to source control

#### Tier 2 Risk Mitigation

**If Entity Framework 6 Issues Arise:**

- **Option A (Recommended for this migration):** Address EF6 compatibility issues
  - Document workarounds
  - Accept some limitations
  - Plan future EF Core migration separately

- **Option B (Extended scope):** Migrate to EF Core during this tier
  - Requires pattern changes (DbModelBuilder ? ModelBuilder)
  - Different migration tooling
  - Better long-term support
  - Significantly extends effort

**Fallback:** If blocking issues with EF6, revert Tier 2, reassess migration approach

---

### Tier 3: WebApiExample.WebApp

#### Current State

- **Target Framework:** net48
- **SDK-style:** False (legacy Web Application Project format)
- **Project Type:** Web Application Project (WAP)
- **Dependencies:** 2 (Common, DataStore - both on net10.0 after Tiers 1-2)
- **Dependants:** 1 (WebApp.Tests)
- **Files:** 92
- **Lines of Code:** 3,612
- **Packages:** 19 packages requiring updates/removal/replacement
- **Risk Level:** ?? High (ASP.NET Framework ? Core conversion, architectural changes)

#### Target State

- **Target Framework:** net10.0
- **SDK-style:** True
- **Project Type:** ASP.NET Core Web Application
- **Package Updates:** 19 packages (see detailed table below)

#### Migration Steps

##### 1. Prerequisites

- ? **Tier 1 Complete:** WebApiExample.Common on net10.0 and stable
- ? **Tier 2 Complete:** WebApiExample.DataStore on net10.0 and stable
- ? .NET 10.0 SDK installed
- ? Working on branch: `upgrade-to-NET10`

##### 2. Project File Conversion

**Convert to SDK-style Web Application:**

This is the most complex conversion - Web Application Projects (WAP) have significantly different structure in ASP.NET Core.

**Expected Changes:**
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  
  <ItemGroup>
    <ProjectReference Include="..\WebApiExample.Common\WebApiExample.Common.csproj" />
    <ProjectReference Include="..\WebApiExample.DataStore\WebApiExample.DataStore.csproj" />
  </ItemGroup>
  
  <!-- Package references - see Package Updates section -->
</Project>
```

**Key Differences:**
- Sdk="Microsoft.NET.Sdk.Web" (not just Sdk)
- No need for explicit ASP.NET Core framework references (implicit)
- Content files (wwwroot) automatically included

##### 3. Package Updates

**Tier 3 Package Updates Table:**

| Package | Current Version | Target Version | Action | Reason |
|---------|----------------|----------------|--------|---------|
| **Security Vulnerabilities** |
| bootstrap | 3.3.7 | 5.3.8 | UPDATE | ?? Security vulnerability (CVE) |
| jQuery | 3.3.1 | 3.7.1 | UPDATE | ?? Security vulnerability (CVE) |
| **Recommended Upgrades** |
| EntityFramework | 6.4.4 | 6.5.1 | UPDATE | Better .NET Core compatibility |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | UPDATE | Compatibility and features |
| System.Runtime.CompilerServices.Unsafe | 4.5.2 | 6.1.2 | UPDATE | Framework compatibility |
| **Framework-Included (Remove)** |
| Microsoft.AspNet.Mvc | 5.2.4 | - | REMOVE | Functionality in ASP.NET Core framework |
| Microsoft.AspNet.Razor | 3.2.4 | - | REMOVE | Functionality in ASP.NET Core framework |
| Microsoft.AspNet.WebApi | 5.2.4 | - | REMOVE | Functionality in ASP.NET Core framework |
| Microsoft.AspNet.WebPages | 3.2.4 | - | REMOVE | Functionality in ASP.NET Core framework |
| Microsoft.CodeDom.Providers.DotNetCompilerPlatform | 2.0.0 | - | REMOVE | Not needed in .NET Core (Roslyn built-in) |
| Microsoft.Web.Infrastructure | 1.0.0.0 | - | REMOVE | ASP.NET Framework infrastructure |
| **Incompatible (Remove/Replace)** |
| Microsoft.AspNet.WebApi.Core | 5.2.7 | - | REMOVE | Use ASP.NET Core controllers |
| Microsoft.AspNet.WebApi.WebHost | 5.2.4 | - | REMOVE | Use ASP.NET Core hosting |
| Microsoft.AspNet.Web.Optimization | 1.1.3 | - | REMOVE | Replaced with alternative bundling (see Architectural Changes) |
| Unity.WebAPI | 5.4.0 | - | REMOVE | Use ASP.NET Core DI or Unity.Microsoft.DependencyInjection |
| Antlr | 3.5.0.2 | Antlr4 4.6.6 | REPLACE | Antlr3 not supported, upgrade to Antlr4 |
| **Compatible (Keep)** |
| Microsoft.AspNet.WebApi.Client | 5.2.7 | - | KEEP | HTTP client utilities still useful |
| Microsoft.AspNet.WebApi.HelpPage | 5.2.4 | - | KEEP or REMOVE | Consider replacing with Swagger/OpenAPI |
| Unity | 5.11.10 | - | KEEP or UPDATE | Can keep if using Unity.Microsoft.DependencyInjection |
| Modernizr | 2.8.3 | - | KEEP | Front-end library, no change needed |
| WebGrease | 1.6.0 | - | KEEP or REMOVE | Used by bundling; may remove if removing bundling |

##### 4. Expected Breaking Changes

**?? Major Architectural Changes Required:**

###### A. Application Initialization: Global.asax.cs ? Program.cs

**Current (ASP.NET Framework):**
- Application startup in Global.asax.cs
- Application_Start(), RegisterRoutes(), RegisterBundles(), etc.

**Target (ASP.NET Core):**
- Program.cs with minimal hosting model
- Middleware pipeline configuration

**Migration Tasks:**
1. Create new Program.cs file
2. Migrate Application_Start logic to Program.cs
3. Migrate route registration to controller routing
4. Migrate DI container setup
5. Migrate configuration loading
6. Remove Global.asax.cs after migration

**Example Structure:**
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();

// Migrate Unity DI registrations here (or use built-in DI)

var app = builder.Build();

// Configure middleware pipeline
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

###### B. Bundling and Minification: System.Web.Optimization ? Alternatives

**Current:**
- System.Web.Optimization (Microsoft.AspNet.Web.Optimization package)
- BundleConfig.cs with script/style bundles
- @Scripts.Render(), @Styles.Render() in views

**Problem:** No direct equivalent in ASP.NET Core

**Migration Options:**

**Option 1 (Simplest - Recommended for MVP):** Direct Link Tags
- Remove bundling entirely
- Use individual `<script>` and `<link>` tags
- Acceptable for small applications

**Option 2 (Build-time):** Webpack, Gulp, or similar
- Modern front-end build tooling
- Better optimization but higher setup effort

**Option 3 (Runtime):** WebOptimizer package
- NuGet: LigerShark.WebOptimizer
- Similar to old bundling, ASP.NET Core compatible

**Recommended Approach:** Start with Option 1 (unblock migration), consider Option 3 or 2 as enhancement

###### C. Dependency Injection: Unity.WebAPI ? ASP.NET Core DI

**Current:**
- Unity container with Unity.WebAPI integration
- DI registered in Global.asax or UnityConfig

**Problem:** Unity.WebAPI not compatible with ASP.NET Core

**Migration Options:**

**Option 1 (Simplest - Recommended):** Use Built-in ASP.NET Core DI
- Services.AddScoped/AddSingleton/AddTransient in Program.cs
- Zero additional packages
- May require adjusting lifetime scopes

**Option 2:** Unity.Microsoft.DependencyInjection
- NuGet: Unity.Microsoft.DependencyInjection
- Keeps Unity container, integrates with ASP.NET Core
- Use if Unity-specific features required

**Migration Tasks:**
1. Audit existing Unity registrations
2. Choose Option 1 or 2
3. Re-register services in Program.cs using chosen approach
4. Update any Unity-specific patterns (if applicable)

###### D. Controllers and Routing

**ASP.NET Framework ? ASP.NET Core Differences:**

- **Namespace:** `System.Web.Mvc` / `System.Web.Http` ? `Microsoft.AspNetCore.Mvc`
- **Base Class:** `ApiController` ? `ControllerBase` (for API controllers)
- **Attributes:** `[RoutePrefix]`, `[Route]` ? Same but slightly different conventions
- **Action Results:** Mostly compatible but some changes (e.g., `Ok()` vs `new OkResult()`)

**Migration Tasks:**
1. Update using statements: `using Microsoft.AspNetCore.Mvc;`
2. Change base class: `public class MyController : ControllerBase`
3. Review attribute routing (mostly compatible)
4. Test action results (mostly compatible)

###### E. Configuration: Web.config ? appsettings.json

**Current:**
- Web.config for app settings, connection strings
- ConfigurationManager.AppSettings

**Target:**
- appsettings.json (and appsettings.Development.json)
- IConfiguration dependency injection

**Migration Tasks:**
1. Create appsettings.json
2. Move connection strings from web.config
3. Move app settings from web.config
4. Update code to inject IConfiguration instead of ConfigurationManager

##### 5. Code Modifications

**Expected File Changes:**

1. **Create New Files:**
   - `Program.cs` - Application entry point
   - `appsettings.json` - Configuration
   - `appsettings.Development.json` - Development configuration

2. **Modify Existing Files:**
   - **Controllers** (Controllers/*.cs):
     - Update using statements
     - Change base class to ControllerBase
     - Verify attribute routing
   - **Views** (if any):
     - Update bundling references to direct links
     - Update tag helpers (if using Razor)

3. **Remove Files:**
   - `Global.asax.cs` - Replaced by Program.cs
   - `Global.asax` - Not needed
   - `Web.config` - Mostly replaced by appsettings.json (may keep for IIS settings)
   - `BundleConfig.cs` - If removing bundling
   - `UnityConfig.cs` - If moving to built-in DI

**Code Pattern Updates:**

```csharp
// OLD: ASP.NET Framework
using System.Web.Http;
public class ValuesController : ApiController
{
    public IHttpActionResult Get()
    {
        return Ok(new[] { "value1", "value2" });
    }
}

// NEW: ASP.NET Core
using Microsoft.AspNetCore.Mvc;
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

##### 6. Testing Strategy

**Build Validation:**
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] All package updates applied successfully
- [ ] No package dependency conflicts

**Functional Testing:**
- [ ] Application starts successfully
- [ ] Routing works (API endpoints accessible)
- [ ] Controllers respond correctly
- [ ] Static files served (CSS, JS, images)
- [ ] Database access functional (via DataStore)
- [ ] Dependency injection resolves services correctly

**Security Testing:**
- [ ] Bootstrap updated to 5.3.8 (vulnerability fixed)
- [ ] jQuery updated to 3.7.1 (vulnerability fixed)
- [ ] No remaining security warnings from NuGet

**Integration Testing:**
- [ ] Common and DataStore project references work
- [ ] End-to-end API calls succeed
- [ ] Response formats correct (JSON serialization)

**Regression Testing:**
- [ ] All existing API endpoints still functional
- [ ] Business logic unchanged
- [ ] Data access patterns work

##### 7. Validation Checklist

- [ ] SDK-style conversion successful (Sdk="Microsoft.NET.Sdk.Web")
- [ ] TargetFramework set to net10.0
- [ ] All package updates applied (19 packages addressed)
- [ ] Security vulnerabilities fixed (bootstrap, jQuery)
- [ ] Program.cs created with proper configuration
- [ ] Global.asax.cs logic migrated
- [ ] Bundling/minification addressed (Option 1/2/3 chosen)
- [ ] Dependency injection migrated (Unity ? Core DI or Unity adapter)
- [ ] Controllers updated to ASP.NET Core patterns
- [ ] Configuration migrated (Web.config ? appsettings.json)
- [ ] Build succeeds with zero errors
- [ ] Build succeeds with zero warnings
- [ ] Application runs and serves requests
- [ ] All tests pass (manual or automated)
- [ ] No package dependency conflicts
- [ ] Project references intact (Common, DataStore)
- [ ] Git commits created with clear messages

#### Tier 3 Completion Criteria

? **Ready to proceed to Tier 4 when:**
- WebApiExample.WebApp builds cleanly on net10.0
- All 19 packages updated/removed/replaced
- Security vulnerabilities addressed
- Application starts and runs successfully
- All API endpoints functional
- No errors or warnings
- Dependent project (WebApp.Tests, still on net48) can reference upgraded WebApp
- Changes committed to source control

#### Tier 3 Risk Mitigation

**This is the highest-risk tier** - multiple architectural changes converging. Mitigation strategies:

1. **Incremental Approach Within Tier:**
   - Phase 3a: SDK conversion + package updates only
   - Phase 3b: Global.asax ? Program.cs migration
   - Phase 3c: Bundling replacement
   - Phase 3d: DI migration
   - Phase 3e: Final validation

2. **Fallback Options:**
   - If bundling blocks: Use direct link tags (simplest)
   - If Unity DI blocks: Use built-in DI (simplest)
   - If major issues: Revert tier, reassess, seek expert consultation

3. **Knowledge Resources:**
   - Microsoft ASP.NET Core migration documentation
   - Community migration guides (ASP.NET Framework ? Core)
   - Stack Overflow for specific error patterns

---

### Tier 4: WebApiExample.WebApp.Tests

#### Current State

- **Target Framework:** net48
- **SDK-style:** False (legacy project format)
- **Project Type:** ClassLibrary (Test Project)
- **Dependencies:** 3 (Common, DataStore, WebApp - all on net10.0 after Tiers 1-3)
- **Dependants:** 0 (leaf node in consumption tree)
- **Files:** 4
- **Lines of Code:** 393
- **Packages:** 14 packages requiring updates/removal
- **Risk Level:** ?? Medium (test framework compatibility, many package updates)

#### Target State

- **Target Framework:** net10.0
- **SDK-style:** True
- **Package Updates:** 14 packages (mostly framework-included packages to remove)

#### Migration Steps

##### 1. Prerequisites

- ? **Tier 1 Complete:** WebApiExample.Common on net10.0 and stable
- ? **Tier 2 Complete:** WebApiExample.DataStore on net10.0 and stable
- ? **Tier 3 Complete:** WebApiExample.WebApp on net10.0 and stable
- ? .NET 10.0 SDK installed
- ? Working on branch: `upgrade-to-NET10`

##### 2. Project File Conversion

**Convert to SDK-style:**

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>
  
  <ItemGroup>
    <ProjectReference Include="..\WebApiExample.Common\WebApiExample.Common.csproj" />
    <ProjectReference Include="..\WebApiExample.DataStore\WebApiExample.DataStore.csproj" />
    <ProjectReference Include="..\WebApiExample.WebApp\WebApiExample.WebApp.csproj" />
  </ItemGroup>
  
  <!-- Package references - see Package Updates section -->
</Project>
```

##### 3. Package Updates

**Tier 4 Package Updates Table:**

| Package | Current Version | Target Version | Action | Reason |
|---------|----------------|----------------|--------|---------|
| **Framework-Included (Remove)** |
| Microsoft.AspNet.Mvc | 5.2.7 | - | REMOVE | Incompatible with .NET Core |
| Microsoft.AspNet.Razor | 3.2.7 | - | REMOVE | Incompatible with .NET Core |
| Microsoft.AspNet.WebApi.Core | 5.2.7 | - | REMOVE | Incompatible with .NET Core |
| Microsoft.AspNet.WebApi.WebHost | 5.2.7 | - | REMOVE | Incompatible with .NET Core |
| Microsoft.AspNet.WebPages | 3.2.7 | - | REMOVE | Incompatible with .NET Core |
| Microsoft.Web.Infrastructure | 1.0.0.0 | - | REMOVE | Not needed in .NET Core |
| System.Buffers | 4.5.1 | - | REMOVE | Included in .NET Core framework |
| System.Memory | 4.5.4 | - | REMOVE | Included in .NET Core framework |
| System.Numerics.Vectors | 4.5.0 | - | REMOVE | Included in .NET Core framework |
| System.Runtime.InteropServices.RuntimeInformation | 4.3.0 | - | REMOVE | Included in .NET Core framework |
| System.Threading.Tasks.Extensions | 4.5.4 | - | REMOVE | Included in .NET Core framework |
| **Recommended Upgrades** |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | UPDATE | Better compatibility and features |
| System.Runtime.CompilerServices.Unsafe | 4.5.3 | 6.1.2 | UPDATE | Framework compatibility |
| **Compatible (Keep)** |
| Microsoft.AspNet.WebApi.Client | 5.2.7 | - | KEEP | HTTP client utilities still useful |
| Castle.Core | 4.4.0 | - | KEEP | Moq dependency, compatible |
| DiffEngine | 6.4.9 | - | KEEP | Test utility, compatible |
| EmptyFiles | 2.3.3 | - | KEEP | Test utility, compatible |
| Microsoft.CSharp | 4.7.0 | - | KEEP | Dynamic language support |
| Moq | 4.16.1 | - | KEEP | Mocking framework, compatible |
| Shouldly | 4.0.3 | - | KEEP | Assertion library, compatible |
| xunit | 2.4.1 | - | KEEP | Test framework, compatible |
| xunit.* (all) | 2.4.x | - | KEEP | xUnit packages, compatible |

**Summary:** 11 packages to remove, 2 to update, 11 to keep

##### 4. Expected Breaking Changes

**Potential Issues:**

###### A. Test Host Changes

**ASP.NET Framework ? ASP.NET Core Testing:**
- In-memory test server patterns may differ
- WebApplicationFactory<T> is standard for ASP.NET Core integration tests
- May need Microsoft.AspNetCore.Mvc.Testing package

**Migration:**
- If tests use HttpServer or similar: Replace with WebApplicationFactory
- If tests mock HttpContext: Update to ASP.NET Core HttpContext patterns

###### B. Package Removals Impact

**Microsoft.AspNet.* Package Removals:**
- Tests referencing ASP.NET Framework types will break
- Need to update to ASP.NET Core equivalents

**System.* Package Removals:**
- No action needed - types still available via framework

###### C. Test Patterns

**Controller Testing:**
```csharp
// OLD: ASP.NET Framework
var controller = new MyController();
var result = controller.Get() as OkNegotiatedContentResult<MyModel>;

// NEW: ASP.NET Core
var controller = new MyController();
var result = controller.Get() as OkObjectResult;
var model = result.Value as MyModel;
```

##### 5. Code Modifications

**Expected Code Changes:**

1. **Using Statements:**
   - Remove: `using System.Web.Http;`, `using System.Web.Mvc;`
   - Add: `using Microsoft.AspNetCore.Mvc;`

2. **Test Setup:**
   - Update test initialization if setting up HttpContext or similar
   - Use ASP.NET Core test patterns

3. **Assertions:**
   - Update result type assertions (OkNegotiatedContentResult ? OkObjectResult)
   - Verify Shouldly, xUnit assertions still work (should be compatible)

4. **Integration Tests (if any):**
   - Replace HttpServer with WebApplicationFactory
   - Add Microsoft.AspNetCore.Mvc.Testing if needed

**Example Integration Test Migration:**
```csharp
// If using integration tests, consider:
public class IntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    
    public IntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }
    
    [Fact]
    public async Task GetValues_ReturnsSuccess()
    {
        var response = await _client.GetAsync("/api/values");
        response.EnsureSuccessStatusCode();
    }
}
```

##### 6. Testing Strategy

**Build Validation:**
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] All package updates/removals applied successfully
- [ ] No package dependency conflicts

**Test Execution:**
- [ ] All unit tests run successfully
- [ ] All unit tests pass (same count as before migration)
- [ ] No skipped or failing tests
- [ ] Test coverage maintained

**Test Framework Validation:**
- [ ] xUnit runner discovers all tests
- [ ] Moq mocking works as expected
- [ ] Shouldly assertions functional
- [ ] Test isolation maintained

**Integration Testing (if applicable):**
- [ ] Integration tests run against .NET 10.0 WebApp
- [ ] HTTP calls to WebApp succeed
- [ ] Test database operations work

**Project Reference Validation:**
- [ ] References to Common, DataStore, WebApp work correctly
- [ ] All three dependencies on net10.0

##### 7. Validation Checklist

- [ ] SDK-style conversion successful
- [ ] TargetFramework set to net10.0
- [ ] IsTestProject property set to true
- [ ] All 14 packages addressed (11 removed, 2 updated, 11 kept)
- [ ] Using statements updated
- [ ] Test code updated for ASP.NET Core patterns
- [ ] Build succeeds with zero errors
- [ ] Build succeeds with zero warnings
- [ ] All tests discovered by runner
- [ ] All tests pass (100% pass rate)
- [ ] No package dependency conflicts
- [ ] Project references intact (Common, DataStore, WebApp)
- [ ] Git commit created with clear message

#### Tier 4 Completion Criteria

? **Migration Complete When:**
- WebApiExample.WebApp.Tests builds cleanly on net10.0
- All 14 packages updated/removed appropriately
- All tests discovered and passing
- No errors or warnings
- All project references functional
- Test coverage maintained (same test count as before)
- Changes committed to source control

**This marks the completion of the entire migration** - all 4 projects now on .NET 10.0!

#### Tier 4 Risk Mitigation

**If Tests Fail After Migration:**

1. **Categorize Failures:**
   - Compilation errors (missing types) ? Update using statements, types
   - Runtime errors (test setup) ? Update test initialization patterns
   - Assertion failures (logic) ? Verify WebApp functionality first (likely WebApp issue)

2. **Incremental Approach:**
   - Fix compilation errors first (get to building state)
   - Fix one failing test at a time
   - Use failing test patterns to fix similar tests

3. **Fallback:**
   - If specific test cannot be fixed: Document as known issue, skip test temporarily
   - If entire test suite broken: Revert tier, reassess test patterns

**Test Failure Decision Tree:**
- ? Test fails due to code change (expected) ? Fix assertion
- ? Test fails due to framework change ? Update test pattern
- ? Test fails but WebApp works ? Likely test issue, update test
- ? Test fails and WebApp broken ? WebApp issue, address in Tier 3

## Package Update Reference

### Consolidated Package Updates by Tier

This section provides a centralized reference for all package changes across the solution.

#### Tier 1: WebApiExample.Common

**No package updates required** - project has no NuGet dependencies.

#### Tier 2: WebApiExample.DataStore

| Package | Current Version | Target Version | Action |
|---------|----------------|----------------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | UPDATE |

**Total:** 1 package update

#### Tier 3: WebApiExample.WebApp

**Security Vulnerabilities (Critical):**

| Package | Current Version | Target Version | Action |
|---------|----------------|----------------|--------|
| bootstrap | 3.3.7 | 5.3.8 | UPDATE |
| jQuery | 3.3.1 | 3.7.1 | UPDATE |

**Recommended Updates:**

| Package | Current Version | Target Version | Action |
|---------|----------------|----------------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | UPDATE |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | UPDATE |
| System.Runtime.CompilerServices.Unsafe | 4.5.2 | 6.1.2 | UPDATE |

**Framework-Included (Remove):**

| Package | Current Version | Action | Replacement |
|---------|----------------|--------|-------------|
| Microsoft.AspNet.Mvc | 5.2.4 | REMOVE | ASP.NET Core framework |
| Microsoft.AspNet.Razor | 3.2.4 | REMOVE | ASP.NET Core framework |
| Microsoft.AspNet.WebApi | 5.2.4 | REMOVE | ASP.NET Core framework |
| Microsoft.AspNet.WebPages | 3.2.4 | REMOVE | ASP.NET Core framework |
| Microsoft.CodeDom.Providers.DotNetCompilerPlatform | 2.0.0 | REMOVE | Roslyn built into .NET Core |
| Microsoft.Web.Infrastructure | 1.0.0.0 | REMOVE | Not needed |

**Incompatible (Remove/Replace):**

| Package | Current Version | Action | Replacement/Notes |
|---------|----------------|--------|-------------------|
| Microsoft.AspNet.WebApi.Core | 5.2.7 | REMOVE | Use ASP.NET Core controllers |
| Microsoft.AspNet.WebApi.WebHost | 5.2.4 | REMOVE | Use ASP.NET Core hosting |
| Microsoft.AspNet.Web.Optimization | 1.1.3 | REMOVE | See bundling alternatives |
| Unity.WebAPI | 5.4.0 | REMOVE | Use ASP.NET Core DI or Unity adapter |
| Antlr | 3.5.0.2 | REPLACE | Replace with Antlr4 4.6.6 |

**Compatible (Keep):**

| Package | Current Version | Action | Notes |
|---------|----------------|--------|-------|
| Microsoft.AspNet.WebApi.Client | 5.2.7 | KEEP | HTTP client utilities |
| Microsoft.AspNet.WebApi.HelpPage | 5.2.4 | KEEP/REMOVE | Consider Swagger instead |
| Unity | 5.11.10 | KEEP/UPDATE | If using Unity.Microsoft.DependencyInjection |
| Modernizr | 2.8.3 | KEEP | Front-end library |
| WebGrease | 1.6.0 | KEEP/REMOVE | Bundling dependency, may remove |

**Total:** 19 packages addressed (5 updates, 10 removals, 1 replacement, 3 decisions)

#### Tier 4: WebApiExample.WebApp.Tests

**Framework-Included (Remove):**

| Package | Current Version | Action |
|---------|----------------|--------|
| Microsoft.AspNet.Mvc | 5.2.7 | REMOVE |
| Microsoft.AspNet.Razor | 3.2.7 | REMOVE |
| Microsoft.AspNet.WebApi.Core | 5.2.7 | REMOVE |
| Microsoft.AspNet.WebApi.WebHost | 5.2.7 | REMOVE |
| Microsoft.AspNet.WebPages | 3.2.7 | REMOVE |
| Microsoft.Web.Infrastructure | 1.0.0.0 | REMOVE |
| System.Buffers | 4.5.1 | REMOVE |
| System.Memory | 4.5.4 | REMOVE |
| System.Numerics.Vectors | 4.5.0 | REMOVE |
| System.Runtime.InteropServices.RuntimeInformation | 4.3.0 | REMOVE |
| System.Threading.Tasks.Extensions | 4.5.4 | REMOVE |

**Recommended Updates:**

| Package | Current Version | Target Version | Action |
|---------|----------------|----------------|--------|
| Newtonsoft.Json | 11.0.1 | 13.0.4 | UPDATE |
| System.Runtime.CompilerServices.Unsafe | 4.5.3 | 6.1.2 | UPDATE |

**Compatible (Keep):**

| Package | Current Version | Action |
|---------|----------------|--------|
| Microsoft.AspNet.WebApi.Client | 5.2.7 | KEEP |
| Castle.Core | 4.4.0 | KEEP |
| DiffEngine | 6.4.9 | KEEP |
| EmptyFiles | 2.3.3 | KEEP |
| Microsoft.CSharp | 4.7.0 | KEEP |
| Moq | 4.16.1 | KEEP |
| Shouldly | 4.0.3 | KEEP |
| xunit | 2.4.1 | KEEP |
| xunit.* (all packages) | 2.4.x | KEEP |

**Total:** 14 packages addressed (2 updates, 11 removals, 11 kept)

### Solution-Wide Package Summary

| Action | Count | Packages |
|--------|-------|----------|
| **UPDATE** | 6 unique | EntityFramework, bootstrap, jQuery, Newtonsoft.Json, System.Runtime.CompilerServices.Unsafe (2 projects) |
| **REMOVE** | 17 unique | Microsoft.AspNet.* (8), System.* (5), Microsoft.Web.Infrastructure, Microsoft.CodeDom.*, Unity.WebAPI, WebGrease |
| **REPLACE** | 1 | Antlr ? Antlr4 |
| **KEEP** | 22 | xUnit suite, Moq, Shouldly, Unity, Modernizr, etc. |
| **Total Packages** | 46 | 14 require action |

---

## Breaking Changes Catalog

### Framework-Level Breaking Changes

#### 1. .NET Framework 4.8 ? .NET 10.0

**Major API Differences:**

- **System.Web namespace:** Not available in .NET Core
  - No HttpContext.Current
  - No Server.MapPath
  - No HttpRuntime
  
- **Configuration:** ConfigurationManager ? IConfiguration (DI)
- **App Domains:** Limited support (mostly not needed)
- **Remoting:** Not supported (use modern alternatives)

**Impact:** Low for this solution (web-focused, not using advanced .NET Framework features)

#### 2. ASP.NET Framework ? ASP.NET Core

**Application Model Changes:**

| ASP.NET Framework | ASP.NET Core | Breaking? |
|-------------------|--------------|-----------|
| Global.asax.cs | Program.cs | ? Yes - requires migration |
| Web.config | appsettings.json | ? Yes - requires migration |
| System.Web.Http.ApiController | Microsoft.AspNetCore.Mvc.ControllerBase | ? Yes - namespace change |
| RouteConfig | app.MapControllers() | ? Yes - pattern change |
| BundleConfig | Alternative bundling | ? Yes - no direct replacement |

**Hosting Model:**
- IIS in-process/out-of-process ? Kestrel with optional IIS reverse proxy
- Application lifecycle completely different
- Middleware pipeline instead of HTTP modules

**Impact:** High for WebApp project - requires significant architectural changes

### Package-Level Breaking Changes

#### 1. Entity Framework 6.4.4 ? 6.5.1

**Breaking Changes:** Minimal

- EF 6.5.1 is mainly a compatibility release for .NET Core
- No major API changes expected
- Configuration model difference (app.config ? code-based)

**Impact:** Low - mostly configuration adjustments

#### 2. bootstrap 3.3.7 ? 5.3.8

**Breaking Changes:** Major

Bootstrap 3 ? 5 is a major version jump with significant changes:

- **Grid System:** .col-xs-* ? .col-*
- **Components:** Many renamed or restructured
- **jQuery Dependency:** Bootstrap 5 no longer requires jQuery
- **Responsive Utilities:** Different class names
- **JavaScript:** Changed plugin initialization

**Migration Path:**
- Review Bootstrap 5 migration guide
- Update HTML/CSS to Bootstrap 5 syntax
- Test responsive layouts
- May require front-end developer assistance

**Impact:** Medium to High - depends on Bootstrap usage depth

#### 3. jQuery 3.3.1 ? 3.7.1

**Breaking Changes:** Minimal

jQuery 3.3 ? 3.7 is a minor version increment:

- Mostly bug fixes and compatibility improvements
- Deprecated methods may have been removed (check release notes)
- Generally backward compatible

**Impact:** Low - mostly compatible

#### 4. Newtonsoft.Json 11.0.1 ? 13.0.4

**Breaking Changes:** Minimal

- Version 13 maintains backward compatibility
- Performance improvements
- Better .NET Core integration
- Some edge cases may behave differently (date serialization, nulls)

**Impact:** Low - mostly compatible

#### 5. Antlr 3.5.0.2 ? Antlr4 4.6.6

**Breaking Changes:** Major

Antlr 3 ? Antlr 4 is a major rewrite:

- Grammar syntax changed
- Runtime API completely different
- Parser generation process changed

**Migration Path:**
- Regenerate grammars with Antlr4
- Update code using parser APIs
- Requires understanding of grammar usage in solution

**Impact:** High IF Antlr is actively used; Unknown IF it's a transitive dependency

**?? Investigation Required:** Determine if solution directly uses Antlr or if it's a dependency of another package (likely WebGrease). If transitive, may be removable.

### ASP.NET Core-Specific Breaking Changes

#### 1. Controller Base Class

```csharp
// BEFORE
public class MyController : ApiController { }

// AFTER
public class MyController : ControllerBase { }
```

**Impact:** All controllers need base class change

#### 2. Action Results

```csharp
// BEFORE
return Ok(data); // Returns OkNegotiatedContentResult<T>

// AFTER  
return Ok(data); // Returns OkObjectResult
```

**Impact:** Test assertions may need updates

#### 3. HTTP Context

```csharp
// BEFORE
var user = HttpContext.Current.User;

// AFTER
var user = HttpContext.User; // Injected or from ControllerBase.HttpContext
```

**Impact:** Any direct HttpContext.Current usage must be refactored

#### 4. Configuration Access

```csharp
// BEFORE
var setting = ConfigurationManager.AppSettings["MySetting"];

// AFTER (inject IConfiguration)
private readonly IConfiguration _config;
public MyController(IConfiguration config) { _config = config; }
var setting = _config["MySetting"];
```

**Impact:** All configuration access must use dependency injection

### Dependency Injection Breaking Changes

#### Unity Container ? ASP.NET Core DI

**If migrating to built-in DI:**

```csharp
// BEFORE (Unity)
container.RegisterType<IMyService, MyService>(new HierarchicalLifetimeManager());

// AFTER (ASP.NET Core DI)
services.AddScoped<IMyService, MyService>();
```

**Lifetime Mapping:**
- ContainerControlledLifetimeManager ? AddSingleton
- HierarchicalLifetimeManager ? AddScoped
- TransientLifetimeManager ? AddTransient

**Impact:** All service registrations need conversion

### Known Compatibility Issues

#### 1. Entity Framework 6 on .NET Core

**Known Limitations:**
- Some providers (e.g., Oracle) may not support .NET Core
- Async operations may behave slightly differently
- No designer support in Visual Studio for .NET Core projects

**Workaround:** These are generally acceptable for most applications; monitor for edge cases

#### 2. System.Web.Optimization (No Replacement)

**Issue:** No built-in bundling/minification in ASP.NET Core

**Options:**
- Use direct link tags (accept larger payload)
- Use WebOptimizer NuGet package
- Use build-time bundling (webpack, gulp)

#### 3. Unity.WebAPI (No Direct Port)

**Issue:** Unity.WebAPI package specifically for ASP.NET Framework

**Options:**
- Use Unity.Microsoft.DependencyInjection adapter
- Migrate to ASP.NET Core built-in DI

### Breaking Change Mitigation Summary

| Breaking Change | Severity | Mitigation Approach |
|-----------------|----------|---------------------|
| Global.asax ? Program.cs | High | Documented migration pattern in plan |
| Web.config ? appsettings.json | Medium | Direct mapping of settings |
| ApiController ? ControllerBase | Low | Find/replace + using statement update |
| Bundling removal | Medium | Start with direct links, enhance later |
| Unity.WebAPI | Medium | Built-in DI or Unity adapter |
| Bootstrap 3 ? 5 | Medium-High | Follow Bootstrap migration guide |
| Antlr 3 ? 4 | Unknown | Investigate usage depth first |
| EF6 configuration | Low | Code-based config instead of app.config |

## Risk Management

### High-Level Risk Assessment

| Tier | Projects | Risk Level | Primary Risks | Mitigation Strategy |
|------|----------|------------|---------------|---------------------|
| Tier 1 | Common | ?? Low | Minimal - no packages, simple models | Straightforward SDK conversion, validate API compatibility |
| Tier 2 | DataStore | ?? Medium | Entity Framework 6 compatibility on .NET Core | Test EF6 operations thoroughly, consider EF Core migration path if issues |
| Tier 3 | WebApp | ?? High | ASP.NET Framework ? Core conversion, bundling, DI, security vulnerabilities | Detailed architectural migration plan, incremental testing, address CVEs |
| Tier 4 | Tests | ?? Medium | Test framework compatibility, package updates | Update test patterns, validate test coverage maintained |

### Critical Risk Factors

#### ?? Security Vulnerabilities (HIGH PRIORITY)

**Affected Packages:**
- `bootstrap` 3.3.7 (vulnerable) ? 5.3.8 (secure) in WebApp
- `jquery` 3.3.1 (vulnerable) ? 3.7.1 (secure) in WebApp

**Risk:** Exploitable security issues in production
**Mitigation:** 
- Address in Tier 3 (WebApp) migration
- Update to secure versions as part of package update phase
- Validate functionality with updated versions
- Review breaking changes in bootstrap 3?5 migration guide

#### ?? Architectural Breaking Changes (HIGH COMPLEXITY)

**ASP.NET Framework ? ASP.NET Core Conversion (Tier 3):**

1. **Global.asax.cs Initialization**
   - Risk: Application startup logic incompatible
   - Mitigation: Migrate to Program.cs with minimal hosting model, map routes to controllers

2. **System.Web.Optimization Bundling/Minification**
   - Risk: No direct replacement in .NET Core
   - Mitigation: Replace with static link tags or build-time bundling (webpack, gulp)

3. **Unity.WebAPI Dependency Injection**
   - Risk: Unity.WebAPI not compatible with ASP.NET Core
   - Mitigation: Migrate to built-in ASP.NET Core DI or update Unity integration approach

#### ?? Entity Framework 6 Compatibility (MEDIUM RISK)

**Affected:** Tier 2 (DataStore)

- **Risk:** EF6 runs on .NET Core but with limitations
- **Mitigation:** 
  - Test all EF6 operations after migration
  - Validate database connectivity and queries
  - Document any behavioral differences
  - Consider future migration to EF Core (out of scope for this plan)

#### ?? Package Incompatibilities (MEDIUM RISK)

**Incompatible Packages Requiring Removal/Replacement:**
- Microsoft.AspNet.Mvc ? Functionality in ASP.NET Core framework
- Microsoft.AspNet.Razor ? Functionality in ASP.NET Core framework
- Microsoft.AspNet.WebApi ? Functionality in ASP.NET Core framework
- Microsoft.AspNet.WebPages ? Functionality in ASP.NET Core framework
- Microsoft.AspNet.WebApi.Core ? Replaced by ASP.NET Core controllers
- Microsoft.AspNet.WebApi.WebHost ? Replaced by ASP.NET Core hosting
- Microsoft.AspNet.Web.Optimization ? Replace with alternative bundling
- Microsoft.Web.Infrastructure ? Functionality in framework

**Mitigation:** Remove packages, rely on framework, update code to use Core APIs

### Contingency Plans

#### If Entity Framework 6 Issues Arise (Tier 2)

**Option A:** Continue with EF6 on .NET Core (supported but not recommended long-term)
- Document limitations
- Plan future EF Core migration

**Option B:** Migrate to EF Core during this upgrade
- Higher effort, extends timeline
- Better long-term compatibility
- Requires data access pattern updates

**Recommendation:** Start with Option A, collect data, plan Option B for future sprint

#### If Bundling Replacement Blocks (Tier 3)

**Option A:** Use direct link tags (simplest)
- Remove bundling entirely
- Reference individual CSS/JS files
- Accept slightly larger payload (acceptable for small app)

**Option B:** Build-time bundling
- Integrate webpack/gulp/other bundler
- Higher effort, better optimization
- Requires build pipeline changes

**Recommendation:** Start with Option A for unblocking, consider Option B as enhancement

#### If Unity.WebAPI Cannot Be Replaced

**Option A:** Use built-in ASP.NET Core DI
- Simplest, zero additional packages
- Requires updating registration code
- May need pattern adjustments if Unity-specific features used

**Option B:** Use Unity.Microsoft.DependencyInjection adapter
- Maintains Unity container
- Integrates with ASP.NET Core DI
- Minimal code changes

**Recommendation:** Evaluate DI usage complexity, prefer Option A unless Unity-specific features critical

## Testing & Validation Strategy

### Multi-Level Testing Approach

This migration follows a tiered testing strategy aligned with the bottom-up dependency migration approach. Each tier is validated before proceeding to the next, ensuring cumulative stability.

---

### Tier 1 Testing: WebApiExample.Common

**Objective:** Validate that the foundational shared library works on .NET 10.0

#### Smoke Tests (Quick Validation)

- [ ] Project builds successfully
- [ ] No build errors
- [ ] No build warnings
- [ ] Project file is valid SDK-style format

#### Unit Testing

- [ ] No unit tests in this project (models/utilities only)
- [ ] If utilities exist, verify they function correctly

#### API Compatibility Testing

- [ ] All public types remain accessible
- [ ] No breaking changes in public API surface
- [ ] Serialization works (if models use serialization attributes)

#### Integration Testing

- [ ] Dependent projects (DataStore, WebApp, WebApp.Tests on net48) can reference Common
- [ ] Solution builds with mixed frameworks (Common on net10.0, others on net48)
- [ ] No package restore conflicts

**Pass Criteria:** ? All smoke tests pass, integration tests confirm dependent projects unaffected

---

### Tier 2 Testing: WebApiExample.DataStore

**Objective:** Validate that data access layer works on .NET 10.0 with Entity Framework 6.5.1

#### Smoke Tests

- [ ] Project builds successfully
- [ ] No build errors
- [ ] No build warnings
- [ ] EntityFramework 6.5.1 package restores correctly
- [ ] Common project reference intact

#### Entity Framework Validation

- [ ] DbContext can be instantiated
- [ ] Database connection succeeds (test connection string)
- [ ] Basic CRUD operations work:
  - [ ] Create entity
  - [ ] Read entity
  - [ ] Update entity
  - [ ] Delete entity
- [ ] LINQ queries execute correctly
- [ ] Migrations apply (if project uses EF migrations)

#### Unit Testing

- [ ] No unit tests in this project (or run if they exist)
- [ ] Data access patterns function correctly

#### Integration Testing

- [ ] Common + DataStore build together
- [ ] Dependent projects (WebApp, WebApp.Tests on net48) can reference DataStore
- [ ] Solution builds with mixed frameworks

**Pass Criteria:** ? All EF operations functional, dependent projects unaffected

---

### Tier 3 Testing: WebApiExample.WebApp

**Objective:** Validate that ASP.NET Core application works on .NET 10.0

This is the most critical testing phase due to the architectural complexity.

#### Smoke Tests

- [ ] Project builds successfully
- [ ] No build errors
- [ ] No build warnings
- [ ] All 19 package updates/removals/replacements applied
- [ ] Project references intact (Common, DataStore)

#### Application Startup Testing

- [ ] Application starts without errors
- [ ] Program.cs executes successfully
- [ ] Middleware pipeline configured correctly
- [ ] Dependency injection container builds
- [ ] No startup exceptions

#### Functional Testing - API Endpoints

Test each API endpoint individually:

- [ ] GET endpoints respond with 200 OK
- [ ] POST endpoints accept data and respond correctly
- [ ] PUT endpoints update data correctly
- [ ] DELETE endpoints remove data correctly
- [ ] Error handling works (404, 500, etc.)
- [ ] Response formats correct (JSON serialization)

#### Functional Testing - Static Files

- [ ] CSS files served correctly
- [ ] JavaScript files served correctly
- [ ] Images/fonts served correctly
- [ ] Bundling/minification approach works (or direct links)

#### Data Access Testing

- [ ] Controllers can access DataStore
- [ ] Database queries succeed
- [ ] Entity Framework operations work through API layer
- [ ] Transactions work correctly

#### Dependency Injection Testing

- [ ] Services resolve from DI container
- [ ] Scoped lifetimes work correctly
- [ ] Singleton services persist
- [ ] Transient services instantiate correctly

#### Security Testing

- [ ] Bootstrap 5.3.8 loaded (verify in browser DevTools)
- [ ] jQuery 3.7.1 loaded (verify in browser DevTools)
- [ ] No security warnings from NuGet packages
- [ ] HTTPS redirection works (if configured)

#### Performance Testing

- [ ] Application response times acceptable
- [ ] No obvious performance regressions
- [ ] Memory usage reasonable

#### Integration Testing

- [ ] Common + DataStore + WebApp work together
- [ ] Dependent project (WebApp.Tests on net48) can reference WebApp
- [ ] Solution builds with mixed frameworks

#### Regression Testing

**Test critical user workflows end-to-end:**
- [ ] Workflow 1: [Define based on application domain]
- [ ] Workflow 2: [Define based on application domain]
- [ ] Workflow 3: [Define based on application domain]

**Verify no regressions in:**
- [ ] Business logic
- [ ] Data validation
- [ ] Error handling
- [ ] User experience

**Pass Criteria:** ? Application runs, all endpoints functional, security issues resolved, no regressions

---

### Tier 4 Testing: WebApiExample.WebApp.Tests

**Objective:** Validate that all tests pass on .NET 10.0

#### Smoke Tests

- [ ] Project builds successfully
- [ ] No build errors
- [ ] No build warnings
- [ ] All 14 package updates/removals applied
- [ ] Project references intact (Common, DataStore, WebApp)

#### Test Discovery

- [ ] xUnit test runner discovers all tests
- [ ] Test count matches pre-migration count
- [ ] No tests missing or hidden

#### Unit Test Execution

- [ ] All unit tests execute
- [ ] All unit tests pass (100% pass rate)
- [ ] No skipped tests (unless intentionally skipped before)
- [ ] No flaky tests introduced

#### Integration Test Execution (if applicable)

- [ ] Integration tests run against .NET 10.0 WebApp
- [ ] HTTP calls to WebApp succeed
- [ ] Test database operations work
- [ ] All integration tests pass

#### Test Framework Validation

- [ ] xUnit framework works correctly
- [ ] Moq mocking library functions
- [ ] Shouldly assertions work
- [ ] Test output/logging functional

#### Coverage Validation

- [ ] Test coverage maintained (same % as before, or better)
- [ ] No reduction in tested code paths

**Pass Criteria:** ? All tests pass, coverage maintained, test count unchanged

---

### Solution-Wide Final Validation

**After all tiers complete, perform comprehensive solution testing:**

#### Build Validation

- [ ] Clean solution
- [ ] Build entire solution
- [ ] Zero errors
- [ ] Zero warnings
- [ ] All projects on net10.0
- [ ] No package conflicts

#### End-to-End Testing

- [ ] Full application workflows execute successfully
- [ ] Data flows from WebApp ? DataStore ? Database
- [ ] API responses correct and complete
- [ ] No data corruption or loss

#### Performance Validation

- [ ] Application startup time acceptable
- [ ] API response times within SLA
- [ ] Database query performance acceptable
- [ ] No memory leaks detected

#### Security Validation

- [ ] All security vulnerabilities addressed
- [ ] No new security warnings
- [ ] NuGet audit clean
- [ ] HTTPS configuration correct

#### Documentation Validation

- [ ] README updated (if applicable)
- [ ] Migration notes documented
- [ ] Known issues documented
- [ ] Deployment guide updated (if applicable)

---

### Testing Checklist Summary

**Tier 1 - Common:**
- ? 4 smoke tests
- ? API compatibility validation
- ? Integration with dependents

**Tier 2 - DataStore:**
- ? 5 smoke tests
- ? 6 EF validation tests
- ? Integration with dependents

**Tier 3 - WebApp:**
- ? 5 smoke tests
- ? 6 startup tests
- ? API endpoint testing
- ? Static file testing
- ? Data access testing
- ? DI testing
- ? Security testing
- ? Regression testing

**Tier 4 - Tests:**
- ? 5 smoke tests
- ? Test discovery
- ? Test execution (unit + integration)
- ? Coverage validation

**Solution-Wide:**
- ? Build validation
- ? End-to-end testing
- ? Performance validation
- ? Security validation

---

### Automated vs Manual Testing

**Automated Testing:**
- Unit tests (xUnit in WebApp.Tests)
- Build process (MSBuild/dotnet build)
- Package restore (dotnet restore)

**Manual Testing Required:**
- Application startup verification
- Browser-based UI testing (if applicable)
- API endpoint testing (Postman/curl)
- Visual inspection of static files
- Performance observation

**Recommended:** Create Postman collection or automated API test suite for regression testing.

---

### Test Failure Response Plan

**If Tests Fail During Migration:**

1. **Categorize the Failure:**
   - Build error (compilation)
   - Runtime error (exception)
   - Logic error (assertion failure)

2. **Isolate the Cause:**
   - Is it tier-specific or cascading?
   - Is it code, configuration, or environment?

3. **Mitigation Options:**
   - Fix immediately (if simple)
   - Document as known issue (if complex)
   - Rollback tier (if blocking)

4. **Decision Tree:**
   - 1-2 test failures: Fix individually, continue
   - 3-5 test failures: Fix batch, validate, continue
   - >5 test failures: Revert tier, reassess approach

**Never proceed to next tier with failing tests in current tier.**

## Complexity & Effort Assessment

### Relative Complexity by Tier

| Tier | Project | Complexity | LOC | Dependencies | Package Updates | Rationale |
|------|---------|------------|-----|--------------|-----------------|-----------|
| Tier 1 | Common | ?? Low | 61 | 0 | 0 | Simple models, no packages, no dependencies |
| Tier 2 | DataStore | ?? Low | 110 | 1 | 2 | Single EF6 package update, straightforward data access |
| Tier 3 | WebApp | ?? High | 3,612 | 2 | 19 | Complex ASP.NET conversion, architectural changes, security fixes |
| Tier 4 | Tests | ?? Medium | 393 | 3 | 14 | Many package updates, test pattern adjustments |

### Phase Complexity Assessment

**Phase 1 - Tier 1 (Common):**
- **Complexity:** Low
- **Effort Factors:** 
  - SDK-style conversion (simple)
  - TargetFramework change (straightforward)
  - No package updates needed
  - No code changes expected
- **Dependencies:** None to wait for
- **Risk:** Minimal

**Phase 2 - Tier 2 (DataStore):**
- **Complexity:** Low-Medium
- **Effort Factors:**
  - SDK-style conversion (simple)
  - TargetFramework change
  - EntityFramework 6.4.4 ? 6.5.1 update
  - EF6 compatibility testing
- **Dependencies:** Tier 1 complete
- **Risk:** Entity Framework compatibility edge cases

**Phase 3 - Tier 3 (WebApp):**
- **Complexity:** High
- **Effort Factors:**
  - SDK-style conversion for Web Application Project (complex)
  - TargetFramework change
  - 19 package updates/removals/replacements
  - Global.asax ? Program.cs migration
  - Bundling/optimization replacement
  - Unity.WebAPI ? ASP.NET Core DI migration
  - ASP.NET Framework ? ASP.NET Core patterns
  - Security vulnerability remediation
- **Dependencies:** Tier 1 and Tier 2 complete
- **Risk:** Multiple architectural changes, high integration surface

**Phase 4 - Tier 4 (Tests):**
- **Complexity:** Medium
- **Effort Factors:**
  - SDK-style conversion
  - TargetFramework change
  - 14 package updates (many framework-included packages to remove)
  - Test pattern updates for .NET Core
  - Moq, xUnit compatibility validation
- **Dependencies:** All application tiers complete
- **Risk:** Test framework compatibility, coverage maintenance

### Resource Requirements

**Skill Levels Needed:**

- **Tier 1-2:** Junior to mid-level developer
  - SDK-style project migration
  - Basic package updates
  - Compilation error resolution

- **Tier 3:** Senior developer with ASP.NET Core experience
  - ASP.NET Framework ? Core migration expertise
  - Dependency injection patterns
  - Web API controller migration
  - Security best practices

- **Tier 4:** Mid to senior developer with testing experience
  - Test framework updates
  - Mocking library compatibility
  - Integration test patterns

**Parallel Capacity:**

Not applicable - all tiers contain single projects, no parallelization within tiers. Migration is strictly sequential.

**Knowledge Transfer:**

- Lessons from Tier 1-2 inform Tier 3 approach (SDK conversion patterns)
- Tier 3 architectural decisions impact Tier 4 test setup
- Document decisions and patterns as tiers complete

### Complexity Rating Methodology

**Low Complexity Criteria:**
- < 500 LOC
- < 5 package updates
- No architectural changes
- Clear migration path

**Medium Complexity Criteria:**
- 500-1,000 LOC
- 5-15 package updates
- Some pattern adjustments needed
- Well-documented migration path

**High Complexity Criteria:**
- > 1,000 LOC
- > 15 package updates
- Architectural changes required
- Multiple breaking changes
- Security vulnerabilities present

## Source Control Strategy

### Branching Strategy

**Branch Structure:**

```
main (or master)
  ??? upgrade-to-NET10 (feature branch)
       ??? commit: Tier 1 complete - Common migrated to net10.0
       ??? commit: Tier 2 complete - DataStore migrated to net10.0
       ??? commit: Tier 3 complete - WebApp migrated to net10.0
       ??? commit: Tier 4 complete - WebApp.Tests migrated to net10.0
```

**Active Branch:** `upgrade-to-NET10` (already created and checked out)

**Source Branch:** `main`

**Merge Target:** `main` (after all tiers complete and validated)

---

### Commit Strategy

**Commit Frequency:** One commit per tier completion

**Rationale:**
- Each tier is a logical milestone
- Allows rollback to previous stable tier if issues arise
- Creates clear history of migration progress
- Enables code review at natural boundaries

#### Commit Template

Use this message format for tier commits:

```
[Tier X] Migrate <ProjectName> to .NET 10.0

- Convert to SDK-style project format
- Update TargetFramework to net10.0
- <Package updates specific to tier>
- <Architectural changes specific to tier>
- All tests passing
- Validated with dependent projects

Closes: <issue number if tracking>
```

#### Example Commit Messages

**Tier 1 Commit:**
```
[Tier 1] Migrate WebApiExample.Common to .NET 10.0

- Convert to SDK-style project format
- Update TargetFramework to net10.0
- No package updates required (no dependencies)
- All APIs remain compatible
- Validated with dependent projects (DataStore, WebApp, WebApp.Tests)

Closes: #123
```

**Tier 2 Commit:**
```
[Tier 2] Migrate WebApiExample.DataStore to .NET 10.0

- Convert to SDK-style project format
- Update TargetFramework to net10.0
- Update EntityFramework 6.4.4 ? 6.5.1
- Migrate EF configuration from app.config to code-based
- All EF operations validated
- Validated with dependent projects (WebApp, WebApp.Tests)

Closes: #124
```

**Tier 3 Commit:**
```
[Tier 3] Migrate WebApiExample.WebApp to .NET 10.0

- Convert to SDK-style Web project format
- Update TargetFramework to net10.0
- Migrate Global.asax ? Program.cs with minimal hosting
- Replace System.Web.Optimization bundling with direct links
- Migrate Unity.WebAPI ? ASP.NET Core DI
- Update 19 packages (removed incompatible, updated vulnerable)
- Fix security vulnerabilities: bootstrap 5.3.8, jQuery 3.7.1
- Update controllers to ASP.NET Core patterns
- Migrate Web.config ? appsettings.json
- Application runs and all endpoints functional
- Validated with dependent project (WebApp.Tests)

Closes: #125
```

**Tier 4 Commit:**
```
[Tier 4] Migrate WebApiExample.WebApp.Tests to .NET 10.0

- Convert to SDK-style test project format
- Update TargetFramework to net10.0
- Remove 11 framework-included packages
- Update Newtonsoft.Json 13.0.4, System.Runtime.CompilerServices.Unsafe 6.1.2
- Update test patterns for ASP.NET Core
- All tests passing (100% pass rate)
- Test coverage maintained

Migration Complete - All projects on .NET 10.0

Closes: #126
```

---

### Working with Changes

#### Before Starting Each Tier

```bash
# Ensure on correct branch
git checkout upgrade-to-NET10

# Ensure branch is clean
git status

# Pull latest changes (if working in team)
git pull origin upgrade-to-NET10
```

#### After Completing Each Tier

```bash
# Stage all changes
git add .

# Commit with descriptive message
git commit -m "[Tier X] Migrate <ProjectName> to .NET 10.0

- <changes summary>
- All tests passing"

# Push to remote (optional, for backup or team collaboration)
git push origin upgrade-to-NET10
```

#### If Tier Fails and Needs Rollback

```bash
# Discard uncommitted changes
git reset --hard HEAD

# Or rollback to previous tier commit
git reset --hard HEAD~1  # Rollback one commit

# Force push if already pushed (use with caution)
git push origin upgrade-to-NET10 --force
```

---

### Code Review and Merge Process

#### Pull Request Requirements

**When:** After all 4 tiers complete and validated

**PR Title:** `Migrate WebApiExample solution from .NET Framework 4.8 to .NET 10.0`

**PR Description Template:**

```markdown
## Migration Summary

This PR migrates the entire WebApiExample solution from .NET Framework 4.8 to .NET 10.0 using a bottom-up, tier-by-tier approach.

## Changes Overview

- **Tier 1:** WebApiExample.Common ? net10.0
- **Tier 2:** WebApiExample.DataStore ? net10.0 (EF 6.5.1)
- **Tier 3:** WebApiExample.WebApp ? net10.0 (ASP.NET Core)
- **Tier 4:** WebApiExample.WebApp.Tests ? net10.0

## Key Architectural Changes

- ASP.NET Framework ? ASP.NET Core
- Global.asax ? Program.cs
- Web.config ? appsettings.json
- Unity.WebAPI ? ASP.NET Core DI
- System.Web.Optimization ? Direct link tags

## Security Fixes

- ? bootstrap 3.3.7 ? 5.3.8 (CVE fixes)
- ? jQuery 3.3.1 ? 3.7.1 (CVE fixes)

## Package Updates

- 6 packages updated
- 17 packages removed (incompatible or framework-included)
- 1 package replaced (Antlr ? Antlr4)
- 22 packages kept (compatible)

## Testing

- ? All projects build without errors or warnings
- ? All unit tests pass (100% pass rate)
- ? Application starts and runs successfully
- ? All API endpoints functional
- ? No security vulnerabilities

## Validation Checklist

- [ ] Code reviewed by team
- [ ] All tier commits reviewed individually
- [ ] Build passes in CI/CD (if applicable)
- [ ] Manual smoke testing performed
- [ ] Performance acceptable
- [ ] Documentation updated

## Breaking Changes

See BREAKING_CHANGES.md for detailed list (if created)

## Migration Plan

Reference: `.github/upgrades/plan.md`

Assessment: `.github/upgrades/assessment.md`
```

#### Review Process

**Recommended Review Approach:**

1. **Review by Tier:** Examine each tier commit individually
   - Tier 1: Simple, quick review
   - Tier 2: Focus on EF changes
   - Tier 3: Deep review (most complex)
   - Tier 4: Test changes review

2. **Focus Areas for Review:**
   - Project file changes (SDK-style correctness)
   - Package updates (security vulnerabilities addressed?)
   - Architectural changes (Program.cs, DI, configuration)
   - Controller changes (ASP.NET Core patterns correct?)
   - Test updates (all passing? coverage maintained?)

3. **Testing by Reviewer:**
   - Pull branch
   - Build solution
   - Run tests
   - Start application
   - Test key endpoints manually

#### Merge Criteria

**Merge to `main` ONLY when:**

- ? All 4 tiers complete
- ? All builds successful
- ? All tests passing
- ? Code review approved (at least 1-2 reviewers)
- ? Manual smoke testing complete
- ? Security vulnerabilities resolved
- ? No outstanding blocking issues
- ? Documentation updated

**Merge Strategy:** Squash or preserve tier commits based on team preference

- **Squash:** Single commit in main (cleaner history)
- **Preserve:** Keep all 4 tier commits (detailed history)

**Recommended:** Preserve tier commits for traceability

---

### Branch Cleanup

**After Successful Merge:**

```bash
# Switch back to main
git checkout main

# Pull merged changes
git pull origin main

# Delete local feature branch
git branch -d upgrade-to-NET10

# Delete remote feature branch (optional)
git push origin --delete upgrade-to-NET10
```

---

### Handling Conflicts (Team Environment)

If working in a team and others are committing to `main`:

**Strategy:** Rebase periodically to incorporate main changes

```bash
# Switch to upgrade branch
git checkout upgrade-to-NET10

# Fetch latest main
git fetch origin main

# Rebase onto main (do this between tiers, not mid-tier)
git rebase origin/main

# Resolve conflicts if any
# Then continue migration
```

**Recommendation:** Coordinate with team to minimize conflicts during migration period.

---

### Emergency Rollback Plan

**If Migration Must Be Abandoned:**

```bash
# Option 1: Keep branch for future attempt
# Just switch back to main
git checkout main

# Option 2: Delete branch entirely
git checkout main
git branch -D upgrade-to-NET10
git push origin --delete upgrade-to-NET10
```

**Recovery:** Branch and commits preserved on remote (if pushed), can resume later.

---

### Source Control Best Practices for This Migration

1. **Commit Granularity:** One commit per tier (4 total)
2. **Commit Messages:** Descriptive, include validation status
3. **Push Frequency:** After each tier (for backup)
4. **Branch Lifespan:** Delete after merge to main
5. **Code Review:** Required before merge
6. **Merge Timing:** Only when all tiers complete
7. **Rollback Strategy:** Clear rollback points at each tier

## Success Criteria

### Technical Success Criteria

The migration is considered **technically successful** when all of the following conditions are met:

#### Project-Level Criteria

**All Projects (Common, DataStore, WebApp, WebApp.Tests):**

- ? **Target Framework:** All projects targeting `net10.0`
- ? **SDK-Style:** All projects converted to SDK-style format
- ? **Build Success:** Every project builds without errors
- ? **Build Warnings:** Zero build warnings across solution
- ? **Project References:** All inter-project references functional

#### Package-Level Criteria

**Package Updates Applied:**

- ? **EntityFramework:** Updated to 6.5.1 (DataStore, WebApp)
- ? **bootstrap:** Updated to 5.3.8 (WebApp) - **Security critical**
- ? **jQuery:** Updated to 3.7.1 (WebApp) - **Security critical**
- ? **Newtonsoft.Json:** Updated to 13.0.4 (WebApp, Tests)
- ? **System.Runtime.CompilerServices.Unsafe:** Updated to 6.1.2 (WebApp, Tests)
- ? **Antlr:** Replaced with Antlr4 4.6.6 (WebApp, if used)

**Package Removals Completed:**

- ? All 17 incompatible/framework-included packages removed
- ? No references to `Microsoft.AspNet.*` packages remain
- ? No references to `System.Web` namespace in code

**Package Health:**

- ? **Zero Security Vulnerabilities:** NuGet audit shows no vulnerabilities
- ? **Zero Dependency Conflicts:** No package version conflicts
- ? **All Packages Restore:** `dotnet restore` succeeds without warnings

#### Code-Level Criteria

**ASP.NET Core Migration (WebApp):**

- ? **Program.cs:** Application entry point created and functional
- ? **No Global.asax:** Legacy application startup removed
- ? **appsettings.json:** Configuration migrated from Web.config
- ? **Controllers:** All controllers updated to ASP.NET Core patterns
- ? **Dependency Injection:** DI configured and working (built-in or Unity adapter)
- ? **Middleware Pipeline:** Properly configured (routing, static files, etc.)

**Entity Framework (DataStore):**

- ? **EF 6.5.1 Functional:** DbContext operational on .NET Core
- ? **Database Connectivity:** Connection to database successful
- ? **CRUD Operations:** Create, Read, Update, Delete all work
- ? **Migrations:** EF migrations apply (if used)

**Testing (WebApp.Tests):**

- ? **All Tests Run:** xUnit discovers all tests
- ? **All Tests Pass:** 100% pass rate (no failures)
- ? **Test Count Maintained:** Same number of tests as before migration
- ? **Coverage Maintained:** Code coverage percentage unchanged or improved

#### Application-Level Criteria

**WebApp Functionality:**

- ? **Application Starts:** `dotnet run` starts application without errors
- ? **All Endpoints Functional:** Every API endpoint responds correctly
- ? **Static Files Served:** CSS, JS, images accessible
- ? **Data Access Works:** API can query/modify database via DataStore
- ? **Error Handling:** Application handles errors gracefully (404, 500, etc.)

**Performance:**

- ? **Startup Time:** Application startup time acceptable (< 5 seconds)
- ? **Response Times:** API response times within expected range
- ? **No Memory Leaks:** Application memory usage stable over time

---

### Quality Success Criteria

#### Code Quality

- ? **No Code Smells Introduced:** Refactoring maintains or improves code quality
- ? **Consistent Patterns:** ASP.NET Core patterns applied consistently
- ? **Modern Conventions:** Code follows .NET 10.0 best practices
- ? **Readable:** Code remains maintainable and understandable

#### Test Coverage

- ? **Coverage Percentage:** Maintained at pre-migration levels (or improved)
- ? **No Missing Tests:** All originally tested code paths still tested
- ? **New Tests Added:** If new code introduced, tests added for it

#### Documentation

- ? **Migration Documented:** This plan and assessment serve as documentation
- ? **Breaking Changes Documented:** Known breaking changes cataloged
- ? **Known Issues Documented:** Any limitations or workarounds noted
- ? **README Updated:** Project README reflects .NET 10.0 target (if applicable)

---

### Process Success Criteria

#### Bottom-Up Strategy Adherence

- ? **Tier Order Followed:** Projects migrated in strict dependency order
  1. Common (Tier 1)
  2. DataStore (Tier 2)
  3. WebApp (Tier 3)
  4. WebApp.Tests (Tier 4)

- ? **Tier Validation:** Each tier validated before proceeding to next

- ? **No Tier Skipped:** All tiers completed in sequence

- ? **No Multi-Targeting:** Projects not left in multi-targeting state

#### Source Control Compliance

- ? **Branch Strategy Followed:** All work done on `upgrade-to-NET10` branch

- ? **Commit Strategy Followed:** One commit per tier (4 commits total)

- ? **Commit Messages Clear:** Each commit describes changes and validation

- ? **Code Review Completed:** PR reviewed and approved before merge

- ? **Merge to Main:** Changes successfully merged to main branch

---

### Acceptance Success Criteria

These criteria define when the migration can be considered **complete and accepted**:

#### Stakeholder Acceptance

- ? **Functionality Verified:** Business stakeholders confirm application works as expected

- ? **No Regressions:** No loss of existing features or capabilities

- ? **Performance Acceptable:** Application performance meets or exceeds expectations

#### Deployment Readiness

- ? **Deployment Tested:** Application successfully deploys to target environment (staging/production)

- ? **Configuration Validated:** appsettings.json properly configured for environments

- ? **Monitoring Configured:** Application logging and monitoring functional

#### Team Readiness

- ? **Team Trained:** Development team understands .NET 10.0 and ASP.NET Core patterns

- ? **Documentation Accessible:** Migration plan, assessment, and guides available to team

- ? **Support Plan:** Plan in place for post-migration issues

---

### Definition of Done

**The migration is DONE when:**

1. ? **All Technical Criteria Met:** Every technical checkbox above is checked

2. ? **All Quality Criteria Met:** Code quality, test coverage, documentation complete

3. ? **All Process Criteria Met:** Bottom-up strategy followed, source control compliant

4. ? **All Acceptance Criteria Met:** Stakeholders accept, deployment ready, team ready

5. ? **Zero Blocking Issues:** No outstanding issues that prevent production deployment

6. ? **Merge Complete:** PR merged to main branch

7. ? **Deployment Successful:** Application running in production (or staging) on .NET 10.0

---

### Validation Checklist

Use this final checklist to confirm migration success:

#### Pre-Merge Validation

- [ ] All 4 tiers complete (Common, DataStore, WebApp, Tests)
- [ ] All projects on net10.0
- [ ] All builds successful (zero errors, zero warnings)
- [ ] All tests passing (100% pass rate)
- [ ] All package updates applied
- [ ] All security vulnerabilities resolved
- [ ] Application runs successfully
- [ ] All API endpoints functional
- [ ] Manual smoke testing passed
- [ ] Code review completed and approved

#### Post-Merge Validation

- [ ] Main branch builds successfully
- [ ] CI/CD pipeline passes (if applicable)
- [ ] Deployment to staging successful
- [ ] Staging environment validated
- [ ] Performance acceptable in staging
- [ ] Stakeholder sign-off obtained

#### Production Deployment Validation

- [ ] Deployment to production successful
- [ ] Application running in production
- [ ] Monitoring shows healthy metrics
- [ ] No errors in production logs
- [ ] User acceptance testing passed
- [ ] Migration marked complete

---

### Success Metrics

**Quantifiable Success Indicators:**

| Metric | Pre-Migration | Post-Migration | Status |
|--------|---------------|----------------|--------|
| Projects on .NET 10.0 | 0/4 (0%) | 4/4 (100%) | ? |
| Build Errors | 0 | 0 | ? |
| Build Warnings | X | 0 | ? |
| Security Vulnerabilities | 2 | 0 | ? |
| Test Pass Rate | X% | X% (maintained) | ? |
| Code Coverage | X% | X% (maintained) | ? |
| API Endpoints Functional | X/X | X/X (all) | ? |
| Application Startup Time | Xs | Xs (acceptable) | ? |

**Qualitative Success Indicators:**

- ? Team confidence in .NET 10.0 application
- ? Stakeholder satisfaction with migration
- ? No post-migration rollback needed
- ? Improved developer experience (SDK-style projects, modern tooling)
- ? Long-term maintainability improved

---

### Migration Sign-Off

**Migration considered officially complete when signed off by:**

- [ ] **Tech Lead / Architect:** Confirms technical criteria met
- [ ] **QA Lead:** Confirms all tests passing, quality maintained
- [ ] **Product Owner:** Confirms functionality and acceptance criteria met
- [ ] **DevOps Lead:** Confirms deployment successful and stable

**Date Completed:** _______________

**Final Status:** ? **SUCCESS** / ? **INCOMPLETE** / ?? **PARTIAL SUCCESS**

---

### Post-Migration Recommendations

**After successful migration, consider:**

1. **Future EF Core Migration:** Plan migration from EF 6 to EF Core for better .NET Core integration
2. **Bundling Enhancement:** Evaluate WebOptimizer or build-time bundling for better optimization
3. **API Documentation:** Implement Swagger/OpenAPI for API documentation
4. **Performance Tuning:** Profile application and optimize for .NET 10.0
5. **Monitoring Enhancement:** Leverage .NET 10.0 monitoring and telemetry features
6. **Further Modernization:** Consider minimal APIs, source generators, or other .NET 10.0 features

**These are enhancements, not requirements for migration success.**
