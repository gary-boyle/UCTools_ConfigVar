# UCTools ConfigVariables

A robust configuration variable system for Unity games, originally adapted from Unity's FPS Sample project. This system provides a centralized, type-safe way to manage game settings, console variables, and persistent configuration data.

## Overview

The ConfigVar system allows you to declare configuration variables using attributes and automatically handles type conversion, persistence, console integration, and change detection. All variables are managed through a global registry with support for various behavioral flags.

## Features

- **Attribute-based Declaration**: Clean, declarative syntax for defining config variables
- **Type-safe Access**: Automatic conversion between string, int, and float values
- **Selective Persistence**: Save only variables marked with the `Save` flag
- **Change Detection**: Efficient system for reacting to value changes
- **Console Integration**: Runtime modification through console commands
- **Name Validation**: Ensures consistent naming conventions
- **Behavioral Flags**: Control save behavior, cheat protection, and network replication

## Console Integration

This system is designed to work  with the **[UCTools_CommandConsole](https://github.com/gary-boyle/UCTools_CommandConsole)** repository. When both systems are used together, all ConfigVars automatically become available as console commands, providing powerful runtime debugging and configuration capabilities.

### Benefits of Using Both Systems Together

- **Runtime Modification**: Change any ConfigVar value through the console at runtime
- **Auto-completion**: Console provides tab completion for all registered ConfigVar names
- **Help System**: Built-in help displays variable descriptions and current values
- **Batch Operations**: Execute multiple variable changes from config files
- **Debug Workflow**: Quickly test different settings without recompiling

### Console Usage Examples

```
// View current value and description
> help player.health
player.health = "100" - Player maximum health

// Change a value
> player.health 150

// Save current settings
> save_config config.cfg

// Load settings from file
> exec config.cfg

// Reset to defaults
> reset_all_config
```

## Installation

1. Copy the `UCTools_ConfigVariables` folder into your Unity project's `Assets` folder
2. (Optional but recommended) Install [UCTools_CommandConsole](https://github.com/gary-boyle/UCTools_CommandConsole) for runtime console access
3. Initialize the system during application startup:

```csharp
void Awake()
{
    UCTools_ConfigVariables.ConfigVar.Init();
    // If using console system, it will automatically discover all ConfigVars
}
```

## Basic Usage

### 1. Declare Variables

```csharp
using UCTools_ConfigVariables;

public class GameSettings
{
    [ConfigVar(Name = "player.health", DefaultValue = "100", Description = "Player maximum health")]
    public static ConfigVar playerHealth;
    
    [ConfigVar(Name = "graphics.quality", DefaultValue = "2", Description = "Graphics quality level", Flags = ConfigFlags.Save)]
    public static ConfigVar graphicsQuality;
    
    [ConfigVar(Name = "debug.godmode", DefaultValue = "0", Description = "Enable god mode", Flags = ConfigFlags.Cheat)]
    public static ConfigVar godMode;
}
```

### 2. Access Values

```csharp
// Get values with automatic type conversion
int health = GameSettings.playerHealth.IntValue;
float healthFloat = GameSettings.playerHealth.FloatValue;
string healthString = GameSettings.playerHealth.Value;

// Set values
GameSettings.playerHealth.Value = "150";
```

### 3. Change Detection

```csharp
void Update()
{
    if (GameSettings.graphicsQuality.ChangeCheck())
    {
        // Graphics quality was changed - update rendering settings
        ApplyGraphicsSettings();
    }
}
```

### 4. Persistence

```csharp
// Save all variables marked with ConfigFlags.Save
ConfigVar.Save("config.cfg");

// Save only if variables have actually changed (more efficient)
ConfigVar.SaveChangedVars("config.cfg");

// Reset all variables to defaults
ConfigVar.ResetAllToDefault();
```

## Configuration Flags

| Flag | Description |
|------|-------------|
| `None` | Default behavior - temporary variable |
| `Save` | Variable persists between sessions |
| `Cheat` | Can only be modified when cheats are enabled |
| `ServerInfo` | Sent from server to clients |
| `ClientInfo` | Sent from client to server |
| `User` | Created dynamically by user |

Flags can be combined: `ConfigFlags.Save | ConfigFlags.ServerInfo`

## When to Use This System

### ✅ Good Use Cases

- **Game Settings & Preferences**: Graphics quality, audio levels, key bindings
- **Console Variables**: Debug flags, developer tools, runtime tweaks
- **Server Configuration**: Game rules, player limits, map rotation
- **Player Preferences**: Name, preferred color, gameplay options
- **Development Tools**: Debug overlays, profiling flags, test parameters
- **Persistent Game State**: Unlocked levels, high scores, achievement progress
- **Runtime Debugging**: Variables that need console access for testing

### ❌ When NOT to Use This System

- **Frequently Changing Data**: Player position, health, ammunition (use regular variables)
- **Complex Data Structures**: Use ScriptableObjects or custom serialization
- **Sensitive Data**: Passwords, API keys (use secure storage methods)
- **Large Binary Data**: Textures, audio clips, models (use Unity's asset system)
- **Real-time Performance Critical**: Variables accessed every frame (cache the values)
- **Temporary Runtime State**: UI state, temporary flags (use regular instance variables)
- **Data That Shouldn't Be User-Accessible**: Internal state that users shouldn't modify via console

## Pros and Cons

### Pros

- **Centralized Management**: All config variables in one global registry
- **Type Safety**: Automatic conversion with fallback to sensible defaults
- **Attribute-based**: Clean, declarative syntax reduces boilerplate
- **Selective Persistence**: Save only what needs to be saved
- **Console Integration**: Built-in support for runtime modification (with UCTools_CommandConsole)
- **Change Detection**: Efficient system for reacting to changes
- **Validation**: Automatic name validation prevents conflicts
- **Flexible Flags**: Fine-grained control over variable behavior
- **Development Workflow**: Seamless integration with debugging tools

### Cons

- **Reflection Overhead**: Initial setup uses reflection (one-time cost)
- **Global State**: All variables are global, which can complicate testing
- **String-based Storage**: Everything stored as strings internally
- **Memory Usage**: Caches multiple representations of each value
- **No Type Enforcement**: Can't prevent assignment of invalid string values
- **Limited Data Types**: Only supports string, int, and float natively
- **Initialization Order**: Must call `Init()` before using any variables
- **Console Exposure**: All ConfigVars become console-accessible (consider security implications)

## Integration with UCTools_CommandConsole

When used together with the CommandConsole system, ConfigVars provide several additional benefits:

### Automatic Command Registration
```csharp
// No additional code needed - ConfigVars automatically become console commands
[ConfigVar(Name = "game.difficulty", DefaultValue = "1", Description = "Game difficulty level")]
static ConfigVar gameDifficulty;

// Now available in console as:
// > game.difficulty 2
// > help game.difficulty
```

### Configuration File Support
The console system can execute configuration files that use ConfigVar syntax:
```
// config.cfg
player.health "200"
graphics.quality "3"
debug.showfps "1"
```

Load with: `> exec config.cfg`

### Development Workflow
```csharp
public class DebugSettings
{
    [ConfigVar(Name = "debug.showfps", DefaultValue = "0", Description = "Show FPS counter")]
    static ConfigVar showFPS;
    
    [ConfigVar(Name = "debug.wireframe", DefaultValue = "0", Description = "Wireframe rendering mode")]
    static ConfigVar wireframe;
    
    void Update()
    {
        if (showFPS.ChangeCheck())
            fpsCounter.gameObject.SetActive(showFPS.IntValue > 0);
            
        if (wireframe.ChangeCheck())
            Camera.main.GetComponent<Camera>().SetReplacementShader(
                wireframe.IntValue > 0 ? wireframeShader : null, "");
    }
}
```

Now you can toggle these at runtime:
```
> debug.showfps 1     // Enable FPS counter
> debug.wireframe 1   // Enable wireframe mode
```

## Advanced Examples

### Custom Variable Registration

```csharp
// Create variables at runtime
var dynamicVar = new ConfigVar("runtime.test", "A test variable", "default", ConfigFlags.User);
ConfigVar.RegisterConfigVar(dynamicVar);
// Automatically available in console if CommandConsole is present
```

## Related Projects

- **[UCTools_CommandConsole](https://github.com/gary-boyle/UCTools_CommandConsole)**: Developer console system that integrates seamlessly with ConfigVars

## License

This system is adapted from Unity's FPS Sample project. Please refer to Unity's licensing terms for usage rights.

## Contributing

Feel free to submit issues and enhancement requests. This is a community-maintained adaptation of Unity's original system.
