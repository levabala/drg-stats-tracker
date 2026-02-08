# Development Environment Setup

This guide walks you through setting up the development environment for the DRG Stats Tracker mod.

## Prerequisites

- **Windows 10/11** (required for Unreal Engine 4.27.2)
- **~50GB free disk space** (for UE4 + project)
- **Visual Studio 2019** or newer (for C++ compilation)
- **Epic Games Launcher** (to download Unreal Engine)

## Step 1: Install Epic Games Launcher

1. Download from [epicgames.com](https://www.epicgames.com/store/download)
2. Install and create/login to your Epic account

## Step 2: Install Unreal Engine 4.27.2

**Important**: DRG uses UE4 version **4.27.2** specifically. Using a different version will cause compatibility issues.

1. Open Epic Games Launcher
2. Go to **Unreal Engine** tab
3. Click **Library**
4. Click the **+** button to add a new engine version
5. Click the dropdown arrow next to "Install" and select version **4.27.2**
6. Configure installation:
   - **Uncheck** all optional components except "Core"
   - This reduces install size from 30GB+ to ~13GB
7. Click Install and wait for completion

## Step 3: Install Visual Studio

UE4 requires Visual Studio for C++ compilation.

1. Download [Visual Studio 2019 Community](https://visualstudio.microsoft.com/vs/older-downloads/) (free)
2. During installation, select these workloads:
   - **Desktop development with C++**
   - **Game development with C++**
3. In Individual Components, ensure these are selected:
   - MSVC v142 build tools
   - Windows 10 SDK (latest)
   - C++ CMake tools

## Step 4: Clone FSD-Template

The FSD-Template contains all the C++ class stubs needed for DRG modding.

```bash
# Clone the template repository
git clone https://github.com/DRG-Modding/FSD-Template.git

# Navigate into it
cd FSD-Template
```

## Step 5: Build FSD-Template

1. Double-click `FSD.uproject` to open in UE4
2. If prompted to rebuild, click **Yes**
3. Wait for shaders to compile (this takes a while on first run)
4. Once loaded, go to **File > Generate Visual Studio project files**
5. Open the generated `.sln` file in Visual Studio
6. Build the solution (Build > Build Solution or Ctrl+Shift+B)

## Step 6: Download Basic Modding Tools

1. Join the [DRG Modding Discord](https://discord.gg/zQMKGTStfa)
2. Go to `#learn-tools` channel
3. Download **[Tool] Basic Modding Tools** zip
4. Extract to a convenient location (e.g., `C:\DRGModding\Tools`)

The zip contains:
- **DRGPacker** - For packing/unpacking .pak files
- **EmptyContentHierarchy** - Empty folder structure for mods
- **UAssetGUI** - For viewing/editing asset files
- **UModel** - For extracting textures, sounds, models
- **FModel** - For viewing asset contents as JSON

## Step 7: Extract Game Files (Optional but Recommended)

Extracting the game files helps you understand the structure and find reference implementations.

1. Locate your DRG installation:
   ```
   Steam\steamapps\common\Deep Rock Galactic\FSD\Content\Paks\FSD-WindowsNoEditor.pak
   ```
2. Drag `FSD-WindowsNoEditor.pak` onto `DRGPacker\_Unpack.bat`
3. Wait for extraction to complete
4. Extracted files will be in `DRGPacker\FSD-WindowsNoEditor\`

## Step 8: Download Header Dumps

The header dumps show you what C++ classes and functions are available.

```bash
git clone https://github.com/DRG-Modding/Header-Dumps.git
```

Open these files in VS Code or similar to search for functions:
- `FSD.hpp` - Main game classes (most of what you need)
- Other `.hpp` files for specific systems

## Step 9: Create Your Mod Project Structure

In your FSD-Template project:

1. Navigate to `Content/`
2. Create a new folder: `ModStatsTracker`
3. This is where all your mod's blueprints will go

Your folder structure should look like:
```
FSD-Template/
├── FSD.uproject
├── Source/
│   └── FSD/
│       └── ... (C++ stubs)
└── Content/
    └── ModStatsTracker/    <- Your mod folder
        ├── InitCave.uasset
        ├── InitSpacerig.uasset
        └── ... (other blueprints)
```

## Step 10: Verify Setup

1. Open the FSD-Template project in UE4
2. In Content Browser, navigate to your `ModStatsTracker` folder
3. Right-click > Blueprint Class > Actor
4. Name it `TestActor`
5. If this works without errors, your setup is complete!

## Troubleshooting

### "Failed to compile" errors
- Ensure Visual Studio 2019 with C++ workloads is installed
- Try: File > Refresh Visual Studio Project
- Rebuild the solution in Visual Studio

### Shader compilation takes forever
- This is normal on first run
- Can take 30-60 minutes depending on your hardware
- Subsequent loads will be faster

### "Module 'FSD' not found" error
- Make sure you opened `FSD.uproject`, not another project
- Rebuild the Visual Studio solution

### UE4 version mismatch
- Must use exactly version 4.27.2
- Check in Epic Games Launcher > Library

## Next Steps

Once your environment is set up, proceed to:

1. [IMPLEMENTATION.md](IMPLEMENTATION.md) - Step-by-step blueprint creation
2. [../blueprints/](../blueprints/) - Detailed blueprint logic for each component

## Quick Reference

| Tool | Purpose |
|------|---------|
| UE4 4.27.2 | Create blueprint assets |
| Visual Studio | Compile C++ (for FSD-Template) |
| DRGPacker | Pack mods into .pak files |
| UAssetGUI | View/edit asset values |
| FModel | View asset contents as JSON |
| UModel | Extract textures, sounds, models |

## Resources

- [DRG Modding Discord](https://discord.gg/zQMKGTStfa) - Ask questions in #mod-questions
- [FSD-Template Repo](https://github.com/DRG-Modding/FSD-Template)
- [Header Dumps](https://github.com/DRG-Modding/Header-Dumps)
- [Blueprint Modding Guide](https://drg.mod.io/guides/how-to-blueprint-mod)
