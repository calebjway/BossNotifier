# SPT Mod Migration Guide: 3.x to 4.0.4

This guide provides step-by-step instructions for migrating SPT mods from version 3.x to 4.0.4.

> **Important**: This guide covers TWO different types of mods:
> - **Server-Side Mods**: TypeScript (3.x) → C# Server Mods (4.0.4)
> - **Client-Side Mods**: BepInEx C# (3.x) → BepInEx C# (4.0.4)

Choose the section that matches your mod type below.

---

# Part 1: Client-Side BepInEx Mod Migration (3.x → 4.0.4)

If your mod is a **BepInEx plugin** (runs in the game client, uses `BaseUnityPlugin`, patches game methods), follow this section.

## Overview of Changes for Client Mods

SPT 4.0.4 client-side changes:
- **Target Framework**: .NET Framework 4.7.1 → .NET Framework 4.8.1
- **New References**: `spt-common.dll` added
- **Updated APIs**: New boss types and game mechanics
- **Build System**: Modern SDK-style .csproj recommended

## Prerequisites

- .NET SDK 9.0 or later ([Download](https://dotnet.microsoft.com/en-us/download/dotnet/9.0))
- Visual Studio Code, Visual Studio 2022, or JetBrains Rider
- SPT 4.0.4 client installation

## Migration Steps for Client-Side Mods

### 1. Update Project File (.csproj)

**Option A: Convert to Modern SDK-Style (Recommended for VSCode)**

Replace your old `.csproj` with this modern format:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net481</TargetFramework>
    <AssemblyName>YourModName</AssemblyName>
    <RootNamespace>YourModName</RootNamespace>
    <Version>2.0.0</Version>
    <LangVersion>latest</LangVersion>
    <OutputType>Library</OutputType>
    <GenerateAssemblyInfo>false</GenerateAssemblyInfo>
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Release|AnyCPU'">
    <Optimize>True</Optimize>
    <DebugType>none</DebugType>
  </PropertyGroup>

  <PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
    <DebugType>full</DebugType>
    <Optimize>False</Optimize>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NETFramework.ReferenceAssemblies.net481" Version="1.0.3">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <Reference Include="0Harmony">
      <HintPath>References\0Harmony.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="Assembly-CSharp">
      <HintPath>References\Assembly-CSharp.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="BepInEx">
      <HintPath>References\BepInEx.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="Comfort">
      <HintPath>References\Comfort.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="spt-common">
      <HintPath>References\spt-common.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="spt-reflection">
      <HintPath>References\spt-reflection.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="UnityEngine">
      <HintPath>References\UnityEngine.dll</HintPath>
      <Private>False</Private>
    </Reference>
    <Reference Include="UnityEngine.CoreModule">
      <HintPath>References\UnityEngine.CoreModule.dll</HintPath>
      <Private>False</Private>
    </Reference>
  </ItemGroup>

  <ItemGroup>
    <Reference Update="System">
      <Private>False</Private>
    </Reference>
  </ItemGroup>

  <Target Name="PostBuild" AfterTargets="PostBuildEvent">
    <MakeDir Directories="$(TargetDir)BepInEx\plugins\" />
    <Copy SourceFiles="$(TargetPath)" DestinationFiles="$(TargetDir)BepInEx\plugins\$(TargetFileName)" />
    <Exec Command="powershell.exe -command &quot;Compress-Archive -Force -Path '$(TargetDir)BepInEx' -DestinationPath '$(TargetDir)$(TargetName).zip'&quot;" />
  </Target>

</Project>
```

**Option B: Update Existing Old-Style .csproj**

If you prefer to keep the old format, just update:
- `<TargetFrameworkVersion>v4.7.1</TargetFrameworkVersion>` → `<TargetFrameworkVersion>v4.8.1</TargetFrameworkVersion>`
- Add `<Reference Include="spt-common"><HintPath>References\spt-common.dll</HintPath></Reference>`

### 2. Update Reference DLLs

Copy updated DLLs from SPT 4.0.4 installation to your `References/` folder:

```bash
# From SPT 4.0.4 installation, copy:
BepInEx/core/0Harmony.dll
BepInEx/core/BepInEx.dll
BepInEx/plugins/spt/spt-common.dll          # NEW - Required!
BepInEx/plugins/spt/spt-reflection.dll
EscapeFromTarkov_Data/Managed/Assembly-CSharp.dll
EscapeFromTarkov_Data/Managed/Comfort.dll
EscapeFromTarkov_Data/Managed/UnityEngine.dll
EscapeFromTarkov_Data/Managed/UnityEngine.CoreModule.dll
```

### 3. Update Plugin Version

Update your `[BepInPlugin]` attribute to reflect the new version:

```csharp
// Before
[BepInPlugin("com.yourname.modname", "ModName", "1.x.x")]

// After
[BepInPlugin("com.yourname.modname", "ModName", "2.0.0")]
```

### 4. Check for New Game Types/APIs

SPT 4.0.4 may have new boss types, zones, or game mechanics. Example new boss types:

```csharp
// New in 4.0.4:
WildSpawnType.bossBoarSniper
WildSpawnType.bossTagillaAgro
WildSpawnType.bossKillaAgro
WildSpawnType.sectantPredvestnik
WildSpawnType.sectantPrizrak
WildSpawnType.sectantOni
```

Review `Assembly-CSharp.dll` using a decompiler (like dnSpy) to find new types.

### 5. Build and Test

**Using dotnet CLI (VSCode/Terminal):**
```bash
dotnet build -c Release
```

**Using Visual Studio:**
- Press F6 or Build → Build Solution

**Output:**
- DLL: `bin/Release/net481/YourModName.dll`
- Package: `bin/Release/net481/YourModName.zip`

### 6. Install and Test

1. Copy `bin/Release/net481/YourModName.dll` to SPT's `BepInEx/plugins/` folder
2. Launch SPT and check the BepInEx console for errors
3. Test your mod's functionality in-game

## Client-Side Migration Checklist

- [ ] Updated .csproj target framework to net481
- [ ] Added spt-common.dll reference
- [ ] Copied all updated DLLs from SPT 4.0.4 to References folder
- [ ] Updated plugin version to 2.0.0+
- [ ] Checked for new game types/APIs in Assembly-CSharp
- [ ] Built successfully without errors
- [ ] Tested in SPT 4.0.4 client
- [ ] All features work as expected

---

# Part 2: Server-Side Mod Migration (TypeScript 3.x → C# 4.0.4)

If your mod is a **server-side mod** (TypeScript, uses dependency injection, modifies server database/routes), follow this section.

## Overview of Changes for Server Mods

SPT 4.0.4 represents a complete architectural shift:
- **Language**: TypeScript → C#
- **Runtime**: Node.js → .NET 9
- **Build System**: npm/webpack → dotnet build
- **Module System**: CommonJS → .NET assemblies (DLLs)
- **Dependency Injection**: tsyringe (TS) → Native DI container (C#)

## Prerequisites

- .NET 9 SDK ([Download](https://dotnet.microsoft.com/en-us/download/dotnet/9.0))
- Visual Studio 2022 Community or JetBrains Rider (recommended)
- Basic understanding of C# syntax
- SPT 4.0.4 server installation

## Migration Steps

### 1. Organize Existing Files

Create an `Old_ModFiles` directory and move all TypeScript-related files:
```bash
mkdir Old_ModFiles
mv src types package.json package-lock.json tsconfig.json build.mjs *.ts Old_ModFiles/
```

Keep:
- `config/` folder (will need to convert JSON5 to JSON)
- `LICENSE` file
- `README.md` (will need updating)
- Any image/asset files

### 2. Create ModMetadata Class

In SPT 4.0.4, `package.json` is replaced by a `ModMetadata` record in your main C# file:

```csharp
using SPTarkov.Server.Core.Models.Spt.Mod;

public record ModMetadata : AbstractModMetadata
{
    public override string ModGuid { get; init; } = "com.yourname.modname";
    public override string Name { get; init; } = "Your Mod Name";
    public override string Author { get; init; } = "Your Name";
    public override List<string>? Contributors { get; init; }
    public override SemanticVersioning.Version Version { get; init; } = new("2.0.0");
    public override SemanticVersioning.Range SptVersion { get; init; } = new("~4.0.0");
    public override List<string>? Incompatibilities { get; init; }
    public override Dictionary<string, SemanticVersioning.Range>? ModDependencies { get; init; }
    public override string? Url { get; init; } = "https://github.com/yourname/modname";
    public override bool? IsBundleMod { get; init; } = false;
    public override string? License { get; init; } = "MIT";
}
```

**Important**: Use reverse domain notation for ModGuid (e.g., `com.refringe.customraidtimes`)

### 3. Create Main Entry Point Class

SPT 4.0.4 uses dependency injection with the `[Injectable]` attribute and `IOnLoad` interface:

```csharp
using SPTarkov.DI.Annotations;
using SPTarkov.Server.Core.DI;
using SPTarkov.Server.Core.Helpers;
using SPTarkov.Server.Core.Models.Utils;
using SPTarkov.Server.Core.Services;
using System.Reflection;

[Injectable(TypePriority = OnLoadOrder.PostDBModLoader + 1)]
public class YourModMain(
    ISptLogger<YourModMain> logger,
    ModHelper modHelper,
    DatabaseService databaseService) : IOnLoad
{
    public Task OnLoad()
    {
        // Your mod initialization code here
        logger.Success("Mod loaded successfully!");
        return Task.CompletedTask;
    }
}
```

**Load Order Options**:
- `OnLoadOrder.PreSptModLoader + 1` - Before database loads
- `OnLoadOrder.PostDBModLoader + 1` - After database loads (most common)

### 4. Configuration Handling

#### Convert JSON5 to JSON
SPT 4.0.4 uses standard JSON (no comments or trailing commas):

**Before (JSON5)**:
```json5
{
    enabled: true, // Enable the mod
    debug: false,
}
```

**After (JSON)**:
```json
{
    "enabled": true,
    "debug": false
}
```

#### Create Configuration Model Classes

```csharp
using System.Text.Json.Serialization;

public class ModConfig
{
    [JsonPropertyName("enabled")]
    public bool Enabled { get; set; } = true;

    [JsonPropertyName("debug")]
    public bool Debug { get; set; } = false;
}
```

#### Load Configuration

```csharp
var pathToMod = modHelper.GetAbsolutePathToModFolder(Assembly.GetExecutingAssembly());
var config = modHelper.GetJsonDataFromFile<ModConfig>(pathToMod, "config/config.json");
```

### 5. Database Access

#### TypeScript (3.x):
```typescript
const locations = container.resolve<DatabaseServer>("DatabaseServer").getTables().locations;
const customs = locations.bigmap.base;
```

#### C# (4.0.4):
```csharp
var locations = databaseService.GetLocations();
var customs = locations.Bigmap; // Note: Property names are PascalCase
```

**Important Property Name Changes**:
- `locations.rezervbase` → `locations.RezervBase` (capital B)
- Most properties follow PascalCase in C#

### 6. Working with Dynamic Types

When you don't have exact type definitions, use `dynamic`:

```csharp
private readonly dynamic _location;

public void ProcessLocation()
{
    // Accessing properties works with dynamic
    int escapeTime = (int)_location.Base.EscapeTimeLimit;

    // Always cast when assigning to dynamic properties
    _location.Base.EscapeTimeLimit = (int)newTime;
}
```

**Critical**: When using `dynamic`, always explicitly cast values when assigning to properties to avoid runtime type conversion errors.

### 7. Common Type Conversions

#### Enums
```csharp
// SPT 4.0.4 uses enums instead of strings
string passageReq = exit.PassageRequirement?.ToString() ?? "";
if (passageReq.Equals("Train", StringComparison.OrdinalIgnoreCase))
{
    // Handle train exit
}
```

#### Logger
```typescript
// TypeScript
this.logger.log("Message", "cyan");
```

```csharp
// C#
logger.Info("Message");
logger.Success("Message");
logger.Warning("Message");
logger.Error("Message");
logger.Critical("Message");
logger.Debug("Message"); // Only to file, not console
```

### 8. Create .csproj File

Create `YourMod.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <RootNamespace>YourModName</RootNamespace>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <OutputType>Library</OutputType>
    <Version>2.0.0</Version>
    <OutputPath>bin\$(Configuration)\$(ProjectName)\$(AssemblyName)\</OutputPath>
    <AppendTargetFrameworkToOutputPath>false</AppendTargetFrameworkToOutputPath>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SPTarkov.Common" Version="4.0.3" />
    <PackageReference Include="SPTarkov.DI" Version="4.0.3" />
    <PackageReference Include="SPTarkov.Server.Core" Version="4.0.3" />
  </ItemGroup>

  <!-- Copy config folder to output directory -->
  <ItemGroup>
    <None Include="config\**\*">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>
  </ItemGroup>

</Project>
```

### 9. File Organization

Recommended C# project structure:
```
YourMod/
├── YourMod.cs              # Main entry point + ModMetadata
├── YourMod.csproj          # Project file
├── YourMod.sln             # Solution file (optional)
├── config/
│   └── config.json         # Configuration file
├── Models/                 # Data models and config classes
│   └── ModConfig.cs
├── Processors/             # Business logic classes
│   └── DataProcessor.cs
├── Adjusters/              # Classes that modify data
│   └── ItemAdjuster.cs
├── Old_ModFiles/           # Original TypeScript files
└── README.md
```

### 10. Building

```bash
# Build in Release mode
dotnet build YourMod.csproj -c Release

# Output will be in: bin/Release/YourMod/
```

### 11. Installation

1. Copy the entire `bin/Release/YourMod/` folder
2. Paste into SPT's `user/mods/` directory
3. Rename to include version: `YourMod-2.0.0/`
4. Start SPT server

### 12. Common Pitfalls & Solutions

#### Problem: "Cannot implicitly convert type 'double' to 'int'"
**Solution**: Always cast when using `dynamic`:
```csharp
// Wrong
exit.MinTime = trainArriveEarliest;

// Correct
exit.MinTime = (int)trainArriveEarliest;
```

#### Problem: "does not contain a definition for 'ToLower'"
**Solution**: Property is an enum, convert to string first:
```csharp
// Wrong
if (exit.PassageRequirement?.ToLower() == "train")

// Correct
string passageReq = exit.PassageRequirement?.ToString() ?? "";
if (passageReq.Equals("Train", StringComparison.OrdinalIgnoreCase))
```

#### Problem: "does not contain a definition for 'Rezervbase'"
**Solution**: Check property casing - SPT 4.0.4 uses PascalCase:
```csharp
// Wrong
_locations.Rezervbase

// Correct
_locations.RezervBase
```

#### Problem: Build succeeds but mod doesn't load
**Solution**: Check these:
1. ModGuid is unique and follows reverse domain notation
2. `OnLoad()` method returns `Task.CompletedTask`
3. Config file is valid JSON (no comments, no trailing commas)
4. Injectable attribute has correct TypePriority

### 13. Testing Checklist

- [ ] Mod builds without errors
- [ ] Server starts without errors
- [ ] Mod appears in server logs
- [ ] Configuration loads correctly
- [ ] Core functionality works
- [ ] Error handling works (test with invalid config)
- [ ] Debug logging works (if implemented)

### 14. Useful References

- **SPT Modding Wiki**: https://wiki.sp-tarkov.com/
- **Server Mod Examples**: https://github.com/sp-tarkov/server-mod-examples
- **.NET 9 Docs**: https://learn.microsoft.com/en-us/dotnet/

### 15. Migration Example: Route Registration

#### TypeScript (3.x):
```typescript
staticRouterModService.registerStaticRouter(
    "MyRoute",
    [
        {
            url: "/client/custom/route",
            action: async (url, info, sessionId, output) => {
                // Handle route
                return output;
            }
        }
    ],
    "MyRoute"
);
```

#### C# (4.0.4):
```csharp
using SPTarkov.Server.Core.Models.Common;
using SPTarkov.Server.Core.Utils;

[Injectable]
public class CustomStaticRouter : StaticRouter
{
    private static HttpResponseUtil _httpResponseUtil;

    public CustomStaticRouter(
        JsonUtil jsonUtil,
        HttpResponseUtil httpResponseUtil) : base(jsonUtil, GetCustomRoutes())
    {
        _httpResponseUtil = httpResponseUtil;
    }

    private static List<RouteAction> GetCustomRoutes()
    {
        return
        [
            new RouteAction(
                "/client/custom/route",
                async (url, info, sessionId, output) =>
                    await HandleRoute(url, info, sessionId)
            )
        ];
    }

    private static ValueTask<string> HandleRoute(string url, IRequestData info, MongoId sessionId)
    {
        // Handle route
        return new ValueTask<string>(_httpResponseUtil.NullResponse());
    }
}
```

## Final Notes

- **Test thoroughly** - SPT 4.0.4 has different behaviors
- **Use `dynamic` sparingly** - Only when types are unknown
- **Cast explicitly** - When working with dynamic types
- **Follow C# conventions** - PascalCase for properties/methods
- **Update version number** - Increment major version (1.x → 2.0)
- **Update README** - Document new requirements and changes

## Credits

This migration guide is based on the CustomRaidTimes mod migration from SPT 3.11 to 4.0.4.
