# .NET Framework 4.8 to .NET 10.0 Migration Plan

## Table of Contents

- [Executive Summary](#executive-summary)
- [Migration Strategy](#migration-strategy)
- [Detailed Dependency Analysis](#detailed-dependency-analysis)
- [Project-by-Project Plans](#project-by-project-plans)
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

### Scenario Description

This plan guides the migration of the **WebApiExample** solution from **.NET Framework 4.8** to **.NET 10.0 (LTS)**. The solution consists of 4 projects totaling approximately 4,176 lines of code across 70 files. This migration involves converting all projects from legacy project format to SDK-style, upgrading the target framework, updating NuGet packages (including addressing security vulnerabilities), and migrating ASP.NET Web API to ASP.NET Core.

### Scope

**Projects Affected:**
- `WebApiExample.Common` - Shared models and utilities (61 LOC, 0 dependencies)
- `WebApiExample.DataStore` - Data access layer with Entity Framework (110 LOC, 1 dependency)
- `WebApiExample.WebApp` - ASP.NET Web API application (3,612 LOC, 2 dependencies) **[HIGH COMPLEXITY]**
- `WebApiExample.WebApp.Tests` - Unit tests (393 LOC, 3 dependencies)

**Current State:**
- All projects: .NET Framework 4.8, legacy (non-SDK-style) project format
- 46 total NuGet packages, 14 requiring updates
- 8 incompatible packages requiring replacement
- **Security vulnerabilities** in bootstrap (3.3.7), jQuery (3.3.1), and Newtonsoft.Json (11.0.1/6.0.4)

**Target State:**
- All projects: .NET 10.0, SDK-style project format
- Modern ASP.NET Core Web API replacing ASP.NET Framework Web API
- All packages updated to compatible versions with security vulnerabilities resolved
- Bundling/minification replaced with modern alternatives

### Complexity Assessment

**Discovered Metrics:**
- **Total Projects**: 4 (small solution)
- **Dependency Depth**: 3 levels (clear hierarchy, no cycles)
- **High-Risk Projects**: 1 (WebApiExample.WebApp - 3,612 LOC, 19 package issues, ASP.NET migration)
- **Security Vulnerabilities**: 3 packages with known CVEs
- **Package Complexity**: 8 incompatible packages, 6 upgrades recommended

**Complexity Classification: Medium**

**Justification:**
- Small project count favors simpler migration
- Clear dependency structure (no circular dependencies)
- **However**: ASP.NET Framework ? ASP.NET Core migration adds significant complexity
- WebApiExample.WebApp requires substantial changes (Global.asax ? Program.cs, bundling/minification replacement, Web.config transformation)
- Security vulnerabilities require immediate attention
- Mixed package compatibility requiring careful sequencing

### Critical Issues

**Security Vulnerabilities (CRITICAL - Must Address):**
1. **bootstrap** 3.3.7 ? 5.3.8 (WebApiExample.WebApp)
2. **jQuery** 3.3.1 ? 3.7.1 (WebApiExample.WebApp)
3. **Newtonsoft.Json** 11.0.1 ? 13.0.4 (WebApiExample.WebApp)
4. **Newtonsoft.Json** 6.0.4 ? 13.0.4 (WebApiExample.WebApp.Tests)

**Incompatible Packages Requiring Replacement:**
- Microsoft.AspNet.Mvc (5.2.4/5.2.7) ? Built into ASP.NET Core
- Microsoft.AspNet.WebApi.Core (5.2.7) ? Built into ASP.NET Core
- Microsoft.AspNet.WebApi.WebHost (5.2.4/5.2.7) ? Built into ASP.NET Core
- Microsoft.AspNet.Web.Optimization (1.1.3) ? Must be replaced with modern bundling
- Unity.WebAPI (5.4.0) ? Replace with Microsoft.Extensions.DependencyInjection

**ASP.NET-Specific Migrations:**
- Global.asax.cs ? Program.cs/Startup.cs pattern
- System.Web.Optimization bundling ? Modern alternatives (webpack, or direct HTML references)
- Web.config ? appsettings.json + Program.cs configuration

### Selected Strategy: Bottom-Up (Dependency-First)

**Rationale:**
- Clear dependency hierarchy makes bottom-up approach natural
- Each tier builds on stable, already-upgraded foundation
- Minimizes risk by validating each tier before proceeding
- No multi-targeting complexity needed
- WebApp migration complexity warrants isolated focus in its own tier

**Iteration Strategy: Phase-Based**
- **Phase 1 (Iteration 2.1)**: Foundation sections (Dependency Analysis, Migration Strategy)
- **Phase 2 (Iterations 2.2-2.3)**: Project stubs, Risk/Complexity overview
- **Phase 3 (Iterations 3.1-3.4)**: Detailed plans per tier
  - Tier 1: WebApiExample.Common (simple)
  - Tier 2: WebApiExample.DataStore (simple with EF upgrade)
  - Tier 3: WebApiExample.WebApp (complex - dedicated iteration)
  - Tier 4: WebApiExample.WebApp.Tests (moderate)
- **Final Iteration**: Success Criteria, Source Control Strategy

**Expected Remaining Iterations**: 7 total (including this one: 3 completed, 4 remaining)

## Migration Strategy

### Approach: Incremental Bottom-Up Migration

**Selected Approach**: Incremental, tier-by-tier migration starting from leaf projects and progressing upward through the dependency chain.

**Justification**:

1. **Clear Dependency Hierarchy**: The 4-tier structure with no circular dependencies is ideal for bottom-up migration
2. **Risk Mitigation**: Upgrading dependencies first ensures each tier builds on a stable, already-migrated foundation
3. **No Multi-Targeting Needed**: Dependencies are always on the same or newer framework than their consumers
4. **Isolated Validation**: Each tier can be fully tested and stabilized before moving to the next
5. **Complexity Management**: High-complexity WebApp project gets isolated focus in Tier 3
6. **Learning Curve**: Lessons from simpler tiers (Common, DataStore) inform the complex WebApp migration

### Bottom-Up Strategy Application

**Ordering Principles**:
- **Tier 1**: Projects with zero internal dependencies (Common)
- **Tier 2**: Projects depending only on Tier 1 (DataStore)
- **Tier 3**: Projects depending on Tiers 1 & 2 (WebApp)
- **Tier 4**: Projects depending on all previous tiers (Tests)

**Execution Flow Per Tier**:

Each tier follows this pattern:

1. **Preparation** (Review & Planning)
   - Review tier's project structure and dependencies
   - Verify lower tiers are stable
   - Identify tier-specific risks

2. **Update** (Conversion & Package Management)
   - Convert project file(s) to SDK-style
   - Update target framework to net10.0
   - Update/remove/replace NuGet packages
   - Fix compilation errors

3. **Code Migration** (Breaking Changes & Features)
   - Address breaking changes from framework/package updates
   - Migrate framework-specific features (e.g., Global.asax ? Program.cs)
   - Update configuration files

4. **Testing** (Tier Validation)
   - Build tier project(s)
   - Run tier unit tests
   - Verify integration with lower tiers
   - Validate consumers (higher tiers on old framework) still compatible

5. **Stabilization** (Review & Documentation)
   - Address any issues found
   - Document lessons learned
   - Mark tier complete
   - Proceed to next tier

### Parallel vs Sequential Execution

**Sequential Execution Required**:
- Each tier contains only **one project**
- Cannot start Tier N+1 until Tier N is validated and stable
- Strict ordering: Tier 1 ? Tier 2 ? Tier 3 ? Tier 4

**No Parallelization Opportunities**:
- Single-project tiers eliminate parallel execution within tiers
- Dependency chain prevents cross-tier parallelization

### Migration Phases

**Phase 1: Tier 1 - Foundation Layer**
- **Project**: WebApiExample.Common
- **Scope**: Shared models, constants, utilities
- **Complexity**: Low
- **Key Activities**: SDK conversion, framework update
- **Benefits Unlocked**: Foundation on net10.0 ready for higher tiers

**Phase 2: Tier 2 - Data Access Layer**
- **Project**: WebApiExample.DataStore
- **Scope**: Entity Framework data access
- **Complexity**: Low-Medium
- **Key Activities**: SDK conversion, framework update, EF 6.5.1 upgrade
- **Benefits Unlocked**: Data layer on net10.0, EF improvements

**Phase 3: Tier 3 - Application Layer**
- **Project**: WebApiExample.WebApp
- **Scope**: ASP.NET Web API ? ASP.NET Core migration
- **Complexity**: High
- **Key Activities**: 
  - SDK conversion (WAP ? Web SDK)
  - Framework update
  - ASP.NET Framework ? Core migration
  - Global.asax ? Program.cs
  - Bundling/minification replacement
  - Unity ? built-in DI migration
  - Package compatibility resolution
  - Security vulnerability fixes
- **Benefits Unlocked**: Modern ASP.NET Core application on net10.0

**Phase 4: Tier 4 - Test Layer**
- **Project**: WebApiExample.WebApp.Tests
- **Scope**: Unit tests for WebApp
- **Complexity**: Low-Medium
- **Key Activities**: SDK conversion, framework update, package updates, test compatibility
- **Benefits Unlocked**: Full solution on net10.0, all tests passing

### Deployment Cadence

**Recommended Approach**: Deploy after each tier completion (4 deployments)

**Rationale**:
- Each tier is independently verifiable
- Allows for early feedback and issue detection
- Reduces risk compared to "big bang" deployment
- **However**: Since Tests project is not deployed, actual deployments are:
  - After Tier 1: Common library
  - After Tier 2: Common + DataStore libraries
  - After Tier 3: Full application (Common + DataStore + WebApp)
  - After Tier 4: No new deployment, validation only

**Alternative**: Single deployment after Tier 3 (before tests), with Tier 4 as final validation.

### Tier Completion Criteria

**Tier is considered "complete" when**:
1. All projects in tier converted to SDK-style
2. All projects in tier target net10.0
3. All package updates applied
4. All compilation errors resolved
5. All compilation warnings addressed
6. All tier-specific tests pass
7. Integration with lower tiers verified
8. No regressions in lower tiers detected
9. Higher tiers (still on net48) continue to function

### Between-Tier Validation

**After completing each tier, verify**:
- ? All projects in tier build successfully
- ? No build errors or warnings
- ? All tier-specific tests pass
- ? No regressions in lower tiers (re-run their tests)
- ? Higher tiers (still on net48) still build and function (transitional compatibility)

## Detailed Dependency Analysis

### Dependency Graph Structure

The solution has a clear 4-tier dependency hierarchy with no circular dependencies, making it ideal for bottom-up migration:

```
Tier 4: [WebApp.Tests]
         ?
Tier 3: [WebApp]
         ?
Tier 2: [DataStore]
         ?
Tier 1: [Common]
```

**Tier Determination Logic:**

1. **Tier 1 (Leaf - No Dependencies)**: 
   - **WebApiExample.Common** - Has zero project dependencies, depended on by all other projects
   - Contains shared models, constants, and utility classes
   - Foundation for entire solution

2. **Tier 2 (Depends Only on Tier 1)**:
   - **WebApiExample.DataStore** - Depends only on Common
   - Data access layer with Entity Framework
   - 2 projects depend on it (WebApp, WebApp.Tests)

3. **Tier 3 (Depends on Tier 1 & 2)**:
   - **WebApiExample.WebApp** - Main application depending on Common + DataStore
   - ASP.NET Web API application
   - 1 project depends on it (WebApp.Tests)

4. **Tier 4 (Test Project - Depends on All)**:
   - **WebApiExample.WebApp.Tests** - Depends on Common, DataStore, and WebApp
   - No projects depend on it
   - Must be migrated last

### Project Groupings by Migration Phase

**Phase 1: Tier 1 - Foundation (1 project)**
- WebApiExample.Common
- **Risk**: Low (no dependencies, simple models/utilities)
- **Unlocks**: All other projects can proceed once stable

**Phase 2: Tier 2 - Data Layer (1 project)**
- WebApiExample.DataStore
- **Risk**: Low-Medium (Entity Framework upgrade)
- **Unlocks**: Application and test projects

**Phase 3: Tier 3 - Application (1 project)**
- WebApiExample.WebApp
- **Risk**: High (ASP.NET Framework ? Core migration, bundling replacement, many package changes)
- **Unlocks**: Test project can be migrated

**Phase 4: Tier 4 - Tests (1 project)**
- WebApiExample.WebApp.Tests
- **Risk**: Low-Medium (package updates, test framework compatibility)
- **Completion**: Full solution migrated

### Critical Path Identification

**Critical Path**: Common ? DataStore ? WebApp ? Tests

This is the only path through the solution, making sequencing straightforward:
1. **Cannot skip any tier** - each tier depends on the previous
2. **No parallelization opportunities** - single project per tier
3. **Serial execution required** - must complete tiers in order

**Bottleneck**: Tier 3 (WebApiExample.WebApp) is the complexity bottleneck due to ASP.NET Framework ? Core migration requirements.

### Tier Dependencies Detail

| Tier | Projects | Depends On | Depended On By | Complexity |
|------|----------|------------|----------------|------------|
| 1 | Common | None | DataStore, WebApp, Tests | Low |
| 2 | DataStore | Common | WebApp, Tests | Low-Medium |
| 3 | WebApp | Common, DataStore | Tests | High |
| 4 | Tests | Common, DataStore, WebApp | None | Low-Medium |

### Validation Points Between Tiers

**After Tier 1 (Common)**:
- ? Common project builds successfully on net10.0
- ? No build warnings
- ? All types remain accessible (API compatibility)

**After Tier 2 (DataStore)**:
- ? DataStore builds successfully on net10.0
- ? Entity Framework 6.5.1 functions correctly
- ? Database connectivity verified
- ? Common project still builds (no regressions)

**After Tier 3 (WebApp)**:
- ? WebApp builds successfully as ASP.NET Core on net10.0
- ? Application starts and responds to requests
- ? Dependency injection works (Unity ? built-in DI)
- ? All API endpoints accessible
- ? Lower tiers (Common, DataStore) still build

**After Tier 4 (Tests)**:
- ? All tests build successfully on net10.0
- ? All tests pass
- ? Full solution builds without errors or warnings

## Project-by-Project Plans

### Tier 1: WebApiExample.Common

**Current State**: .NET Framework 4.8, Legacy project format, 61 LOC, 0 dependencies, 3 dependants

**Target State**: .NET 10.0, SDK-style project format

**Complexity**: Low

---

#### Migration Steps

##### 1. Prerequisites

**Verify:**
- ? No dependencies to migrate first (leaf node)
- ? .NET 10.0 SDK installed
- ? Visual Studio or VS Code with C# extension
- ? Git working directory clean (changes committed)

**Backup:**
- Create snapshot/commit before conversion: `git commit -m "Pre-migration: WebApiExample.Common"`

##### 2. Convert Project to SDK-Style

**Action**: Convert legacy .csproj to SDK-style format

**Method**: Use automated conversion tool or manual conversion

**Automated Approach** (Recommended):
```bash
# Using dotnet try-convert tool
dotnet tool install -g try-convert
cd WebApiExample.Common
try-convert -p WebApiExample.Common.csproj
```

**Manual Verification After Conversion**:
- Verify all 3 files included in project
- Check that `<TargetFramework>` is set (will update in next step)
- Ensure project compiles

**Expected SDK-Style Structure**:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <ProjectReference Include="..\[dependencies if any]" />
  </ItemGroup>
</Project>
```

##### 3. Update Target Framework

**Action**: Change `<TargetFramework>` from `net48` to `net10.0`

**Edit**: `WebApiExample.Common\WebApiExample.Common.csproj`

**Change**:
```xml
<!-- FROM -->
<TargetFramework>net48</TargetFramework>

<!-- TO -->
<TargetFramework>net10.0</TargetFramework>
```

##### 4. Package Updates

**No package updates required** - WebApiExample.Common has zero NuGet package dependencies.

##### 5. Expected Breaking Changes

**None Expected** - This tier contains:
- Simple models/POCOs
- Utility classes
- Constants

**Potential Issues**:
- ?? If any code uses .NET Framework-specific APIs (e.g., `System.Configuration`), they will need replacement
- ?? If any code depends on `System.Web` types, those are incompatible

**Assessment Review**: According to assessment.md, 10 APIs analyzed, all marked as ? Compatible. No breaking changes expected.

##### 6. Code Modifications

**Expected**: Minimal to none

**Review Areas**:
1. **Namespace usage** - Verify all `using` directives resolve
2. **API compatibility** - All 10 APIs marked compatible in assessment
3. **Build warnings** - Address any new warnings

**If Issues Found**:
- Replace removed APIs with .NET 10.0 equivalents
- Use Microsoft.DotNet.UpgradeAssistant guidance
- Consult .NET API browser for replacements

##### 7. Build and Validate

**Build Project**:
```bash
cd WebApiExample.Common
dotnet build
```

**Success Criteria**:
- ? Build completes with 0 errors
- ? Build completes with 0 warnings (or justified warnings only)
- ? Output assembly produced in `bin\Debug\net10.0\`

**Verify**:
- All 3 files compiled
- No missing references
- Assembly metadata shows .NET 10.0 target

##### 8. Testing Strategy

**Unit Tests**: None identified for Common project in assessment

**Validation Approach**:
- ? Build succeeds
- ? No warnings
- ? Visual inspection of public API (no inadvertent changes)
- ? Higher-tier projects (DataStore, WebApp, Tests) can still reference Common (verify in next tier)

**Integration Validation**:
- Will be validated when dependent projects (Tier 2-4) are migrated
- All 3 dependants must successfully reference the upgraded Common

##### 9. Validation Checklist

- [ ] Project converted to SDK-style format
- [ ] Target framework updated to net10.0
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] All source files included in build
- [ ] Output assembly targets .NET 10.0
- [ ] Git commit created: `git commit -m "Tier 1: Migrated WebApiExample.Common to .NET 10.0"`

##### 10. Rollback Plan

**If Migration Fails**:
```bash
# Revert to previous commit
git reset --hard HEAD~1
```

**Alternative**: Keep original .csproj as `.csproj.backup` before conversion

---

#### Tier 1 Completion Criteria

- ? WebApiExample.Common builds on net10.0 without errors or warnings
- ? All source files accounted for and compiling
- ? Project ready for consumption by Tier 2 (DataStore)
- ? Changes committed to source control

### Tier 2: WebApiExample.DataStore

**Current State**: .NET Framework 4.8, Legacy project format, 110 LOC, 1 dependency (Common), 2 dependants

**Target State**: .NET 10.0, SDK-style project format, Entity Framework 6.5.1

**Complexity**: Low-Medium

---

#### Migration Steps

##### 1. Prerequisites

**Verify:**
- ? Tier 1 (WebApiExample.Common) successfully migrated to net10.0
- ? Common project builds without errors
- ? Git working directory clean

**Dependencies Check**:
- WebApiExample.Common (Tier 1) - ? Must be on net10.0 before proceeding

##### 2. Convert Project to SDK-Style

**Action**: Convert legacy .csproj to SDK-style format

**Method**: Use automated conversion tool

```bash
cd WebApiExample.DataStore
try-convert -p WebApiExample.DataStore.csproj
```

**Manual Verification**:
- Verify all 3 files included
- Verify project reference to WebApiExample.Common preserved
- Verify EntityFramework package reference present

##### 3. Update Target Framework

**Action**: Change `<TargetFramework>` from `net48` to `net10.0`

**Edit**: `WebApiExample.DataStore\WebApiExample.DataStore.csproj`

**Change**:
```xml
<TargetFramework>net10.0</TargetFramework>
```

##### 4. Package Updates

**Packages to Update (Tier 2 Scope):**

| Package | Current Version | Target Version | Reason |
|---------|----------------|----------------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | Recommended upgrade for .NET 10.0 compatibility and improvements |

**Update Process**:

**Option 1: Edit .csproj directly**
```xml
<PackageReference Include="EntityFramework" Version="6.5.1" />
```

**Option 2: Use Package Manager**
```bash
dotnet add package EntityFramework --version 6.5.1
```

**Restore Packages**:
```bash
dotnet restore
```

##### 5. Expected Breaking Changes

**Entity Framework 6.4.4 ? 6.5.1**:

**Low Risk Upgrade** - Minor version bump, primarily bug fixes and compatibility improvements.

**Potential Issues**:
- EF 6.5.1 includes .NET 6+ support improvements
- Minimal breaking changes expected in minor version update
- Check for any LINQ query behavior differences (rare)

**Reference**: Review [Entity Framework 6.5.1 release notes](https://github.com/dotnet/ef6/releases) for specific changes

**Other Considerations**:
- Database connectivity: Should remain unchanged
- Migration scripts: Should continue working
- DbContext configuration: No changes expected

##### 6. Code Modifications

**Expected**: Minimal to none

**Review Areas**:

1. **DbContext Configuration**
   - Verify connection string handling
   - Check any custom conventions or configurations
   - File: Likely in a context class (e.g., `MyDbContext.cs`)

2. **Entity Configurations**
   - Fluent API configurations should remain compatible
   - Data annotations unchanged

3. **Migration Scripts** (if present)
   - Located in: `Migrations\` folder (if exists)
   - Verify migrations still compile

4. **Database Initialization**
   - Check for any custom initializers
   - Verify seed data logic

**No API Breaking Changes Expected**: Assessment shows 0 API issues for this project.

##### 7. Build and Validate

**Build Project**:
```bash
cd WebApiExample.DataStore
dotnet build
```

**Success Criteria**:
- ? Build completes with 0 errors
- ? Build completes with 0 warnings
- ? EntityFramework 6.5.1 package restored successfully
- ? Reference to WebApiExample.Common (net10.0) resolved

##### 8. Testing Strategy

**Database Connectivity Test**:

If connection string available, test basic operations:

```csharp
// Example validation (manual or automated)
using (var context = new YourDbContext())
{
    // Test connection
    var canConnect = context.Database.Exists();
    
    // Test simple query (if applicable)
    var count = context.YourEntity.Count();
}
```

**Unit Tests**: 
- No unit tests identified specifically for DataStore in assessment
- Unit tests in WebApp.Tests may cover DataStore (will validate in Tier 4)

**Validation Checklist**:
- [ ] Project builds successfully
- [ ] EntityFramework 6.5.1 referenced
- [ ] DbContext compiles
- [ ] Database connection verified (if possible)
- [ ] No SQL generation issues
- [ ] Migration scripts still compile (if present)

##### 9. Integration Validation

**Verify with Tier 1**:
- DataStore correctly references Common (net10.0)
- No version conflicts between tiers

**Prepare for Tier 3**:
- DataStore ready to be consumed by WebApp (Tier 3)
- Both Common and DataStore on net10.0 create stable foundation

##### 10. Validation Checklist

- [ ] Project converted to SDK-style format
- [ ] Target framework updated to net10.0
- [ ] EntityFramework updated to 6.5.1
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] References to WebApiExample.Common (net10.0) resolved
- [ ] Database operations validated (if feasible)
- [ ] Git commit created: `git commit -m "Tier 2: Migrated WebApiExample.DataStore to .NET 10.0, upgraded EF to 6.5.1"`

##### 11. Rollback Plan

**If Migration Fails**:
```bash
git reset --hard HEAD~1
```

**If EF 6.5.1 Issues Found**:
- Downgrade to 6.4.4 (still compatible with net10.0)
- Document issue and investigate separately
- Proceed with migration using 6.4.4

---

#### Tier 2 Completion Criteria

- ? WebApiExample.DataStore builds on net10.0 without errors or warnings
- ? EntityFramework 6.5.1 successfully integrated
- ? Database operations functional (if tested)
- ? Integration with Tier 1 (Common) verified
- ? Ready for consumption by Tier 3 (WebApp)
- ? Changes committed to source control

### Tier 3: WebApiExample.WebApp

**Current State**: .NET Framework 4.8, Legacy WAP project format, 3,612 LOC, 2 dependencies (Common, DataStore), 1 dependant (Tests)

**Target State**: .NET 10.0, SDK-style Web project format, ASP.NET Core Web API

**Complexity**: High

---

#### Migration Steps

##### 1. Prerequisites

**Verify:**
- ? Tier 1 (WebApiExample.Common) on net10.0
- ? Tier 2 (WebApiExample.DataStore) on net10.0 with EF 6.5.1
- ? Both lower tiers build successfully
- ? .NET 10.0 SDK with ASP.NET Core runtime installed
- ? Git working directory clean

**Review Current Structure**:
- Identify all controllers in `Controllers\` folder
- Identify Global.asax.cs initialization code
- Identify bundling configuration (likely `App_Start\BundleConfig.cs`)
- Identify Unity DI configuration (likely `App_Start\UnityConfig.cs`)
- Review Web.config for app settings and connection strings

##### 2. Convert Project to SDK-Style (Web SDK)

**Action**: Convert WAP (Web Application Project) to SDK-style Web project

**Important**: ASP.NET Framework Web API ? ASP.NET Core Web API is more complex than simple SDK conversion. Consider incremental approach:

**Approach 1: Convert then Migrate** (Recommended)
1. Convert to SDK-style targeting net48 first
2. Verify it still works
3. Then migrate to net10.0 + ASP.NET Core

**Approach 2: Create New ASP.NET Core Project**
1. Create new ASP.NET Core Web API project targeting net10.0
2. Migrate code incrementally from old project
3. More work but cleaner result

**For this plan, using Approach 1:**

```bash
cd WebApiExample.WebApp
try-convert -p WebApiExample.WebApp.csproj
```

**Expected Issues with WAP Conversion**:
- `*.csproj.user` files
- Web.config embedded resources
- Content files (HTML, CSS, JS) may need explicit inclusion
- `App_Start\` folder files may need adjustment

**Manual Adjustments After Conversion**:
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
    <!-- Temporarily keep net48 for validation -->
  </PropertyGroup>
  
  <!-- Project references -->
  <ItemGroup>
    <ProjectReference Include="..\WebApiExample.Common\WebApiExample.Common.csproj" />
    <ProjectReference Include="..\WebApiExample.DataStore\WebApiExample.DataStore.csproj" />
  </ItemGroup>
  
  <!-- Content files - may need to be explicit -->
  <ItemGroup>
    <Content Include="Web.config" />
    <!-- Add other content files as needed -->
  </ItemGroup>
</Project>
```

**Verify Intermediate State**:
```bash
dotnet build
# Should still build on net48 after SDK conversion
```

##### 3. Update Target Framework & Add ASP.NET Core

**Action**: Change to net10.0 and add ASP.NET Core framework reference

**Edit**: `WebApiExample.WebApp\WebApiExample.WebApp.csproj`

**Change**:
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
</Project>
```

**Note**: `Microsoft.NET.Sdk.Web` SDK automatically includes ASP.NET Core framework.

##### 4. Package Updates (Critical - 19 packages affected)

**Security Vulnerabilities - IMMEDIATE PRIORITY:**

| Package | Current | Target | Action | Reason |
|---------|---------|--------|--------|--------|
| bootstrap | 3.3.7 | 5.3.8 | Update | ?? Security vulnerability |
| jQuery | 3.3.1 | 3.7.1 | Update | ?? Security vulnerability |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | Update | ?? Security vulnerability, recommended upgrade |

**Incompatible Packages - REMOVE (functionality included in ASP.NET Core):**

| Package | Current | Action | Replacement |
|---------|---------|--------|-------------|
| Microsoft.AspNet.Mvc | 5.2.4 | Remove | Built into ASP.NET Core |
| Microsoft.AspNet.Razor | 3.2.4 | Remove | Built into ASP.NET Core |
| Microsoft.AspNet.WebApi | 5.2.4 | Remove | Built into ASP.NET Core |
| Microsoft.AspNet.WebApi.Core | 5.2.7 | Remove | Built into ASP.NET Core |
| Microsoft.AspNet.WebApi.WebHost | 5.2.4 | Remove | Built into ASP.NET Core |
| Microsoft.AspNet.WebPages | 3.2.4 | Remove | Built into ASP.NET Core |
| Microsoft.CodeDom.Providers.DotNetCompilerPlatform | 2.0.0 | Remove | Not needed in .NET 10.0 |
| Microsoft.Web.Infrastructure | 1.0.0.0 | Remove | Not needed in ASP.NET Core |

**Incompatible Packages - REPLACE:**

| Package | Current | Action | Replacement |
|---------|---------|--------|-------------|
| Microsoft.AspNet.Web.Optimization | 1.1.3 | Remove | Use direct HTML references or modern bundler |
| Unity.WebAPI | 5.4.0 | Remove | Use built-in Microsoft.Extensions.DependencyInjection |
| Antlr | 3.5.0.2 | Replace | Antlr4 (4.6.6) if still needed - **verify actual usage first** |

**Compatible Packages - UPGRADE:**

| Package | Current | Target | Reason |
|---------|---------|--------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | Already updated in Tier 2 (DataStore), consistency |
| System.Runtime.CompilerServices.Unsafe | 4.5.2 | 6.1.2 | Recommended upgrade |

**Compatible Packages - RETAIN:**

| Package | Version | Action |
|---------|---------|--------|
| Microsoft.AspNet.WebApi.Client | 5.2.7 | Keep (compatible with .NET 10.0) |
| Microsoft.AspNet.WebApi.HelpPage | 5.2.4 | Keep (if using HelpPage for API docs) |
| Unity | 5.11.10 | Keep (if needed, but prefer built-in DI) |
| WebGrease | 1.6.0 | Keep (compatible, though may not need it) |
| Modernizr | 2.8.3 | Keep (compatible) |

**Update Process**:

1. **Remove incompatible packages** (done via .csproj edit - remove `<PackageReference>` lines)
2. **Update security-critical packages**:
   ```bash
   dotnet add package bootstrap --version 5.3.8
   dotnet add package jQuery --version 3.7.1
   dotnet add package Newtonsoft.Json --version 13.0.4
   dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2
   ```

3. **Verify Antlr usage** - Search codebase for `Antlr` references:
   - If used: Replace with Antlr4 4.6.6
   - If unused: Remove package

##### 5. Global.asax.cs ? Program.cs Migration

**Current Structure** (ASP.NET Framework):
```csharp
// Global.asax.cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        GlobalConfiguration.Configure(WebApiConfig.Register);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
        UnityConfig.RegisterComponents();
    }
}
```

**Target Structure** (ASP.NET Core):

**Create** `Program.cs`:
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddControllers();

// Add Entity Framework DbContext (if needed)
// builder.Services.AddDbContext<YourDbContext>(options =>
//     options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Migrate Unity DI registrations here
// Example:
// builder.Services.AddScoped<IYourService, YourService>();

// Add API documentation (if using HelpPage)
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseStaticFiles(); // If serving static content

app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Migration Tasks**:
1. ? Delete `Global.asax` and `Global.asax.cs` files
2. ? Create `Program.cs` with above template
3. ? Migrate WebApiConfig route configurations (see next section)
4. ? Migrate Unity DI registrations to `builder.Services`
5. ? Remove bundling configuration (BundleConfig) - replace separately

##### 6. WebApiConfig ? Program.cs Route Configuration

**Current** (likely in `App_Start\WebApiConfig.cs`):
```csharp
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        config.MapHttpAttributeRoutes();
        
        config.Routes.MapHttpRoute(
            name: "DefaultApi",
            routeTemplate: "api/{controller}/{id}",
            defaults: new { id = RouteParameter.Optional }
        );
    }
}
```

**Target** (integrated in `Program.cs`):
```csharp
// Routes defined via attributes on controllers, e.g.:
// [Route("api/[controller]")]
// [ApiController]
// public class ValuesController : ControllerBase { }

// Or add conventional routing in Program.cs:
app.MapControllerRoute(
    name: "default",
    pattern: "api/{controller}/{id?}");
```

**Action**:
- Review `App_Start\WebApiConfig.cs` for custom routes
- Migrate to attribute routing on controllers or conventional routing in Program.cs
- **Recommended**: Use attribute routing (`[Route]` attributes) - more explicit and ASP.NET Core standard

##### 7. Unity DI ? Microsoft.Extensions.DependencyInjection

**Current** (likely in `App_Start\UnityConfig.cs`):
```csharp
public static class UnityConfig
{
    public static void RegisterComponents()
    {
        var container = new UnityContainer();
        
        // Register types
        container.RegisterType<IMyService, MyService>();
        
        GlobalConfiguration.Configuration.DependencyResolver = 
            new UnityDependencyResolver(container);
    }
}
```

**Target** (in `Program.cs`):
```csharp
// In builder.Services section
builder.Services.AddScoped<IMyService, MyService>();
// Or AddSingleton, AddTransient depending on lifetime
```

**Lifetime Mapping**:
- Unity `RegisterType<I, T>()` (default: transient) ? `AddTransient<I, T>()`
- Unity `RegisterType<I, T>(new ContainerControlledLifetimeManager())` ? `AddSingleton<I, T>()`
- Unity `RegisterType<I, T>(new HierarchicalLifetimeManager())` ? `AddScoped<I, T>()`

**Migration Tasks**:
1. Review `App_Start\UnityConfig.cs` for all registrations
2. Map each registration to built-in DI
3. Add registrations to `Program.cs`
4. Delete `UnityConfig.cs` and `Unity.WebAPI` package
5. Update controllers to use constructor injection (should already be doing this)

##### 8. Bundling & Minification Replacement

**Current** (likely `App_Start\BundleConfig.cs`):
```csharp
public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
            "~/Scripts/jquery-{version}.js"));
        
        bundles.Add(new StyleBundle("~/Content/css").Include(
            "~/Content/bootstrap.css",
            "~/Content/site.css"));
    }
}
```

**Target Options**:

**Option 1: Direct HTML References** (Simplest for migration):
```html
<!-- In _Layout.cshtml or relevant views -->
<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
<link rel="stylesheet" href="~/css/site.css" />
<script src="~/lib/jquery/dist/jquery.min.js"></script>
```

**Option 2: Use LibMan** (Library Manager):
```bash
# Install client-side libraries
libman install jquery@3.7.1 -d wwwroot/lib/jquery
libman install bootstrap@5.3.8 -d wwwroot/lib/bootstrap
```

**Option 3: Modern Bundler** (Future enhancement - webpack, Vite, etc.)

**Recommended for Initial Migration**: Option 1 (direct references)

**Migration Tasks**:
1. Move static files to `wwwroot\` folder (ASP.NET Core convention)
   - `Scripts\` ? `wwwroot\lib\` or `wwwroot\js\`
   - `Content\` ? `wwwroot\css\`
2. Update HTML references in views
3. Delete `App_Start\BundleConfig.cs`
4. Remove `Microsoft.AspNet.Web.Optimization` package
5. Add `app.UseStaticFiles();` to Program.cs

##### 9. Web.config ? appsettings.json

**Current** (`Web.config`):
```xml
<configuration>
  <appSettings>
    <add key="SomeKey" value="SomeValue" />
  </appSettings>
  <connectionStrings>
    <add name="DefaultConnection" 
         connectionString="Server=...;Database=...;" 
         providerName="System.Data.SqlClient" />
  </connectionStrings>
</configuration>
```

**Target** (`appsettings.json`):
```json
{
  "AppSettings": {
    "SomeKey": "SomeValue"
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=...;"
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

**Migration Tasks**:
1. Create `appsettings.json` in project root
2. Create `appsettings.Development.json` for dev-specific settings
3. Migrate `<appSettings>` to JSON
4. Migrate `<connectionStrings>` to JSON
5. Update code to use `IConfiguration` instead of `ConfigurationManager`:

```csharp
// Old
var value = ConfigurationManager.AppSettings["SomeKey"];

// New (inject IConfiguration)
public class MyController : ControllerBase
{
    private readonly IConfiguration _configuration;
    
    public MyController(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public IActionResult Get()
    {
        var value = _configuration["AppSettings:SomeKey"];
        // ...
    }
}
```

**Web.config in ASP.NET Core**:
- Keep minimal `web.config` only for IIS deployment configuration
- Most application config moves to `appsettings.json`

##### 10. Controller Updates

**Current** (ASP.NET Web API):
```csharp
using System.Web.Http;

public class ValuesController : ApiController
{
    public IHttpActionResult Get()
    {
        return Ok(new[] { "value1", "value2" });
    }
}
```

**Target** (ASP.NET Core):
```csharp
using Microsoft.AspNetCore.Mvc;

[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new[] { "value1", "value2" });
    }
}
```

**Key Changes**:
- `using System.Web.Http` ? `using Microsoft.AspNetCore.Mvc`
- `ApiController` ? `ControllerBase`
- `IHttpActionResult` ? `IActionResult`
- Add `[Route]` and `[ApiController]` attributes
- Use `[HttpGet]`, `[HttpPost]`, etc. attributes

**Migration Tasks**:
1. Update all controller base classes
2. Update using directives
3. Add routing attributes
4. Update return types
5. Test each endpoint after migration

##### 11. Expected Breaking Changes

**System.Web Removal**:
- `System.Web.Http` ? `Microsoft.AspNetCore.Mvc`
- `HttpContext.Current` ? Use `HttpContext` from controller
- `HttpContext.Request` ? `Request` property on controller
- `HttpContext.Response` ? `Response` property on controller

**Configuration**:
- `ConfigurationManager` ? `IConfiguration` (injected)

**Dependency Injection**:
- Unity container ? Built-in DI container
- `IDependencyResolver` ? `IServiceProvider`

**JSON Serialization**:
- May need to configure Newtonsoft.Json explicitly if using specific settings:
  ```csharp
  builder.Services.AddControllers()
      .AddNewtonsoftJson(options => {
          // Configure Newtonsoft.Json settings if needed
      });
  ```
  Requires: `dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson`

**Bootstrap 3 ? 5**:
- CSS class changes (e.g., `panel` ? `card`, `hidden` ? `d-none`)
- Review and update HTML/Razor views
- Test UI after update

##### 12. Build and Resolve Errors

**Build Project**:
```bash
cd WebApiExample.WebApp
dotnet build
```

**Expected Errors**:
- Namespace not found (`System.Web.*`)
- Type not found (`ApiController`, `IHttpActionResult`)
- Configuration APIs (`ConfigurationManager`)

**Resolution Strategy**:
1. Address namespace/using directive errors
2. Update controller base classes and return types
3. Replace configuration access
4. Remove references to removed packages
5. Build iteratively, fixing errors in batches

**Success Criteria**:
- ? Build completes with 0 errors
- ? Warnings addressed or documented
- ? All controllers compile

##### 13. Runtime Testing

**Start Application**:
```bash
dotnet run
```

**Validation**:
1. ? Application starts without exceptions
2. ? Swagger UI accessible (if added) at `/swagger`
3. ? Test each API endpoint:
   - GET requests
   - POST requests
   - PUT/PATCH requests
   - DELETE requests
4. ? Dependency injection works (services instantiate correctly)
5. ? Database connectivity (if using DataStore)
6. ? Configuration values read correctly
7. ? Static files serve correctly (CSS, JS)
8. ? No runtime exceptions in logs

**Manual Testing Checklist**:
- [ ] All API endpoints respond
- [ ] Correct HTTP status codes returned
- [ ] JSON serialization/deserialization works
- [ ] Database operations function
- [ ] Error handling works
- [ ] Logging works

##### 14. UI Validation (if applicable)

**Bootstrap 5 Changes**:
- Test all pages/views
- Verify layout not broken
- Update CSS classes as needed

**jQuery 3.7.1**:
- Test all JavaScript functionality
- Verify no console errors
- Check AJAX calls still work

##### 15. Validation Checklist

- [ ] Project converted to SDK-style Web project
- [ ] Target framework updated to net10.0
- [ ] All incompatible packages removed
- [ ] All security-vulnerable packages updated
- [ ] ASP.NET Core framework reference added
- [ ] Global.asax deleted, Program.cs created
- [ ] Routes migrated (attribute routing or conventional)
- [ ] Unity DI migrated to built-in DI
- [ ] Bundling replaced with direct references or LibMan
- [ ] Web.config migrated to appsettings.json
- [ ] All controllers updated to ControllerBase
- [ ] All using directives updated
- [ ] Project builds without errors
- [ ] Project builds without warnings (or warnings justified)
- [ ] Application starts successfully
- [ ] All API endpoints tested and functional
- [ ] Database operations work
- [ ] Static files serve correctly
- [ ] UI displays correctly (if applicable)
- [ ] Git commit created: `git commit -m "Tier 3: Migrated WebApiExample.WebApp to ASP.NET Core on .NET 10.0"`

##### 16. Rollback Plan

**If Critical Blocker Found**:
```bash
git reset --hard HEAD~1
```

**If Partial Success**:
- Keep progress in feature branch
- Document blockers
- Seek assistance for specific issues
- Consider incremental approach (migrate one controller at a time in separate branch)

---

#### Tier 3 Completion Criteria

- ? WebApiExample.WebApp builds on net10.0 as ASP.NET Core Web API
- ? All security vulnerabilities addressed
- ? All incompatible packages removed/replaced
- ? Application starts and runs without errors
- ? All API endpoints functional and tested
- ? Dependency injection works correctly
- ? Database operations functional (integration with DataStore)
- ? Static files serve correctly
- ? Integration with Tier 1 (Common) and Tier 2 (DataStore) verified
- ? Ready for testing by Tier 4 (Tests)
- ? Changes committed to source control

**Note**: This is the most complex tier. Budget sufficient time for testing and issue resolution.

### Tier 4: WebApiExample.WebApp.Tests

**Current State**: .NET Framework 4.8, Legacy project format, 393 LOC, 3 dependencies (Common, DataStore, WebApp), 0 dependants

**Target State**: .NET 10.0, SDK-style test project format

**Complexity**: Low-Medium

---

#### Migration Steps

##### 1. Prerequisites

**Verify:**
- ? Tier 1 (WebApiExample.Common) on net10.0
- ? Tier 2 (WebApiExample.DataStore) on net10.0
- ? Tier 3 (WebApiExample.WebApp) on net10.0 as ASP.NET Core
- ? All tiers build successfully
- ? WebApp application runs and all endpoints functional
- ? Git working directory clean

**Dependencies Check**:
All three dependencies must be successfully migrated before proceeding with tests.

##### 2. Convert Project to SDK-Style

**Action**: Convert legacy test project to SDK-style format

```bash
cd WebApiExample.WebApp.Tests
try-convert -p WebApiExample.WebApp.Tests.csproj
```

**Manual Verification**:
- Verify all 4 test files included
- Verify project references to Common, DataStore, WebApp preserved
- Verify xUnit packages preserved

##### 3. Update Target Framework

**Action**: Change `<TargetFramework>` from `net48` to `net10.0`

**Edit**: `WebApiExample.WebApp.Tests\WebApiExample.WebApp.Tests.csproj`

**Change**:
```xml
<TargetFramework>net10.0</TargetFramework>
```

##### 4. Package Updates

**Packages to Update/Remove (Tier 4 Scope):**

**Security Vulnerabilities - CRITICAL:**

| Package | Current | Target | Action | Reason |
|---------|---------|--------|--------|--------|
| Newtonsoft.Json | 6.0.4 | 13.0.4 | Update | ?? Security vulnerability |

**Incompatible Packages - REMOVE (built into framework):**

| Package | Current | Action | Replacement |
|---------|---------|--------|-------------|
| Microsoft.AspNet.Mvc | 5.2.7 | Remove | Not needed for tests |
| Microsoft.AspNet.Razor | 3.2.7 | Remove | Not needed for tests |
| Microsoft.AspNet.WebApi.Core | 5.2.7 | Remove | Not needed for tests |
| Microsoft.AspNet.WebApi.WebHost | 5.2.7 | Remove | Not needed for tests |
| Microsoft.AspNet.WebPages | 3.2.7 | Remove | Not needed for tests |
| Microsoft.Web.Infrastructure | 1.0.0.0 | Remove | Not needed for tests |
| System.Buffers | 4.5.1 | Remove | Included in .NET 10.0 |
| System.Memory | 4.5.4 | Remove | Included in .NET 10.0 |
| System.Numerics.Vectors | 4.5.0 | Remove | Included in .NET 10.0 |
| System.Runtime.InteropServices.RuntimeInformation | 4.3.0 | Remove | Included in .NET 10.0 |
| System.Threading.Tasks.Extensions | 4.5.4 | Remove | Included in .NET 10.0 |

**Recommended Updates:**

| Package | Current | Target | Reason |
|---------|---------|--------|--------|
| System.Runtime.CompilerServices.Unsafe | 4.5.3 | 6.1.2 | Recommended upgrade |

**Compatible Packages - RETAIN:**

| Package | Version | Action |
|---------|---------|--------|
| Microsoft.AspNet.WebApi.Client | 5.2.7 | Keep (compatible) |
| Castle.Core | 4.4.0 | Keep (compatible, used by Moq) |
| DiffEngine | 6.4.9 | Keep (compatible) |
| EmptyFiles | 2.3.3 | Keep (compatible) |
| Microsoft.CSharp | 4.7.0 | Keep (compatible) |
| Moq | 4.16.1 | Keep (compatible mocking framework) |
| Shouldly | 4.0.3 | Keep (compatible assertion library) |
| xunit | 2.4.1 | Keep (compatible test framework) |
| xunit.abstractions | 2.0.3 | Keep |
| xunit.analyzers | 0.10.0 | Keep |
| xunit.assert | 2.4.1 | Keep |
| xunit.core | 2.4.1 | Keep |
| xunit.extensibility.core | 2.4.1 | Keep |
| xunit.extensibility.execution | 2.4.1 | Keep |
| xunit.runner.console | 2.4.1 | Keep |
| xunit.runner.visualstudio | 2.4.3 | Keep (test adapter for VS) |

**Update Process**:

1. **Remove incompatible packages** (edit .csproj, remove `<PackageReference>` lines for packages listed as "Remove")

2. **Update security-critical package**:
   ```bash
   dotnet add package Newtonsoft.Json --version 13.0.4
   dotnet add package System.Runtime.CompilerServices.Unsafe --version 6.1.2
   ```

3. **Restore packages**:
   ```bash
   dotnet restore
   ```

##### 5. Expected Breaking Changes

**ASP.NET Framework ? ASP.NET Core Testing Changes:**

**Potential Issues:**
1. **Controller instantiation** - Tests may need updates if they directly instantiate controllers:
   ```csharp
   // Old (may still work)
   var controller = new MyController(dependencies...);
   
   // If using HttpContext, may need to mock differently
   ```

2. **HttpContext mocking** - ASP.NET Core uses different `HttpContext`:
   ```csharp
   // Old
   var httpContext = new Mock<HttpContextBase>();
   
   // New (if needed)
   var httpContext = new DefaultHttpContext();
   ```

3. **IHttpActionResult ? IActionResult**:
   ```csharp
   // Old
   IHttpActionResult result = controller.Get();
   
   // New
   IActionResult result = controller.Get();
   ```

**xUnit Compatibility**:
- xUnit 2.4.1 is compatible with .NET 10.0
- No breaking changes expected in test framework itself

**Moq Compatibility**:
- Moq 4.16.1 is compatible with .NET 10.0
- No changes needed for mocking

##### 6. Code Modifications

**Review Test Files**:

1. **Update using directives**:
   ```csharp
   // Remove
   using System.Web.Http;
   using System.Web.Http.Results;
   
   // Add (if needed)
   using Microsoft.AspNetCore.Mvc;
   ```

2. **Update controller references**:
   - Ensure tests reference migrated WebApp controllers
   - Update base class expectations (`ApiController` ? `ControllerBase`)

3. **Update assertion types**:
   ```csharp
   // Old
   var okResult = result as OkNegotiatedContentResult<Model>;
   
   // New
   var okResult = result as OkObjectResult;
   var model = okResult.Value as Model;
   ```

4. **Check for ConfigurationManager usage**:
   - If tests use `ConfigurationManager`, replace with configuration mocks
   - Inject `IConfiguration` mock if needed

##### 7. Build and Validate

**Build Project**:
```bash
cd WebApiExample.WebApp.Tests
dotnet build
```

**Success Criteria**:
- ? Build completes with 0 errors
- ? Build completes with 0 warnings (or justified warnings)
- ? All test files compile
- ? References to WebApp, DataStore, Common resolved

##### 8. Run Tests

**Execute All Tests**:
```bash
dotnet test
```

**Success Criteria**:
- ? All tests discovered by test runner
- ? All tests pass (or understand why they fail)
- ? No test infrastructure errors

**If Tests Fail**:
1. Review failure messages
2. Identify category of failure:
   - Type mismatch (IHttpActionResult ? IActionResult)
   - Assertion differences
   - Mock behavior changes
   - WebApp behavioral changes
3. Update tests to match new framework behavior
4. Re-run tests until all pass

**Test Categories to Verify**:
- Unit tests for controllers
- Integration tests (if any)
- Mocked dependency tests
- Data access tests (using DataStore)

##### 9. Integration Validation

**Full Solution Validation**:

After Tier 4 completion, verify entire solution:

```bash
# Build entire solution
dotnet build WebApiExample.sln

# Run all tests
dotnet test WebApiExample.sln
```

**Success Criteria**:
- ? All 4 projects build successfully
- ? No errors across solution
- ? No warnings across solution (or all justified)
- ? All tests pass

##### 10. Validation Checklist

- [ ] Project converted to SDK-style format
- [ ] Target framework updated to net10.0
- [ ] Newtonsoft.Json updated to 13.0.4 (security fix)
- [ ] System.Runtime.CompilerServices.Unsafe updated to 6.1.2
- [ ] All incompatible packages removed
- [ ] All framework-included packages removed
- [ ] Project builds without errors
- [ ] Project builds without warnings
- [ ] References to Common, DataStore, WebApp (all net10.0) resolved
- [ ] All tests discovered by test runner
- [ ] All tests pass
- [ ] Full solution builds successfully
- [ ] Git commit created: `git commit -m "Tier 4: Migrated WebApiExample.WebApp.Tests to .NET 10.0"`

##### 11. Rollback Plan

**If Tests Cannot Be Fixed**:
```bash
git reset --hard HEAD~1
```

**If Specific Test Issues**:
- Isolate failing tests
- Skip temporarily with `[Fact(Skip = "Reason")]`
- Document issues
- Fix incrementally

---

#### Tier 4 Completion Criteria

- ? WebApiExample.WebApp.Tests builds on net10.0
- ? Security vulnerability in Newtonsoft.Json addressed
- ? All incompatible and framework-included packages removed
- ? All tests pass
- ? Integration with all dependencies (Common, DataStore, WebApp) verified
- ? Full solution builds without errors or warnings
- ? Changes committed to source control

---

#### Full Migration Completion

**After Tier 4, the entire solution is successfully migrated to .NET 10.0:**

? All 4 projects targeting net10.0
? All projects using SDK-style format
? All security vulnerabilities addressed
? All incompatible packages removed/replaced
? ASP.NET Framework ? ASP.NET Core migration complete
? All tests passing
? Full solution builds and runs successfully

**Proceed to final validation and success criteria verification.**

## Package Update Reference

### Consolidated Package Updates by Tier

**Tier 1 (WebApiExample.Common)**:
- No package updates required

**Tier 2 (WebApiExample.DataStore)**:
| Package | Current Version | Target Version | Reason |
|---------|----------------|----------------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | Recommended upgrade for .NET 10.0 compatibility |

**Tier 3 (WebApiExample.WebApp)**:

*Security Vulnerabilities (CRITICAL):*
| Package | Current Version | Target Version | CVE/Risk |
|---------|----------------|----------------|----------|
| bootstrap | 3.3.7 | 5.3.8 | Known security vulnerabilities |
| jQuery | 3.3.1 | 3.7.1 | XSS and prototype pollution vulnerabilities |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | Deserialization vulnerabilities |

*Incompatible - Remove (built into ASP.NET Core):*
- Microsoft.AspNet.Mvc (5.2.4)
- Microsoft.AspNet.Razor (3.2.4)
- Microsoft.AspNet.WebApi (5.2.4)
- Microsoft.AspNet.WebApi.Core (5.2.7)
- Microsoft.AspNet.WebApi.WebHost (5.2.4)
- Microsoft.AspNet.WebPages (3.2.4)
- Microsoft.CodeDom.Providers.DotNetCompilerPlatform (2.0.0)
- Microsoft.Web.Infrastructure (1.0.0.0)

*Incompatible - Replace:*
| Package | Current Version | Replacement |
|---------|----------------|-------------|
| Microsoft.AspNet.Web.Optimization | 1.1.3 | Direct HTML references or modern bundler |
| Unity.WebAPI | 5.4.0 | Microsoft.Extensions.DependencyInjection (built-in) |
| Antlr | 3.5.0.2 | Antlr4 (4.6.6) if needed, or remove if unused |

*Recommended Updates:*
| Package | Current Version | Target Version | Reason |
|---------|----------------|----------------|--------|
| EntityFramework | 6.4.4 | 6.5.1 | Consistency with DataStore |
| System.Runtime.CompilerServices.Unsafe | 4.5.2 | 6.1.2 | Recommended upgrade |

*Compatible - Retain:*
- Microsoft.AspNet.WebApi.Client (5.2.7)
- Microsoft.AspNet.WebApi.HelpPage (5.2.4)
- Unity (5.11.10) - optional, prefer built-in DI
- WebGrease (1.6.0)
- Modernizr (2.8.3)

**Tier 4 (WebApiExample.WebApp.Tests)**:

*Security Vulnerabilities:*
| Package | Current Version | Target Version | CVE/Risk |
|---------|----------------|----------------|----------|
| Newtonsoft.Json | 6.0.4 | 13.0.4 | Deserialization vulnerabilities |

*Incompatible - Remove:*
- Microsoft.AspNet.Mvc (5.2.7)
- Microsoft.AspNet.Razor (3.2.7)
- Microsoft.AspNet.WebApi.Core (5.2.7)
- Microsoft.AspNet.WebApi.WebHost (5.2.7)
- Microsoft.AspNet.WebPages (3.2.7)
- Microsoft.Web.Infrastructure (1.0.0.0)
- System.Buffers (4.5.1)
- System.Memory (4.5.4)
- System.Numerics.Vectors (4.5.0)
- System.Runtime.InteropServices.RuntimeInformation (4.3.0)
- System.Threading.Tasks.Extensions (4.5.4)

*Recommended Updates:*
| Package | Current Version | Target Version | Reason |
|---------|----------------|----------------|--------|
| System.Runtime.CompilerServices.Unsafe | 4.5.3 | 6.1.2 | Recommended upgrade |

*Compatible - Retain (Testing Frameworks):*
- Castle.Core (4.4.0)
- DiffEngine (6.4.9)
- EmptyFiles (2.3.3)
- Microsoft.AspNet.WebApi.Client (5.2.7)
- Microsoft.CSharp (4.7.0)
- Moq (4.16.1)
- Shouldly (4.0.3)
- xunit (2.4.1)
- xunit.* packages (all 2.4.x versions)

## Breaking Changes Catalog

### .NET Framework 4.8 ? .NET 10.0 Breaking Changes

#### API Removals

**System.Web Namespace Removal**:
- **Impact**: High - Affects WebApp project significantly
- **APIs Removed**:
  - `System.Web.Http.*` (ASP.NET Web API)
  - `System.Web.Mvc.*` (ASP.NET MVC)
  - `System.Web.Optimization.*` (Bundling/Minification)
  - `HttpContext.Current`
  - `HttpContextBase`
  - `ConfigurationManager` (from System.Configuration)
- **Replacement**:
  - Use `Microsoft.AspNetCore.Mvc` namespace
  - Use `HttpContext` from controller context
  - Use `IConfiguration` dependency injection
- **Affected Files**: All controllers, Global.asax.cs, configuration code

**Configuration API Changes**:
- **Old**: `ConfigurationManager.AppSettings["key"]`
- **New**: `IConfiguration["key"]` (injected dependency)
- **Impact**: Medium - Requires code changes wherever configuration accessed
- **Migration**: Inject `IConfiguration`, update all access points

#### Behavior Changes

**Dependency Injection Paradigm**:
- **Old**: Unity container explicitly configured, resolver set on `GlobalConfiguration`
- **New**: Built-in DI container, services registered in `Program.cs`
- **Impact**: High - Requires DI configuration migration
- **Migration**: Map all Unity registrations to `builder.Services` methods

**Application Initialization**:
- **Old**: `Global.asax.cs` with `Application_Start()` event
- **New**: `Program.cs` with `WebApplication` builder pattern
- **Impact**: High - Architectural change
- **Migration**: Move initialization logic to Program.cs

**Routing**:
- **Old**: `RouteConfig` with conventional routing, or `MapHttpAttributeRoutes()`
- **New**: Attribute routing on controllers, or `MapControllers()` in Program.cs
- **Impact**: Medium - Requires route configuration review
- **Migration**: Prefer attribute routing with `[Route]` and `[HttpGet]`, etc.

**Static Files**:
- **Old**: `Content\` and `Scripts\` folders automatically served
- **New**: `wwwroot\` folder convention, requires `app.UseStaticFiles()` middleware
- **Impact**: Medium - File structure change
- **Migration**: Move static files to `wwwroot\`, update HTML references

**Bundling & Minification**:
- **Old**: `System.Web.Optimization` with `BundleConfig`
- **New**: No built-in bundling, use direct references or external tools
- **Impact**: Medium - Configuration change
- **Migration**: Replace with direct `<script>` and `<link>` tags initially

#### Package-Specific Breaking Changes

**Entity Framework 6.4.4 ? 6.5.1**:
- **Impact**: Low - Minor version update, minimal breaking changes
- **Potential Issues**: 
  - LINQ query evaluation differences (rare)
  - Performance improvements may expose existing issues
- **Mitigation**: Test all database operations thoroughly

**Newtonsoft.Json 11.0.1/6.0.4 ? 13.0.4**:
- **Impact**: Low-Medium - Major version jump
- **Known Breaking Changes**:
  - Default `DateParseHandling` changed
  - Floating-point number parsing stricter
  - Some serialization edge cases changed
- **Mitigation**: 
  - Test JSON serialization/deserialization
  - Configure `JsonSerializerSettings` explicitly if needed
  - Consider migrating to `System.Text.Json` in future (ASP.NET Core default)

**bootstrap 3.3.7 ? 5.3.8**:
- **Impact**: High - Major version jump (3.x ? 5.x)
- **Breaking CSS Changes**:
  - `.panel` ? `.card`
  - `.well` ? `.card` or custom class
  - `.hidden` ? `.d-none`
  - `.show` ? `.d-block`, `.d-inline`, etc.
  - Grid system changes (some classes renamed)
  - Form control classes updated
  - Button classes remain mostly compatible
- **Mitigation**: 
  - Review all HTML/Razor views
  - Update CSS classes systematically
  - Test UI thoroughly
  - Refer to [Bootstrap Migration Guide](https://getbootstrap.com/docs/5.3/migration/)

**jQuery 3.3.1 ? 3.7.1**:
- **Impact**: Low - Patch version update within 3.x
- **Breaking Changes**: Minimal (3.3.1 ? 3.7.1 is backward compatible)
- **Mitigation**: Test JavaScript functionality, check for deprecation warnings in console

#### Controller & Action Result Changes

**Base Class Changes**:
```csharp
// OLD
public class MyController : ApiController
{
    public IHttpActionResult Get() { ... }
}

// NEW
[ApiController]
[Route("api/[controller]")]
public class MyController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() { ... }
}
```

**Return Type Changes**:
- `IHttpActionResult` ? `IActionResult`
- `OkNegotiatedContentResult<T>` ? `OkObjectResult`
- `BadRequestResult` ? `BadRequestResult` (compatible)
- `NotFoundResult` ? `NotFoundResult` (compatible)
- Helper methods mostly compatible: `Ok()`, `BadRequest()`, `NotFound()`, etc.

#### Configuration File Changes

**Web.config ? appsettings.json**:
- **Impact**: High - Complete configuration paradigm shift
- **Changes**:
  - XML ? JSON format
  - `<appSettings>` ? JSON object
  - `<connectionStrings>` ? `"ConnectionStrings"` JSON object
  - System.web configuration no longer applicable (ASP.NET Core uses different middleware model)
- **Web.config Retained For**: IIS deployment settings only

### Expected vs. Possible Breaking Changes

**Expected (High Confidence)**:
- ? System.Web removal requires namespace updates
- ? ApiController ? ControllerBase requires base class changes
- ? Global.asax ? Program.cs requires initialization migration
- ? Unity ? Built-in DI requires DI configuration migration
- ? Bootstrap 3 ? 5 requires CSS class updates

**Possible (Lower Confidence)**:
- ?? Newtonsoft.Json serialization behavior differences
- ?? EF 6.5.1 LINQ query evaluation changes
- ?? Custom HTTP modules/handlers (if any) need ASP.NET Core middleware equivalents
- ?? Custom build targets in legacy .csproj may need recreation
- ?? Application-specific configuration edge cases

### Mitigation Strategy

1. **Tier-by-tier validation** catches breaking changes early
2. **Comprehensive testing** after each tier
3. **Incremental updates** (security fixes first, then framework packages)
4. **Maintain old branch** for comparison/rollback
5. **Document issues** as discovered for future reference

## Risk Management

### High-Risk Changes

| Project | Risk Level | Description | Mitigation Strategy |
|---------|------------|-------------|---------------------|
| WebApiExample.WebApp | ?? High | ASP.NET Framework ? ASP.NET Core migration involves significant architectural changes (Global.asax ? Program.cs, Web.config ? appsettings.json, different middleware pipeline) | Dedicate separate tier/phase; thorough testing of all endpoints; manual validation of application startup and DI container; incremental feature migration |
| WebApiExample.WebApp | ?? High | Bundling/minification replacement (System.Web.Optimization incompatible) | Replace with direct HTML references initially (simplest); consider modern bundling tools (webpack, Vite) in future iteration; validate all static assets load correctly |
| WebApiExample.WebApp | ?? High | Unity DI container ? Microsoft.Extensions.DependencyInjection migration | Map all Unity registrations to built-in DI; test all controllers receive correct dependencies; verify lifetime scopes (singleton, scoped, transient) match original behavior |
| WebApiExample.WebApp | ?? Medium | 19 package updates/removals/replacements | Update packages one category at a time (security first, then framework packages, then utilities); build and test after each category |
| WebApiExample.DataStore | ?? Medium | Entity Framework 6.4.4 ? 6.5.1 upgrade | Review EF 6.5.1 release notes; test all database operations; verify migration scripts still work; check for behavioral changes in LINQ queries |
| All Projects | ?? Medium | SDK-style project conversion (legacy ? modern format) | Use automated conversion tool; manually verify all files included; check for custom build steps or targets that need migration |

### Security Vulnerabilities

**Critical - Must Remediate Immediately:**

| Package | Current Version | Target Version | Projects Affected | CVE/Risk |
|---------|----------------|----------------|-------------------|----------|
| bootstrap | 3.3.7 | 5.3.8 | WebApiExample.WebApp | Known security vulnerabilities in 3.x versions |
| jQuery | 3.3.1 | 3.7.1 | WebApiExample.WebApp | Multiple XSS and prototype pollution vulnerabilities in 3.3.x |
| Newtonsoft.Json | 11.0.1 | 13.0.4 | WebApiExample.WebApp | Deserialization vulnerabilities in versions < 13.0.1 |
| Newtonsoft.Json | 6.0.4 | 13.0.4 | WebApiExample.WebApp.Tests | Deserialization vulnerabilities in versions < 13.0.1 |

**Remediation Plan:**
1. Update Newtonsoft.Json immediately (critical deserialization risk)
2. Update jQuery to 3.7.1 (XSS risks)
3. Update bootstrap to 5.3.8 (note: may require HTML/CSS updates due to breaking changes in bootstrap 4.x ? 5.x)
4. Test all JavaScript functionality after jQuery update
5. Verify all UI components after bootstrap update

### Contingency Plans

**Scenario: SDK Conversion Fails to Include All Files**
- **Alternative**: Manually review legacy .csproj and ensure all `<Compile>`, `<Content>`, and `<None>` items are accounted for
- **Fallback**: Create SDK-style project from scratch, copy settings/references manually
- **Validation**: Compare file count and build output before/after conversion

**Scenario: Entity Framework 6.5.1 Introduces Breaking Changes**
- **Alternative**: Remain on 6.4.4 temporarily (still compatible with net10.0)
- **Investigation**: Review EF 6.5.1 release notes and GitHub issues for known problems
- **Rollback**: Downgrade to 6.4.4 if critical issues found

**Scenario: ASP.NET Core Migration Encounters Blocker**
- **Alternative**: Consider multi-targeting (net48 + net10.0) temporarily
- **Incremental Approach**: Migrate one controller at a time if feasible
- **Rollback**: Maintain net48 version in separate branch while troubleshooting
- **Escalation**: Seek community/Microsoft support for specific blockers

**Scenario: Unity ? Microsoft DI Migration Too Complex**
- **Alternative**: Use Unity.Microsoft.DependencyInjection adapter package temporarily
- **Long-term**: Gradually migrate registrations to built-in DI
- **Validation**: Ensure all controllers and services instantiate correctly

**Scenario: Bundling/Minification Replacement Causes Asset Loading Issues**
- **Immediate Fix**: Use direct `<script>` and `<link>` references (unminified if needed)
- **Alternative**: Use LibMan or npm for client-side package management
- **Future Enhancement**: Implement webpack or Vite in separate iteration

**Scenario: Tests Fail After Migration**
- **Investigation**: Check for framework behavior differences (e.g., async/await handling, null reference behavior)
- **Tooling**: Use .NET Portability Analyzer to identify API differences
- **Mitigation**: Update test assertions/mocks to match new framework behavior

### Risk Factors by Tier

**Tier 1 (Common) - Low Risk:**
- Simple models and utilities
- No external dependencies
- No framework-specific features
- **Key Risk**: Ensure all consuming projects can reference updated Common

**Tier 2 (DataStore) - Low-Medium Risk:**
- Entity Framework upgrade (minor version)
- Database connectivity unchanged
- **Key Risk**: EF 6.5.1 compatibility, database operation validation

**Tier 3 (WebApp) - High Risk:**
- Framework paradigm shift (ASP.NET Framework ? Core)
- Many incompatible packages
- Architectural changes required
- Security vulnerabilities to address
- **Key Risk**: Application functionality preservation, endpoint compatibility

**Tier 4 (Tests) - Low-Medium Risk:**
- Test framework compatibility
- Package updates
- **Key Risk**: Test behavior changes, mock compatibility

## Testing & Validation Strategy

### Multi-Level Testing Approach

This migration employs a comprehensive, tier-by-tier testing strategy to ensure stability at each stage.

---

### Tier 1: WebApiExample.Common

#### Smoke Tests
- [x] **Build Validation**
  - Project builds on net10.0 without errors
  - No build warnings
  - Output assembly generated in `bin\Debug\net10.0\`

- [x] **API Surface Validation**
  - All public types remain accessible
  - No inadvertent API changes
  - Namespace resolution works

#### Unit Tests
- None identified for Common project
- Validation deferred to integration with higher tiers

#### Integration Tests
- **Validation**: Higher tiers (DataStore, WebApp, Tests) can reference Common
- **Performed During**: Tier 2, 3, 4 migrations

---

### Tier 2: WebApiExample.DataStore

#### Smoke Tests
- [x] **Build Validation**
  - Project builds on net10.0 without errors
  - No build warnings
  - EntityFramework 6.5.1 restored successfully
  - References to Common (net10.0) resolved

- [x] **Dependency Validation**
  - No package conflicts
  - All dependencies compatible

#### Database Tests
- [x] **Connection Test**
  ```csharp
  using (var context = new YourDbContext())
  {
      var canConnect = context.Database.Exists();
      Assert.True(canConnect);
  }
  ```

- [x] **Basic Query Test**
  - Simple LINQ query executes without error
  - Record count retrieval works
  - No SQL generation issues

- [x] **Migration Compatibility** (if applicable)
  - Migration scripts still compile
  - Database schema matches expectations

#### Unit Tests
- Deferred to Tier 4 (WebApp.Tests) if tests exist there
- Manual validation sufficient for this tier

#### Integration Tests
- **Validation**: DataStore integrates with Common successfully
- **Validation**: WebApp (Tier 3) can consume DataStore

---

### Tier 3: WebApiExample.WebApp

#### Smoke Tests
- [x] **Build Validation**
  - Project builds as ASP.NET Core on net10.0 without errors
  - Warnings addressed or justified
  - All controllers compile
  - Static files included in output

- [x] **Startup Validation**
  ```bash
  dotnet run
  ```
  - Application starts without exceptions
  - No runtime errors in startup logs
  - Kestrel server listening on expected port

- [x] **DI Container Validation**
  - All services registered successfully
  - No circular dependencies
  - Controllers instantiate via DI

#### API Endpoint Tests

**For Each Controller/Endpoint**:
- [x] **GET Requests**
  - Endpoint responds with correct status code
  - Response payload structure correct
  - JSON serialization works

- [x] **POST Requests**
  - Endpoint accepts payload
  - Validation works
  - Data persisted correctly (if applicable)
  - Correct status code returned

- [x] **PUT/PATCH Requests** (if applicable)
  - Update operations work
  - Validation applied correctly

- [x] **DELETE Requests** (if applicable)
  - Delete operations work
  - Correct status code returned

**Manual Testing Checklist**:
```bash
# Example with curl or Postman
curl http://localhost:5000/api/values
curl -X POST http://localhost:5000/api/values -H "Content-Type: application/json" -d '{"value":"test"}'
```

#### Database Integration Tests
- [x] **CRUD Operations**
  - Create: New entities saved via WebApp endpoints
  - Read: Entities retrieved via WebApp endpoints
  - Update: Entities modified via WebApp endpoints
  - Delete: Entities removed via WebApp endpoints

- [x] **Transaction Handling**
  - Successful transactions commit
  - Failed operations rollback

#### UI/Static File Tests (if applicable)
- [x] **Static Files Served**
  - CSS files load from `wwwroot\css\`
  - JavaScript files load from `wwwroot\js\` or `wwwroot\lib\`
  - No 404 errors for static assets

- [x] **Bootstrap 5 Compatibility**
  - Layout renders correctly
  - Components display properly
  - Responsive behavior works

- [x] **jQuery Functionality**
  - JavaScript executes without errors
  - AJAX calls work
  - No console errors

#### Configuration Tests
- [x] **Settings Read**
  - `appsettings.json` values accessible via `IConfiguration`
  - Environment-specific settings work (`appsettings.Development.json`)
  - Connection strings read correctly

#### Security Tests
- [x] **Vulnerability Remediation**
  - bootstrap 5.3.8 confirmed (no vulnerable 3.x)
  - jQuery 3.7.1 confirmed (no vulnerable 3.3.1)
  - Newtonsoft.Json 13.0.4 confirmed (no vulnerable 11.0.1)

---

### Tier 4: WebApiExample.WebApp.Tests

#### Smoke Tests
- [x] **Build Validation**
  - Test project builds on net10.0 without errors
  - No build warnings
  - References to Common, DataStore, WebApp (all net10.0) resolved

- [x] **Test Discovery**
  ```bash
  dotnet test --list-tests
  ```
  - All test methods discovered by xUnit
  - No test discovery errors

#### Unit Test Execution
- [x] **Run All Tests**
  ```bash
  dotnet test
  ```
  - All tests execute
  - Test results reported correctly

- [x] **Test Pass Rate**
  - Target: 100% pass rate
  - If failures: Investigate, categorize, fix

- [x] **Test Categories** (if organized):
  - Controller tests
  - Service tests
  - Data access tests
  - Integration tests

#### Test-Specific Validations
- [x] **Mocking Compatibility**
  - Moq 4.16.1 works with net10.0
  - Mock setups still valid
  - Verification works correctly

- [x] **Assertion Libraries**
  - Shouldly assertions work
  - xUnit assertions work

- [x] **Test Fixtures**
  - Test setup/teardown executes correctly
  - Shared context works

---

### Full Solution Validation (Post-Tier 4)

#### Comprehensive Build
```bash
cd C:\Repos\Demo\net48-web-api-example
dotnet build WebApiExample.sln
```

**Success Criteria**:
- [x] All 4 projects build successfully
- [x] Zero errors across solution
- [x] Zero warnings across solution (or all justified/documented)
- [x] Total build time acceptable

#### Comprehensive Test Run
```bash
dotnet test WebApiExample.sln
```

**Success Criteria**:
- [x] All test projects discovered
- [x] All tests pass (100% pass rate)
- [x] No test infrastructure errors
- [x] Test execution time acceptable

#### End-to-End Scenario Tests

**Scenario 1: Happy Path CRUD**
1. Start WebApp: `dotnet run --project WebApiExample.WebApp`
2. Create entity via POST
3. Retrieve entity via GET
4. Update entity via PUT
5. Delete entity via DELETE
6. Verify all operations succeed

**Scenario 2: Error Handling**
1. POST with invalid data ? 400 Bad Request
2. GET non-existent entity ? 404 Not Found
3. Verify error responses formatted correctly

**Scenario 3: Database Persistence**
1. Create multiple entities
2. Stop and restart application
3. Verify entities persisted (if using real database)

#### Performance Validation
- [x] **Startup Time**: Application starts within acceptable timeframe
- [x] **Response Time**: API endpoints respond within acceptable SLA
- [x] **Memory Usage**: No obvious memory leaks or excessive allocations
- [x] **Database Query Performance**: No significant regression vs. .NET Framework version

#### Security Validation
- [x] **Vulnerability Scan**
  ```bash
  dotnet list package --vulnerable
  ```
  - Zero vulnerabilities reported

- [x] **Package Audit**
  - All packages at secure versions
  - No deprecated packages (or justified if retained)

---

### Test Failure Handling

**If Tests Fail**:

1. **Categorize Failure**:
   - Build error ? Fix compilation issues
   - Test infrastructure error ? Fix test setup
   - Assertion failure ? Investigate behavioral change
   - Timeout ? Performance issue or deadlock

2. **Investigate Root Cause**:
   - Compare .NET Framework vs .NET 10.0 behavior
   - Check for breaking changes in dependencies
   - Review migration changes for unintended side effects

3. **Resolution Approach**:
   - **Option 1**: Update test to match new framework behavior (if behavior change is correct)
   - **Option 2**: Fix code to maintain original behavior (if test was correct)
   - **Option 3**: Document known difference, create tracking issue

4. **Regression Prevention**:
   - Add test for discovered issue
   - Document breaking change in catalog
   - Update migration plan if needed

---

### Test Documentation

**After Each Tier**:
- Document test results (pass/fail counts)
- Note any issues discovered
- Record time spent on testing
- Update lessons learned

**Final Test Report Should Include**:
- Total tests executed
- Pass rate
- Time to execute
- Any known issues or limitations
- Performance baseline for future comparison

## Complexity & Effort Assessment

### Per-Project Complexity

| Project | Complexity | Dependencies | Package Updates | Code Changes | Risk Level | Rationale |
|---------|------------|--------------|-----------------|--------------|------------|-----------|
| WebApiExample.Common | ?? Low | 0 | 0 | Minimal | Low | Small codebase (61 LOC), no packages, simple models/utilities, no framework-specific features |
| WebApiExample.DataStore | ?? Medium | 1 | 2 | Minor | Low-Medium | Small codebase (110 LOC), EF upgrade (6.4.4?6.5.1), database operations need validation |
| WebApiExample.WebApp | ?? High | 2 | 19 | Major | High | Large codebase (3,612 LOC), ASP.NET Framework?Core migration, 8 incompatible packages, architectural changes, security fixes |
| WebApiExample.WebApp.Tests | ?? Medium | 3 | 14 | Moderate | Low-Medium | Medium codebase (393 LOC), many package updates, test framework compatibility, depends on migrated WebApp |

### Phase Complexity Assessment

**Phase 1: Tier 1 (Common)**
- **Complexity**: ?? Low
- **Effort**: Minimal - straightforward SDK conversion and framework update
- **Duration**: Short
- **Challenges**: None expected
- **Dependencies**: No blockers

**Phase 2: Tier 2 (DataStore)**
- **Complexity**: ?? Medium
- **Effort**: Moderate - SDK conversion, framework update, EF upgrade, database validation
- **Duration**: Short-Medium
- **Challenges**: Entity Framework 6.5.1 compatibility verification
- **Dependencies**: Requires Tier 1 completion

**Phase 3: Tier 3 (WebApp)**
- **Complexity**: ?? High
- **Effort**: Significant - ASP.NET migration, package replacement, architectural changes, security fixes
- **Duration**: Long (majority of migration effort)
- **Challenges**:
  - Global.asax ? Program.cs migration
  - Bundling/minification replacement
  - Unity ? built-in DI conversion
  - Web.config ? appsettings.json
  - 19 package updates/removals/replacements
  - Endpoint compatibility validation
  - UI asset loading verification
- **Dependencies**: Requires Tier 1 & 2 completion

**Phase 4: Tier 4 (Tests)**
- **Complexity**: ?? Medium
- **Effort**: Moderate - SDK conversion, framework update, many package updates, test compatibility
- **Duration**: Medium
- **Challenges**: Test behavior compatibility, mock updates, assertion adjustments
- **Dependencies**: Requires Tier 1, 2, & 3 completion

### Resource Requirements

**Skill Levels Required:**

- **Tier 1 & 2**: Junior-to-mid level developer familiar with .NET SDK-style projects
- **Tier 3**: Senior developer with experience in:
  - ASP.NET Framework and ASP.NET Core
  - Dependency injection patterns
  - Web API migration
  - Client-side asset management
  - Security vulnerability remediation
- **Tier 4**: Mid-level developer with testing experience

**Parallel Capacity**: N/A - Sequential execution required due to single-project tiers

**Recommended Staffing**: 1-2 developers (1 primary for Tier 3 complexity, optional second for reviews/pairing)

### Relative Effort Distribution

```
Tier 1 (Common):        ???????????????????? 10%
Tier 2 (DataStore):     ???????????????????? 20%
Tier 3 (WebApp):        ???????????????????? 55%
Tier 4 (Tests):         ???????????????????? 15%
```

**Note**: Effort percentages are relative estimates based on complexity factors. Actual effort depends on:
- Team familiarity with ASP.NET Core
- Discovery of unforeseen breaking changes
- Testing thoroughness requirements
- Issue resolution time

### Incremental Benefits

**After Tier 1 Completion:**
- ? Shared models/utilities on .NET 10.0
- ? Foundation stable for higher tiers
- ? Lessons learned from SDK conversion

**After Tier 2 Completion:**
- ? Data layer on .NET 10.0
- ? Entity Framework 6.5.1 benefits
- ? Database operations validated on new framework

**After Tier 3 Completion:**
- ? Modern ASP.NET Core Web API on .NET 10.0
- ? Security vulnerabilities remediated
- ? Incompatible package dependencies resolved
- ? Application functional on modern framework
- ? Ready for production deployment

**After Tier 4 Completion:**
- ? Full solution on .NET 10.0
- ? All tests passing
- ? Comprehensive validation complete
- ? Migration fully verified

## Source Control Strategy

### Branching Strategy

**Branch Structure**:

```
main (original: .NET Framework 4.8)
  ??? upgrade-to-NET10 (migration work happens here)
       ??? feature/tier-1-common (optional: isolate tier work)
       ??? feature/tier-2-datastore (optional)
       ??? feature/tier-3-webapp (optional)
       ??? feature/tier-4-tests (optional)
```

**Primary Approach**: Work directly on `upgrade-to-NET10` branch

**Alternative Approach**: Create sub-branches per tier for very cautious/reviewable approach

**Selected**: **Primary Approach** (single `upgrade-to-NET10` branch)
- Simpler workflow
- Tier-by-tier commits provide granular history
- Can still rollback individual tiers via Git history

---

### Commit Strategy

#### Commit Frequency

**Per Tier**:
- **Minimum**: 1 commit per tier completion
- **Recommended**: Multiple commits per tier at logical checkpoints

**Checkpoint Examples**:
1. After SDK conversion: `git commit -m "Tier X: Convert to SDK-style"`
2. After framework update: `git commit -m "Tier X: Update to net10.0"`
3. After package updates: `git commit -m "Tier X: Update packages"`
4. After code changes: `git commit -m "Tier X: Migrate code to ASP.NET Core"`
5. After validation: `git commit -m "Tier X: All tests passing"`

#### Commit Message Format

**Template**:
```
Tier X: <Descriptive summary>

- Detail 1
- Detail 2
- Detail 3

[Optional: Notes about issues encountered or decisions made]
```

**Examples**:

```
Tier 1: Migrated WebApiExample.Common to .NET 10.0

- Converted project to SDK-style
- Updated TargetFramework to net10.0
- Verified build succeeds
```

```
Tier 3: Migrated WebApiExample.WebApp to ASP.NET Core on .NET 10.0

- Converted WAP to SDK-style Web project
- Migrated Global.asax to Program.cs
- Replaced Unity DI with built-in DependencyInjection
- Replaced System.Web.Optimization bundling with direct HTML references
- Updated all controllers to ControllerBase
- Migrated Web.config to appsettings.json
- Updated bootstrap 3.3.7 ? 5.3.8 (security fix)
- Updated jQuery 3.3.1 ? 3.7.1 (security fix)
- Updated Newtonsoft.Json 11.0.1 ? 13.0.4 (security fix)
- All API endpoints tested and functional
```

#### Tier Completion Commits

**Required Commits** (minimum):

1. **After Tier 1**: 
   ```bash
   git add .
   git commit -m "Tier 1: Migrated WebApiExample.Common to .NET 10.0"
   ```

2. **After Tier 2**:
   ```bash
   git add .
   git commit -m "Tier 2: Migrated WebApiExample.DataStore to .NET 10.0, upgraded EF to 6.5.1"
   ```

3. **After Tier 3**:
   ```bash
   git add .
   git commit -m "Tier 3: Migrated WebApiExample.WebApp to ASP.NET Core on .NET 10.0"
   ```

4. **After Tier 4**:
   ```bash
   git add .
   git commit -m "Tier 4: Migrated WebApiExample.WebApp.Tests to .NET 10.0

All projects successfully migrated from .NET Framework 4.8 to .NET 10.0.
All security vulnerabilities addressed. All tests passing."
   ```

---

### Review and Merge Process

#### Pull Request Requirements

**When Ready to Merge** `upgrade-to-NET10` ? `main`:

**PR Title**: 
```
Upgrade entire solution from .NET Framework 4.8 to .NET 10.0
```

**PR Description Template**:
```markdown
## Summary
This PR migrates the entire WebApiExample solution from .NET Framework 4.8 to .NET 10.0.

## Migration Details

### Projects Migrated (4)
- ? WebApiExample.Common (Tier 1)
- ? WebApiExample.DataStore (Tier 2)
- ? WebApiExample.WebApp (Tier 3)
- ? WebApiExample.WebApp.Tests (Tier 4)

### Key Changes
- All projects converted to SDK-style
- All projects target .NET 10.0
- ASP.NET Web API ? ASP.NET Core Web API
- Global.asax ? Program.cs
- Unity DI ? Microsoft.Extensions.DependencyInjection
- System.Web.Optimization ? Direct HTML references
- Web.config ? appsettings.json

### Security Fixes
- ? bootstrap 3.3.7 ? 5.3.8
- ? jQuery 3.3.1 ? 3.7.1
- ? Newtonsoft.Json 11.0.1/6.0.4 ? 13.0.4

### Package Updates
- EntityFramework 6.4.4 ? 6.5.1
- System.Runtime.CompilerServices.Unsafe 4.5.2/4.5.3 ? 6.1.2
- Removed 20+ incompatible packages (replaced with framework features)

### Testing
- ? All projects build successfully
- ? All tests passing (100%)
- ? All API endpoints tested and functional
- ? No security vulnerabilities
- ? Full solution builds without errors or warnings

### Breaking Changes
See [Breaking Changes Catalog](/.github/upgrades/plan.md#breaking-changes-catalog) in plan.md

### Documentation
- Assessment: `.github/upgrades/assessment.md`
- Plan: `.github/upgrades/plan.md`

## Testing Checklist
- [ ] Code review completed
- [ ] All automated tests pass
- [ ] Manual smoke tests performed
- [ ] Security scan clean (`dotnet list package --vulnerable`)
- [ ] Performance acceptable
- [ ] Documentation reviewed

## Deployment Notes
[Add any deployment-specific instructions or considerations]

## Rollback Plan
If issues discovered post-merge:
1. Revert this PR: `git revert <merge-commit>`
2. Or checkout main branch from before merge
3. Investigate issues on upgrade-to-NET10 branch
4. Fix and re-merge when ready
```

#### Review Checklist

**Code Reviewer Should Verify**:

- [ ] All 4 tier commits present in history
- [ ] Commit messages clear and descriptive
- [ ] `.csproj` files correctly formatted (SDK-style)
- [ ] All target frameworks set to `net10.0`
- [ ] Security-vulnerable packages updated
- [ ] Incompatible packages removed
- [ ] `Program.cs` includes all necessary DI registrations
- [ ] Controllers updated to `ControllerBase`
- [ ] `appsettings.json` contains necessary configuration
- [ ] Static files in `wwwroot\` folder
- [ ] No `Global.asax` or `Web.config` (except IIS deployment config if needed)
- [ ] Tests still meaningful (not just passing with empty implementations)
- [ ] No obvious security issues introduced
- [ ] Code quality maintained

#### Merge Criteria

**Required**:
- ? All projects build successfully
- ? All tests passing
- ? No security vulnerabilities
- ? Code review approved
- ? Manual testing completed

**Recommended**:
- ? Performance validated (no significant regression)
- ? UI tested (if applicable)
- ? Database operations validated

**Merge Command**:
```bash
# From main branch
git merge upgrade-to-NET10 --no-ff
git push origin main
```

**Post-Merge**:
1. Tag the release:
   ```bash
   git tag -a v2.0.0-net10.0 -m "Migrated to .NET 10.0"
   git push origin v2.0.0-net10.0
   ```

2. Archive or delete upgrade branch (optional):
   ```bash
   git branch -d upgrade-to-NET10  # Local
   git push origin --delete upgrade-to-NET10  # Remote
   ```

---

### Rollback Strategy

#### Before Merge (On upgrade-to-NET10 branch)

**Rollback Entire Migration**:
```bash
git checkout main
git branch -D upgrade-to-NET10  # Discard upgrade work
```

**Rollback to Specific Tier**:
```bash
# Find commit hash of tier completion
git log --oneline --grep="Tier 2"

# Reset to that commit
git reset --hard <commit-hash>

# Continue from that point
```

#### After Merge (On main branch)

**Revert Entire Migration**:
```bash
# Revert the merge commit
git revert -m 1 <merge-commit-hash>
git push origin main
```

**Selective Revert**:
```bash
# Revert specific commits (be cautious with dependencies)
git revert <commit-hash>
```

---

### Git Workflow Summary

**Initial Setup**:
```bash
git checkout main
git pull origin main
git checkout -b upgrade-to-NET10
```

**During Migration** (after each tier):
```bash
git add .
git commit -m "Tier X: <description>"
git push origin upgrade-to-NET10
```

**Final Merge**:
```bash
git checkout main
git pull origin main
git merge upgrade-to-NET10 --no-ff
git push origin main
git tag -a v2.0.0-net10.0 -m "Migrated to .NET 10.0"
git push origin v2.0.0-net10.0
```

---

### Backup & Safety

**Before Starting Migration**:
1. Ensure remote backup of `main` branch
2. Optionally create backup branch:
   ```bash
   git checkout -b backup/pre-net10-migration main
   git push origin backup/pre-net10-migration
   ```

**During Migration**:
- Push `upgrade-to-NET10` branch regularly to remote
- Keep local and remote in sync

**After Completion**:
- Maintain `upgrade-to-NET10` branch for reference (at least temporarily)
- Keep tags for version milestones

## Success Criteria

### Migration is considered **COMPLETE** and **SUCCESSFUL** when all criteria below are met:

---

### Technical Criteria

#### Framework & Project Structure
- [x] **All projects target .NET 10.0**
  - WebApiExample.Common: net10.0
  - WebApiExample.DataStore: net10.0
  - WebApiExample.WebApp: net10.0
  - WebApiExample.WebApp.Tests: net10.0

- [x] **All projects use SDK-style format**
  - Legacy `.csproj` format eliminated
  - Modern `<Project Sdk="...">` format adopted

- [x] **ASP.NET Core migration complete**
  - WebApiExample.WebApp uses `Microsoft.NET.Sdk.Web`
  - ASP.NET Core Web API replaces ASP.NET Framework Web API
  - Program.cs created, Global.asax removed

#### Package Management
- [x] **All recommended package updates applied**
  - EntityFramework: 6.4.4 ? 6.5.1 ?
  - Newtonsoft.Json: 11.0.1/6.0.4 ? 13.0.4 ?
  - System.Runtime.CompilerServices.Unsafe: 4.5.2/4.5.3 ? 6.1.2 ?

- [x] **All security vulnerabilities addressed**
  - bootstrap: 3.3.7 ? 5.3.8 ?
  - jQuery: 3.3.1 ? 3.7.1 ?
  - Newtonsoft.Json: 11.0.1/6.0.4 ? 13.0.4 ?
  - Vulnerability scan clean:
    ```bash
    dotnet list package --vulnerable
    # Expected: No vulnerabilities found
    ```

- [x] **All incompatible packages removed or replaced**
  - Microsoft.AspNet.* packages removed (built into ASP.NET Core)
  - Unity.WebAPI removed (replaced with built-in DI)
  - Microsoft.AspNet.Web.Optimization removed (replaced with direct references)
  - System.Web.* dependencies eliminated

#### Build & Compilation
- [x] **Full solution builds successfully**
  ```bash
  dotnet build WebApiExample.sln
  # Expected: Build succeeded. 0 Error(s)
  ```

- [x] **Zero build errors across all projects**

- [x] **Zero build warnings** (or all warnings justified and documented)

- [x] **All projects restore packages successfully**
  ```bash
  dotnet restore WebApiExample.sln
  # Expected: Restore completed successfully
  ```

#### Testing
- [x] **All automated tests pass**
  ```bash
  dotnet test WebApiExample.sln
  # Expected: Passed! - All tests pass with 100% success rate
  ```

- [x] **No test infrastructure errors**
  - Test discovery works
  - Test execution completes
  - Test results reported correctly

- [x] **Integration tests validate tier dependencies**
  - Common integrates with DataStore ?
  - DataStore integrates with WebApp ?
  - WebApp integrates with Tests ?

---

### Quality Criteria

#### Code Quality
- [x] **Code quality maintained or improved**
  - No intentional workarounds or hacks
  - Clean, idiomatic .NET 10.0 code
  - Dependency injection patterns followed

- [x] **Test coverage maintained**
  - Existing tests still validate functionality
  - New tests added for migration-related changes (if applicable)

- [x] **Documentation updated**
  - README.md reflects .NET 10.0 (if applicable)
  - Architecture docs updated (if applicable)
  - API documentation regenerated (if using Swagger/OpenAPI)

#### Functionality
- [x] **Application functionality preserved**
  - All API endpoints functional
  - CRUD operations work correctly
  - Business logic unchanged (unless intentionally improved)

- [x] **Database operations functional**
  - Connections succeed
  - Queries execute correctly
  - Migrations compatible (if using EF migrations)

- [x] **Configuration works**
  - appsettings.json read correctly
  - Environment-specific settings work
  - Connection strings resolve

- [x] **Dependency injection functional**
  - All services registered
  - Controllers receive dependencies
  - Scopes (singleton, scoped, transient) correct

- [x] **Static file serving works** (if applicable)
  - CSS files load
  - JavaScript files load
  - Images/fonts load

---

### Process Criteria

#### Bottom-Up Strategy Adherence
- [x] **Tier 1 (Common) completed before Tier 2**
  - Common migrated first ?
  - Validated before proceeding ?

- [x] **Tier 2 (DataStore) completed before Tier 3**
  - DataStore migrated after Common ?
  - EF upgraded ?
  - Validated before proceeding ?

- [x] **Tier 3 (WebApp) completed before Tier 4**
  - WebApp migrated after Common & DataStore ?
  - ASP.NET Core fully implemented ?
  - Validated before proceeding ?

- [x] **Tier 4 (Tests) completed last**
  - Tests migrated after all dependencies ?
  - All tests pass ?

- [x] **Each tier validated before proceeding**
  - Build validation ?
  - Functional validation ?
  - No regressions in lower tiers ?

#### Source Control
- [x] **All changes committed**
  - Tier 1 commit exists
  - Tier 2 commit exists
  - Tier 3 commit exists
  - Tier 4 commit exists

- [x] **Commit messages clear and descriptive**
  - Each commit explains what was done
  - Major changes documented in commit body

- [x] **Migration branch clean**
  - No uncommitted changes
  - No merge conflicts
  - Ready for merge to main

---

### Operational Criteria

#### Security
- [x] **No known security vulnerabilities**
  - Package vulnerability scan clean
  - No deprecated packages with known CVEs
  - Security best practices followed

- [x] **Authentication/Authorization works** (if applicable)
  - Login functionality preserved
  - Authorization policies enforced
  - Token validation works (if using JWT/OAuth)

#### Performance
- [x] **Performance acceptable**
  - Application startup time reasonable
  - API response times acceptable
  - No obvious performance regressions
  - Database query performance maintained

- [x] **Resource usage acceptable**
  - Memory usage within reasonable bounds
  - CPU usage within reasonable bounds
  - No memory leaks detected

#### Deployment Readiness
- [x] **Application runs on target environment**
  - Runs locally (development)
  - Runs in test environment (if applicable)
  - Ready for production deployment

- [x] **Deployment documentation ready**
  - Deployment steps documented
  - Prerequisites listed (.NET 10.0 SDK/runtime required)
  - Configuration changes documented

---

### Verification Commands

**Run these commands to verify success criteria**:

```bash
# 1. Restore packages
dotnet restore WebApiExample.sln

# 2. Build solution
dotnet build WebApiExample.sln --configuration Release

# 3. Run tests
dotnet test WebApiExample.sln --configuration Release

# 4. Check for vulnerabilities
dotnet list package --vulnerable --include-transitive

# 5. Run application
cd WebApiExample.WebApp
dotnet run --configuration Release

# 6. Verify version
dotnet --list-runtimes | grep 10.0
# Expected: Microsoft.NETCore.App 10.0.x

# 7. Check project targets
grep -r "<TargetFramework>" --include="*.csproj"
# Expected: All showing net10.0
```

---

### Final Checklist

Before declaring migration complete, verify:

#### Tier 1 (Common)
- [ ] Converted to SDK-style
- [ ] Targets net10.0
- [ ] Builds successfully
- [ ] No warnings

#### Tier 2 (DataStore)
- [ ] Converted to SDK-style
- [ ] Targets net10.0
- [ ] EntityFramework 6.5.1
- [ ] Builds successfully
- [ ] Database operations work

#### Tier 3 (WebApp)
- [ ] Converted to SDK-style Web project
- [ ] Targets net10.0
- [ ] ASP.NET Core implementation complete
- [ ] Program.cs created, Global.asax removed
- [ ] All controllers migrated to ControllerBase
- [ ] Unity ? built-in DI complete
- [ ] Bundling replaced
- [ ] Web.config ? appsettings.json complete
- [ ] All security vulnerabilities fixed
- [ ] Builds successfully
- [ ] Application starts and runs
- [ ] All API endpoints tested

#### Tier 4 (Tests)
- [ ] Converted to SDK-style
- [ ] Targets net10.0
- [ ] All packages updated/removed
- [ ] Security vulnerability fixed
- [ ] Builds successfully
- [ ] All tests pass

#### Full Solution
- [ ] Solution builds without errors
- [ ] Solution builds without warnings
- [ ] All tests pass (100%)
- [ ] No security vulnerabilities
- [ ] Performance acceptable
- [ ] All commits pushed
- [ ] Pull request created (if using PR workflow)
- [ ] Code review completed (if applicable)
- [ ] Ready to merge to main

---

### Definition of Done

**The .NET Framework 4.8 to .NET 10.0 migration is DONE when**:

? All technical criteria met
? All quality criteria met
? All process criteria met (Bottom-Up Strategy followed)
? All operational criteria met
? All verification commands pass
? Final checklist complete
? Team consensus that migration is production-ready

**At this point**: 
- Merge `upgrade-to-NET10` branch to `main`
- Tag release: `v2.0.0-net10.0`
- Deploy to production (if ready)
- Celebrate! ??

---

### Post-Migration Activities (Not Required for Success Criteria)

**Recommended Future Enhancements**:
- Migrate from Newtonsoft.Json to System.Text.Json (ASP.NET Core default)
- Implement modern bundling (webpack, Vite)
- Replace Entity Framework 6 with EF Core (for full .NET Core experience)
- Add health checks
- Implement structured logging
- Add OpenTelemetry/Application Insights
- Review and optimize DI registrations
- Update to newer bootstrap/jQuery versions as released
- Implement minimal APIs (if appropriate for use case)
